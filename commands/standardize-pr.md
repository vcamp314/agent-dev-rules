# Standardize pull request

Developer command. Bring an existing pull request up to the developer bar. Use it for a `/submit` review request or for any other pull request. Do not run `/feature-start` to receive work that is already in the diff.

The text after this command is a pull request URL or number.

## Checkout

1. `gh pr view` and `gh pr checkout` the pull request (requires `gh` authenticated). If the head branch cannot be checked out, stop and say so.
2. Do not merge it.

## Tests

Run the relevant test commands. Fix failures. After the first failing run, allow **at most 3** fix-and-rerun cycles. If tests are still red, stop, report the failures, and do not start the critic fix loop.

## Critics

When tests are green, follow `multi-critic-protocol.mdc`: launch the five critics in parallel, auto-fix only `[BLOCKER]` findings, at most 3 fix cycles.

Stop and ask the user when a blocker is a product or design choice (what the change should do, or a hard-to-reverse tradeoff). Do not silently rewrite that choice. Apply clear defect fixes (incorrect behavior, missing tests the rules require, security mistakes) within the cycle cap.

If blockers remain after 3 cycles, stop and report them. Do not claim the pull request is ready.

## Notes

Write or update design notes only when the diff has a real design fork, per `documentation.mdc`. Do not run a requirements confirmation interview.

## Update the pull request

Push to the head branch when the remote accepts it. When it does not (a fork you cannot push to), push a branch on the base repo and point the pull request at that branch (`gh pr checkout` / retarget as `gh` allows). Never force-push `main` or `master`.

Update the pull request body so these sections match the code **after** the fixes:

- **At-a-glance summary**
- **Must review (human)**
- **Safe to skim / trust automation**
- **Concerns & residual risk**
- **Tests** — commands and results

Use the Review triage categories in `feature-workflow.mdc`. An empty Must-review section means the critics ran and passed.

This pull request is then the production candidate. Use `/feature-start` only for work you are originating, or when this approach should be discarded and rebuilt.

Print the pull request URL, branch, remaining blockers if any, and test results.
