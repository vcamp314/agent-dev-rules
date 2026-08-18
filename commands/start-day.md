# Start day

Put agents to work on the **approved** shortlist produced by `/plan-day`. Execute **Phase B (Dispatch)** of `.cursor/rules/daily-planning.mdc`, using config from `.cursor/rules/backlog-sources.mdc`. If either file is missing, fetch it from https://github.com/vcamp314/agent-dev-rules before continuing.

## Do this

1. Read today's approved shortlist from the configured plan store (default: the tracking GitHub issue in `home_repo`). **Do not re-plan.** If none exists for today, stop and tell me to run `/plan-day` first.
2. For each not-yet-done task, dispatch a worker to implement it, bounded by a **concurrency cap** so review/deploy stays feasible within the day's budget.
3. Each worker follows `.cursor/rules/feature-workflow.mdc` for its linked backlog item and **stops at the manual-QA / review handoff — it does not commit** (commit stays with `/commit-push` or an explicit ask).
4. Update each task's status in the plan store as workers finish or block (check the box; note branch or blocker).
5. Summarize per task: what was done, where the change lives, the remaining human review + deploy step, and anything blocked.

## Cursor / cloud notes

- Dispatch workers as Cursor **background agents** (one per task), each on its **own branch**, so they run in parallel up to the concurrency cap. Prefer one branch (and later one PR) per task.
- Because agents are ephemeral/isolated, the plan store is the only reliable hand-off from `/plan-day` — read task state from it, and write status back to it.
- Deployment to the testing environment remains a **human** step (the budget is sized for it); do not mark tasks deployed.

Treat the rest of my message after this command as scoping overrides (e.g. limit which tasks to start, or set the concurrency cap).
