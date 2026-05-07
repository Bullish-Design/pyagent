# Development Testing With Browser Harness (pyagent)

This skill is currently a placeholder for future browser-harness work in `pyagent`.

## Current repo status
- No `scripts/ensure_chrome_debug.sh` exists in this repo.
- No browser-harness demo scripts are currently scaffolded.

## Rule when adding browser-harness workflows
When browser-based demos/tests are introduced:
1. Add a repo-local bootstrap script (for example `scripts/ensure_chrome_debug.sh`).
2. Document required env vars in this skill and `AGENTS.md`.
3. Run all browser-harness commands via `devenv shell -- ...`.

## Until then
Do not run Browsee/Remora-specific commands copied from other repos in `pyagent`.
