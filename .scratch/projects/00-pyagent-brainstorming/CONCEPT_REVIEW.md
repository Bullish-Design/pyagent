# CONCEPT_REVIEW.md

# Review and Analysis of the pi-agent-py Concept Document

**Revision 2** — Updated after studying the Pi ecosystem overview (`pi-overview.md`)

---

## 1. Overall assessment

The concept document is comprehensive, well-structured, and demonstrates deep understanding of both the Pi ecosystem and the Python developer experience it aims to serve. It makes sound architectural decisions—particularly the RPC-first/SDK-bridge-second progression and the Pydantic-first contract boundary. The Nix/devenv commitment is ambitious but appropriate for a project spanning Python, Node, and an external agent runtime.

Now that the target is clearly understood as `pi-mono` / `pi-coding-agent` (Mario Zechner / Earendil), several aspects of the concept become sharper—and several new considerations emerge.

**Verdict:** Strong foundational concept that is well-aligned with Pi's actual architecture and philosophy. A few areas need sharper focus before implementation, particularly around the RPC protocol surface, Pi's extension-only tool registration model, and the naming/ownership transition currently underway.

---

## 2. Strengths

### 2.1 Clear architectural layering

The four-layer architecture (domain → transport → runtime → governance) is well-motivated. Each layer has a distinct responsibility and the boundaries are clean. The transport abstraction in particular is the right structural decision—it makes the MVP-to-bridge migration a transport swap rather than an API rewrite.

### 2.2 Honest scoping

The document is disciplined about what is deferred (V3 extension generation, automatic self-training) and why. The explicit non-goals section prevents scope creep. The MVP criteria are concrete and testable.

### 2.3 Pydantic as the contract boundary

Using Pydantic for both public API schemas and internal persistence schemas is the right call. It gives you validation, serialization, schema generation, and IDE support in one move. The distinction between "stable ergonomic public schema" and "rich loss-aware internal schema" shows mature thinking about API design.

### 2.4 Environment-as-architecture

Treating Nix/devenv as an architectural component rather than a convenience feature is the right philosophy for a project that spans multiple runtimes. This mirrors Pi's own environment-centric philosophy: Pi treats the working directory, context files, and loaded extensions/skills as part of the agent's identity. The concept's devenv approach is spiritually aligned with Pi's "environment over prompt" worldview.

### 2.5 Trace-first design

Building trace capture and normalization into the MVP is strategically sound—especially given that Pi already has a trace/session export story (`pi-share-hf`, Hugging Face datasets). The concept's trace normalization layer can complement Pi's existing export tooling by providing a Python-native analytical layer over the same underlying JSONL session data.

### 2.6 Alignment with Pi's actual integration model

The concept correctly identifies Pi's intended embedding surfaces:
- RPC mode (LF-delimited JSONL over stdin/stdout, documented for non-Node integrations)
- SDK embedding via `pi-agent-core` (with real-world precedent in OpenClaw)

This is not speculative—these are documented, supported integration paths.

### 2.7 Multi-provider awareness

The concept's `ProviderRef` / `ModelRef` abstractions align with Pi's `pi-ai` layer, which treats multi-provider, mid-session model switching as a first-class capability. The Python wrapper inherits this flexibility naturally.

---

## 3. Weaknesses and concerns

### 3.1 The concept should name Pi explicitly

The concept document never identifies "Pi" as `pi-coding-agent` from `badlogic/pi-mono` (transitioning to `@earendil/pi` under `earendil-works`). While the concept was clearly written with full knowledge of the ecosystem, an explicit identification section would:

- Anchor the document to concrete versioned artifacts
- Acknowledge the naming transition (important for dependency management)
- Reference specific source URLs for protocol and SDK documentation

**Recommendation:** Add a "Target Runtime" section that names `pi-mono` / `pi-coding-agent`, its current package scope (`@mariozechner/pi-coding-agent`), its announced future scope (`@earendil/pi`), and links to the RPC/SDK documentation.

### 3.2 RPC protocol surface is clearer than the concept implies, but still needs documentation

The pi-overview confirms that RPC mode uses **LF-delimited JSONL over stdin/stdout**. The framing question from the original review is answered. However, the concept still needs:

