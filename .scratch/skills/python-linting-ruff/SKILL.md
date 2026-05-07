# Python Linting (Ruff) for pyagent

Use this skill when changing Python code in this repository.

## Commands
- `devenv shell -- uv run lint`
- `devenv shell -- uv run lint_fix`
- `devenv shell -- uv run format`
- `devenv shell -- uv run format_check`

## Scope
- Default target is repository root via configured scripts.
- Ruff config is read from `pyproject.toml`.

## Enforcement
Before finalizing Python edits, run:
1. `devenv shell -- uv run lint`
2. `devenv shell -- uv run format_check`
