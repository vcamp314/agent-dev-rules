# Design decisions

Decisions for **this** repository (agent-dev-rules) and the workflow rules it ships. Prefer edit-in-place when circumstances change.

Inspired in part by Simon Willison’s *The Perfect Commit* (implementation + tests + documentation + issue context in one coherent change). The normative behavior lives in `commands/commit-push.md` and related rules — not in an external URL — so the workflow stays usable if that page disappears. Historical reference: `https://simonwillison.net/2022/Oct/29/the-perfect-commit/`

---

### Systematic project documentation (README + docs/)

- **Context:** Long Ask/Plan deliberation about features was evaporating; future agents re-litigated discarded options. No prior rule covered design-decision docs (only operational “document in README” notes).
- **Chosen:**
  - Short `README.md` landing pages at every level (repo, service, feature ownership folder).
  - Depth in a sibling `docs/` folder, created lazily; README links to it.
  - Feature-owned docs **colocated** under the ownership folder (`README.md` + optional `docs/decisions.md`).
  - System-wide docs at repo-root `docs/` (or `backend/<service>/docs/` in a monorepo when the concern is one service).
  - Fixes/improvements **must** update matching docs in the same change.
  - Encode this in `documentation.mdc`, wire into `feature-workflow` (step after requirements) and `/new-project` / bootstrap.
- **Alternatives rejected:**
  - **Chat-only / commit-message essays** — not discoverable in-repo; don’t survive as a stable map for agents.
  - **Wiki or docs outside the repo** — drifts from code; harder to review with the same PR.
  - **One fat README everywhere** — hard to scan; GitHub subdirectory README works better as a short landing page.
  - **Always create empty `docs/`** — ceremony; create only when depth warrants it.
  - **Separate `DECISIONS.md` without a README index** — less visible when browsing folders on GitHub.
  - **Put all deliberation only in GitHub issues** — great for thread/media; weaker clone/blame archival than in-repo decision records. Issues complement docs (see perfect commit) rather than replace them.
- **Consequences:** Consumer projects install `documentation.mdc`; greenfield scaffolds root README + `docs/architecture.md` + `docs/decisions.md`. Agents are steered away from reintroducing rejected paths unless **Revisit if** matches.
- **Revisit if:** A host platform stops rendering subdirectory READMEs usefully; or teams standardize on a different ADR tooling that should replace the markdown template.

---

### Perfect commit gate on `/commit-push`

- **Context:** Feature workflow produced code, tests, and docs, but `/commit-push` only staged “this thread’s files” with a why-focused message — no completeness check, no issue link. Wanted alignment with the perfect-commit idea without depending on an external article in the command text.
- **Chosen:**
  - Default path: verify **implementation + tests + docs + GitHub issue** before commit; fix gaps or stop and report.
  - Issue from conversation/prompt → else `gh issue create` (background, state of play, decisions summary + links to in-repo docs) → else stop and ask.
  - Commit message references the issue (`Fixes #N` / `See #N`).
  - Prefer focused/atomic commits; split mixed concerns.
  - Bypass: `/commit-push --simple` (or equivalent wording), or agent auto-classifies trivial changes (typo, format-only, chore with no behavior change).
  - Normative text stays self-contained in `commands/commit-push.md` (no required live link to the article).
- **Alternatives rejected:**
  - **Issue-only context, no in-repo docs** — loses durable “why” next to code; we keep both (docs durable, issue for thread).
  - **Long commit messages as the primary archive** — poorer discoverability than docs + issue; harder for agents to find later.
  - **Always require perfect commits** — too heavy for typos/nits; bypass + auto-simple path required.
  - **Cookiecutter / template repo for perfect commits** — out of scope; behavior belongs in the slash command.
- **Consequences:** Agents need `gh` authenticated on the user’s machine to auto-create issues. Incomplete bundles should not be pushed on the perfect path.
- **Revisit if:** Cursor or GitHub workflows make issue creation unreliable for the user; or the team adopts a different tracker (then generalize “issue” to that tracker).

---

### Day-one test harness on greenfield