- A catalog of command types the Python layer will send
- A catalog of event types the Python layer will receive
- Examples of actual protocol exchanges (prompt → events → response)
- Confirmation of whether the protocol supports correlation IDs natively or if the Python layer must add them

The SDK docs and source code (`packages/coding-agent/docs/sdk.md`, `packages/coding-agent/src/cli/args.ts`) should contain enough to reverse-engineer these.

**Recommendation:** Create `RPC_PROTOCOL.md` by examining Pi's source and running a manual RPC session with logging enabled. This should be Milestone 0 work.

### 3.3 Python-side tool registration faces a hard constraint

This is the most important new insight from the ecosystem overview.

Pi's custom tool registration happens exclusively through **TypeScript extensions**. Extensions can register tools, intercept tool calls, and alter control flow—but they are TS modules loaded by the Pi runtime. There is no documented mechanism for registering tools from an external RPC client.

This means:
- The Python library **cannot register Python-defined tools** that Pi calls back into unless a mechanism is built for this (e.g., a companion TS extension that bridges tool calls back to the Python process)
- The library is fundamentally an **observer and prompter**, not a bidirectional tool provider
- If Python-side tools are desired, they require either:
  - A custom Pi extension that acts as a bridge (sends tool calls to Python, returns results)
  - Abuse of the prompt/response cycle to simulate tool behavior
  - Contributing an upstream feature to Pi for external tool registration

**Recommendation:** The concept should explicitly acknowledge this limitation. For the MVP, the library is a control plane that sends prompts and consumes events. Python-side tool registration should be declared as a future bridge-dependent feature requiring a companion TS extension, not a core MVP capability.

### 3.4 Sync wrapper complexity is underestimated

The document mentions "sync and async clients" as if the sync wrapper is straightforward. In practice, wrapping an async-first library with a sync API in Python is fraught:

- Running `asyncio.run()` inside an existing event loop raises `RuntimeError`
- Thread-based approaches add complexity and potential deadlocks
- Libraries like `anyio` or `greenlet`-based approaches each have tradeoffs
- Jupyter notebooks and other async-aware environments need special handling

**Recommendation:** Decide early whether the sync wrapper uses `asyncio.run()` in a background thread (like `httpx`), a `greenlet`-based approach, or whether "sync" simply means "blocking calls that internally use asyncio". Document the chosen strategy and its limitations.

### 3.5 SQLModel as the persistence layer may be premature

The document recommends SQLModel for persistence. Concerns:

- SQLModel is a relatively thin wrapper over SQLAlchemy with Pydantic integration, but it has known limitations (incomplete type support, less active maintenance than SQLAlchemy itself)
- For the MVP, the storage requirements (traces, sessions, events) could be satisfied with simpler approaches (SQLite via raw SQLAlchemy, or even just structured JSON/JSONL files)
- Pi already stores sessions as JSONL—the Python layer could build directly on that format for raw storage and only add indexed queries where needed

**Recommendation:** Consider whether MVP persistence could use Pi's native JSONL format for raw session data + a lightweight SQLite index for queries. Defer the full SQLModel schema to the storage milestone.

### 3.6 The bridge design needs clearer justification

Section 7 of the concept outlines four possible bridge shapes and recommends JSON-over-stdio for consistency. But given what we now know about Pi:

- `pi-agent-core` is the actual runtime package, separate from the CLI
- The SDK lets you call `createAgentSession`, manage `SessionManager`, and select tools programmatically
- The bridge would essentially be a thin Node service that exposes `pi-agent-core` primitives over a protocol the Python layer can consume

The key question is: **what does the bridge unlock that RPC cannot?**

Based on the ecosystem overview, the answer is:
1. **Direct session lifecycle control** — create, continue, fork, list sessions programmatically without CLI intermediation
2. **Tool set selection** — choose which tools are available per session (the SDK supports this)
3. **Extension participation** — potentially load extensions and observe their events
4. **Richer metadata access** — session tree navigation, compaction state, loaded context/skills

**Recommendation:** Document these specific capabilities as the bridge's value proposition, distinct from RPC.

### 3.7 No error taxonomy or failure mode analysis

The document describes the happy path thoroughly but barely mentions:

