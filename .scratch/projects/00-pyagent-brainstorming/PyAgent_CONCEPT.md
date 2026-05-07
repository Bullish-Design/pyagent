# CONCEPT.md

# pi-agent-py

A Pythonic, Pydantic-first interface library for the Pi Agent framework and ecosystem, built with a reproducible Nix/devenv.sh workflow.

---

## 1. Executive summary

`pi-agent-py` is a Python control-plane and integration library for the Pi Agent ecosystem. Its goal is to give Python developers a clean, typed, Pythonic API for using Pi as an agent runtime while preserving Pi’s native strengths: its execution harness, session model, skills, prompt templates, extensions, and programmatic integration surfaces.

The project will start with **Pi RPC mode** as the MVP integration path because RPC is explicitly intended for embedding Pi in non-Node applications via a JSON protocol over stdin/stdout. The long-term target is a **Node bridge built on the Pi SDK**, which provides richer direct access to Pi session and runtime primitives such as `createAgentSession` and `SessionManager`. The project will intentionally stop there unless a later need emerges for deeper custom TypeScript extension/package generation.

The library is not meant to replace Pi. It is meant to provide:

- a **Pythonic and Pydantic-first public API**
- a **stable abstraction layer** over Pi’s integration surfaces
- a **typed orchestration and persistence layer** for traces, sessions, tools, instructions, and evaluations
- a **controlled self-improvement pipeline** that can later support LoRA fine-tuning based on curated outcomes
- a **fully reproducible development and production workflow** using `devenv.sh` on NixOS

In short, the concept is:

> **Use Pi as the execution engine and build a Python-native control plane around it.**

---

## 2. Core goal

The core goal is to create a Python library that makes Pi feel native to Python developers while remaining faithful to Pi’s actual architecture.

That means:

- Pi should remain the underlying agent runtime.
- Python should become the primary interface for application developers, orchestrators, evaluators, and training workflows.
- Pydantic should define the canonical schema for public objects and persisted internal artifacts.
- Nix + `devenv.sh` should define the canonical development, test, build, and deployment environment.

The resulting system should let a developer:

- start and manage Pi sessions from Python
- send prompts and receive typed responses/events
- stream tool execution updates
- preserve, replay, and analyze Pi sessions
- compose instruction packs and skill references in Python
- store traces and evaluations in a database
- later curate high-quality traces for LoRA fine-tuning
- move from a subprocess-based RPC integration to a richer Node bridge without breaking the Python API

---

## 3. Why this should exist

Pi is TypeScript-first. That is a strength for Pi, but it creates friction for Python-first teams that want:

- strong typed models for application state
- clean orchestration in Python
- Python-native data processing and evaluation workflows
- direct integration with Python ML tooling
- database-backed agent analytics and dataset generation

A Python wrapper that merely forwards raw JSON is not enough. The missing layer is a **Python domain model** that normalizes Pi concepts into stable objects and workflows.

This project exists to fill that gap.

---

## 4. Design principles

### 4.1 Pi is the runtime, not the target for reimplementation

The library should not reimplement Pi’s execution engine in Python. Pi already provides:

- an interactive and headless agent runtime
- a programmatic RPC protocol
- an SDK for embedding in Node.js applications
- a sessions model with tree-structured JSONL storage
- a skills, prompt-template, and extension ecosystem

The library should wrap and extend these capabilities rather than compete with them.

### 4.2 Pydantic is the contract boundary

All public-facing Python objects should be represented by Pydantic models. This keeps the API:

- explicit
- testable
- serializable
- schema-friendly
- resilient to upstream JSON shape changes

### 4.3 RPC first, SDK bridge second

The MVP should use Pi’s RPC mode because it is designed for non-Node integrations and has a simple, inspectable JSON-over-stdio protocol.

The full target should use a Node bridge over the Pi SDK for richer control and more direct embedding.

### 4.4 Nix/devenv is not a convenience feature; it is part of the architecture

The project will be built on NixOS with `devenv.sh`, which means the environment should be designed as a reproducible system artifact from day one.