- **Context:** Bootstrap required a runnable hello-path and optional testing rules; a project could ship without a green `npm test` / `go test` / `pytest` / `cargo test`. Perfect-commit style work is easier when a harness already exists.
- **Chosen:** Always install `testing.mdc` on greenfield (+ `frontend-testing` when React). Scaffold the stack’s unit runner with **at least one minimal passing test**, run it once during scaffold, document the command on the README. CI remains optional. Full E2E/Playwright need not block day one.
- **Alternatives rejected:**
  - **Testing remains opt-in forever** — too easy to defer the harness; later “first test” is expensive.
  - **Mandate full E2E on day one** — overkill for blank scaffolds; unit smoke is enough.
  - **Ship cookiecutter templates from this repo** — user preferred a rule mandate, not a template product.
- **Consequences:** Decision tree asks about CI, not “tests now?”. Presets always list `testing`.
- **Revisit if:** A stack has no practical unit runner for a hello scaffold (unlikely); or day-one policy should differ for throwaway spikes (use an explicit escape hatch if that becomes common).

---

### Modular presets over a mega-rule

- **Context:** One file with every stack’s conventions pollutes context and steers agents toward the wrong stack.
- **Chosen:** Split rules by include boundary; bootstrap emits an exact file list; User Rules hold only bootstrap (plus optional one-liners).
- **Alternatives rejected:** Dumping the full pack into global User Rules; single mega `.mdc` with conditionals for every stack.
- **Consequences:** `/sync-rules` and `/new-project` must keep workflow files (`feature-workflow`, `documentation`, `multi-critic-protocol`, `testing`) in the always-install set.
- **Revisit if:** Cursor gains first-class remote rule subsets that replace manual preset lists.

---

### Daily planning: cross-project shortlist + remote plan store

- **Context:** Wanted a start-of-day command to review backlogged tasks and propose a prioritized plan (by **urgency × impact**), plus a second command to put agents to work on the approved shortlist. Priority evidence is mainly labels/dates, but stale metadata is common. Planning must span **multiple projects/repos** and will usually run from a Cursor cloud agent with **no project in context**; the two commands run as **separate, ephemeral, isolated** sessions.
- **Chosen:**
  - Split into tool-agnostic **rules** (`daily-planning.mdc` protocol + `backlog-sources.mdc` config) and Cursor-specific **commands** (`plan-day`, `start-day`); no Cursor terms leak into the rules.
  - Priority = `urgency × impact` from structured signals, with a **secondary, raise-only text scan** for cues like "ASAP" that outrun stale labels (capped, negation-aware, always surfaced with the quote).
  - Capacity-bound the shortlist to a **human review→deploy budget** (default ~4h, serial) via a **swappable estimator** (default educated guess; future statistical buckets fit the same contract). Overridable by inline prompt or project rule.
  - Store the approved shortlist in a **remote, project-independent plan store** — default a tracking GitHub issue in a designated `home_repo` (alternatives: committed file, gist) — so a later, separate session can read it.
  - Ship an **`/init-planning`** setup command: install the planning rules/commands into a chosen `home_repo` and fill the placeholders (backlog repos, home repo, store, budget) from supplied values, prompting for anything missing. Keeps `backlog-sources.mdc` as the schema source of truth; the command only fills what the user confirms.
  - Opt-in: not added to bootstrap presets; listed in README + catalog and syncable via `/sync-rules`.
- **Alternatives rejected:**
  - **Local `.cursor/plans/{date}.md` hand-off** — fails cross-project (no project context) and across ephemeral/isolated cloud agents (no shared disk between the two commands).
  - **Text-only or label-only priority** — text-only is noisy/unauditable; label-only misses stale metadata. Structured-primary + capped raise-only text scan balances both.
  - **Bound the day by agent time** — the real constraint is human review + deploy bandwidth, so budget in human hours.
  - **Cursor-specific rules** — would violate the tool-agnostic-body principle; Cursor specifics (cloud ephemerality, background-agent dispatch) live only in the command files.
  - **Force it into bootstrap** — extra machinery most projects don't need; keep opt-in.
- **Consequences:** Needs `gh` authenticated and a designated `home_repo` (or a gist fallback) reachable from any agent. Estimates are explicitly conservative guesses, not commitments; deployment stays a human step within the budget.
- **Revisit if:** A statistical estimator replaces the default guess (slot behind the estimator contract); or a non-GitHub tracker becomes primary (generalize the store beyond issue/file/gist).

