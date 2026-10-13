STATUS: DONE - contrib/child-process-call-facts ready for owner review (fixes #2571, #2572); explanation added
UPDATED: 2026-10-13T13:52:40Z
BRANCH: contrib/child-process-call-facts
MODE: Auto

## State

- The branch `contrib/child-process-call-facts` holds one commit, `cb1a936`, authored by ahmed0magdy with a `Co-Authored-By: Claude Opus 5.5` trailer. It is pushed to the fork; no PR is open.
- PR title: `fix(javascript): read child-process options and kill signals as written`. Fixes #2571. Fixes #2572.
- The PR notes, the validation list and the inline-script issue draft are as reported earlier in chat.
- AI disclosure line: "AI-assisted: implemented and tested with Claude Code; I reviewed the diff and validation results."

## Explain it to the owner

### 1. The bugs

REA reads JavaScript as text, without running it. When the code starts a child process (Node's `child_process` module), REA records facts about the call: whether an environment (`env`) was passed, which `stdio` mode was chosen, and which signal `kill()` sends. Two of those facts came out wrong.

- **#2571, options lost.** In Node, the args array is optional, so `spawn(cmd, options)` is valid. REA always looked for the options object in the third argument, so it missed options given second.
  - Input: `spawn("tool", { env: { A: "1" }, stdio: "pipe" })`
  - Wrong output: `environmentSupplied: false`, `stdioMode: "default"`
  - Right output: `environmentSupplied: true`, `stdioMode: "pipe"`
- **#2572, invented signal.** `child.kill()` with no argument sends SIGTERM. REA also reported SIGTERM when the argument was something it could not read, such as a function parameter. That reports a guess as a fact.
  - Input: `function run(sig) { child.kill(sig) }`
  - Wrong output: `signalName: "SIGTERM"`
  - Right output: `signalName: null` (unknown)

  `child.kill()` still gives `"SIGTERM"`, and `child.kill("SIGINT")` still gives `"SIGINT"`.

### 2. How the fix works

Everything is in `src/domain/javascript/javascriptSemanticChildProcesses.ts`. That module finds `child_process` calls in the parsed syntax tree (Babel AST) and builds the records. It only reads syntax.

1. **`spawnOptions(candidate)`** picks the argument that holds the options.
   - Before: it always took index 1 for `exec` and index 2 for `spawn`, `execFile` and `fork`.
   - Now, for `spawn`, `execFile` and `fork`: if the second argument is an inline object literal (`{ ... }`), that is the options. Otherwise it uses the third argument, as before. `exec` is unchanged.
   - This matches Node itself, which treats a non-array object in the args position as the options.
   - Its callers, `immutableSpawn` → `objectHasProperty(options, "env")` and `stdioMode(options)`, did not change. They now just receive the right object.
2. **`killSignalName(args)`** is a new small helper. It is called from `childInteraction`, where the inline `?? "SIGTERM"` used to be.
   - If `kill` has no arguments, it returns `"SIGTERM"` (Node's documented default).
   - Otherwise it returns `dataEffectLiteralString(args[0])`, the existing helper. That helper returns the string for an exact string literal (or a template string with no `${}`) and null for anything else.

**Why this approach and not the alternatives:**
- **Not "always read argument 1 when it is an object".** A variable in that position, as in `spawn(cmd, someVar, {...})`, could be either args or options. The fix only reads argument 1 when it is clearly an object literal, so the existing three-argument behavior is untouched.
- **No new field.** The IR already allows `signalName: string | null`, and the graph output already writes `signal_name` as given. Null is the codebase's normal way to say "not established", so no contract or schema change was needed.
- **`resolution` left alone.** `resolution` describes whether the `kill` call is linked to exactly one child process, not whether the signal is known. Issue #2572 says the same. Changing it would mix two meanings.
- **No guessing for numbers.** `kill(9)` now gives null rather than a name. Mapping numbers to names would be new behavior, not part of this bug.

### 3. Why it is correct

The two new tests are in `src/domain/javascript/javascriptSemanticAnalysis.native_bindings.test.ts`, in a block named "child-process call forms". They are the issue reporters' own tests plus their controls.

**What they prove:**
- **Options test.** The same three calls (`spawn`, `execFile`, `fork`) give the same correct facts in both forms, with and without an explicit `[]` args array.
- **No invented options.** Calls with no options still report no env and "default" stdio.
- **`execFile` stdio.** `execFile` reports "default" stdio even with options, because it has no stdio option.
- **All results stay complete.** Every call keeps `resolution: "complete"`.
- **Signal test.** A parameter signal gives null, an omitted signal gives SIGTERM, and `"SIGINT"` gives SIGINT. All three stay linked to the one child process.

Both tests failed on unchanged main (2 failed, 3 passed) and pass with the fix. Separately, I ran the whole REA CLI on a small fixture: the output graph shows `environment_supplied: true`, `stdio_mode: "pipe"`, and a signal node with `signal_name: null`.

**What they do NOT cover:**
- An options object held in a variable, as in `spawn(cmd, opts)`. It is still not recovered, the same limit as before the fix.
- `kill(9)` or `kill(undefined)`. Both now give null; no test pins that.
- `exec`, which is unchanged and has no new test.
- Real Node behavior. Nothing is executed, and the Node semantics come from Node's documentation.

### 4. Likely maintainer questions

1. **Why null instead of SIGTERM for an unknown signal?**
   REA's rules (AGENTS.md) say unknowns must stay unknown. SIGTERM is only certain when no argument is passed. The IR field was already nullable.
2. **Why not change `resolution` to "partial" for an unknown signal?**
   `resolution` is about which child process the call is linked to, and that link is still exact. The issue explicitly says not to reuse it for the signal.
3. **What about `spawn(cmd, optsVariable)`?**
   It is still not recovered. Only inline object literals are read, both before and after the change. Telling args from options in a variable would need value tracking, which is out of scope here.
4. **Could an object literal in argument 1 really be args?**
   No. Node's args must be an array, and Node itself treats a non-array object there as the options.
5. **Does this change any schema or generated catalog?**
   No. No type, tool schema or generated file changed. That is why `docs:check` was not needed.

### 5. Risks and how they were checked

- **Three-argument calls could regress.** Argument 1 is only treated as options when it is an object literal, and argument 1 in a three-argument call is an array or a variable. The existing test (`spawn` with `["--mode","fast"]` and options) still passes.
- **Consumers could break on a null signal.** The graph projection (`src/application/javascript/JavaScriptSemanticGraphChildProcessProjection.ts`, `addSignalNode`) passes the value through as the label and `signal_name`. Null labels are already used for listener nodes, and the CLI run produced a valid graph with a null signal node.
- **Wider regressions.** All 62 JavaScript domain and application test files pass (1,439 tests), and so does `npm run check:fast` (typecheck and lint).
- **Lint or format.** The first `check:fast` run failed a lint rule (function longer than 150 lines). Moving the new tests into their own `describe` block fixed it. oxfmt and oxlint pass in the commit hooks.

### 6. Read the diff in this order (about 10 minutes)

1. `src/domain/javascript/javascriptSemanticChildProcesses.ts`, `spawnOptions`: the new rule ("argument 1 is the options if it is an object literal, except for `exec`, which always uses argument 1").
2. The same file, `killSignalName` and its call in `childInteraction` (the `signalName:` line): SIGTERM only when `args.length === 0`.
3. `src/domain/javascript/javascriptSemanticIr.ts`, `JavaScriptSemanticChildProcessInteraction.signalName` (about line 286): it is already `string | null`. Nothing changed here; this just confirms there is no contract change.
4. `src/domain/javascript/javascriptSemanticAnalysis.native_bindings.test.ts`, the "child-process call forms" describe block: the two-argument and three-argument forms, the no-options control, and the three signal cases.
5. Optional: `src/application/javascript/JavaScriptSemanticGraphChildProcessProjection.ts`, `addSignalNode`, to see how `signal_name` reaches the output unchanged.
