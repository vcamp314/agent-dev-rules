# Init planning

Set up (or refresh) a **planning home repo** for the daily task-management workflow: install the planning rules and commands into it and fill their placeholders with your values. This is the one-time setup that `/plan-day` and `/start-day` depend on. Config schema and placeholder meanings are defined in `backlog-sources.mdc` — treat that file as the source of truth for field names.

## Inputs

Read values from the rest of my message (accept `key=value` pairs or plain wording). For any **required** value that is missing, list exactly what you need and **ask me** before writing anything. Required unless a default is shown:

- `home_repo` — the planning home repo `<owner>/<name>` that will hold the cross-project config + daily plans.
- `backlog_repos` — one or more `<owner>/<repo>` to pull backlog from (comma/space separated). Multiple expected (cross-project).
- `source_type` — `github` (default) | `linear` | `other`. For non-GitHub, also collect that tracker's minimal fields (e.g. Linear `teams`, `states`) and note it's an extension.
- `store_type` — `github_issue` (default) | `github_file` | `gist`.
- `daily_review_deploy_budget` — default `4h`. Also accept `max_tasks`, `estimator` if I provide them.

Echo the resolved values and **confirm with me** before writing.

## Do this

1. **Resolve inputs** (above). Ask for anything required that's missing; confirm the final set.
2. **Ensure the rules/commands exist locally.** Copy `daily-planning.mdc` and `backlog-sources.mdc` into the target's `.cursor/rules/`, and `plan-day.md`, `start-day.md`, and this `init-planning.md` into `.cursor/commands/`. If any are missing, fetch them from https://github.com/vcamp314/agent-dev-rules first.
3. **Fill placeholders** in the copied `backlog-sources.mdc` only (the protocol rule has none):
   - replace the `sources[].github.repos` placeholder list (`<owner>/<repo-a>`, `<owner>/<repo-b>`) with `backlog_repos`;
   - set `planning.home_repo` to `home_repo`;
   - set `planning.store.type` (and `title`/`path`/`labels` as relevant) to `store_type`;
   - set `capacity.daily_review_deploy_budget` (and `max_tasks`/`estimator` if provided);
   - for `linear`/`other`, uncomment and fill that source block and replace `<TEAM_KEY>`.
   Leave every other field at its default. **Do not invent values** — only write what I supplied or confirmed.
4. **Verify** no unresolved placeholders remain in the **active** (non-commented) config — no `<owner>`, `<repo`, `<name>`, or `<TEAM_KEY>` on any line that isn't a `#` comment example — and the config block still parses. Commented extension examples (e.g. the `linear` block) may keep their placeholders.
5. **Prepare the store (idempotent, non-destructive):** for `github_issue`, ensure the `day-plan` label exists in `home_repo`; for `github_file`, ensure the `plans/` directory exists. Skip anything that already exists.
6. **Persist to the home repo.** Because planning/execution run as separate, project-less sessions, the filled config must be reachable remotely: commit the filled `.cursor/rules/` + `.cursor/commands/` into `home_repo` (or, if I'm not working inside its checkout, tell me the exact files to commit there). For the `gist` store, write the config to the gist instead.
7. **Report** the resolved config, where each file was written, and remind me that inline-prompt and project-level overrides still apply per `backlog-sources.mdc` precedence.

## Cursor / cloud notes

- Use the authenticated `gh` CLI for repo/label/issue and gist operations.
- If `home_repo` does not exist yet, offer to create it with `gh repo create` — **ask first**; never create or push to a repo without my confirmation.
- The point of persisting into `home_repo` is that later ephemeral, project-less agents (`/plan-day`, `/start-day`) can fetch the same config and plan store regardless of which product repo they're near.

Treat the rest of my message after this command as the input values (e.g. `home_repo=me/planning backlog_repos=me/web,me/api budget=6h`).