---

### Triage items into executor-typed tasks (agent queue + human list)

- **Context:** Planning was extended to cover non-code work (chores, paperwork, finances) that agents can't do, and even code items imply human-only steps (code review, QA/deploy). A flat "issue shortlist" couldn't express "this issue → some agent work + some human work", nor keep the human's day realistic once agents start generating reviews.
- **Chosen:**
  - Model an **item** (a code issue, or a personal chore) as triaging into one or more **tasks**, each with an `executor` (`agent` | `agent-assist` | `human`), `depends_on`, `status`, and an inherited priority. Pipelines per item `kind` live in `backlog-sources.mdc` (`triage.pipelines`).
  - Code pipeline: `implement (agent) → auto_review (agent, via multi-critic) → human_review (human) → apply_fixes (agent) → qa_deploy (human)`, so automated critics run before the human review task is emitted.
  - Planning produces **two queues**: an **agent queue** (bounded by `agent_concurrency`) and the **human's single ordered task list** (bounded by `human_budget`). `/plan-day` ends by handing the user their ordered list.
  - **The human day is the scarce resource:** `human_budget` bounds the whole human list, and selecting agent work must **reserve** budget for the `human_review`/`qa_deploy` it induces (`reserve_for_induced_reviews`). Agent time is not charged to the human budget.
  - The human list is a **projection** re-rendered from per-task status (source of truth on each item via `task_storage` sub-issues/checklist), shown in **actionable-now** vs **expected-later** bands. As agents finish, `/start-day` materializes the induced human task and re-renders (pull, no daemon).
  - Personal items are **stored as issues in the private `home_repo`** (a `manual` source); chores typed into `/plan-day` are persisted there on approval (deduped), so they survive to later days.
  - **Scalability seam (deferred):** task `assignee` (today `agent` | `human:me`) + per-item `visibility`, all private by default — multi-person and public/private split can be layered on without changing the item→task model.
- **Alternatives rejected:**
  - **Flat issue shortlist with a per-issue human/agent flag** — can't represent one issue producing both agent and human work, nor the review/deploy that agent work induces.
  - **Dispatch every task through `feature-workflow`** — chores and human code review can't run TDD/critics; `human` tasks must never be dispatched to a coding worker.
  - **Bound the day by agent concurrency alone** — ignores that each dispatched feature manufactures human review/deploy load; the day would blow up even though "agents did the work".
  - **Push-update the human list from finishing agents** — fragile across ephemeral, isolated agents; re-render as a projection on demand (pull) instead.
  - **Personal tasks in committed files as the primary backlog** — lose native open/closed lifecycle, query, comments, and per-item concurrency; keep files only for recurring templates if needed.
  - **Build multi-person now** — introduces public/private data separation prematurely; defer behind the `assignee`/`visibility` seam.
- **Consequences:** `home_repo` must be **private** (holds finance/paperwork). `capacity` config generalized from `daily_review_deploy_budget` to `human_budget` + `agent_concurrency` + `reserve_for_induced_reviews`; estimator semantics vary by executor. Per-item tasks are stored as sub-issues/checklists; the day-plan issue is a re-rendered view, not the source of truth.
- **Revisit if:** Task volume outgrows sub-issues/checklists (consider a projects board); or a real-time updater replaces pull-refresh; or multi-person collaboration is prioritized (activate the `assignee`/`visibility` seam with a public/private policy).

---

### This repo’s own docs

- **Context:** After adding `documentation.mdc`, agent-dev-rules itself had no `docs/` capturing the above deliberation.
- **Chosen:** Root `docs/architecture.md` + `docs/decisions.md`; README gains a Further reading section and keeps install/presets/catalog as the consumer landing page.
- **Alternatives rejected:** Leaving decisions only in chat history; stuffing all of this back into the root README.
- **Consequences:** Future workflow changes should update `docs/decisions.md` (or linked topic files) in the same change.
- **Revisit if:** The README becomes too long again — promote presets/catalog into `docs/` and leave only quickstart on the landing page.

---

### Explicit injectable seams on backends (with examples)

