STATUS: WORKING - #2571 + #2572 fixed and tested locally; pushing contrib branch next
UPDATED: 2026-10-13T07:29:35Z
BRANCH: contrib/child-process-call-facts
MODE: Auto

## Progress

- Branch `contrib/child-process-call-facts` created from main (60f771a).
- Regression tests from the #2571 and #2572 issue bodies were added to `src/domain/javascript/javascriptSemanticAnalysis.native_bindings.test.ts`. They fail on unfixed main (2 failed, 3 passed).
- Fix in `src/domain/javascript/javascriptSemanticChildProcesses.ts`. Focused tests, all 62 JavaScript domain/application test files, and `npm run check:fast` pass.
- Next: push the branch, then post the PR notes and the inline-script issue draft here with STATUS: DONE.
