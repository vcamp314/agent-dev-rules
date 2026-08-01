# Commit push

Commit and push **only this thread’s** work. Do not wait for extra confirmation beyond what git safety requires.

## Steps

1. Identify files that belong to **this conversation’s change** (diff vs branch base, and/or paths you edited in-thread). Ignore unrelated dirty or untracked files (local scripts, experiments, etc.).
2. `git add` **only those paths**.
3. Draft a concise commit message focused on **why / outcome intent**, not a file list. Prefer 1–2 sentences. Match the repo’s recent commit style (`git log`).
4. Commit with a HEREDOC message. Never skip hooks. Never update git config.
5. Push: `git push -u origin HEAD` if no upstream; else `git push`. Never `--force` to `main`/`master`. Do not amend unless the user’s/commit safety rules explicitly allow and conditions are met.
6. Print the branch name and remind the user to open the PR on GitHub (PR creation stays manual unless they ask otherwise).

If there is nothing to commit for this thread, say so and stop.