That includes:

- pinned toolchains
- pinned Python and Node runtimes
- reproducible dev shells
- deterministic CI tasks
- service orchestration for local integration testing
- controlled secrets injection
- reproducible packaging and container outputs

### 4.5 Stable Python API, swappable transport layer

The Python API should remain stable even if the underlying transport changes from RPC subprocesses to a Node bridge.

Transport concerns should be internal.

### 4.6 Self-improvement must be curated and governed

If the system later supports trace-based training, only validated, high-quality traces should be admitted into the training corpus. The system must avoid recursive self-training without evaluation gates.

---

## 5. High-level architecture

The recommended architecture has four layers.

### 5.1 Layer 1: Python domain layer

This is the canonical semantic layer of the library. It contains Pydantic models representing:

- client configuration
- session configuration
- prompt requests and responses
- events
- tools
- skills
- instruction packs
- traces
- evaluations
- dataset examples
- model/adapter metadata

This layer defines the stable API contract.

### 5.2 Layer 2: transport layer

This layer knows how to talk to Pi.

It should provide a common internal interface such as:

- `start_session`
- `send_prompt`
- `stream_events`
- `close`
- `interrupt`
- `resume`

Transport implementations:

- **MVP:** `RpcTransport`
- **Final target:** `NodeSdkTransport`

### 5.3 Layer 3: runtime/application layer

This layer provides developer-facing operations:

- session lifecycle
- fluent synchronous and asynchronous APIs
- trace capture
- session replay
- orchestration across agents
- skill/instruction resolution
- storage integration
- evaluation workflows

### 5.4 Layer 4: governance and training layer

This layer is not part of the MVP, but the concept should reserve space for it.

It includes:

- trace quality scoring
- regression benchmarks
- evaluator pipelines
- curated dataset extraction
- LoRA training jobs
- candidate promotion and rollback

---

## 6. Why MVP starts with RPC

Pi’s RPC mode is specifically intended for embedding Pi in other applications through a JSON protocol over stdin/stdout. That makes it the best MVP integration for Python.

### 6.1 Advantages of RPC for MVP

- simple subprocess model
- no embedded TypeScript runtime inside the Python package
- easy to inspect and debug
- easy to record and replay raw protocol traffic
- minimal coupling to internal TypeScript implementation details
- can be used immediately from Python with asyncio subprocesses

### 6.2 What the MVP should support through RPC

- starting a Pi process in RPC mode
- sending prompt commands
- receiving request/response acknowledgements
- streaming message updates
- streaming tool execution events
- collecting final outputs
- session persistence configuration
- graceful shutdown and timeout handling
- transcript and raw event capture

### 6.3 Limits of RPC

RPC is excellent for the initial integration, but some deeper capabilities are better handled through the SDK.

Examples of things that may be easier or richer with an SDK-backed bridge:

- more direct session lifecycle manipulation
- richer resource loading and prompt template integration
- more ergonomic access to Pi service/runtime abstractions
- advanced embedded use cases

That is why the architecture should be designed to graduate to a Node bridge later without changing the Python public API.

---

## 7. Why the full target is a Node bridge over the Pi SDK

Pi’s SDK is the most natural way to embed Pi into a Node/TypeScript application. It exposes session and runtime primitives such as `createAgentSession` and `SessionManager`, and documents session strategies including in-memory, persistent creation, continuing recent sessions, and opening specific session files.

A Node bridge is the right full target because it lets Python keep its clean control-plane role while gaining access to richer Pi-native capabilities.

### 7.1 Benefits of a Node bridge

- uses Pi the way it was designed to be embedded programmatically
- reduces dependence on subprocess protocol specifics
- allows richer SDK-level features without exposing TS internals to Python users
- gives a place to expose higher-level endpoints for session management and asset discovery
- still keeps the Python public API stable

### 7.2 Bridge shape

The Node bridge can be implemented as one of:

- a subprocess sidecar with JSON-over-stdio
- a local HTTP server
- a local WebSocket service
- a Unix-domain-socket service