- **Context:** Greenfield backends scaffolded from these rules often wired concrete integration clients into features. The plug-and-play / test-fake pattern existed only implicitly (folder trees, “define interfaces where used”). Frontend does not need the same DI rule. Separately: whether transport should depend on a service interface (chatserver-style `ServiceInterface`) for unit-testing handlers.
- **Chosen:**
  - Mandatory **collaborator** ports with compact ✅/❌ examples in `go-core`, `rust-core` (backends only; not WASM engines), and FastAPI stack rules.
  - Hard must-bullets in every Go/Rust backend stack architecture file pointing at those examples.
  - Rust collaborator ports **default to static generics** (`Service<M: Trait>`). `Arc<dyn Trait>` / runtime swap is an exception, not a scaffold rule.
  - **Service-as-port is situational:** allow when there are two real implementations or a cross-package/usecase consumer that must not import the concrete. Handlers/routers may take a concrete `Service` by default.
  - Do **not** require a service interface solely to mock the service for handler/router unit tests — keep transport thin; cover with L2 + fake collaborators (`testing.mdc`).
  - Leave `frontend-core` unchanged for this pattern.
- **Alternatives rejected:**
  - **Frontend DI rules** — React services are API client modules; plug-and-play there is rare.
  - **`code-craft` only** — too broad; would pollute frontend context.
  - **New `backend-core.mdc`** — unnecessary; language cores + stack musts suffice.
  - **Default Rust to `Arc<dyn Trait>`** — overkill when one concrete type per process (env/test) is the normal case.
  - **Always interface the feature Service** — YAGNI with one impl; invites handler unit tests behind mocks that conflict with the testing strategy.
  - **Service ports only (no collaborator ports)** — misses the seam actually swapped in prod/L2 (integrations/repos).
- **Consequences:** Scaffold and feature work must introduce collaborator ports at use sites and wire concretes at composition roots; L2 fakes implement those ports. Service interfaces appear only when product/composition needs them.
- **Revisit if:** A product needs mid-process runtime swap of integrations often enough to justify a first-class `Arc<dyn>` rule; or multiple real service implementations become the common case and a stronger default is warranted.

---

### Frontend URL state without locking a router library

- **Context:** `frontend-core` told agents to use React Router’s `useSearchParams` for URL state; apps may use other routers (e.g. TanStack Router).
- **Chosen:** Prefer shareable UI state in **URL query parameters**; use the app’s existing router search/query APIs; do not invent a second source of truth. Name React Router / TanStack Router only as examples.
- **Alternatives rejected:** Hard-requiring React Router `useSearchParams`.
- **Consequences:** Agents pick the project’s router APIs; URL remains the shareable-state default where reasonable.
- **Revisit if:** The stack standardizes on one router and a single API should be named for consistency.

---

### Vitest as default unit runner for Vite frontends

- **Context:** Rules previously said Jest (or Jest/Vitest vaguely). Vite apps align better with Vitest (same resolve/aliases, lighter setup).
- **Chosen:** **Vitest + RTL** is the default for Vite/React; **Jest + RTL** only for non-Vite or legacy Jest-standardized apps — do not run both on Vite.
- **Alternatives rejected:** Keeping Jest as the primary frontend unit runner alongside Vite.
- **Consequences:** Greenfield Vite scaffolds and `frontend-testing` / bootstrap / README point at Vitest.
- **Revisit if:** A non-Vite React bundler becomes the default scaffold.

---

### Review triage on the developer handoff

- **Context:** Human review is the scarce step once agents draft most of the code. The feature handoff asked for manual QA and a docs list, and `/commit-push` suggested a pull-request body without saying what a human must read.
- **Chosen:** Every non-trivial developer handoff, and the suggested pull-request body, includes At-a-glance, Must review, Safe to skim, and Concerns & residual risk. Categories and the “if unsure, Must review” default live in `feature-workflow.mdc` (**Review triage**). `/commit-push` reuses that classification. Simple commits get a one-line class.
- **Alternatives rejected:**
  - **A separate always-on triage rule file** — the handoff already exists; another file would duplicate the developer path.
  - **Skipping triage on the developer path and using it only for business PRs** — the bottleneck is the same for both.
- **Consequences:** Agents classify the diff before asking for review. The starting categories can move after incidents or repeated clean merges, recorded in the consuming project's decision log.
- **Revisit if:** A critic or CI label should emit the classification instead of the parent agent.

