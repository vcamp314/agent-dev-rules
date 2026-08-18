# AGENTS.md

## Cursor Cloud specific instructions

This repository (`agent-dev-rules`) is **documentation-only**. It ships a Cursor
rules pack, not an application:

- `rules/*.mdc` — architecture / craft / workflow rules (Markdown + YAML frontmatter).
- `agents/*.md` — multi-critic subagent prompts.
- `commands/*.md` — Cursor slash commands.
- `README.md`, `LICENSE`.

Non-obvious consequences for setup/run/test:

- **No dependencies, package manager, build step, test framework, linter config, or
  CI** exist in this repo, so there is nothing to install and nothing to compile or
  serve. The update script is intentionally a no-op. Do not add a `package.json`,
  `requirements.txt`, or similar just to satisfy tooling expectations.
- The "product" is consumed by Cursor: rule bodies are the deliverable. Files are
  used by copying the preset subset into a consuming project's `.cursor/rules/`
  (and `agents/*.md` into `.cursor/agents/`, `commands/*.md` into `.cursor/commands/`),
  per `rules/bootstrap-select-rules.mdc`. **Do not load every rule** — only the
  preset that matches the target project's shape.
- The closest equivalent to "lint/test" is a content-integrity check: every
  `rules/*.mdc` must have a valid `---` frontmatter block with a non-empty
  `description`, and every rule named in the README File catalog must exist (and
  vice-versa). Keep the README catalog and bootstrap preset tables in sync with the
  files in `rules/` when adding or renaming a rule.
- Frontmatter matters: `.mdc` rules use `description` + `alwaysApply`; `agents/*.md`
  use `name` + `description` + `model` + `readonly`. Preserve these when editing.