For consistency with the MVP, a JSON-over-stdio subprocess sidecar is the least disruptive progression.

### 7.3 What the bridge should expose

At minimum:

- create session
- continue recent session
- open session file
- list sessions
- prompt session
- stream session events
- get session metadata
- inspect or resolve resources/skills/prompts
- close/interrupt/resume session

---

## 8. Pi concepts and how they should map into Python

The wrapper should not merely rename Pi objects. It should normalize them into Python-native, typed objects.

### 8.1 Sessions

Pi sessions are stored as JSONL files and form a tree structure through `id` and `parentId`. The wrapper should expose this as a typed Python session and trace model.

Suggested Python concept split:

- `PiSessionHandle`: a live runtime session
- `PiSessionSnapshot`: a persisted/replayable session view
- `SessionBranch`: a typed branch representation
- `Trace`: a normalized runtime trace derived from events and persisted messages

### 8.2 Skills

Pi already has a skills model. In Python, skills should be represented as references and metadata, not reimplemented execution artifacts.

Suggested concepts:

- `PiSkillRef`
- `SkillSource`
- `ResolvedSkill`
- `SkillBundle`

The wrapper should be able to:

- reference existing Pi skills
- validate their metadata
- compose sets of skills into session configs
- optionally generate local manifest-style assets where appropriate

### 8.3 Prompt templates and instructions

Prompt templates and instructions should become first-class Python assets.

Suggested concepts:

- `InstructionPack`
- `InstructionRef`
- `PromptTemplateRef`
- `PromptPolicy`

This lets application code reason about behavior in a typed way.

### 8.4 Events

Pi RPC provides streamed events including message lifecycle and tool execution lifecycle updates. The wrapper should define discriminated Pydantic event unions so that application code can consume them naturally.

Examples:

- `AgentStartEvent`
- `TurnStartEvent`
- `MessageUpdateEvent`
- `ToolExecutionStartEvent`
- `ToolExecutionEndEvent`
- `AgentEndEvent`
- `ExtensionErrorEvent`

### 8.5 Model/provider/runtime config

The wrapper should make provider/model/session settings explicit.

Suggested concepts:

- `ProviderRef`
- `ModelRef`
- `ThinkingLevel`
- `QueueBehavior`
- `SessionPersistenceMode`

---

## 9. Public Python API design

The Python API should feel idiomatic rather than CLI-shaped.

### 9.1 API goals

- typed configuration
- context-manager-friendly sessions
- sync and async clients
- event streaming as iterators/async iterators
- explicit exceptions
- minimal hidden global state

### 9.2 Example user experience

```python
from pi_agent_py import PiClient, PiSessionConfig, PromptRequest

client = PiClient.from_env()

config = PiSessionConfig(
    working_dir=".",
    provider="anthropic",
    model="anthropic/claude-sonnet",
)

with client.session(config) as session:
    response = session.prompt(
        PromptRequest(message="Analyze this repository and propose a refactor plan")
    )

print(response.final_text)
```

### 9.3 Async example

```python
from pi_agent_py import AsyncPiClient

client = AsyncPiClient.from_env()

async with client.session(config) as session:
    await session.prompt("Inspect the codebase")
    async for event in session.events():
        print(event)
```

### 9.4 Recommended top-level objects

- `PiClient`
- `AsyncPiClient`
- `PiSession`
- `AsyncPiSession`
- `PiSessionConfig`
- `PromptRequest`
- `PromptResponse`
- `PiEvent`
- `PiTrace`
- `InstructionPack`
- `PiSkillRef`

---

## 10. Pydantic-first schema strategy

Pydantic should be used for two distinct purposes.

### 10.1 Public API schemas

These are the developer-facing models.

Examples:

- `PiClientConfig`
- `PiSessionConfig`
- `PromptRequest`
- `PromptResponse`
- `EventEnvelope`

### 10.2 Internal persistence schemas

These are the normalized records that the library stores or analyzes.

