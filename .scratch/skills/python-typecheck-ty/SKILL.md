# Python Type Checking (Ty) for pyagent

Use this skill when edits may affect typed interfaces, models, protocols, or public APIs.

## Command
- `devenv shell -- uv run typecheck`

## Scope
- Current script target is `src`.
- If package layout changes, update `[tool.uv.scripts].typecheck` in `pyproject.toml`.

## Enforcement
Before finalizing relevant Python edits, run:
1. `devenv shell -- uv run typecheck`
