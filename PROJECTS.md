# Open-source project notes

[Back to my profile](https://github.com/RyderFreeman4Logos)

I build primarily in Rust, concentrating on the boundaries where agent systems meet real state: source evidence, tool effects, process lifetimes, persistence, and constrained inference hardware. This page gives a closer view of the implementation behind the [profile overview](README.md).

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

## Evidence, archives, and language access

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

### hymt

[Repository](https://github.com/RyderFreeman4Logos/hymt) · Original project · Apache-2.0

Rust translation libraries and a terminal-oriented CLI for Hy-MT2. The workspace separates token-aware segmentation, model transport, caching, translation, CLI integration, and benchmarking.

For Markdown, source offsets distinguish syntax that must be preserved from text eligible for translation. The transport layer handles incremental SSE, including split UTF-8, multiline events, truncation, and dropped connections. The CLI supports batch documents, command-output translation, translated manual pages, and bilingual document synchronization.

[Markdown structure handling](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/markdown_ownership.rs) · [Structure regression tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/tests/source_structure.rs) · [Streaming protocol tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-client/src/sse_protocol_tests.rs)

### Documentation projects

- **[ADK-Rust Simplified Chinese documentation](https://github.com/RyderFreeman4Logos/adk-rust-zh-cn):** an mdBook translation of upstream documentation, with an English source snapshot and SHA-256 source manifest.
- **[Hermes Agent bilingual documentation](https://github.com/RyderFreeman4Logos/hermes-agent-docs-i18n):** an English/Chinese mdBook mirror with incremental translation through `hymt` and source-file hash tracking.

These are community documentation projects; the linked upstream projects retain their own authorship and governance.

## Working connections between projects

| Consumer | Reused component | Implementation evidence |
| --- | --- | --- |
| `pbi` | `workflow-adk` and `workflow-runtime` from `adk-workflow-kit`, plus explicit ADK-Rust patches | [Cargo dependencies](https://github.com/RyderFreeman4Logos/pbi/blob/main/Cargo.toml) |
| `telegram-archive-bot` | `hymt` translation libraries through a separately built bridge | [Translation integration](https://github.com/RyderFreeman4Logos/telegram-archive-bot/blob/main/docs/translation.md) |
| `telegram-archive-bot` | `md2audio` speech adapter | [Speech pipeline](https://github.com/RyderFreeman4Logos/telegram-archive-bot/blob/main/docs/speech.md) |
| `gb10-services` | `llm-guard-proxy` for model routing, observability, and recovery | [Service configuration](https://github.com/RyderFreeman4Logos/gb10-services/blob/main/README.md) |
| Hermes documentation mirror | `hymt` for incremental English-to-Chinese translation | [Sync workflow](https://github.com/RyderFreeman4Logos/hermes-agent-docs-i18n/blob/master/README.md) |

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
