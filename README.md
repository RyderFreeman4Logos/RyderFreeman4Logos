# Ryder Freeman

**Independent open-source maintainer · Rust systems · Agent tooling · Local AI · Translation**

I build tools that make AI workflows **inspectable, recoverable, and practical to run on your own infrastructure**. My work spans agent execution, source-grounded retrieval, local inference, translation systems, and durable archives. Rust is my primary language; Python, Linux, Docker, and systemd connect the pieces.

I develop with coding agents and publish the implementation, design decisions, and regression cases alongside the code.

[Development workflow](#development-workflow) · [Selected projects](#selected-projects) · [Maintained forks](#maintained-forks) · [Upstream contributions](#merged-upstream-contributions) · [Technical project notes](https://github.com/RyderFreeman4Logos/RyderFreeman4Logos/blob/main/PROJECTS.md) · [Contact](https://t.me/Ryder_support_bot)

## Development workflow

I use a **design-first, issue-driven** approach to AI-assisted software development. First I work through requirements, architecture, trade-offs, and failure cases with frontier AI agents until I approve the design. I turn that specification into a dependency-ordered set of GitHub feature issues—often generated automatically—with implementation requirements and acceptance tests. **Codex now works through those issues in sequence**, a role I previously filled with Hermes Agent.

My goal is a repeatable path from an approved design to tested software that supports **long, largely unattended development runs**. Closing every issue is an objective, not a substitute for verification: tests, review, and recorded acceptance evidence determine whether the software meets its specification. See an [ordered 38-item plan](https://github.com/RyderFreeman4Logos/adk-workflow-kit/issues/235) and an [accepted issue with reproducible evidence](https://github.com/RyderFreeman4Logos/akasha-otsuge/issues/3).

## Selected projects

### [adk-workflow-kit](https://github.com/RyderFreeman4Logos/adk-workflow-kit)

**Recoverable agent workflows in Rust.** Compiles declarative TOML workflows into validated execution graphs on ADK-Rust. Implements durable checkpoints, resume and inspection, offline replay, and explicit capability policies for tool execution.

The interesting work is in the failure paths: retaining tool-effect history, checking compatibility before resuming, and testing recovery after a real process crash.

[Architecture](https://github.com/RyderFreeman4Logos/adk-workflow-kit/tree/main/docs/architecture) · [Crash/resume tests](https://github.com/RyderFreeman4Logos/adk-workflow-kit/blob/main/crates/workflowctl/tests/m1_12_destructive_resume.rs) · [Maintenance](https://github.com/RyderFreeman4Logos/adk-workflow-kit/commits/main/)

### [hymt](https://github.com/RyderFreeman4Logos/hymt)

**Neural translation engineered for actual developer workflows—not just a model API wrapper.** A Rust workspace and CLI for Hy-MT2, covering Markdown/document trees, terminal output, man/info pages, and optional Telegram translation.

- **Source-owned Markdown.** Parser byte ranges protect syntax such as code fences, YAML frontmatter, link destinations and reference definitions while exposing text and link labels for translation. Mixed-language planning can preserve confidently detected Chinese-family paragraphs byte-for-byte. Oversized protected blocks remain intact.
- **Full-request token budgeting.** Plans against the model tokenizer, rendered chat template, per-request context, output allowance, and retry reserve—not naive character splits. Missing tokenizer metadata triggers an explicit conservative estimate or strict refusal.
- **Robust streaming and backends.** Pipe-friendly completed-paragraph output keeps data on `stdout` and progress on `stderr`, with retries for interrupted responses and protocol truncation. Explicit llama.cpp/vLLM adapters respect their different sampling semantics; preflight also detects and suppresses a specific incorrect EOS token reported by some llama.cpp services.
- **Evidence-aware reuse and measurable performance.** Optional segment caching is bound to verified runtime/prompt identity; complete validated runs can be replayed locally from persisted evidence without contacting the model. SQLite history supports ETA estimation, and a versioned benchmark suite measures quality, exact-token preservation, syntax validity, truncation, and latency.

**Try it:** `hymt translate-doc README.md -l zh` · `hymt exec -- cargo test` · `hymt man git-rebase`

[Markdown ownership code](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/markdown_ownership.rs) · [Backend/EOS regression tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-client/src/backend_preflight_tests.rs) · [Recovery and replay tests](https://github.com/RyderFreeman4Logos/hymt/blob/main/crates/hymt-translate/src/translate/tests/retained_original_tests.rs) · [Benchmark methodology](https://github.com/RyderFreeman4Logos/hymt/blob/main/benchmarks/README.md)

### [llm-guard-proxy](https://github.com/RyderFreeman4Logos/llm-guard-proxy) + [gb10-services](https://github.com/RyderFreeman4Logos/gb10-services)

**Local inference that can be operated and diagnosed.** A Rust OpenAI-compatible gateway adds bounded queues, retries, streaming lifecycle handling, loop detection, and observability. The companion GB10 stack brings together chat, embedding, and reranking services with pinned deployment assets and recovery procedures.

Highlights include persisted recovery records, checks tied to a specific service instance, and a low-memory emergency operation that avoids allocation.

[Proxy architecture](https://github.com/RyderFreeman4Logos/llm-guard-proxy/blob/main/docs/architecture.md) · [Emergency-path test](https://github.com/RyderFreeman4Logos/llm-guard-proxy/blob/main/crates/llm-guard-proxy-host-guardian/tests/no_alloc.rs) · [Deployment and experiments](https://github.com/RyderFreeman4Logos/gb10-services)

### [Verbatim](https://github.com/RyderFreeman4Logos/verbatim)

**Document retrieval with traceable evidence.** A local Rust daemon and CLI for searching documents and code through lexical and vector retrieval, with optional reranking. Results carry stable evidence IDs and source locators; generated answers can be checked against their cited evidence.

The design keeps original passages, derived material, and generated answers distinguishable, with explicit storage and recovery contracts.

[Evidence model](https://github.com/RyderFreeman4Logos/verbatim/blob/main/docs/safety-model.md) · [Durability](https://github.com/RyderFreeman4Logos/verbatim/blob/main/docs/sqlite-durability.md) · [Implementation](https://github.com/RyderFreeman4Logos/verbatim/tree/main/crates)

### [telegram-archive-bot](https://github.com/RyderFreeman4Logos/telegram-archive-bot)

**Durable source archives and audio delivery in Rust.** Preserves canonical source revisions in SQLite and immutable content-addressed files, with recoverable jobs, validated media, backup/restore, and a deterministic Git projection. Integrates translation through `hymt` and audio through `md2audio`.

Currently a **release candidate**, with explicit provider and client acceptance checks.

[Release candidates](https://github.com/RyderFreeman4Logos/telegram-archive-bot/releases) · [Storage and crash tests](https://github.com/RyderFreeman4Logos/telegram-archive-bot/blob/main/src/storage.rs) · [Recovery](https://github.com/RyderFreeman4Logos/telegram-archive-bot/blob/main/docs/recovery.md)

## Focused tools

| Project | What it provides |
| --- | --- |
| [pbi](https://github.com/RyderFreeman4Logos/pbi) | Native Rust source search with bounded BM25 ranking, parser-based extraction, and optional model synthesis with citations checked against source spans. |
| [skillcfg](https://github.com/RyderFreeman4Logos/skillcfg) | Shared TOML preferences across agent skills, symlink-aware discovery, dependency validation, and configuration impact inspection. |
| [mesh-proxy](https://github.com/RyderFreeman4Logos/mesh-proxy) | Iroh/QUIC service networking for TCP and Unix sockets, with invitation-based joining and persisted routes. |
| [md2audio](https://github.com/RyderFreeman4Logos/md2audio) | Markdown-to-audio workflows with subtitles, optional video, and an explicitly selected offline speech backend. |
| [openwebui-skill](https://github.com/RyderFreeman4Logos/openwebui-skill) | A Rust CLI for agent conversations that remain visible in Open WebUI, with local JSONL history. |

These projects share working components: `pbi` uses `adk-workflow-kit`, and the archive bot integrates `hymt` and `md2audio`. See the [project notes](https://github.com/RyderFreeman4Logos/RyderFreeman4Logos/blob/main/PROJECTS.md) for source links, smaller utilities, and earlier security work.

## Maintained forks

- **[ADK-Rust](https://github.com/RyderFreeman4Logos/adk-rust)** — My fork of [zavora-ai/adk-rust](https://github.com/zavora-ai/adk-rust). Work includes [subprocess ownership and cleanup across cancellation](https://github.com/RyderFreeman4Logos/adk-rust/pull/34), [preserving terminal streaming output](https://github.com/RyderFreeman4Logos/adk-rust/pull/29), and [retaining caller-provided JSON Schemas](https://github.com/RyderFreeman4Logos/adk-rust/pull/16).
- **[Hermes Agent](https://github.com/RyderFreeman4Logos/hermes-agent)** — My fork of [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent). A maintained patch stack for long-running agents: cache-aware context handling, bounded compression, session recovery, and process lifecycle fixes. The [integration ledger](https://github.com/RyderFreeman4Logos/hermes-agent/pull/188) connects retained behavior to source patches and validation records.

## Merged upstream contributions

| Upstream | Contributions accepted by its maintainers |
| --- | --- |
| [rspow](https://github.com/zolagonano/rspow) | Configurable proof-of-work difficulty, concurrent solving, EquiX proof bundles, and feature-gated backends: [#1](https://github.com/zolagonano/rspow/pull/1), [#2](https://github.com/zolagonano/rspow/pull/2), [#3](https://github.com/zolagonano/rspow/pull/3), [#4](https://github.com/zolagonano/rspow/pull/4). |
| [nazgul](https://github.com/edwinhere/nazgul) | Conditional Serde serialization support for CLSAG ring signatures: [#3](https://github.com/edwinhere/nazgul/pull/3). |

## In development

**[akasha-otsuge](https://github.com/RyderFreeman4Logos/akasha-otsuge)** explores local, traceable memory across coding agents. The current Rust foundation implements immutable source evidence, atomic versioned storage, durable idempotent writes, and checksum-verified backup/restore. Host integrations and retrieval remain on the MVP roadmap.

## Documentation and collaboration

I also publish [ADK-Rust documentation in Simplified Chinese](https://github.com/RyderFreeman4Logos/adk-rust-zh-cn), with upstream source snapshots and a SHA-256 source manifest for traceability.

For bugs and technical discussion, open an issue in the relevant repository. For direct contact: **[Telegram](https://t.me/Ryder_support_bot)**.
