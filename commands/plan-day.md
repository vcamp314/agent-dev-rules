# Plan day

Review the backlog(s) and propose a prioritized, capacity-bounded shortlist for **today**. Execute **Phase A (Plan)** of `.cursor/rules/daily-planning.mdc`, using config from `.cursor/rules/backlog-sources.mdc`. If either file is missing, fetch it from https://github.com/vcamp314/agent-dev-rules before continuing.

## Do this

1. Resolve effective config (sources, priority, estimator, budget, plan store). Honor precedence: **this prompt's text > project override > central default**. Echo the key choices you used.
2. Gather open backlog items across **all** configured sources/repos (cross-project) and merge them.
3. Score `urgency × impact` from labels/dates/milestones, then apply the **secondary text scan** (raise-only; quote the phrase when it changes a tier).
4. Estimate the **human** review→deploy effort per candidate and fill the shortlist up to the daily budget (leave headroom for same-day deploy).
5. Present the ordered shortlist with scores, rationale, text-scan flags, estimates, and the running total vs budget; list what was deferred and why.
6. **Iterate**: treat the rest of my message (and my replies) as prioritization/scoping context or overrides. Re-plan until I explicitly approve.
7. On approval, write the final shortlist to the configured plan store and tell me where it is.

## Cursor / cloud notes

- I will often run this from a Cursor cloud agent with **no specific project in context** — the shortlist spans every backlog. Do not rely on a project folder.
- Cursor cloud agents are **ephemeral and isolated**, and `/start-day` will usually run as a *different* agent on a *different* VM. So the approved shortlist **must** go to the remote plan store (default: a tracking GitHub issue in the configured `home_repo`), never to local disk. Use the authenticated `gh` CLI for GitHub sources and the store.

Treat the rest of my message after this command as extra prioritization context / overrides.