Examples:

- `TraceRecord`
- `TraceStep`
- `SessionSnapshot`
- `EvaluationRecord`
- `TrainingExample`
- `PreferencePair`

### 10.3 Guiding rule

The public schema should be stable and ergonomic.

The internal schema should be rich and loss-aware. It should preserve raw protocol payloads where useful, but also normalize them for analytics and replay.

---

## 11. Transport abstraction

The transport layer should be hidden from users but explicit in the architecture.

### 11.1 Internal protocol

Conceptually:

```python
class PiTransport:
    async def start(self, config): ...
    async def prompt(self, request): ...
    async def events(self): ...
    async def close(self): ...
```

### 11.2 `RpcTransport`

Responsibilities:

- spawn `pi --mode rpc`
- frame outbound JSONL commands
- parse inbound JSONL responses/events
- correlate responses with command IDs
- buffer streamed events
- handle cancellation, restart, and teardown
- capture raw traffic for debugging and replay

### 11.3 `NodeSdkTransport`

Responsibilities:

- launch or connect to a local Node bridge
- call bridge APIs for session lifecycle and prompting
- receive normalized events
- preserve bridge/runtime metadata
- expose SDK-backed features through the same Python abstractions

### 11.4 Why this matters

The transport abstraction is what makes the MVP-to-final migration possible without a public breaking change.

---

## 12. Storage and persistence strategy

The system should keep both raw and normalized artifacts.

### 12.1 Raw artifacts

- raw Pi session files
- raw RPC command/response/event logs
- raw bridge request/response/event logs

### 12.2 Normalized artifacts

- session metadata rows
- trace rows
- trace steps
- event summaries
- evaluations
- dataset examples
- model adapter metadata

### 12.3 Database recommendation

Use `SQLModel` as the database layer for:

- sessions
- traces
- events
- evaluations
- datasets
- training runs
- model deployments

This aligns with a Pythonic persistence stack and supports a typed repository pattern.

---

## 13. Session and trace model

Pi’s session format is valuable because it supports branching through a tree of entries linked by `id` and `parentId`. The Python library should lean into that rather than flatten it away.

### 13.1 Why branch-aware sessions matter

- replaying agent decisions
- comparing alternative outcomes from the same state
- debugging regressions
- extracting preferred vs rejected trajectories
- supporting preference-pair generation later

### 13.2 Normalized trace structure

A normalized trace should include:

- task metadata
- session metadata
- active instructions and skills
- model/provider configuration
- step-by-step events
- tool execution records
- final answer
- outcome labels
- evaluator scores

### 13.3 Why trace normalization matters

Raw session JSONL is good as a source artifact. A normalized trace is better for:

- analytics
- search
- benchmarking
- dataset extraction
- auditing
- training governance

---

## 14. Skill and instruction strategy

The Python layer should treat skills and instructions as separate but related concepts.

### 14.1 Skills

Skills are Pi-native reusable capabilities or assets. In Python, they should be modeled as references with metadata.

### 14.2 Instructions

Instructions are policy-like behavioral overlays that should be easy to define, compose, and version in Python.

### 14.3 Recommended distinction

- **Skills**: references to Pi-native resources
- **Instructions**: Python-owned behavioral assets that may resolve into prompt content or policy bundles

### 14.4 Why this matters

It lets the Python library be opinionated about behavior without pretending to own Pi’s full extension model.

---

## 15. NixOS + devenv.sh as a first-class architectural advantage

The project will be developed using `devenv.sh` on NixOS, and this should materially shape the design.

This is not just about easier onboarding. It is about making the environment part of the product’s correctness story.

### 15.1 Why devenv.sh fits this project

The project depends on multiple ecosystems at once:

- Python
- Node.js / npm or pnpm
- the Pi CLI/runtime
- optional databases and local services
- code generation, linting, formatting, and test tooling

`devenv.sh` is well-suited because it provides declarative developer environments using Nix and supports packages, tasks, processes, services, secrets, git hooks, profiles, tests, and outputs.

