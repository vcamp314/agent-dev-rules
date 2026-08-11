# New project

Follow the project bootstrap protocol in [agent-dev-rules](https://github.com/vcamp314/agent-dev-rules) `rules/bootstrap-select-rules.mdc` (also pasteable as a Cursor User Rule). Documentation conventions: `rules/documentation.mdc`. Testing day-one: `rules/testing.mdc`.

## Do this

1. If architecture is not yet agreed, stay in / switch to **Plan mode**: ask the decision-tree questions (frontend-only vs monorepo vs single service; stacks; WASM/HTTP/protobuf; CI). Capture rejected alternatives as you go (they become `docs/` decision records). **Testing is always on** for greenfield (harness + minimal passing test); only CI remains optional in the tree.
2. Emit the exact `.mdc` file list to install, always including `feature-workflow.mdc`, `multi-critic-protocol.mdc`, `documentation.mdc`, and `testing.mdc` (+ `frontend-testing.mdc` when React is present).
3. After the user approves the plan, in **Agent mode**: fetch those files from `https://github.com/vcamp314/agent-dev-rules` into `.cursor/rules/`.
4. Scaffold a blank runnable app from the stack rules; init git/`main` if needed; stack `.gitignore`.
5. **Testing (required from day one)** — per `testing.mdc`: wire the stack’s test runner and add **at least one minimal passing test**; run it once and confirm green; document the exact command on the root README.
6. **Documentation (required for greenfield)** — per `documentation.mdc`:
   - Root `README.md`: short what/why, quickstart (compose/run/test one-liners), links into `docs/`.
   - Root `docs/architecture.md`: chosen shape/stacks and layout.
   - Root `docs/decisions.md` (or `docs/decisions/<topic>.md`): greenfield deliberation — options considered, why rejected, revisit-if.
   - Optional `docs/testing.md` if test-stack notes exceed README bullets; still keep command one-liners discoverable from the README.
   - Do **not** invent feature-local READMEs until features exist.
7. Optionally add CI per `ci-github`.
8. Remind the user to install/sync critic agents to `~/.cursor/agents/` (or `.cursor/agents/`) from the repo’s `agents/` folder if missing.

Do not put the full architecture pack into global User Rules—only bootstrap belongs there globally.
