# agent-dev-rules

Modular architecture / craft rules and a Cursor-native DevOps workflow for polyglot apps: frontend (React), Go / Python / Rust HTTP & gRPC backends, and Rust WASM beside React.

Canonical repo: https://github.com/vcamp314/agent-dev-rules

```text
agent-dev-rules/
├── README.md
├── docs/            # architecture + design decisions (this repo)
├── rules/           # architecture + bootstrap + feature workflow
├── agents/          # multi-critic subagent prompts
└── commands/        # Cursor slash commands
```

Files ship as `.mdc` (Markdown + optional YAML frontmatter) or `.md` for agents/commands. The guidance is the body; strip frontmatter or rename when using other tools.

**Do not load every rule.** Include only the [preset](#presets-what-to-include) that matches your project shape.

## Further reading

- [Architecture](docs/architecture.md) — repo layout, modules, install boundaries
- [Design decisions](docs/decisions.md) — why documentation, perfect commit, day-one tests, presets

---

## Cursor workflow (install)

```text
Plan /new-project  →  fetch preset into .cursor/rules/
       ↓
 /feature-start    →  git branch → requirements → docs → TDD → critics → manual QA
       ↓
 /commit-push      →  perfect commit (code+tests+docs+issue) → push
       ↓
 You open PR on GitHub
```

### Where each artifact goes

| Artifact | Cursor location | Source in this repo |
|----------|-----------------|---------------------|
| Bootstrap / rule selection | **User Rules** (Customize → Rules) — paste `rules/bootstrap-select-rules.mdc` body | `rules/bootstrap-select-rules.mdc` |
| Architecture stack rules | Project `.cursor/rules/*.mdc` | `rules/*.mdc` (preset subset) |
| Feature + docs + multi-critic protocols | Project `.cursor/rules/` | `feature-workflow.mdc`, `documentation.mdc`, `multi-critic-protocol.mdc` |
| Critic subagents | Prefer `~/.cursor/agents/*.md` (all projects) | `agents/*.md` |
| Slash commands | Prefer `~/.cursor/commands/*.md` | `commands/*.md` |

Do **not** put the full architecture pack in User Rules (context bloat / wrong-stack pollution). Global = bootstrap (+ optional one-liner: prefer `feature-workflow` when implementing features; only commit via `/commit-push` or an explicit ask).

### Install steps (macOS)

1. **User Rules:** copy the body of `rules/bootstrap-select-rules.mdc` into Cursor → Customize → Rules.
2. **Commands (global):**
   ```bash
   mkdir -p ~/.cursor/commands
   cp commands/*.md ~/.cursor/commands/
   ```
3. **Critic agents (global):**
   ```bash
   mkdir -p ~/.cursor/agents
   cp agents/*.md ~/.cursor/agents/
   ```
4. **Per project:** run `/new-project` (or follow bootstrap) so the agent fetches the preset `.mdc` files into `.cursor/rules/`, including `feature-workflow.mdc`, `documentation.mdc`, `multi-critic-protocol.mdc`, and `testing.mdc`.

Alternatively use Cursor **Add Rule → Remote Rule (GitHub)** for this repo once per machine/team; bootstrap still tells the agent which subset to use.

### Slash commands

| Command | Purpose |
|---------|---------|
| `/new-project` | Plan preset → fetch rules → scaffold (incl. day-1 passing test) → root README + `docs/` |
| `/feature-start` | Full feature/fix loop (requirements, design docs, TDD, multi-critic, QA handoff) |
| `/commit-push` | Perfect commit (code+tests+docs+issue) or `--simple` bypass; push |
| `/sync-rules` | Refresh rules/agents/commands from this repo |
| `/init-planning` *(opt-in)* | One-time setup: install planning rules into a home repo and fill placeholders (asks for any values not supplied) |
| `/plan-day` *(opt-in)* | Review cross-project backlog → prioritize (urgency×impact + text scan) → capacity-bounded shortlist → approve |
| `/start-day` *(opt-in)* | Dispatch bounded per-task workers against today's approved shortlist |

Daily planning is **opt-in** (not installed by bootstrap): `/plan-day` + `/start-day` are driven by the tool-agnostic `daily-planning.mdc` protocol and the `backlog-sources.mdc` config (backlog sources, priority/estimator, and a remote plan store for cross-project use). Run `/init-planning` once to install those two rules plus the commands into a designated home repo and fill in their placeholders (backlog repos, home repo, store, budget); it prompts for anything you don't pass inline.

### Feature loop (summary)

1. Git prep (untracked local files allowed; stop only on real conflicts).
2. Requirements in two groups: **User-specified** vs **Inferred** (infer only sensible gaps); confirm before tests.
3. Design documentation per `documentation.mdc` (colocated feature README/`docs/`; system-wide root or service `docs/`; decisions + rejected alternatives). Fixes update docs for modified behavior.
4. Red → green TDD per `testing.mdc`.
5. Verify: max **3** fix-and-rerun cycles, then human intervention.
6. Multi-critic in parallel; max **3** blocker fix cycles.
7. Manual QA; list docs touched; commit only with `/commit-push` (issue link; `--simple` for trivial).

---

## How the architecture modules fit together

```text
code-craft          ← language-agnostic craft (always useful)
     │
     ├── go-core / rust-core     ← language craft (when that language is used)
     ├── http-api-contract       ← /<audience>/v1 routes + uuid7 in HTTP payloads
     ├── testing                 ← E2E / L1 / L2 / unit strategy
     ├── ci-github               ← GitHub Flow + Actions (when tests run)
     ├── backend-usecases        ← rare cross-domain orchestration (optional)
     ├── monorepo-architecture   ← repo layout when frontend + backend share a repo
     │
     ├── frontend-core (+ frontend-http-api | frontend-protobuf-gen | frontend-rust-wasm | frontend-testing)
     └── backend-*-architecture-patterns   ← one stack file per service type
```

| Layer | Files | Role |
|--------|--------|------|
| Shared craft | `code-craft`, `go-core`, `rust-core` | How to write code (incl. typing + composition) — not folder trees |
| Shared contracts | `http-api-contract`, `backend-usecases`, `testing`, `ci-github` | HTTP paths/IDs; usecases; what/how to test; when CI runs |
| Repo layout | `monorepo-architecture` | Top-level `frontend/`, `backend/`, `protobuf/`, compose, test env |
| Frontend | `frontend-core`, `frontend-http-api`, `frontend-protobuf-gen`, `frontend-rust-wasm`, `frontend-testing` | React layout + optional API / protobuf / WASM / Playwright+Jest |
| Backend stacks | `backend-go-http-…`, `backend-go-grpc-…`, `backend-python-fastapi-…`, `backend-rust-http-…`, `backend-rust-grpc-…` | In-service folder layout + framework choices |
| Workflow | `bootstrap-select-rules`, `feature-workflow`, `documentation`, `multi-critic-protocol` | Greenfield install + feature delivery + docs + critics |

**Backend vocabulary (aligned across Go / Python / Rust HTTP & gRPC):** `features/`, shared `services/` (after reuse), `integrations/`, `database/`, `common/`, optional `usecases/`.

---

## Quick start (architecture only)

1. Pick a [preset](#presets-what-to-include) below.
2. Copy those files from `rules/` into your project’s agent instructions folder (e.g. `.cursor/rules/`).
3. Ask the agent to scaffold or change code using those conventions.

Only include modules relevant to the work in progress so context stays small and focused.

---

## Presets (what to include)

### 1. Frontend-only (React app)

| Always | Add when… |
|--------|-----------|
| `code-craft`, `frontend-core`, `testing`, `frontend-testing` | — |
| + `http-api-contract`, `frontend-http-api` | App calls an HTTP API |
| + `frontend-protobuf-gen` | App uses generated protobuf/TS clients |
| + `rust-core`, `frontend-rust-wasm` | App has `crates/` + WASM engines |
| + `ci-github` | Adding GitHub Actions / CI |

Also install for Cursor workflow projects: `feature-workflow`, `documentation`, `multi-critic-protocol`.

### 2. Single HTTP API service (no monorepo)

Pick one stack. Always include `code-craft` + `http-api-contract` + `testing`. Add `ci-github` when adding Actions.

**Go (Echo):** `code-craft`, `go-core`, `http-api-contract`, `backend-go-http-architecture-patterns`, `testing`

**Python (FastAPI):** `code-craft`, `http-api-contract`, `backend-python-fastapi-architecture-patterns`, `testing`

**Rust (Axum):** `code-craft`, `rust-core`, `http-api-contract`, `backend-rust-http-architecture-patterns`, `testing`

Add `backend-usecases` only for rare cross-domain orchestration APIs.

### 3. Single gRPC service (no monorepo)

Do **not** include `http-api-contract`. Always include `testing`. Add `ci-github` as needed.

**Go:** `code-craft`, `go-core`, `backend-go-grpc-architecture-patterns`, `testing`

**Rust (Tonic):** `code-craft`, `rust-core`, `backend-rust-grpc-architecture-patterns`, `testing`

### 4. Full monorepo (frontend + backends)

**Always:** `monorepo-architecture`, `code-craft`, `testing`.

Then add per surface you actually have (React / Go HTTP / Go gRPC / Python / Rust HTTP / Rust gRPC / usecases / ci) — see the tables in `rules/bootstrap-select-rules.mdc` for the full matrix. React also gets `frontend-testing`.

---

## File catalog

| File | Use when |
|------|----------|
| `code-craft.mdc` | Any project |
| `go-core.mdc` | Any Go backend work |
| `rust-core.mdc` | Any Rust work (HTTP, gRPC, or WASM) |
| `http-api-contract.mdc` | Defining or calling HTTP APIs |
| `testing.mdc` | Testing strategy; greenfield day-one harness + minimal passing test |
| `ci-github.mdc` | GitHub Flow + Actions |
| `backend-usecases.mdc` | Rare cross-feature orchestration modules |
| `monorepo-architecture.mdc` | Repo has both `frontend/` and `backend/` |
| `frontend-core.mdc` | React / Vite / Tailwind app structure |
| `frontend-http-api.mdc` | Frontend HTTP client + react-query / toasts |
| `frontend-protobuf-gen.mdc` | Frontend consumes generated protobuf/TS |
| `frontend-rust-wasm.mdc` | Frontend `crates/` + wasm-bindgen bridge |
| `frontend-testing.mdc` | Playwright E2E/L1 + Jest/RTL unit tests |
| `backend-go-http-architecture-patterns.mdc` | Go Echo HTTP service layout |
| `backend-go-grpc-architecture-patterns.mdc` | Go gRPC service layout |
| `backend-python-fastapi-architecture-patterns.mdc` | Python FastAPI service layout |
| `backend-rust-http-architecture-patterns.mdc` | Rust Axum HTTP service layout |
| `backend-rust-grpc-architecture-patterns.mdc` | Rust Tonic gRPC service layout |
| `bootstrap-select-rules.mdc` | Greenfield / preset selection (global User Rule) |
| `documentation.mdc` | README + colocated/`docs/` placement; design decisions; keep docs in sync |
| `feature-workflow.mdc` | Feature/fix TDD delivery protocol (includes design docs step) |
| `multi-critic-protocol.mdc` | Parallel critic orchestration after green tests |
| `daily-planning.mdc` | Opt-in daily task protocol: review backlog → prioritize → capacity-bound → dispatch |
| `backlog-sources.mdc` | Opt-in config for daily planning: sources, priority/estimator, remote plan store |

**Agents:** `functional-verifier`, `security-auditor`, `qa-edgecase-verifier`, `architecture-linter`, `test-coverage-auditor`

**Commands:** `new-project`, `feature-start`, `commit-push`, `sync-rules`, `init-planning`, `plan-day`, `start-day`

---

## Design notes

See [docs/architecture.md](docs/architecture.md) and [docs/decisions.md](docs/decisions.md) for layout rationale and recorded decisions (documentation protocol, perfect commit, day-one tests, modular presets).

Short reminders:

- **Split by include boundary**, not by dumping conditionals into one mega-file.
- **Trees live in stack files**; language craft lives in `*-core` / `code-craft`.
- Placeholders like `aserver` / `bserver` are examples — rename to your domain.
- Rules prefer reusable conventions (uuid7, `/<audience>/v1/...`, feature modules). They are not product-specific.

## Contributing

Keep new rules:

1. Focused (one include reason per file when practical).
2. Free of private product/service names.
3. Consistent with existing `features` / `services` / `integrations` vocabulary where they apply.
4. Tool-agnostic body text in architecture rules (Cursor install lives in this README + workflow files).
5. Update [docs/decisions.md](docs/decisions.md) (or a linked topic file) when changing workflow/architecture choices.