### 15.2 Development environment goals

The environment should guarantee:

- a pinned Python interpreter and tooling stack
- a pinned Node.js runtime and package manager
- reproducible availability of Pi CLI and bridge dependencies
- stable local paths and environment variables
- identical dev/test/CI shell behavior
- minimal “works on my machine” failure modes

### 15.3 Production environment goals

The same Nix/devenv definitions should inform production packaging for:

- Python application containers
- bridge service containers
- test runners
- integration environments

### 15.4 Why this is especially important here

This project spans a Python package, a Node bridge, and an external agent runtime. Without environment determinism, integration issues will dominate development.

With Nix/devenv, the project can make the runtime topology explicit and reproducible.

---

## 16. How devenv should be used in practice

### 16.1 Single source of environment truth

The repository should treat `devenv.nix` and related Nix files as the canonical declaration for:

- language runtimes
- package dependencies needed outside language package managers
- development tasks
- integration test topology
- local services
- environment variables
- secrets wiring

### 16.2 Use devenv tasks for workflow standardization

Recommended tasks:

- `format`
- `lint`
- `typecheck`
- `test`
- `test-unit`
- `test-integration`
- `bridge:dev`
- `bridge:test`
- `rpc:smoke`
- `build`
- `package`

This makes local and CI execution consistent.

### 16.3 Use processes for local integration topology

A local development topology may include:

- Python package in editable mode or test mode
- Pi runtime in RPC mode for smoke tests
- Node bridge service for SDK integration testing
- optional database service for trace persistence

`devenv` processes should define and manage these cohesively.

### 16.4 Use services where helpful

If local supporting services are needed, such as PostgreSQL or Redis, they should be managed declaratively through `devenv` rather than manually.

### 16.5 Use git hooks for quality gates

Recommended hooks:

- formatter
- linter
- import sorting
- basic static type checking
- markdown linting
- generated-code consistency checks

### 16.6 Use outputs for reproducible deployables

As the project matures, Nix outputs should be used to create:

- Python package artifacts
- bridge runtime bundles or container images
- integration test environments
- CI-consumable build outputs

### 16.7 Use profiles to separate concerns

Potential profiles:

- `dev`
- `ci`
- `rpc-only`
- `bridge`
- `full`

This keeps the environment lean when only certain project parts are needed.

### 16.8 Use secrets support intentionally

Provider API keys and related credentials should not be handled ad hoc. `devenv` secrets or dotenv integration should be used to define a consistent local-development story without baking secrets into the repo.

---

## 17. Recommended repository layout

```text
pi-agent-py/
  devenv.nix
  devenv.yaml
  flake.nix
  .envrc
  pyproject.toml
  package.json
  pnpm-lock.yaml
  README.md
  docs/
    CONCEPT.md
    architecture/
  src/
    pi_agent_py/
      core/
        models.py
        enums.py
        errors.py
        protocols.py
      transport/
        rpc.py
        node_bridge.py
        framing.py
        events.py
      runtime/
        client.py
        session.py
        streaming.py
        traces.py
        orchestration.py
      assets/
        skills.py
        instructions.py
        prompts.py
      storage/
        tables.py
        engine.py
        repos.py
      evals/
        scorers.py
        benchmarks.py
      training/
        datasets.py
        lora.py
        promotion.py
  bridge/
    src/
      index.ts
      api/
      sdk/
      session/
  tests/
    unit/
    integration/
    fixtures/
```

---

## 18. MVP scope

The MVP should be intentionally narrow and focused.

### 18.1 Included in MVP

- Pydantic core models
- sync and async Python client APIs
- `RpcTransport`
- session startup and shutdown
- prompt submission
- streaming event handling
- response assembly
- raw traffic recording
- normalized trace capture
- SQLModel persistence for sessions/traces/events
- smoke tests against a real Pi RPC process
- reproducible `devenv` workflow for development and CI

### 18.2 Excluded from MVP

