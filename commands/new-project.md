# New project

Follow the project bootstrap protocol in [agent-dev-rules](https://github.com/vcamp314/agent-dev-rules) `rules/bootstrap-select-rules.mdc` (also pasteable as a Cursor User Rule).

## Do this

1. If architecture is not yet agreed, stay in / switch to **Plan mode**: ask the decision-tree questions (frontend-only vs monorepo vs single service; stacks; WASM/HTTP/protobuf; testing/CI).
2. Emit the exact `.mdc` file list to install, always including `feature-workflow.mdc` and `multi-critic-protocol.mdc`.
3. After the user approves the plan, in **Agent mode**: fetch those files from `https://github.com/vcamp314/agent-dev-rules` into `.cursor/rules/`.
4. Scaffold a blank runnable app from the stack rules; init git/`main` if needed; stack `.gitignore`; document local test commands; optionally add CI per `ci-github`.
5. Remind the user to install/sync critic agents to `~/.cursor/agents/` (or `.cursor/agents/`) from the repo’s `agents/` folder if missing.

Do not put the full architecture pack into global User Rules—only bootstrap belongs there globally.
