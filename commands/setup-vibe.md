# Setup vibe

Run **once on a machine that should vibe**: try ideas in a repo without the developer pauses. The text after this command is a GitHub repository URL. This command is not part of `/new-project` or bootstrap.

Do not run it on a developer machine unless the user explicitly says to replace that machine's commands.

## Steps

1. Resolve the URL. If that repo is already cloned and is the current workspace, use it. Otherwise `git clone` it into a directory you confirm with the user, then work in that checkout. Safe to run again on an existing checkout.
2. Copy `rules/business-team-experiments.mdc` from [agent-dev-rules](https://github.com/vcamp314/agent-dev-rules) into `<clone>/.cursor/rules/business-team-experiments.mdc`. If this command is running inside a checkout of agent-dev-rules, copy from that `rules/` directory. Otherwise fetch the raw file from `https://github.com/vcamp314/agent-dev-rules`. The installed copy must have `alwaysApply: true`.
3. Append this line to `<clone>/.git/info/exclude` if it is not already there:

   ```text
   .cursor/rules/business-team-experiments.mdc
   ```

   Do not add the rule to the committed `.gitignore`. Do not commit the rule file.
4. Install commands for this machine only:

   ```bash
   mkdir -p ~/.cursor/commands
   ```

   Copy `commands/submit.md` and `commands/feature-start.md` from agent-dev-rules into `~/.cursor/commands/` (overwrite those two names if present). Do not copy any other command.
5. If `~/.cursor/commands/` contains other command files, list them. Remove them only after the user confirms this machine should have only `/submit` and `/feature-start`.
6. Leave the project's other `.cursor/rules/` files in place (`code-craft`, testing, stack rules, `feature-workflow`, and so on). Do not delete them.

## Reply

Say where the repo is, that the always-on rule is local and uncommitted, and that the machine's commands are `/submit` and `/feature-start`. Remind them that GitHub protection of `main` is a separate admin step.