- Node bridge
- SDK-native session management
- advanced asset/resource discovery
- automated training pipeline
- multi-agent orchestration engine
- custom TypeScript extension generation

### 18.3 MVP success criteria

The MVP is successful if a Python developer can:

- open a reproducible dev shell
- run tests and smoke checks deterministically
- start a Pi session from Python
- send prompts and consume streamed events
- persist and replay traces
- build an application on top of the wrapper without caring about raw JSONL framing

---

## 19. Final target scope

The final target extends the MVP without replacing its public API.

### 19.1 Included in final target

- `NodeSdkTransport`
- bridge lifecycle tooling
- session creation/continuation/open/list via SDK
- richer resource and prompt-template access
- improved session metadata and state introspection
- more advanced trace normalization
- benchmark and evaluation framework
- curated dataset extraction for LoRA training

### 19.2 Explicitly deferred

The project will **not** plan for a V3 centered on custom TypeScript extension/package generation unless future real-world usage proves it necessary.

This deferral is intentional. It keeps the design focused and avoids speculative complexity.

---

## 20. Testing strategy

Testing should be designed for the multi-runtime nature of the project.

### 20.1 Unit tests

- Pydantic model validation
- JSONL framing/parsing
- event normalization
- response assembly
- repository logic

### 20.2 Integration tests

- Python ↔ Pi RPC subprocess
- timeout and cancellation handling
- session persistence behavior
- trace extraction from real sessions

### 20.3 Bridge integration tests

Once the bridge exists:

- Python ↔ bridge ↔ Pi SDK
- session lifecycle operations
- event propagation correctness
- parity tests between RPC and bridge behaviors where applicable

### 20.4 Reproducibility tests

`devenv` should define repeatable test tasks that run identically locally and in CI.

### 20.5 Fixture strategy

Maintain:

- recorded raw RPC streams
- recorded normalized traces
- sample session JSONL files
- bridge response fixtures

This enables deterministic regression testing.

---

## 21. Observability and debugging strategy

This project should be unusually strong in observability because agent integration bugs are often subtle.

### 21.1 Capture raw and normalized artifacts

Always preserve the option to record:

- raw commands
- raw responses
- raw events
- normalized events
- final trace records

### 21.2 Correlation IDs

Every request/response/event chain should have correlation IDs at the Python layer, regardless of transport.

### 21.3 Replay tools

The library should eventually provide replay tooling for:

- raw RPC logs
- bridge event logs
- persisted Pi session files

### 21.4 Why this matters

This makes the system auditable and dramatically reduces debugging time for race conditions, streaming assembly problems, and session-branching issues.

---

## 22. Security and secrets strategy

The project should assume that it will interact with model providers and possibly system tools.

### 22.1 Secrets handling

Secrets should be injected through environment configuration managed by `devenv` and not hard-coded in repo files or test fixtures.

### 22.2 Principle of least privilege

The wrapper should be explicit about:

- provider credentials used
- working directory scope
- session persistence locations
- logging of potentially sensitive prompts/responses

### 22.3 Safe logging modes

Provide configuration for:

- full raw capture
- metadata-only capture
- redacted capture

This matters both for development and future enterprise usage.

---

## 23. Performance considerations

The design should not over-optimize early, but it should avoid obvious bottlenecks.

### 23.1 RPC performance considerations

- incremental JSONL parsing
- non-blocking stdout/stderr reading
- bounded event buffering
- async-first internals even if sync wrappers exist

### 23.2 Bridge performance considerations

- low-overhead local IPC
- backpressure-aware event streaming
- minimal serialization churn between bridge and Python

### 23.3 Trace persistence considerations

- append-friendly storage for raw logs
- normalized indexing for common queries
- separate archival path for large raw artifacts if necessary

---

## 24. Evolution path toward evaluation and training

This project’s long-term direction includes self-improvement, but that must be built on controlled data workflows.

### 24.1 What the wrapper should enable first

- capture traces cleanly
- normalize traces
- score traces with explicit evaluators
- identify high-quality trajectories

### 24.2 What should come later