---

### Business idea lane vs developer ceremony

- **Context:** Some products have business contributors who should try ideas without the developer pauses, while engineers keep `/feature-start`. The same craft rules should apply. An idea must not land on `main` before an engineer raises it to that bar. An earlier cut used `/business-iterate` and `/business-finalize` as opt-in commands and told engineers to re-run `/feature-start` to promote.
- **Chosen:**
  - `feature-workflow` runs only when `/feature-start` is invoked. A plain prompt does not enter it.
  - `business-team-experiments.mdc` is not a preset. `/setup-business <repo-url>` copies it into one clone with `alwaysApply: true`, lists it in `.git/info/exclude`, and does not commit it. That machine's commands are `/submit` and `/feature-start` only.
  - A plain prompt on that machine stays on `feature/business-ideas/<topic>` in the same folders as `main`, runs tests, records failures, and does not commit.
  - `/submit` writes idea-stage assumptions, runs the five critics once without fixing, and opens the pull request even when tests are red. It refuses `main`. It never stages the always-on rule file.
  - `/feature-start` on that machine still runs the full ceremony, then the handoff points at `/submit` instead of `/commit-push`.
  - `/standardize-pr` is the engineer intake for any pull request: fix tests, critic fix loop, update triage. `/feature-start` is for work the engineer originates, or a rebuild.
  - Developer install copies every command except `submit`. `ci-github` stays GitHub Flow. Protecting `main` on the product repo is an admin step, not part of setup.
- **Alternatives rejected:**
  - **Commit the rule with `alwaysApply: false`, then gitignore it** — a committed file stays tracked; a local flip to `alwaysApply: true` can merge and turn the lane on for every developer.
  - **Delete developer workflow rules on the business clone** — stack rules must stay so the agent still follows the real layout; `skip-worktree` deletions are easy to commit by mistake.
  - **`/commit-push` on the business machine** — two finish commands. `/submit` is the only one.
  - **Re-run `/feature-start` to receive a business pull request** — the diff already exists. `/standardize-pr` raises that diff.
  - **Always-on rule inside the developer preset** — personal projects and engineers would be forced onto idea branches.
- **Consequences:** Business iteration is a plain prompt. Engineers meet the result as a pull request. Red tests and unfixed critic blockers are that pull request's Must review until `/standardize-pr` runs.
- **Revisit if:** Business accounts must not have write access to the product repo (use a fork and point `/submit` at it); or `/standardize-pr` should refuse product-choice blockers instead of stopping to ask.

---

### slog as the Go logger; do not add Wire

- **Context:** Go stack rules listed zerolog and Google Wire as optional upgrades. Wire's generate step duplicates what `go build` already checks on manual constructors. slog is in the standard library.
- **Chosen:** New Go code logs with `log/slog` (`go-core`). zerolog only when slog is insufficient or the service already uses it. Do not introduce Google Wire unless the service already has it; do not hand-edit existing generated Wire output. Just and air stay optional.
- **Alternatives rejected:**
  - **Keep “add Wire when the graph hurts”** — agents were introducing a codegen tool the composition root does not need.
  - **Require Just and air on day one** — README test commands already cover the agent path; hot reload is a human convenience.
- **Consequences:** Go HTTP and gRPC trees no longer show `injector.go`. Existing Wire services are left in place.
- **Revisit if:** A service's constructor graph is large enough that manual wiring is a repeated source of review errors Wire would have caught.

---

### Biome for new Vite apps

- **Context:** `ci-github` named eslint as the JavaScript linter, so greenfield Vite scaffolds grew a second toolchain beside the intended single formatter/linter.
- **Chosen:** New Vite apps use **Biome** for lint and format. Keep eslint only when the repo already uses it. Do not run both. `ci-github` runs the project's linter rather than installing eslint by default.
- **Alternatives rejected:** **eslint + Prettier as the greenfield default** — two tools, and Biome matches the Vitest decision (one toolchain that fits Vite).
- **Consequences:** `frontend-core` and `ci-github` agree. Library-default churn beyond this stays out of these rules until the dependency-rules pass.
- **Revisit if:** Biome cannot express a lint rule the team must have on every Vite app.
