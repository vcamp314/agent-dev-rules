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
