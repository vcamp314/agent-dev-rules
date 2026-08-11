# Feature start

Execute the full feature/fix delivery protocol in `.cursor/rules/feature-workflow.mdc` (from agent-dev-rules). If that file is missing, fetch it from https://github.com/vcamp314/agent-dev-rules before continuing. Documentation placement: `.cursor/rules/documentation.mdc` (fetch with the workflow rules if missing).

## Follow every step in order

1. **Git prep** — fetch/integrate `main`; untracked local files are OK (never refuse only for untracked; never auto-gitignore them). Stop only on real merge/checkout conflicts. Create `feature/…` or `fix/…` branch matching repo style.
2. **Requirements** — break the user ask into **User-specified** vs **Inferred** (infer only sensible gaps). Label inferred items. **Pause for confirmation** before tests.
3. **Design documentation** — per `documentation.mdc`: colocated feature `README.md` (+ optional `<feature>/docs/`); system-wide notes in root/`docs/` or service `docs/`. Record design decisions (options rejected + why). **Fixes must update** docs for modified behavior. Pause if a new tradeoff needs a human call.
4. **Test design** — per `testing.mdc` (E2E/L1/L2/unit leaves only).
5. **Red** — write tests; confirm correct failure.
6. **Green** — implement per project architecture rules; compose from leaves; keep docs aligned if decisions change.
7. **Verify** — at most **3** fix-and-rerun cycles after first red→green; then escalate to the user (no multi-critic if still red).
8. **Multi-critic** — only when green; follow `multi-critic-protocol.mdc` (parallel critics; max 3 blocker fix cycles); pass approved requirements + design-doc pointers.
9. **Hand off** — manual QA summary; list docs created/updated; **do not commit** until `/commit-push` (perfect commit + issue unless `--simple`).

Treat the rest of the user’s message after this command as the feature/fix request.
