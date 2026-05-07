# AGENTS.md

Read `.scratch/CRITICAL_RULES.md` first in every session, then `.scratch/REPO_RULES.md`.

Operational reminders:
- Never use subagents.
- Keep project tracking files in `.scratch/projects/<num>-<name>/` up to date.
- Use `devenv shell -- ...` for all environment-dependent CLI commands.
- This includes tests, project scripts, demos, linters/formatters/typecheckers, and app/runtime commands.
- Before the first test/lint/typecheck run in a session, sync dependencies:
  - `devenv shell -- uv sync --extra dev`
- For Python linting/formatting, use Ruff via `uv` scripts:
  - `devenv shell -- uv run lint`
  - `devenv shell -- uv run lint_fix`
  - `devenv shell -- uv run format`
  - `devenv shell -- uv run format_check`
- For Python type checking, use Ty via `uv` scripts:
  - `devenv shell -- uv run typecheck`

Available local skills:
- `.scratch/skills/devenv-skill.md`
- `.scratch/skills/python-linting-ruff/SKILL.md`
- `.scratch/skills/python-typecheck-ty/SKILL.md`
- `.scratch/skills/development-testing-browser-harness/SKILL.md`
- `.scratch/skills/context_library_lookup/SKILL.md`
- `.scratch/skills/lsp-chat-recovery-loop/SKILL.md`
- `.scratch/skills/e2e-neovim-testing.md`
- `.scratch/skills/e2e-live-verification.md`

Quality gate checklist before finalizing Python changes:
1. Run `devenv shell -- uv run lint` for any Python edit.
2. Run `devenv shell -- uv run format_check` for any Python edit.
3. Run `devenv shell -- uv run typecheck` when changing signatures, typed models, protocol/contracts, or public interfaces.
4. If checks fail, fix and rerun until clean; mention any unresolved blockers explicitly.
