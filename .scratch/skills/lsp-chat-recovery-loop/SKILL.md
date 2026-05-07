---
name: lsp-chat-recovery-loop
description: "Template workflow for diagnosing LSP chat-delivery regressions when pyagent introduces an LSP/runtime pipeline."
---

# LSP Chat Recovery Loop (pyagent template)

This skill was adapted from another repo and is intentionally reduced to a template.

## Current repo status
- `pyagent` currently has no LSP chat runtime, no `e2e/` harness, and no log simplifier scripts.

## When to activate this workflow
Use this only after the repository adds:
- an LSP/chat runtime,
- reproducible e2e scenarios,
- structured server/client logs.

## Minimal loop (future)
1. Reproduce with a deterministic scenario.
2. Capture server/client logs for the run.
3. Quantify missing pipeline stages.
4. Write one falsifiable hypothesis.
5. Implement one minimal fix.
6. Re-run scenario and compare markers.
7. Document verification commands and expected evidence.

## Rule
Keep all run artifacts in `.scratch/projects/<project>/runs/<timestamp>/`.
