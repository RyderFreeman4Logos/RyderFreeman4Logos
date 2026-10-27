# Open-source project notes

[Back to my profile](https://github.com/RyderFreeman4Logos)

I build primarily in Rust, concentrating on the boundaries where agent systems meet real state: source evidence, tool effects, process lifetimes, persistence, and constrained inference hardware. This page gives a closer view of the implementation behind the [profile overview](README.md).

## How I build: approved design → ordered issues → verified implementation

My development process is intended to make coding agents useful well beyond an interactive pair-programming session. It is a **design-first, issue-driven workflow** with explicit handoff and acceptance points.

1. **Design with frontier agents.** I discuss requirements, architecture, interfaces, trade-offs, and likely failure modes with capable AI agents. I remain responsible for choosing and approving the specification rather than immediately accepting the first generated implementation.
2. **Turn the agreed design into an issue queue.** I create feature issues, often through automated generation, with implementation order, prerequisite dependencies, scoped requirements, non-goals, deliverables, and testable acceptance conditions. Issues act as persistent contracts that another agent session can pick up without needing the full design conversation.
3. **Execute sequentially with Codex.** Codex is my current implementation agent; I previously used Hermes Agent in this role. The intended execution loop is to implement the next unblocked issue, test it, document evidence, and proceed through the dependency graph. Newly discovered defects can become separate follow-up issues instead of silently weakening the original acceptance conditions.
4. **Verify before claiming completion.** Passing tests, review, explicit acceptance checks, and links to actual artifacts matter more than the issue's closed label. The approach should make it possible to finish an approved issue plan and obtain the intended software while minimizing routine human intervention, but it is **a workflow I am actively validating, not a claim of guaranteed autonomous correctness**.

There is a public trail behind this approach. An [adk-workflow-kit Sentinel issue (#235)](https://github.com/RyderFreeman4Logos/adk-workflow-kit/issues/235) specifies *implementation order 13 of 38*, dependencies, concrete deliverables, security constraints, acceptance criteria, and mandatory tests. An [akasha-otsuge storage decision issue (#3)](https://github.com/RyderFreeman4Logos/akasha-otsuge/issues/3) originated in a versioned specification and was closed with an implementation PR, architectural decision record, benchmark artifacts, recorded measurements, quality gates, and remaining limitations. The latter is an example of substantiated issue acceptance, not a claim that the entire MVP is complete.

**Why I use Codex:** More autonomous execution makes rigorous issue definitions, reproducible regression tests, independent review, and security checks more important. I want the agent to take over routine implementation while I stay responsible for architecture and acceptance—and to make multi-hour, mostly unattended work auditable by others.

## Agent execution and developer tools

### adk-workflow-kit

[Repository](https://github.com/RyderFreeman4Logos/adk-workflow-kit) · Original project · Apache-2.0

The kit turns declarative TOML into a validated intermediate representation and executes it through ADK-Rust. The CLI supports validation, graph inspection, locking, execution, checkpoint recovery, and offline replay.

The maintenance problem is more specific than running a sequence of model calls: a process may crash after a tool has acted, a resumed run may encounter different configuration, or a capability may no longer be available. The implementation records execution identity and tool-effect history, checks compatibility before resuming, and enforces declared capabilities before backend work.

Useful entry points:

- [Architecture and component boundaries](https://github.com/RyderFreeman4Logos/adk-workflow-kit/tree/main/docs/architecture).
- [Checkpoint storage and compatibility checks](https://github.com/RyderFreeman4Logos/adk-workflow-kit/blob/main/crates/workflow-runtime/src/checkpoint.rs).
- [Execution and capability regression cases](https://github.com/RyderFreeman4Logos/adk-workflow-kit/blob/main/crates/workflow-adk/tests/m1_11_execution_checkpoints.rs).
- [Real-process crash and resume fixtures](https://github.com/RyderFreeman4Logos/adk-workflow-kit/blob/main/crates/workflowctl/tests/m1_12_destructive_resume.rs).

The repository documents which execution features are implemented, experimental, planned, or deferred. Offline replay validates recorded, redacted evidence without making provider calls.

### pbi

[Repository](https://github.com/RyderFreeman4Logos/pbi) · Original project · Apache-2.0

A native Rust source-search CLI for coding agents. It combines bounded BM25 ranking and Boolean queries with parser-based extraction and optional model synthesis. The native search path does not require an external search subprocess or a model request.

Its source-admission rules place limits on files, traversal, deadlines, and output. Model citations are checked against admitted source spans. Rust extraction uses syntax parsing to select an enclosing item; approximate fallbacks are identified separately.

[Native search](https://github.com/RyderFreeman4Logos/pbi/blob/main/src/native_search.rs) · [Extraction tests](https://github.com/RyderFreeman4Logos/pbi/blob/main/tests/extract.rs) · [Model integration contract](https://github.com/RyderFreeman4Logos/pbi/blob/main/docs/model-seam.md)

### skillcfg

[Repository](https://github.com/RyderFreeman4Logos/skillcfg) · Original project · Apache-2.0

A headless Rust library and CLI for sharing runtime preferences across agent skills. It discovers conventional Codex, Hermes, Claude, and generic agent skill roots, follows and deduplicates symlinked directories, validates manifests, and explains which consumers reference a configuration key.

A bounded static scanner recognizes supported literal shell references while reporting dynamic or ambiguous cases as unknown. Configuration inspection never executes skill behavior. Visible and opaque preferences control ordinary presentation; this is preference storage, not a secret manager.

[Core library](https://github.com/RyderFreeman4Logos/skillcfg/blob/main/crates/skillcfg-core/src/lib.rs) · [Static scanner](https://github.com/RyderFreeman4Logos/skillcfg/blob/main/crates/skillcfg-core/src/validate/scanner.rs) · [Scanner regression cases](https://github.com/RyderFreeman4Logos/skillcfg/blob/main/crates/skillcfg/tests/scanner.rs)

## Translation engineering: hymt

[Repository](https://github.com/RyderFreeman4Logos/hymt) · Original Rust project · Apache-2.0 · [English documentation](https://github.com/RyderFreeman4Logos/hymt/blob/main/README.md) · [简体中文文档](https://github.com/RyderFreeman4Logos/hymt/blob/main/README.zh-cn.md)

Rather than just calling Hy-MT2 with a sentence and writing the result, `hymt` makes long, structured text and terminal workflows manageable on a self-hosted translation model. The modular Rust workspace separates tokenizer-aware planning, model transport, SQLite caches/history, orchestration, and CLI integrations. It can translate source files, whole directory trees, command output, `man`/`info` documentation, and Telegram messages. A `translate-doc` workflow supports bilingual Markdown maintenance.

**Source ownership instead of naive Markdown splitting.** The Markdown pipeline parses complete-document context and tracks original byte intervals: structural markers, code fences, leading frontmatter, reference definitions, link destinations, and opaque blocks belong to the source, while literal text leaves—including labels inside reference links and images—can be given to the model. Coherent inline paragraphs are sent as a single request when they fit, preserving surrounding context. Oversized protected sections (including tables) are retained as original bytes rather than cut mid-structure. Tests cover escaped punctuation, link normalization and duplicate references, nested inline syntax, exact occurrence matching, and bounds on interval scans.

[Source-owner implementation](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/markdown_ownership.rs) · [Ownership/complexity tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/tests/ownership_interval_tests.rs) · [Structure preservation tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/tests/source_structure.rs)

**Context-aware, inspectable request planning.** A selected Hy-MT2 profile pins the relevant tokenizer; the planner can count the complete rendered chat exchange, including role framing, template, the assistant marker, source material, output reservation, and retries for truncated responses. It accounts for service-wide context divided among parallel slots, not just the server's advertised total. When a verified tokenizer/template is absent, the CLI discloses an approximate conservative budget; strict mode refuses such planning. `--plan` exposes segment decisions and token accounting before requesting translations. For Chinese-family targets only, a deliberate CJK ratio heuristic can keep already-target-language paragraphs as byte-exact source; other targets do not claim generic language identification.

[Tokenizer and segmentation](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-segment/src/split.rs) · [Token and timing tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/tests/timing_tests.rs)

**Streaming with meaningful completion rules.** Data belongs on `stdout` and diagnostic progress on `stderr`, so the output remains useful in pipelines. Completed paragraphs may be published incrementally, but atomic JSON and structurally sensitive output is buffered. The HTTP client handles SSE split across TCP/UTF-8 boundaries, CRLF/LF, multiline events, dropped connections, and unexpected end-of-stream. Transport retry attempts preserve the same admitted logical request and never relabel partial output as a successful complete translation. Protocol-level truncation is handled separately; exhausted completeness retries can preserve the original marked paragraph under the documented warn-only policy, while strict mode or atomic JSON fails. Source preservation here is not a guarantee that the model translates or reproduces all Markdown correctly: completed model output is not silently rewritten to hide semantic or formatting failures.

[SSE protocol tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-client/src/sse_protocol_tests.rs) · [Transport, partial output and source retention](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/tests/retained_original_tests.rs) · [Cache admission/backpressure tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/tests/cache_admission_tests.rs)

**An explicit inference contract.** `llama_cpp`, `vllm` and conservative OpenAI-compatible adapters distinguish omitted sampler values (let the server decide) from explicit numeric or disabled overrides; unsupported wire representations are rejected rather than guessed. Preflight reads available runtime metadata and reports unverified identities instead of inventing model/build/defaults information. It also recognizes a specific llama.cpp metadata inconsistency: an ASCII punctuation `eos_token` alongside a control-marker chat template. It resolves the false token ID through `/tokenize` and applies targeted llama.cpp logit bias without suppressing the real control-marker EOS. Runtime identities are TTL-cached and invalidated when relevant live configuration changes; strict backend preflight is available.

[Backend preflight and EOS tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-client/src/backend_preflight_tests.rs) · [Request fingerprint and inference adapter tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-client/src/backend_payload_tests.rs)

**Cache correctness, recovery, and observability.** Application-level segment and whole-output caches are opt-in. Cache reuse is scoped to source, target, fully rendered prompts, tokenizer/profile, explicit sampling, and verified inference identity. Matching fully completed work can be locally replayed from SQLite with bounded saved proof, without a live preflight; historical metadata does not confer verification for new work. Failed, degraded, or incomplete attempts do not populate the successful replay proof. A separate optional inference-response cache supports trusted local-network consumers with its own identity/epoch controls; it is plaintext and is not an authentication boundary. The CLI tracks historical throughput, percentile-based ETA and completion statistics, with a prompt to file timing-divergence bug reports. File writes use same-directory temporary files and rename-on-success to preserve previous outputs on ordinary failures, while explicitly excluding power-loss durability guarantees.

[Replay tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/tests/retained_original_tests.rs) · [Cache policy tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/tests/cache_admission_tests.rs) · [Timing and config consistency](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/tests/timing_tests.rs)

**Reproducible quality gates, not unsupported benchmark claims.** The versioned translation corpus has more than 50 examples, including Chinese↔English, other target languages, structured Markdown/JSON/YAML/TOML, CLI syntax and adversarial truncation. The benchmark harness can compare Transformers, vLLM and llama.cpp Q6_K/Q4_K_M using chrF, exact protected-token survival, structured parsing, completeness, language residue, and streaming/total latency. It requires explicit opt-in for live inference and records model/tokenizer/backend identities to make regressions reproducible. The repository documents the suite and methodology; it does not imply any unreported benchmark win.

[Benchmark methodology](https://github.com/RyderFreeman4Logos/hymt/blob/main/benchmarks/README.md) · [Versioned corpus](https://github.com/RyderFreeman4Logos/hymt/blob/main/benchmarks/corpus/v1.json) · [Acceptance gates](https://github.com/RyderFreeman4Logos/hymt/blob/main/benchmarks/decision-gates.toml)

**Representative commands:**

```bash
hymt article.md -l zh | less
hymt translate-doc docs/ --recursive
hymt batch docs -l zh --dry-run
hymt exec -- cargo test
hymt man git-rebase
hymt backend inspect
```

These are deployable engineering features and public regression cases, not a claim that model-generated translations are semantically infallible.

## Local inference and operations

### llm-guard-proxy

[Repository](https://github.com/RyderFreeman4Logos/llm-guard-proxy) · Original project · Apache-2.0

A Rust gateway for OpenAI-compatible inference services. It handles named upstreams, bounded concurrency and queues, retries, streaming response lifecycles, loop detection, and operational visibility.

Two areas illustrate the engineering focus. First, recovery must respect whether a response has already become visible to the client. Second, low-memory recovery must remain useful when ordinary allocation or orchestration is unreliable. The code includes an offline response/recovery self-test, persisted recovery receipts, and a Linux guardian with a pre-opened cgroup control descriptor and an allocation-free emergency operation.

[Architecture](https://github.com/RyderFreeman4Logos/llm-guard-proxy/blob/main/docs/architecture.md) · [Response/recovery self-test](https://github.com/RyderFreeman4Logos/llm-guard-proxy/blob/main/crates/llm-guard-proxy/src/proxy/post_await_self_test.rs) · [Allocation-counting test](https://github.com/RyderFreeman4Logos/llm-guard-proxy/blob/main/crates/llm-guard-proxy-host-guardian/tests/no_alloc.rs)

### gb10-services

[Repository](https://github.com/RyderFreeman4Logos/gb10-services) · Original deployment and operations project · Apache-2.0

A reproducible reference layout for a GB10 inference host: chat, embeddings, reranking, the guard proxy, and host monitoring. The repository contains Docker image pins, systemd user units, readiness checks, deployment scripts, recovery procedures, and dated experiment records.

The work addresses shared-memory pressure, service identity, startup ordering, and keeping essential retrieval services available when text generation consumes too much memory. Benchmark records distinguish the tested host and configuration from broader performance claims.

[Service layout and capacity contracts](https://github.com/RyderFreeman4Logos/gb10-services/blob/main/README.md) · [Deployment guidance](https://github.com/RyderFreeman4Logos/gb10-services/tree/main/docs/deployment) · [Regression tests](https://github.com/RyderFreeman4Logos/gb10-services/tree/main/tests) · [Dated research records](https://github.com/RyderFreeman4Logos/gb10-services/tree/main/docs/research)

## Evidence and archives

### Verbatim

[Repository](https://github.com/RyderFreeman4Logos/verbatim) · Original project · Apache-2.0

A local daemon and CLI for document and code retrieval. Lexical and vector search return passages with stable evidence identities and structured locators. Optional reranking and answer generation build on those retrieved sources.

The central design concern is provenance: original source text, OCR, image artifacts, generated captions, and generated answers have different evidentiary status. Citation checking can validate generated answers against retrieved material. Search ranking alone does not establish that every relevant passage has been found.

[Evidence and safety model](https://github.com/RyderFreeman4Logos/verbatim/blob/main/docs/safety-model.md) · [Evidence identity implementation](https://github.com/RyderFreeman4Logos/verbatim/blob/main/crates/verbatim-core/src/api_evidence_identity.rs) · [SQLite durability and recovery](https://github.com/RyderFreeman4Logos/verbatim/blob/main/docs/sqlite-durability.md)

### telegram-archive-bot

[Repository](https://github.com/RyderFreeman4Logos/telegram-archive-bot) · Original Rust implementation · MIT OR Apache-2.0 · Release candidate

A service for preserving Telegram source material and delivering validated audio. Canonical SQLite records and immutable content-addressed files hold source identity and revision history; a Git repository receives a deterministic projection. Recoverable jobs track processing and external delivery separately.

The storage and recovery code includes tests for interrupted publication, concurrent captures, process death, integrity failures, and restoring backups without automatically resuming external side effects. Translation is supplied through a separately built Rust bridge to `hymt`, and speech processing integrates `md2audio`.

[Storage implementation and tests](https://github.com/RyderFreeman4Logos/telegram-archive-bot/blob/main/src/storage.rs) · [Source-revision tests](https://github.com/RyderFreeman4Logos/telegram-archive-bot/blob/main/tests/sources.rs) · [Backup/restore tests](https://github.com/RyderFreeman4Logos/telegram-archive-bot/blob/main/tests/recovery_cli.rs) · [Release status](https://github.com/RyderFreeman4Logos/telegram-archive-bot/releases)

### Documentation projects

- **[ADK-Rust Simplified Chinese documentation](https://github.com/RyderFreeman4Logos/adk-rust-zh-cn):** an mdBook translation of upstream documentation, with an English source snapshot and SHA-256 source manifest.

This is a community documentation project; the linked upstream framework retains its own authorship and governance.

## Working connections between projects

| Consumer | Reused component | Implementation evidence |
| --- | --- | --- |
| `pbi` | `workflow-adk` and `workflow-runtime` from `adk-workflow-kit`, plus explicit ADK-Rust patches | [Cargo dependencies](https://github.com/RyderFreeman4Logos/pbi/blob/main/Cargo.toml) |
| `telegram-archive-bot` | `hymt` translation libraries through a separately built bridge | [Translation integration](https://github.com/RyderFreeman4Logos/telegram-archive-bot/blob/main/docs/translation.md) |
| `telegram-archive-bot` | `md2audio` speech adapter | [Speech pipeline](https://github.com/RyderFreeman4Logos/telegram-archive-bot/blob/main/docs/speech.md) |
| `gb10-services` | `llm-guard-proxy` for model routing, observability, and recovery | [Service configuration](https://github.com/RyderFreeman4Logos/gb10-services/blob/main/README.md) |

## Maintained forks and upstream work

### ADK-Rust fork

[My fork](https://github.com/RyderFreeman4Logos/adk-rust) · [Upstream: zavora-ai/adk-rust](https://github.com/zavora-ai/adk-rust)

My maintained changes concentrate on provider fidelity and execution lifecycle:

- [Subprocess custody across sandbox and Rust execution, #34](https://github.com/RyderFreeman4Logos/adk-rust/pull/34): preserve ownership of process groups, I/O tasks, and temporary resources through cancellation, timeout, and response-delivery failure.
- [Terminal streaming output, #29](https://github.com/RyderFreeman4Logos/adk-rust/pull/29): retain text, reasoning, and tool parts carried in terminal OpenAI-compatible stream events, with redacted malformed-event diagnostics.
- [Schema preservation, #16](https://github.com/RyderFreeman4Logos/adk-rust/pull/16): preserve caller-provided JSON Schema and explicitly reject unsupported strict-mode forms.

These PRs were merged into my fork. Their links identify the implementation and review history without implying upstream acceptance.

### Hermes Agent fork

[My fork](https://github.com/RyderFreeman4Logos/hermes-agent) · [Upstream: NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)

A maintained integration of patches for long-running agents: cache-aware routing, context handling, bounded compression, session recovery, and process lifecycle. The [integration PR, #188](https://github.com/RyderFreeman4Logos/hermes-agent/pull/188), records retained behaviors, source patches, upstream baselines, and validation status.

One focused example is [bounded display-history loading after repeated compression, #422](https://github.com/RyderFreeman4Logos/hermes-agent/pull/422): the display path loads a bounded amount of historical data while model continuation retains the complete active context. The public ledger distinguishes work in the fork from proposals sent upstream.

### Contributions merged by external maintainers

**[rspow](https://github.com/zolagonano/rspow)** accepted four contributions covering proof-of-work difficulty and algorithm dispatch ([#1](https://github.com/zolagonano/rspow/pull/1)), concurrent solving ([#2](https://github.com/zolagonano/rspow/pull/2)), parallel EquiX and proof bundles ([#3](https://github.com/zolagonano/rspow/pull/3)), and feature-gated backends with resumable solving and protocol helpers ([#4](https://github.com/zolagonano/rspow/pull/4)).

**[nazgul](https://github.com/edwinhere/nazgul)** accepted conditional CLSAG Serde support in [#3](https://github.com/edwinhere/nazgul/pull/3). My [separate fork](https://github.com/RyderFreeman4Logos/nazgul) also contains explicit RNG injection, public-only key derivation, and bLSAG API/testing work.

## Current research: akasha-otsuge

[Repository](https://github.com/RyderFreeman4Logos/akasha-otsuge) · Original project · Apache-2.0 · MVP in development

The goal is local, traceable memory across coding agents. The implemented foundation covers immutable source evidence, atomic versioned writes, durable idempotency receipts, scoped provenance and review rules, and checksum-verified backup/restore.

The storage layer includes native transaction tests and subprocess crash-recovery fixtures. Host integrations, the complete daemon/IPC path, and product retrieval remain on the roadmap.

[Storage contract](https://github.com/RyderFreeman4Logos/akasha-otsuge/blob/main/docs/storage.md) · [Provenance and review rules](https://github.com/RyderFreeman4Logos/akasha-otsuge/blob/main/docs/domain.md) · [Crash-recovery fixtures](https://github.com/RyderFreeman4Logos/akasha-otsuge/blob/main/src/storage/crash_tests.rs)

## More tools and earlier work

| Project | Scope and useful entry point |
| --- | --- |
| [mesh-proxy](https://github.com/RyderFreeman4Logos/mesh-proxy) | Rust service networking over Iroh/QUIC, with TCP/Unix-socket forwarding, invitation-based joining, and persisted routes. [Invite lifecycle tests](https://github.com/RyderFreeman4Logos/mesh-proxy/blob/main/crates/mesh-core/tests/invite_flow.rs). |
| [md2audio](https://github.com/RyderFreeman4Logos/md2audio) | Markdown-to-audio with subtitles and optional video; includes explicitly selected offline VITS synthesis for Linux and Android/Termux. [Local speech tests](https://github.com/RyderFreeman4Logos/md2audio/blob/main/tests/test_local_tts.py). |
| [openwebui-skill](https://github.com/RyderFreeman4Logos/openwebui-skill) | Rust CLI for model conversations with native Open WebUI visibility and local JSONL session history. [Session implementation](https://github.com/RyderFreeman4Logos/openwebui-skill/blob/main/src/session.rs). |
| [cmd-proposer](https://github.com/RyderFreeman4Logos/cmd-proposer) | Command proposals grounded in documentation, with policy checks, explicit approval, and an argv-based execution path. [Execution boundary](https://github.com/RyderFreeman4Logos/cmd-proposer/blob/main/crates/cps-exec/src/lib.rs). |
| [model-bridge](https://github.com/RyderFreeman4Logos/model-bridge) | Modular Rust inference relay with provider adapters, streaming, quotas, and cache-affinity routing. [Routing implementation](https://github.com/RyderFreeman4Logos/model-bridge/blob/main/crates/mb-core/src/core_domain/cache_router.rs). |
| [git-bak](https://github.com/RyderFreeman4Logos/git-bak) | Filesystem-to-Git backups with change watching, checkpoints, hooks, and coordinated Git operations. [Checkpoint implementation](https://github.com/RyderFreeman4Logos/git-bak/blob/main/crates/git-bak-core/src/checkpoint.rs). |
| [mvln](https://github.com/RyderFreeman4Logos/mvln) | A Rust move-and-link utility that relocates files while preserving their original paths through symlinks. [Safety regression cases](https://github.com/RyderFreeman4Logos/mvln/blob/main/tests/safety.rs). |
| [libre-pdwise](https://github.com/RyderFreeman4Logos/libre-pdwise) | Rust audio/video transcription tooling with local and cloud ASR options. |
| [whisper-bot-zh](https://github.com/RyderFreeman4Logos/whisper-bot-zh) | Telegram speech transcription with configurable OpenAI-compatible ASR and optional text correction. |
| [OnionVault](https://github.com/OnionVault/onion_vault) | Earlier Rust password-manager prototype exploring hardware-backed signing and per-item layered encryption. Includes reusable vault and signature-to-key crates; [workspace authorship](https://github.com/OnionVault/onion_vault/blob/main/Cargo.toml). |
| [nazgul-zkp](https://github.com/RyderFreeman4Logos/nazgul-zkp) | Experimental Halo2 ring-membership and linkable-signature constructions. Research code, with no claim of an independent cryptographic audit. [Circuit implementation](https://github.com/RyderFreeman4Logos/nazgul-zkp/blob/main/crates/nazgul-zkp/src/zk/circuit.rs). |

---

Project descriptions and contribution status last reviewed **2026-10-26**. Each repository contains its current platform, release, and validation details.
