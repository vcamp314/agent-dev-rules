# Submit

The only way a vibe machine commits, pushes, and asks a developer to review. Developer machines use `/commit-push` instead. This command refuses `main`.

## Branch

1. The current branch must be `feature/business-ideas/<topic>`, `feature/<topic>`, or `fix/<topic>`. If it is `main` or `master`, stop. Do not create a new branch here and do not merge into `main`.
2. `git fetch` `main`. Do not check out `main`.

## Notes

Before committing, write idea-stage notes for this change (not on every earlier prompt). Record what the user asked for, what the agent assumed, and which approach it took. Label them as assumptions, not approved decisions. Follow `documentation.mdc` placement when that file is present: a short feature `README.md`, and a decision record only when there was a real design fork. Do not pause for the user to confirm the notes.

## Tests

Run the relevant test commands. Record pass, fail, and anything not run. Red tests do not stop this command.

## Critics (report only)

Launch these subagents **in parallel**, once. Pass the idea, the diff against `main`, and pointers to the project's stack rules. Do not auto-fix. Do not re-run them. Do not ask the user to resolve findings. If `/feature-start` already applied a fix loop, this pass still does not fix.

- `functional-verifier`
- `security-auditor`
- `qa-edgecase-verifier`
- `architecture-linter`
- `test-coverage-auditor`

If a critic errors or times out, continue and list it as missing.

## Git

1. Commit this branch's files with a simple why-focused message. Do not apply the perfect-commit gate and do not require a GitHub issue.
2. Do not stage `.cursor/rules/business-team-experiments.mdc`.
3. Push: `git push -u origin HEAD` if no upstream, otherwise `git push`. Never push `main` or `master`. Never force-push.

## Pull request

Create a pull request into `main` with `gh pr create` (`gh` must be authenticated). If `gh` is missing or unauthenticated, stop and ask the user to fix that (`gh auth status`). Do not merge.

Title: make clear this is a submission for developer review.

Body:

- **At-a-glance summary** — what the idea changed.
- **Must review (human)** — each critic `[BLOCKER]`, plus red tests. When tests are red, mark any pass line (`SECURITY_PASSED` and the others) as provisional.
- **Safe to skim / trust automation** — areas whose critic returned a pass line and no blocker. An empty Must-review section means the critics ran and passed, not that they were skipped.
- **Concerns & residual risk** — `[NIT]`s, coverage notes, and any critic that did not finish.
- **Tests** — commands, failures, anything not run.
- **Handoff** — this pull request is a review request. An engineer raises it to the developer bar with `/standardize-pr`. Merging before that leaves the idea short of the developer fix loop.

Use **Review triage** categories in `feature-workflow.mdc` when that file is present (fetch it from agent-dev-rules if needed). Classify blockers as Must review. Do not invent a Safe-to-skim area a critic did not actually pass.

Print the pull request URL, branch name, triage sections, and test results.
