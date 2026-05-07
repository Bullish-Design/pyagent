# Skill: devenv.sh Environment (pyagent)

This repository uses `devenv.sh` as the canonical execution environment.

## Rule
Run environment-dependent commands through:

```bash
devenv shell -- <command>
```

This includes tests, scripts, linters, formatters, type checkers, and runtime commands.

## Session-start requirement
Before the first test/lint/typecheck run in each session:

```bash
devenv shell -- uv sync --extra dev
```

## Dependency management
- Never use `uv pip install`.
- Use `uv sync` for dependency changes.

## pyagent standard commands
```bash
devenv shell -- uv run lint
devenv shell -- uv run lint_fix
devenv shell -- uv run format
devenv shell -- uv run format_check
devenv shell -- uv run typecheck
```
