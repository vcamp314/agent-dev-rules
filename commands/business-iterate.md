# Business iterate

Use only when a business contributor invokes this command to try an idea. Developer work stays on `/feature-start` and `feature-workflow`. Invoking this command overrides `feature-workflow`'s confirmation pauses, design-doc gate, multi-critic loop, and three-cycle stop. Architecture, craft, and testing rules in the project's `.cursor/rules/` still apply.

## Branch

1. `git fetch` and integrate latest `main`.
2. If the current branch is already `feature/business-ideas/<topic>`, stay on it.
3. Otherwise create and checkout `feature/business-ideas/<short-kebab>` from `main`. Match an existing topic name when the user is continuing one.
4. Untracked local files are allowed. Stop only on a real merge, rebase, or checkout conflict.

## How to work

Treat the rest of the user's message as the idea to try.

- Write code in the same folders `main` uses (`features/` and the stack's usual layout). Do not add a separate experiment tree.
- Follow installed architecture and craft rules (code-craft, stack layout, how tests are chosen in `testing.mdc`) without asking the user to run the developer protocol.
- Write or update the tests those rules call for. Run the relevant test commands.
- If tests fail, record the command and the failure, then keep going. Do not stop the session the way `feature-workflow` does after three red cycles, and do not block the user on red tests.
- Do not commit or push unless the user asks. If they ask mid-iteration, make a simple why-focused commit. Do not apply the perfect-commit gate.
- Do not open a pull request. That is `/business-finalize`.
- Do not merge this branch to `main`. Promotion onto production paths is a later developer change.

## Reply

Say what changed, where it lives, which tests ran, and any failures (command plus the useful output). Point them at `/business-finalize` when they want a developer to review.
