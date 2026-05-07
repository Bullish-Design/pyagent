# Skill: E2E Editor/Terminal Testing (pyagent template)

This file is a pyagent-oriented placeholder for future E2E harness work.

## Current repo status
- No `e2e/` package is currently present.
- No Neovim/tmux/asciinema scenario runner exists yet.

## Guidance for future adoption
If `pyagent` adds E2E terminal/editor scenarios:
1. Keep harness code inside `e2e/`.
2. Run scenarios with `devenv shell -- ...`.
3. Save recordings/artifacts in a dedicated output directory.
4. Add deterministic assertions first, then optional visual artifacts.
5. Keep commands and paths in this skill repo-local (no cross-repo absolute paths).

## Verification baseline
- Include one smoke scenario.
- Include one scenario for interactive flow under test.
- Document how to run a single scenario and full suite.
