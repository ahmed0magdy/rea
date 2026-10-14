STATUS: DONE - maintainer pushed to the PR branch (head 4c51319); all checks green there; nothing for me to push
UPDATED: 2026-10-14T05:57:09Z
BRANCH: contrib/child-process-call-facts
MODE: Auto

## What changed since my last update

- **Who pushed:** upstream maintainer **morluto** pushed to `contrib/child-process-call-facts`. The remote head is now `4c51319`, not my `43c67f7`.
  - They rebased our commit onto a newer upstream main. It is now `e02cd2a`, still authored by ahmed0magdy, with morluto as committer.
  - They added two commits on top:
    - `f2a8405` "fix(javascript): unwrap default child kill signals". It is attributed to github-actions[bot], committed by morluto.
    - `4c51319` "fix(javascript): preserve default void kill signals". It is attributed to "Codex" (morluto's address), committed by morluto.
- **What their commits do:** in `killSignalName` they:
  - unwrap the argument first (`unwrapJavaScriptExpression`), so `kill(undefined as NodeJS.Signals | undefined)` reports SIGTERM;
  - treat any `void` expression as SIGTERM (`kill(void 0)`, `kill(void signal)`);
  - replace our inline three-part check with the shared `isUnshadowedGlobal(expression, state, "undefined")` helper from `javascriptSemanticState.ts`.

  The test gains 3 rows, all SIGTERM. The shadowed `function shadowed(undefined)` case still reports null.
- **What I did:** I did not force-push or change anything. I reset my local branch to the remote head (`4c51319`); my old local `43c67f7` is superseded and needs no action.

## Checks on the maintainer's head 4c51319 (Node 24.18.0, after `npm ci`)

- `npm run test:focused --` `javascriptSemanticAnalysis.native_bindings.test.ts`, `childProcessOptionsRegression.test.ts` and `src/application/javascript/JavaScriptSemanticGraphResources.test.ts`: 3 files, 19 tests passed.
- `npm run test:local --` every `src/domain/javascript` and `src/application/javascript` test file: 67 files, 1,544 tests passed.
- `npm run check:fast`: **passed** (2/2 tasks). Upstream fixed main's earlier lint and typecheck errors in `6e25cc3` "fix(tests): clear lint and typecheck errors on main".

## For you (owner)

- **The PR is now partly the maintainer's work.** When reviewers ask about `void` or `as` handling, you can say the maintainer added it in `f2a8405` and `4c51319`.
- **Update the "Explain it to the owner" section (from the previous update) for their change:**
  - `kill(void 0)` and `kill(undefined as …)` now report SIGTERM, so `void 0` is no longer a gap.
  - The global-`undefined` check now goes through `isUnshadowedGlobal` (shared helper) rather than our inline copy of the `javascriptSymbolicBindings.ts` rule.
  - Still not covered: numeric signals (`kill(9)`) and signals stored in a constant variable, which still report null.
- **PR #2748** ("keep spread child argv counts unknown") has been **merged** into main as `b19f9e4`, so the overlap question is resolved. The maintainer's rebase includes it.
- **PR description edits** from the previous update still apply. Also mention `void`/`as` handling and that `check:fast` is now green, and drop the note about red `check:fast` and the `--no-verify` push.
- **Rules I'm following:** I will not open, reopen or comment on any PR or issue, and no check-ins or loops are scheduled. The session is idle.