- generate supervised fine-tuning examples
- generate preference pairs from branched trajectories or evaluator comparisons
- train candidate LoRA adapters
- benchmark candidate adapters against held-out tasks
- promote only when improvements are demonstrated

### 24.3 What should not happen

The system should not blindly train on its own outputs just because they exist.

The right pattern is:

- trace capture
- validation
- scoring
- curation
- training
- regression evaluation
- gated promotion

This keeps the “self-evolving” vision grounded and safe.

---

## 25. Risks and mitigations

### 25.1 Risk: transport drift between Pi versions

Mitigation:

- strong transport abstraction
- recorded protocol fixtures
- compatibility test matrix
- version metadata on traces and sessions

### 25.2 Risk: Python API leaks transport details

Mitigation:

- hide framing and protocol details behind typed abstractions
- keep raw payloads available only for debugging and advanced users

### 25.3 Risk: environment complexity across Python + Node + Pi

Mitigation:

- Nix/devenv as the canonical environment definition
- one-command onboarding
- task-based workflows
- pinned toolchains

### 25.4 Risk: premature expansion into full extension ecosystem ownership

Mitigation:

- defer V3 work explicitly
- keep focus on RPC MVP and SDK bridge final target

### 25.5 Risk: training on low-quality traces later

Mitigation:

- build evaluation and curation into the architecture from the beginning
- do not conflate trace collection with training eligibility

---

## 26. Milestones

### Milestone 0: environment foundation

- Nix flake and `devenv` setup
- pinned Python + Node toolchains
- task definitions
- test runner setup
- local secrets strategy

### Milestone 1: Pydantic core + RPC MVP

- core models
- RPC framing/parser
- async transport
- sync wrapper
- streaming event model
- smoke tests against real Pi RPC

### Milestone 2: storage + trace normalization

- SQLModel schema
- trace repositories
- raw artifact capture
- replay helpers

### Milestone 3: bridge prototype

- Node bridge service
- minimal SDK-backed session operations
- parity tests with RPC wrapper

### Milestone 4: full bridge integration

- session lifecycle operations
- resource and template access
- richer introspection
- bridge-based integration suite

### Milestone 5: evaluation and dataset preparation

- evaluator interfaces
- benchmark runs
- dataset extraction from curated traces

---

## 27. Non-goals

The following are intentionally out of scope for the initial concept.

- rewriting Pi in Python
- replacing Pi’s extension ecosystem
- committing to custom TS extension generation now
- automatic self-training without governance
- ad hoc environment setup outside Nix/devenv

---

## 28. Final recommendation

The best implementation path is:

1. Build a **Pydantic-first Python SDK** whose public interface models Pi concepts cleanly.
2. Use **Pi RPC mode** as the MVP transport because it is explicitly designed for embedding in non-Node applications.
3. Normalize Pi sessions, streamed events, and traces into Python-owned models and persistence tables.
4. Use **NixOS + `devenv.sh`** as the canonical environment and workflow layer so the Python package, Pi runtime, and later Node bridge all run reproducibly.
5. Graduate to a **Node bridge over the Pi SDK** as the full target, while keeping the Python public API unchanged.
6. Defer any V3 work around custom TypeScript extension/package generation unless real usage demonstrates the need.

This path is the most technically sound because it aligns with Pi’s actual integration model, respects environment determinism as a system requirement, and leaves room for future evaluation and training workflows without prematurely overbuilding the stack.

---

## 29. Reference notes

The concept above is aligned with the current Pi and devenv documentation:

- Pi documents **RPC mode** as a JSON protocol over stdin/stdout for embedding the coding agent in other applications, IDEs, or custom UIs.
- Pi documents **SDK embedding** for Node.js and describes session creation and management primitives such as `createAgentSession` and `SessionManager`.
- Pi documents **session files** as JSONL files whose entries form a tree via `id` and `parentId`.
- devenv documents a declarative Nix-based environment with support for packages, tasks, processes, services, git hooks, profiles, outputs, and secrets-oriented environment handling.

