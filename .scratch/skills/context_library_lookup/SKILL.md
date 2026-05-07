---
name: context_library_lookup
description: >
  Locate and extract authoritative documentation from this repository's local
  context/vendor materials when they exist.
---

# SKILL: Local Context Lookup for pyagent

## Purpose
Use repository-local docs and vendored materials first, and avoid relying on stale memory.

## Current repo status
- This repo currently does not include a `.context/` directory.
- If `.context/` is later added, treat it as the highest-priority source.

## Workflow
1. Search local docs first (`README.md`, `docs/`, `.scratch/projects/`).
2. If `.context/` exists, prioritize it over external sources.
3. Use `rg -n` to locate symbols and behavior quickly.
4. Answer with exact file paths and line references.

## Search patterns
```bash
rg -n "<symbol-or-topic>" README.md docs .scratch .context 2>/dev/null
rg --files | rg "README|docs|context|design|concept"
```