- What happens when Pi crashes mid-session?
- How are timeout vs. hang vs. OOM failures distinguished?
- What is the retry strategy for transient failures?
- How does the library handle Pi version incompatibilities?
- What happens when Pi's output doesn't match expected schemas?

Pi's extension docs note that tool calls run in parallel and file mutations need queue coordination—this suggests the runtime has real concurrency. The Python layer must be robust to partial/malformed events.

**Recommendation:** Add a section on error handling philosophy, failure classification, and recovery strategies.

### 3.8 No concurrency model discussion

Key unaddressed questions:

- Can multiple sessions run concurrently from one client?
- Does each session spawn its own Pi subprocess (RPC implies yes)?
- What are the resource implications of N concurrent sessions?
- Is there a session pool concept?
- How does backpressure work when events arrive faster than the consumer processes them?
- How does this interact with Pi's own internal parallelism (parallel tool execution)?

**Recommendation:** Define the concurrency model explicitly, even if the MVP only supports one session at a time.

---

## 4. Architectural risks

### 4.1 Naming/ownership transition risk (HIGH — new)

Pi is actively transitioning from `badlogic/pi-mono` / `@mariozechner/*` to `earendil-works/pi` / `@earendil/*`. This means:

- npm package names will change
- Repository URLs will change
- The CLI binary name may change
- Import paths in the SDK will change

Building against a moving target introduces real risk for the devenv lockfile, CI, and any hard-coded package references.

**Mitigation strategy:**
- Pin to a specific git commit or npm version in devenv rather than tracking `latest`
- Abstract the Pi binary path/name behind a configuration layer
- Monitor the transition and have a migration checklist ready
- Consider whether to target the old or new naming from the start

### 4.2 Protocol coupling risk (HIGH)

The entire project depends on Pi's RPC protocol being stable enough to build against. The ecosystem overview confirms RPC is an officially documented mode, which is encouraging. However:

- Pi is evolving rapidly (the project is relatively young)
- There are no documented backward-compatibility guarantees for RPC message schemas
- The protocol may gain features that require Python-side updates

