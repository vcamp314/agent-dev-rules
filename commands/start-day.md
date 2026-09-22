# Start day

Put agents to work on today's **agent queue**, and keep my human task list updated as they finish. Execute **Phase B (Dispatch)** of `.cursor/rules/daily-planning.mdc`, using config from `.cursor/rules/backlog-sources.mdc`. If either file is missing, fetch it from https://github.com/vcamp314/agent-dev-rules before continuing.

## Do this

1. Read today's plan and tasks from the plan store (default: the `Day plan` issue + per-item tasks in the private `home_repo`). **Do not re-plan.** If none exists for today, stop and tell me to run `/plan-day` first.
2. Start workers only for `agent` / `agent-assist` tasks whose dependencies are satisfied, bounded by `agent_concurrency`. **Never dispatch `human` tasks** (chores, human code review, QA/deploy) — those stay on my list.
3. Each code worker follows `.cursor/rules/feature-workflow.mdc`, runs the automated `auto_review` (multi-critic) when green, and **stops at the human-review handoff — it does not commit** (commit stays with `/commit-push` or an explicit ask). `agent-assist` workers produce a draft for my approval.
4. As each worker finishes or blocks: update its task status at the source of truth, **materialize the induced human task** (e.g. `human_review` with the branch/diff link, then `qa_deploy`), and **re-render my ordered task list**, inserting the new task at its priority (+`review_boost`) position. A blocked agent task becomes a human task describing the blocker.
5. Summarize per task: what was done, where the change lives, the new review/deploy item now on my list, and anything blocked. Don't mark anything deployed — QA/deploy is mine.

## Cursor / cloud notes

- Dispatch workers as Cursor **background agents** (one per task), each on its **own branch**, parallel up to the concurrency cap. Prefer one branch (later one PR) per task.
- Agents are ephemeral/isolated, so the private plan store is the only reliable hand-off: read task state from it and write status/new tasks back to it. The human list is a projection re-rendered from task statuses (pull, not push).
- Deployment to the testing environment remains a **human** step sized into `human_budget`.

Treat the rest of my message after this command as scoping overrides (e.g. limit which tasks to start, or set the concurrency cap).
