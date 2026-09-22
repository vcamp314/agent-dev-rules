# Business finalize

Use when a business contributor invokes this command to ask a developer to review the idea branch. Developer `/feature-start` work is unchanged. This command overrides `feature-workflow`'s perfect-commit and verify-stop gates: failing tests are reported, not treated as a reason to refuse the pull request.

## Git

1. The current branch must be `feature/business-ideas/<topic>`. If it is not, stop and say so. Do not create a developer `feature/` branch here.
2. `git fetch` `main`. Do not merge this branch into `main`.
3. If this iteration has uncommitted files, commit them with a simple why-focused message. Do not apply the perfect-commit gate and do not require a GitHub issue.
4. Push: `git push -u origin HEAD` if no upstream, otherwise `git push`. Never force-push to `main`/`master`.

## Tests

Run the relevant test commands for the change. Record pass, fail, and anything not run. Include failing command output that a developer needs. Still open the pull request when tests are red.

## Pull request

Create a pull request into `main` with `gh pr create` (requires `gh` authenticated). If `gh` is missing or unauthenticated, stop and ask the user to fix that (`gh auth status`). Do not merge.

Title: make clear this is a business idea for developer review.

Body, using **Review triage** in `feature-workflow.mdc` (fetch that file from agent-dev-rules if the project does not have it):

- **At-a-glance summary**
- **Must review (human)**
- **Safe to skim / trust automation**
- **Concerns & residual risk**
- **Tests** — commands run, failures, anything not run
- **Handoff** — this branch uses the same folders as `main` and is experiment work. Merging it as-is would put that code on `main`. A developer promotes the behavior they want with the normal feature flow (`/feature-start`) and removes the experiment when it is obsolete. This pull request is the review request, not that promotion.

Print the pull request URL, branch name, triage sections, and test results.
