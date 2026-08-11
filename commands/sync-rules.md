# Sync rules

Refresh architecture and workflow rules from [agent-dev-rules](https://github.com/vcamp314/agent-dev-rules).

## Steps

1. Determine which files are already in this project’s `.cursor/rules/`.
2. Fetch the matching files (and any newly selected preset files the user wants) from the public repo’s `rules/` directory.
3. Show a brief diff summary of what would change; do **not** overwrite silently if the user has local edits—confirm before replacing.
4. Always offer to refresh `feature-workflow.mdc`, `multi-critic-protocol.mdc`, `documentation.mdc`, and `testing.mdc` (+ `frontend-testing.mdc` when applicable) alongside the stack preset.
5. Optionally sync `agents/*.md` into `.cursor/agents/` and remind the user to copy into `~/.cursor/agents/` for global use; same for `commands/` → `~/.cursor/commands/` if they want updated slash commands.

Do not delete unrelated local rules the user added unless they ask.
