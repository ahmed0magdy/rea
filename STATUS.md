STATUS: DONE - conflict resolved by merge, 3 review threads fixed in 05429ce, pushed without force; checks green
UPDATED: 2026-10-14T06:11:17Z
BRANCH: contrib/child-process-call-facts

## Result

- **New head:** `05429ce9c5054eae549dbe41058611d17a1907b0`. It was a normal fast-forward push (`4c51319..05429ce`), with no force.
- **Commits on top of the maintainer's work:**
  - `753d308`: merge of `origin/main` (`7375ab6`) into the branch;
  - `05429ce`: the review fixes.

  Both are authored by ahmed0magdy. The fix commit keeps the `Co-Authored-By: Claude Opus 5.5` trailer; there is no Claude-Session trailer. The maintainer's commits `e02cd2a`, `f2a8405` and `4c51319` are intact; I checked that each is an ancestor of the head.

## Important: main already contains the original PR

- Upstream main commit `04672f2` (morluto, "fix(javascript): preserve builtin identity and share alias traversal") lists **#2830** among its "reviewed fixes". It brought the kill-signal fix itself (`killSignalName` with `void`, `as` and global-`undefined` handling) and the signal test into main.
- That is why the branch conflicted. Main also changed our test: it asserts only `signalName`, and adds a new "configuration value escapes" block after it.
- **How I resolved it:** I took main's side of the test file. It contains every case we had, so nothing was lost. The source file merged without a conflict.
- **What the PR contains now:** after the merge, the PR's net diff against main is **only the review-fix commit `05429ce`** (2 files, +95/-6). Your PR description should say so: the original #2572 fix landed via `04672f2`, and this PR now carries the follow-up for the three review threads.

## The 3 review threads (facts for your replies)

All three are fixed in commit `05429ce`, in `src/domain/javascript/javascriptSemanticChildProcesses.ts`.

**a. greptile, unused `semanticResolutionBlocked` import (line 18)**
- Removed it; nothing in the file uses it any more.
- Main had already dropped it in `04672f2`, but the merge brought it back on this branch, so `05429ce` removes it again. The file now imports `semanticStableInitializer` instead, which fix (c) needs.

**b. Codex P2, `child.kill(...[])` (line 249)**
- New helper `firstRuntimeArgument`. It walks the arguments and expands spreads of **array literals** (also nested ones) to find the first argument Node actually receives.
- Results:
  - `kill(...[])` has no runtime argument, so it reports "SIGTERM";
  - `kill(...["SIGINT"])` reports "SIGINT";
  - an array hole such as `kill(...[,])` passes undefined, so it reports "SIGTERM".
- A spread of anything else (a parameter, or even a `const` array variable) reports null.
- Tested: `...[]`, `...["SIGINT"]`, `...signals` (a parameter) and `...literals` (a const array variable). The last two report null.

**c. Codex P2, `const signal = undefined; child.kill(signal)` (line 256)**
- New helper `immutableAliasValue`. It follows an identifier to its initializer only when the binding is not `mutable` and `semanticStableInitializer` returns a single unconditional initializer with no destructuring projection. It also follows alias chains and guards against cycles.
- This reuses the existing rule other producers already apply, for example `javascriptSemanticPromises.ts` (`binding.mutable`, single initializer, empty projection). The codebase sets `mutable` to false only for `const`; `let`, `var` and any reassignment set it to true.
- Results:
  - `const unset = undefined; kill(unset)` reports "SIGTERM";
  - `const named = "SIGKILL"; const alias = named; kill(alias)` reports "SIGKILL";
  - `let open = undefined; kill(open)` reports null;
  - a reassigned `let` reports null;
  - `const unset = undefined` inside `function shadowed(undefined)` reports null.
- The earlier shadowed-`undefined` case still reports null.

**Tests:** all of the above are in one new test, "expands literal spreads and follows only immutable signal aliases", in the `child-process signals` block of `src/domain/javascript/javascriptSemanticAnalysis.native_bindings.test.ts`. It **fails** on the merge commit's source (1 failed, 10 passed) and passes with the fix.

## Exact results (Node 24.18.0, after `npm ci`)

- `npm run test:focused --` `javascriptSemanticAnalysis.native_bindings.test.ts`, `childProcessOptionsRegression.test.ts` and `src/application/javascript/JavaScriptSemanticGraphResources.test.ts`: 3 files, 26 tests passed.
- `npm run test:local --` every `src/domain/javascript` and `src/application/javascript` test file: 77 files, 1,851 tests passed.
- `npm run check:fast`: passed (2/2). The first run failed typecheck on my own code (TS2345: a `t.Node` passed to `dataEffectLiteralString`); I fixed it with a `t.isExpression` guard before committing.
- The pre-commit hook (oxfmt, oxlint) and the pre-push hook (`check:fast`) both passed on the real push.
- Not run: `docs:check` (no contract, schema or generated metadata changed).

## Housekeeping

- I did not comment on the PR.
- I unsubscribed this session from PR activity on #2830, and no check-ins are scheduled. The session is idle.
