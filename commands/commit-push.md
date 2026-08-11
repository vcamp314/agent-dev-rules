# Commit push

Commit and push **only this thread’s** work. Do not wait for extra confirmation beyond what git safety requires.

Non-trivial commits should bundle **implementation + tests + documentation + an issue link**. Docs stay in-repo per `documentation.mdc`; the GitHub issue holds thread/context and links back to those docs.

## Bypass (not every commit needs to be “perfect”)

Skip the full perfect-commit gate when **either**:

- The user explicitly bypasses: `/commit-push --simple`, or the prompt says `bypass perfect` / `trivial` / `simple commit`, **or**
- You classify the change as **simple**: typo/comment-only, pure formatting, chore with no behavior change, or a docs-only nit that does not alter product behavior. Say briefly that you are using the simple path.

On the simple path: stage thread files → short why-focused message (issue link optional) → commit → push. Still never commit unrelated files.

## Perfect-commit path (default for behavioral / design work)

### 0. Classify and resolve the issue

1. Decide perfect vs simple (above). If perfect:
2. Resolve a **GitHub issue** in this order:
   1. Issue number/URL already in this conversation or in the text after `/commit-push`
   2. Else create one with `gh issue create` (requires `gh` installed and authenticated for the repo’s remote)
   3. Else **stop** and ask the user for an issue URL/number, or to install/auth `gh` (`gh auth status`)

When creating an issue, use a body that captures useful thread context (not a chat dump):

- **Background** — why this change
- **State of play** — what was true before (link key files/docs if helpful)
- **Decisions** — short summary; link in-repo paths under `docs/` or feature `README.md` / `docs/decisions.md`
- **Links** — related PRs, docs, prior issues
- Optional: notes on tests/docs paths touched

Print the issue URL/number and use it below.

### 1. Completeness gate

Before staging, for this thread’s change verify (fix → fix or stop and report):

| Bundle | Expectation |
|--------|-------------|
| Implementation | Intended code changes are present |
| Tests | Behavioral changes include or update tests; relevant tests were run and pass (or explain why none apply) |
| Documentation | Behavior/decision changes updated docs per `documentation.mdc` (or explicitly N/A) |
| Issue | A GitHub issue exists and will be referenced in the commit message |

### 2. Atomic commits

Prefer **one focused commit** for one coherent change (easy to review/revert). If this thread mixed unrelated concerns, split into separate focused commits (each with its own message; share the same issue link when they belong to the same ticket). Do not mix unrelated user work from outside this thread.

### 3. Stage, commit, push

1. Identify files that belong to **this conversation’s change** (diff vs branch base, and/or paths you edited in-thread). Ignore unrelated dirty or untracked files.
2. `git add` **only those paths** (include tests + docs that belong to the change).
3. Draft a concise commit message focused on **why / outcome intent**, not a file list. Prefer 1–2 sentences. Match the repo’s recent commit style (`git log`). **Include the issue reference** (e.g. `Fixes #123` or `See #123` + URL if useful).
4. Commit with a HEREDOC message. Never skip hooks. Never update git config.
5. Push: `git push -u origin HEAD` if no upstream; else `git push`. Never `--force` to `main`/`master`. Do not amend unless the user’s/commit safety rules explicitly allow and conditions are met.
6. Print the branch name, issue URL/number, and remind the user to open the PR on GitHub (PR creation stays manual unless they ask otherwise). Suggest the PR body link the same issue and mention tests/docs briefly.

If there is nothing to commit for this thread, say so and stop.