**Mitigation strategy:**
- Record protocol fixtures and run compatibility tests against them
- Version-lock Pi in the devenv and test against specific versions
- Implement a protocol version detection mechanism (Pi's RPC may include version info)
- Use Pydantic's `model_config = ConfigDict(extra="ignore")` to tolerate unknown fields gracefully

### 4.3 Extension ecosystem asymmetry risk (MEDIUM — new)

Pi's real power comes from TypeScript extensions. The Python library, by design, cannot participate in this extension ecosystem directly. This creates an asymmetry:

- A TS user can add tools, intercept calls, modify UI, and alter control flow
- A Python user can only observe, prompt, and consume events

Over time, if Pi's most valuable capabilities move into extension-space, the Python wrapper may become a second-class citizen.

**Mitigation strategy:**
- The bridge (Milestone 3-4) should be designed as a **companion extension** that exposes extension-level capabilities to the Python layer
- Consider writing a generic "Python bridge extension" for Pi that forwards events and accepts commands from an external process
- Accept that the MVP is observation-and-prompting-only, and plan the bridge as the upgrade path to full participation

### 4.4 Abstraction leakage risk (MEDIUM)

The transport abstraction aims to make RPC and SDK-bridge interchangeable. But Pi's RPC and SDK expose fundamentally different capability sets:

- RPC: prompt/response/events (observer model)
- SDK: full session lifecycle, tool selection, extension loading (participant model)

Feature parity between transports may be impossible without limiting the SDK transport to RPC-equivalent operations.

**Mitigation strategy:**
- Define a "core" capability set that both transports must support
- Allow optional "extended" capabilities per transport, discoverable at runtime
- Document which features require which transport
- Use Python's protocol/ABC system to make this discoverable at the type level

### 4.5 Complexity budget risk (MEDIUM)

The project spans: Python library + Pydantic schemas + async runtime + sync wrappers + persistence + Nix environment + Node bridge + TypeScript SDK/extension integration + trace normalization + evaluation framework. For a team of one (or a small team), this is a large surface area.

**Mitigation strategy:**
- Ruthlessly enforce MVP scope
- Ship Milestone 0 + 1 before touching anything else
- Accept that the bridge (Milestone 3-4) may never ship, and that's OK
- Consider whether the project would be viable as RPC-only permanently
- Leverage Pi's existing session export (`pi-share-hf`) rather than rebuilding export tooling

---

## 5. New insights from the ecosystem overview

### 5.1 Pi's "kernel + extensibility" philosophy validates the concept

The ecosystem overview's central insight—"Pi is a small agent kernel plus an extensibility/distribution system"—directly validates the concept's architecture. The Python library is essentially building a **Python-native shell** around Pi's kernel, analogous to how `pi-coding-agent` is the TypeScript-native shell.

This framing should be made explicit in the concept: `pi-agent-py` is to Python developers what `pi-coding-agent` is to TypeScript developers—an application-layer shell over the same runtime.

### 5.2 Skills interoperability is a strategic opportunity

The `pi-skills` repository is explicitly compatible with Claude Code, Codex CLI, Amp, and Droid—not just Pi. This means:

- Skills defined for the Python library's workflows could be reusable across agent harnesses
- The Python library could include skill authoring/validation tooling that benefits the broader ecosystem
- The concept's `PiSkillRef` / `SkillBundle` models could become a contribution to the wider Agent Skills standard

**Recommendation:** Consider whether the Python library should include skill authoring utilities as a value-add beyond pure Pi integration.

### 5.3 Pi's session model is richer than the concept captures

The ecosystem overview reveals that Pi's session model supports:
- Resume, fork, clone
- Tree navigation with `/tree`
- Bookmarks/labels
- Compaction with structured summaries
- Abandoned-branch summarization

The concept mentions tree structure and branching but doesn't fully leverage all these capabilities in its Python models. Specifically:

- `SessionBranch` should include bookmark/label support
- The replay model should support navigating to specific tree nodes
- Compaction events should be first-class in the event model (they carry structured summaries that are analytically valuable)

### 5.4 Pi's context file system maps directly to configuration

Pi loads context from `AGENTS.md` / `CLAUDE.md` files at global, parent-directory, and project levels. The concept's `InstructionPack` / `InstructionRef` models should account for this layered loading:

- The Python library should be able to specify or override context files
- The library should capture which context files were loaded (for trace reproducibility)
- Configuration resolution (Section 5.3 of the original review) should follow Pi's own hierarchy

### 5.5 `pi-share-hf` provides prior art for trace export

Pi already has tooling (`pi-share-hf`) for exporting sessions to Hugging Face datasets with redaction/review steps. The concept's trace normalization layer should be aware of this:

- Consider whether normalized traces should be exportable in `pi-share-hf`-compatible format
- The concept's `TrainingExample` / `PreferencePair` models could build directly on the HF dataset schema
- There's an opportunity to add Python-native analysis tooling over the existing public dataset

### 5.6 Pi's multi-provider model switching enriches the trace model

`pi-ai` supports mid-session provider/model switching with context transformation. This means a single session trace may span multiple models. The concept's trace normalization should:

- Record model switches as first-class trace events
- Attribute individual steps to specific provider/model pairs
- Enable comparative analysis (same prompt, different models, in the same session tree)

---

## 6. Design questions requiring resolution

### 6.1 Session ownership and lifecycle

Who owns the Pi process? Based on the RPC model:
- Each `PiSession` likely spawns its own Pi subprocess
- The subprocess runs for the duration of the session
- Multiple concurrent sessions = multiple subprocesses

This should be explicit. The concept should also address:
- Maximum concurrent sessions (resource constraint)
- Process reuse across sequential sessions (possible optimization)
- Graceful vs. forced termination

### 6.2 Event delivery guarantees

When streaming events:
- Pi's RPC delivers events over stdout as they occur
- Are events buffered in the Python layer or must they be consumed in real-time?
- What happens if the consumer falls behind? (stdout buffer fills → Pi blocks → deadlock risk)
- Can events be replayed from the raw capture log?

### 6.3 Configuration resolution and context files

The concept mentions `PiClientConfig` and `PiSessionConfig` but doesn't describe how they interact with Pi's own layered context system. Questions:
- Does `PiSessionConfig.working_dir` determine which `AGENTS.md` files Pi loads?
- Can the Python layer inject additional context files?
- Should the library expose which context files Pi loaded (for reproducibility)?
- How do `SYSTEM.md` / `APPEND_SYSTEM.md` overrides interact with the Python config?

### 6.4 Testing without Pi

Can the library be tested without a running Pi instance?

The concept should explicitly define:
- **Mock transport** — for unit testing domain logic without Pi
- **Replay transport** — for deterministic integration testing from recorded fixtures
- **Live transport** — for smoke/acceptance testing against real Pi

This three-tier strategy should be part of the transport layer design, not an afterthought.

### 6.5 Relationship to Pi Packages

Pi Packages bundle extensions, skills, prompt templates, and themes. Questions:
- Should the Python library be able to install/manage Pi Packages?
- Should it be aware of installed packages when constructing sessions?
- Could the Python library itself distribute a companion Pi Package (containing a bridge extension)?

### 6.6 Compaction awareness

Pi compacts long sessions by summarizing older messages. The Python layer should decide:
- Does it receive compaction events via RPC?
- Should it trigger compaction programmatically?
- How does compaction affect trace normalization (the trace should note where summaries replaced raw content)?

---

## 7. Comparison with alternatives

### 7.1 Direct subprocess management

A developer could simply spawn Pi via `subprocess` and parse JSONL. Why use this library?

**Value add:** Typed models, event streaming as async iterators, trace capture and normalization, session management, error classification, replay capability, transport migration path, and Pydantic-based schema validation. The library saves hundreds of lines of boilerplate per project and provides a stable API even as Pi's protocol evolves.

### 7.2 Generic agent frameworks (LangChain, CrewAI, etc.)

These frameworks provide their own agent runtimes. Why not use one of them with Pi as a tool?

**Answer:** Pi is the runtime, not a tool. Pi provides its own session model (tree-structured, with branching/compaction), its own tool system, its own skills/extensions, and its own multi-provider model layer. Wrapping Pi as a "tool" in LangChain would discard everything that makes Pi valuable and reduce it to a subprocess executor.

### 7.3 Using `pi-agent-core` directly from Node

Since Pi is TypeScript-native, why not just use Node?

**Answer:** For Python-first teams, the value is in having Python-native typed models, Python async/await patterns, Python persistence tooling (SQLAlchemy/SQLModel), Python ML/evaluation infrastructure, and Pydantic schemas that integrate with Python web frameworks, data pipelines, and training workflows. The library bridges the ecosystem gap.

### 7.4 Using Pi's JSON/print modes instead of RPC

Pi also has JSON mode and print mode for non-interactive use.

**Answer:** These modes are one-shot: send a prompt, get a response. RPC mode provides a persistent session with streaming events, which is necessary for:
- Long-running tasks with progress visibility
- Multi-turn interactions
- Real-time event capture for traces
- Session lifecycle management (interrupt, resume, etc.)

JSON/print modes could be useful for simple one-shot queries but don't support the full session model the concept targets.

---

## 8. Recommended changes to the concept

### 8.1 Critical (must address before implementation)

1. **Identify Pi explicitly.** Name `pi-coding-agent` from `badlogic/pi-mono`, its current and transitioning package scopes, and link to the RPC/SDK documentation.
2. **Document the RPC protocol.** Run a manual RPC session, capture the JSONL exchange, and create `RPC_PROTOCOL.md` with command/event schemas.
3. **Acknowledge the tool registration constraint.** The Python layer cannot register tools in Pi without a companion TS extension. This limits the MVP to observation-and-prompting. Document this explicitly and plan the bridge as the upgrade path.
4. **Define the concurrency model.** One Pi subprocess per session, resource implications, backpressure strategy for stdout buffering.
5. **Specify sync wrapper strategy.** Choose an approach (background-thread-with-event-loop, like httpx) and document its limitations.

### 8.2 Important (should address during Milestone 0-1)

6. **Add a mock/replay transport** to the transport layer design for testing (three-tier test strategy).
7. **Simplify MVP persistence.** Use Pi's native JSONL for raw storage + SQLite index for queries. Defer full ORM.
8. **Account for the naming transition.** Abstract Pi binary name/path, pin to specific versions, monitor the Earendil migration.
9. **Model compaction events.** Include compaction in the event type union and trace normalization.
10. **Capture loaded context metadata.** Record which `AGENTS.md`/`SYSTEM.md` files Pi loaded per session for trace reproducibility.

### 8.3 Important (should address during Milestone 2-3)

11. **Design the bridge as a companion Pi extension.** The Node bridge isn't just a sidecar—it should be a Pi extension that participates in the runtime and exposes capabilities back to Python.
12. **Align trace export with `pi-share-hf`.** Ensure normalized traces can be exported in HF-compatible format.
13. **Define error handling philosophy.** Classification, recovery, and typed exceptions.
14. **Specify logging framework** (`structlog` recommended for structured JSON logs with correlation IDs).

### 8.3 Nice to have (can defer)

15. **Skill authoring utilities** (validation, scaffolding, testing).
16. **Contribution guide** for non-Nix environments.
17. **OpenTelemetry integration** plan.
18. **Multi-tenancy considerations.**
19. **Performance benchmarks** for event throughput.

---

## 9. Implementation priority recommendation

### Phase 0: Foundation (1-2 weeks)
1. `devenv.nix` with Python (via uv) + Node (for Pi) pinned
2. `pyproject.toml` with uv
3. Basic project structure (`src/pyagent/core/`, `src/pyagent/transport/`)
4. Install Pi in devenv, run a manual RPC session, capture raw JSONL
5. Create `RPC_PROTOCOL.md` from captured traffic
6. Record 3-5 RPC fixtures (simple prompt, tool use, multi-turn, error case)

### Phase 1: Core transport (2-3 weeks)
1. JSONL framing (parser + writer, LF-delimited)
2. Pydantic event type models (discriminated union from fixture analysis)
3. Async RPC transport (spawn Pi subprocess, send commands, yield events, close)
4. Basic `AsyncPiClient` + `AsyncPiSession` with context manager protocol
5. Replay transport for fixture-based testing
6. Integration smoke test against live Pi RPC

### Phase 2: Public API polish (1-2 weeks)
1. Response assembly from event stream
2. Sync wrapper (background thread + event loop, like httpx)
3. Error classification and typed exceptions
4. Configuration resolution (client defaults → session overrides → env vars)
5. Raw traffic recording (JSONL append-log per session)

### Phase 3: Trace normalization + persistence (1-2 weeks)
1. Trace model: steps, tool executions, model info, timing
2. Normalize from raw event stream to `Trace` / `TraceStep` models
3. SQLite-backed trace index (session metadata, searchable)
4. Session tree reconstruction from JSONL (leverage Pi's id/parentId structure)
5. Basic replay from normalized traces

### Phase 4: Stabilization + release
1. Test coverage (unit via mock transport, integration via replay, smoke via live Pi)
2. Minimal documentation (API reference from Pydantic models, getting started guide)
3. CI pipeline via devenv tasks
4. First tagged release (`0.1.0`)

---

## 10. Final verdict

**The concept is architecturally sound, strategically well-positioned, and well-aligned with Pi's actual design philosophy.**

Pi's "minimal kernel + extensibility" approach directly validates the concept's choice to build a Python control plane rather than reimplementing Pi. Pi's documented RPC mode, tree-structured sessions, multi-provider model layer, and SDK embedding support all confirm that the concept is building against intended integration surfaces—not hacking around limitations.

The main risks are:
- **Naming/ownership transition** — building against a project that's actively changing names and homes
- **Tool registration asymmetry** — Python cannot participate as a tool provider without a companion extension
- **Protocol stability** — Pi is young and evolving; RPC schemas may change
- **Complexity budget** — the full vision spans two runtimes and multiple integration modes

The most important strategic insight: **the bridge (Milestone 3-4) should be conceived as a Pi extension, not just a sidecar.** This is what unlocks the full value of the SDK transport—the bridge extension can register tools on behalf of Python, forward extension events, and participate in Pi's lifecycle in ways that raw RPC cannot.

**Recommendation: Proceed to implementation. Address the five critical items in Section 8.1 as Milestone 0 work. The concept is ready.**
