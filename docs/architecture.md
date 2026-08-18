# Architecture

## Purpose

`agent-dev-rules` is a **modular rule pack** and Cursor-native workflow for polyglot apps (React frontend; Go / Python / Rust HTTP & gRPC; optional Rust WASM). Consuming projects install a **preset subset** of `rules/*.mdc`, plus workflow rules and slash commands — not the entire catalog.

Canonical remote: https://github.com/vcamp314/agent-dev-rules

## Repo layout

```text
agent-dev-rules/
├── README.md           # Landing: install, presets, catalog summary
├── docs/               # System-wide docs for this repo
│   ├── architecture.md # This file
│   └── decisions.md    # Design decisions + rejected alternatives
├── rules/              # Architecture + craft + workflow `.mdc` files
├── agents/             # Multi-critic subagent prompts
├── commands/           # Cursor slash commands
└── LICENSE
```

This repository is **not** an application monorepo. There is no `frontend/` / `backend/` here — those trees are conventions the rules teach *other* projects to use.

## Modules

| Area | Location | Role |
|------|----------|------|
| Stack / craft rules | `rules/*.mdc` | Include by preset; bodies are tool-agnostic where practical |
| Greenfield bootstrap | `rules/bootstrap-select-rules.mdc` | Global User Rule; selects preset + scaffold |
| Feature delivery | `rules/feature-workflow.mdc` | Requirements → docs → TDD → critics → handoff |
| Documentation protocol | `rules/documentation.mdc` | README + colocated/`docs/` placement; decisions; sync with code |
| Testing strategy | `rules/testing.mdc` | E2E/L1/L2/unit; **day-one** harness + minimal passing test |
| Multi-critic | `rules/multi-critic-protocol.mdc` + `agents/` | Parallel review after green tests |
| Daily planning (opt-in) | `rules/daily-planning.mdc` + `rules/backlog-sources.mdc` | Cross-project backlog review → prioritized, capacity-bounded shortlist → dispatch; config + remote plan store |
| Slash commands | `commands/` | `/new-project`, `/feature-start`, `/commit-push`, `/sync-rules`, `/plan-day`, `/start-day` |

### Include graph (conceptual)

```text
code-craft
     │
     ├── go-core / rust-core
     ├── http-api-contract
     ├── testing (+ frontend-testing when React)
     ├── ci-github (optional)
     ├── backend-usecases (rare)
     ├── monorepo-architecture (when frontend + backend share a repo)
     ├── frontend-core (+ optional frontend-*)
     └── backend-*-architecture-patterns
```

Workflow overlays (always for Cursor workflow projects): `feature-workflow`, `documentation`, `multi-critic-protocol`, `testing`.

## Install boundaries

| Artifact | Where it lives for the user |
|----------|----------------------------|
| Bootstrap | Cursor **User Rules** (global) |
| Stack + workflow `.mdc` | Project `.cursor/rules/` |
| Critic agents | Prefer `~/.cursor/agents/` |
| Slash commands | Prefer `~/.cursor/commands/` |

Do **not** put the full architecture pack in User Rules (context bloat / wrong-stack pollution).

## Documentation conventions (this repo)

Follow `rules/documentation.mdc`: short README landing page; depth and decisions under `docs/`. Consumer projects get the same protocol via the installed `documentation.mdc` rule.

## Further reading

- [Design decisions](./decisions.md)
- [Root README](../README.md) — install steps, presets, file catalog
