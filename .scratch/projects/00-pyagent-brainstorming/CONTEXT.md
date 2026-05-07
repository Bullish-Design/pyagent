# CONTEXT

Date: 2026-05-06

Repository was initialized from a template and had several copied skills from another repo (Remora/Browsee-specific paths and commands). Those references were incorrect for this codebase state.

Current state:
- `AGENTS.md` now reflects pyagent execution rules and lists all local skill docs present under `.scratch/skills/`.
- Core operational skills were aligned to pyagent:
  - `devenv shell -- uv sync --extra dev` pre-check requirement
  - Ruff scripts via `uv run`
  - Ty typecheck via `uv run typecheck`
- Non-applicable copied skills (browser harness, LSP recovery, live E2E workflows) were converted into pyagent templates with clear "not scaffolded yet" status and future activation criteria.

Next likely step:
- Scaffold project code layout (`src/pyagent`, `tests/`) and then tighten skill docs from placeholders into concrete runnable workflows.
