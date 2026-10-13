STATUS: DONE - contrib/child-process-call-facts re-authored to ahmed0magdy and force-pushed (fixes #2571, #2572)
UPDATED: 2026-10-13T13:49:38Z
BRANCH: contrib/child-process-call-facts
MODE: Auto

## Authorship update

- Set this repo's `user.name` to `ahmed0magdy` and `user.email` to `110637215+ahmed0magdy@users.noreply.github.com`.
- Re-authored the single commit: `9be852c` became `cb1a936`. Author and committer are now ahmed0magdy. The file contents are identical to what was pushed before.
- The `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>` trailer is kept. There is no `Claude-Session:` trailer.
- Re-ran the checks:
  - `npm run test:focused -- src/domain/javascript/javascriptSemanticAnalysis.native_bindings.test.ts`: 5 passed.
  - `npm run check:fast`: 2/2 tasks passed.
- Pushed with `git push --force-with-lease` to `contrib/child-process-call-facts` only. No pull request is open.

## Unchanged from the previous update

- The fix, the PR notes, the AI disclosure and the inline-script issue draft are as reported earlier in chat. Only the commit hash changed: it is now `cb1a936`.
- PR title: `fix(javascript): read child-process options and kill signals as written`. Fixes #2571. Fixes #2572.
- AI disclosure line: "AI-assisted: implemented and tested with Claude Code; I reviewed the diff and validation results."
