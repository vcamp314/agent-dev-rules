# Plan day

Review the backlog(s), triage them into agent vs human work, and hand me back an ordered list of what **I** should do while the agents work. Execute **Phase A (Plan)** of `.cursor/rules/daily-planning.mdc`, using config from `.cursor/rules/backlog-sources.mdc`. If either file is missing, fetch it from https://github.com/vcamp314/agent-dev-rules before continuing.

## Do this

1. Resolve effective config (sources, priority, triage pipelines, estimator, human budget, agent concurrency, plan store). Honor precedence: **this prompt's text > project override > central default**. Echo the key choices.
2. Gather open **items** across all configured sources/repos (cross-project) and the personal `manual` source; merge and de-dupe.
3. Score items `urgency × impact` (structured first, then the raise-only text scan — quote the phrase when it changes a tier).
4. **Triage** each item into executor-typed tasks (`agent` / `agent-assist` / `human`) per its pipeline; tasks inherit the item's priority (review/deploy get the small `review_boost`).
5. Estimate the **human** effort per human/agent-assist task; build the **agent queue** (bounded by concurrency) and the **human list** (bounded by `human_budget`). **Reserve** human budget for the reviews/deploys that agent work will induce — dispatching agents is not free for me.
6. Present the agent queue (with the human load each induces) and my **ordered task list** in two bands: **actionable now** and **expected later today** (pending agent output); show running total vs budget and what's deferred.
7. **Iterate**: treat the rest of my message (and replies) as prioritization/scoping/overrides (incl. "that one's human-only", "today only", "remember this chore"). Re-plan until I explicitly approve.
8. On approval: persist any new personal chores I typed as issues in the private `home_repo` (de-dupe; skip if "today only"), write per-item tasks to the store, render the plan, and **end your reply with my ordered task list**.

## Cursor / cloud notes

- I'll often run this from a Cursor cloud agent with **no project in context** — items span every backlog. Don't rely on a project folder.
- Cloud agents are **ephemeral and isolated**; `/start-day` runs as a *different* agent later. So the plan, tasks, and personal items **must** go to the remote **private** `home_repo` (via `gh`), never local disk. The human list in the `Day plan` issue is a re-rendered projection, not the source of truth.
- Personal/finance/paperwork items are sensitive: keep them in the **private** repo; keep secrets/account numbers out of issue text.

Treat the rest of my message after this command as extra prioritization context, overrides, or new chores to add.
