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

### This repo’s own docs

- **Context:** After adding `documentation.mdc`, agent-dev-rules itself had no `docs/` capturing the above deliberation.
- **Chosen:** Root `docs/architecture.md` + `docs/decisions.md`; README gains a Further reading section and keeps install/presets/catalog as the consumer landing page.
- **Alternatives rejected:** Leaving decisions only in chat history; stuffing all of this back into the root README.
- **Consequences:** Future workflow changes should update `docs/decisions.md` (or linked topic files) in the same change.
- **Revisit if:** The README becomes too long again — promote presets/catalog into `docs/` and leave only quickstart on the landing page.
