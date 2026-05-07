# Skill: E2E Live Verification Loop (pyagent template)

Use this loop after `pyagent` has live/integration E2E scenarios.

## Current repo status
- No live E2E runner is currently present in this repository.

## Future loop
1. Run one scenario against real dependencies.
2. Capture logs/recordings.
3. Compare expected vs observed behavior.
4. Write a concise run report with failure signature.
5. Apply one minimal fix.
6. Re-run and confirm behavior.

## Reporting template
For each run, record:
- Command executed
- PASS/FAIL
- Duration
- Key evidence (logs, markers, output snippets)
- Next action

## Rules
- Keep each fix isolated to one hypothesis.
- Prefer deterministic checks over brittle timing sleeps.
- Keep artifacts and notes under `.scratch/projects/<project>/`.
