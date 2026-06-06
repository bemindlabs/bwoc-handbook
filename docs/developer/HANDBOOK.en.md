---
title: "BWOC Developer Handbook"
version: "1.0.0"
audience: "Framework contributors and extension builders (Rust)"
language: en
counterpart: HANDBOOK.th.md
---

> Thai version: [HANDBOOK.th.md](HANDBOOK.th.md) — Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) — v2.24.0 · Rust 1.85+

# BWOC Developer Handbook

This handbook is for people who build on or contribute to the BWOC Framework source: crate authors, CLI contributors, harness developers, and anyone shipping a PR upstream. If you only want to incarnate and run agents, see [[../agents/HANDBOOK.en.md]].

---

## Contents

1. [The Two Scopes](#1-the-two-scopes)
2. [Crate Map](#2-crate-map)
3. [Dependency-Quarantine Principle](#3-dependency-quarantine-principle)
4. [Build, Test, and Install](#4-build-test-and-install)
5. [CI Gates](#5-ci-gates)
6. [The Nine Hard Rules](#6-the-nine-hard-rules)
7. [Auto-Version Hook](#7-auto-version-hook)
8. [Docs and Bilingual Workflow](#8-docs-and-bilingual-workflow)
9. [Skill Development](#9-skill-development)
10. [Plugin Development](#10-plugin-development)
11. [Contribution Flow](#11-contribution-flow)
12. [Releasing](#12-releasing)
13. [Where to Look Next](#13-where-to-look-next)

---

## 1. The Two Scopes

When you open the framework repo you are looking at two distinct surfaces. Confusing them causes mistakes.

### 1.1 Framework Root

Everything at the top level of [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework):

- `crates/` — the Rust workspace (nine crates, described below).
- `docs/en/` and `docs/th/` — bilingual specification documents (`ARCHITECTURE`, `DESIGN`, `HARNESS`, `SKILLS`, `PLUGINS`, `RELEASING`, `SIGNING`, `WORKSPACE`, `NAMING`, `FLEET-GOVERNANCE`, `GLOSSARY`, `FAQ`, `INCARNATION`, each with `.en.md` and `.th.md`).
- Root-level OSS docs: `README.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `SECURITY.md`, `LICENSE`, `VERSION.md` — English-only by design (see [Rule 3](#rule-3-bilingual-parity)).
- `scripts/install.sh`, `scripts/bump-version.sh` — helper scripts for the release pipeline.
- `.claude/hooks/auto-version.sh` — the patch-bump hook (see [Section 7](#7-auto-version-hook)).
- `modules/` — agent-template, plugin-template, skill-template, plus pre-built `plugins/` and `skills/`.

Edits here go through the full PR and CI gate described in [Section 11](#11-contribution-flow).

### 1.2 Agent Template (`modules/agent-template/`)

The cloneable artifact that `bwoc new` copies when incarnating an agent. It has its own:

- `AGENTS.md` — the single source of truth for all backends. Backend symlinks (`CLAUDE.md`, `AGY.md`, `CODEX.md`, `KIMI.md`, `OLLAMA.md`, `OPENAI.md`) all point here.
- Slot directories: `persona/`, `mindsets/`, `skills/`, `interconnect/`, `memories/`.
- `docs/` (with its own EN/TH pairs: `OVERVIEW`, `PHILOSOPHY`, `PRD`, `SELF-IMPROVEMENT`, `SRS`, `THREAT-MODEL`).
- `scripts/check-agent-neutrality.sh` and `scripts/incarnate.sh` — run inside the template directory, not from the framework root.

The template's `AGENTS.md` is the specification that incarnated agents read at runtime. The framework root's `CLAUDE.md` is for the AI editor working on the framework itself. Do not conflate them.

> **Repo-as-workspace quirk.** The framework repo is itself a BWOC workspace (`.bwoc/workspace.toml` at root) used for end-to-end CLI testing. Its `.gitignore` therefore excludes `.bwoc/`, `agents/`, and `projects/` entirely — the **opposite** of what `bwoc init` writes for user workspaces. The contrast lives in `crates/bwoc-cli/src/init.rs::GITIGNORE_TEMPLATE`.

---

## 2. Crate Map

The workspace has nine crates. Their dependency relationships enforce the quarantine principle described in [Section 3](#3-dependency-quarantine-principle). Current `[workspace.package].version` is `2.24.0`.

### 2.1 Quick Reference Table

| Crate | Binary | Purpose | Permitted deps (non-workspace) | Must NOT contain |
|---|---|---|---|---|
| `bwoc-core` | — | Shared types: manifest, identity, lifecycle phases | `serde`, `serde_json`, `toml`, `thiserror` | async, HTTP, crypto, SQLite, network of any kind |
| `bwoc-signing` | — | ed25519 signing primitives for agent identity proof | `ed25519-dalek`, `rand_core`, `hex`, `serde*`, `thiserror` | async, HTTP, heavy runtime |
| `bwoc-cli` | `bwoc` | The user-facing CLI; localized TH/EN output | `bwoc-core`, `bwoc-signing`, `bwoc-tui`, `clap`, `ratatui`, `crossterm`, `fluent-bundle`, `unic-langid`, `sha2`, `include_dir`, `libc`; `interprocess` (Windows only) | Direct `tokio`/`axum`/`reqwest`; exec heavy binaries as subprocesses instead |
| `bwoc-agent` | `bwoc-agent` | Minimal runtime shipped with each agent; `--serve` daemon over Unix socket + Windows named pipe; `ping`/`status`/`stop` clients | `bwoc-core`, `bwoc-signing`, `fluent-bundle`, `unic-langid`, `ctrlc`, `serde_json`; `interprocess` (Windows only) | Heavy async runtime at the library level |
| `bwoc-harness` | `bwoc-harness` | Self-hosted agentic runtime: OpenAI-compatible provider, agentic loop, core tools, P3 task queue, telemetry, tool-auth, `--chat` event stream | `tokio` (rt-multi-thread), `reqwest`, `keyring`, `landlock` (Linux), optional `opentelemetry` (`otel` feature) | Must stay out of `bwoc-core`; CLI execs it as a subprocess |
| `bwoc-deep-memory` | `bwoc-deep-memory` | Tier-2 memory over local SQLite; semantic recall via embeddings; speaks `wake-up`/`search`/`mine` contract | `rusqlite` (bundled), `reqwest` (blocking), `clap`, `serde*` | Must stay out of `bwoc-core` |
| `bwoc-a2a` | `bwoc-a2a` | Agent2Agent protocol interop pinned to A2A spec v1.0.0; Agent Card, JSON-RPC handlers; axum HTTP listener; outbound client | `axum`, `tokio`, `reqwest`, `async-stream`, `futures-core`, `clap` | Must stay out of `bwoc-core`; CLI execs it as a subprocess |
| `bwoc-mqtt` | `bwoc-mqtt` | MQTT transport for inter-workspace routing: publish envelope, or subscribe and deliver into `inbox.jsonl` | `rumqttc`, `serde_json`, `clap`, `thiserror` | Must stay out of `bwoc-core` |
| `bwoc-tui` | — | Interactive ratatui dashboard and `--chat` client; shared rendering primitives used by `bwoc-cli` and `bwoc-harness` | `ratatui`, `crossterm`, `serde_json` | Heavy async runtime |

### 2.2 Per-Crate Detail

#### `bwoc-core`

**Purpose.** The single source of shared types for the entire framework: `config.manifest.json` schema, agent identity structs, the three lifecycle phases (*uppāda · ṭhiti · vaya*), and error enumerations. Every other crate that needs to understand "what an agent is" imports `bwoc-core`.

**What lives here.** Pure data types, serialization/deserialization (via `serde`), validation logic, and `thiserror`-based error types. Nothing else.

**What must NOT leak in.** Any async executor, HTTP client, cryptographic operation, database driver, network socket, or operating-system-specific capability. The rule is absolute: `bwoc-core` must remain importable by a `no_std` environment in the future without modification, and must be auditable in minutes. Adding `tokio` or `reqwest` here — even as an optional feature — violates the quarantine and will be rejected in review.

**Relationship to others.** `bwoc-cli`, `bwoc-agent`, `bwoc-harness`, `bwoc-a2a`, `bwoc-mqtt`, `bwoc-tui`, and `bwoc-deep-memory` all depend on `bwoc-core`. It is the only crate that forms a true foundation; all others are optional at deployment.

**How to verify.** `cargo tree -p bwoc-core` must show only `serde`, `serde_json`, `toml`, and `thiserror` (plus their transitive pure-Rust deps). Any async or network dep appearing here is a quarantine violation — open a blocking PR review comment.

---

#### `bwoc-signing`

**Purpose.** Ed25519 keypair generation, message signing, and signature verification for agent identity proofs (feature set HV2-4). Used by `bwoc-cli` to sign outbound messages and by `bwoc-agent` to verify inbound ones.

**What lives here.** `ed25519-dalek` keypair and signing, `rand_core` for entropy, `hex` for key material encoding, canonical JSON serialization (sorted keys via `BTreeMap`) of the fields being signed.

**What must NOT leak in.** Any async runtime, HTTP client, or tokio dep. The crate is intentionally synchronous so it can be linked from both the CLI (synchronous) and the agent daemon without dragging in a second async executor.

**Relationship to others.** `bwoc-cli` calls `bwoc trust --keygen` to produce a keypair via this crate. `bwoc-agent` calls the verify path on every inbound envelope. `bwoc-core` does NOT depend on `bwoc-signing` — the trust layer is above the type layer.

**How to verify.** `cargo tree -p bwoc-signing` must contain no tokio, axum, or reqwest nodes.

---

#### `bwoc-cli`

**Purpose.** The `bwoc` binary that users and operators interact with daily: `bwoc new`, `bwoc check`, `bwoc spawn`, `bwoc chat`, `bwoc status`, `bwoc trust`, `bwoc skill`, `bwoc plugin`, and approximately thirty more subcommands. Localized output via `fluent-bundle` (TH and EN bundles embedded at compile time with `include_dir`). Ships pre-built for macOS (aarch64 + x86_64), Linux (x86_64), and Windows (x86_64) in every release.

**What lives here.** CLI argument parsing (`clap`), command dispatch, workspace resolution logic, agent registry read/write (`agents.toml`), `bwoc check` neutrality audit, `bwoc init` workspace scaffold (including the `GITIGNORE_TEMPLATE` constant), `bwoc trust --keygen` keypair generation, and the TUI dashboard (via `bwoc-tui`). On Windows, `interprocess` for named-pipe IPC with `bwoc-agent --serve`.

**What must NOT leak in.** Direct `tokio` or `axum` imports. When a subcommand needs async capability (A2A, harness, MQTT), the CLI execs the corresponding sibling binary as a subprocess (`bwoc-a2a`, `bwoc-harness`, `bwoc-mqtt`). This keeps the CLI's dependency tree synchronous, startup fast, and the binary small. The subprocess pattern is documented in `crates/bwoc-cli/src/spawn.rs`.

**Relationship to others.** `bwoc-cli` depends on `bwoc-core`, `bwoc-signing`, and `bwoc-tui`. It execs `bwoc-agent`, `bwoc-harness`, `bwoc-a2a`, and `bwoc-mqtt` as subprocesses. It never links them directly.

**How to verify.** `cargo tree -p bwoc-cli | grep tokio` must return empty (except the Windows `interprocess` transitive, which is acceptable because it targets a separate cfg gate). Run `cargo build -p bwoc-cli` on all three platforms in CI.

---

#### `bwoc-agent`

**Purpose.** The minimal runtime that ships alongside each incarnated agent. Runs as `bwoc-agent --serve` to start a daemon that listens on a Unix socket (macOS/Linux) or Windows named pipe, then responds to `PING`, `STATUS`, and `STOP` messages from the CLI. Also prints the liveness banner from `config.manifest.json` on startup.

**What lives here.** Daemon entry point, IPC message dispatch, liveness banner rendering from the manifest, signal handling (`ctrlc`), signature verification of inbound envelopes (via `bwoc-signing`), and Fluent localization strings. On Windows: `interprocess` named-pipe server and client.

**What must NOT leak in.** Heavy async frameworks, HTTP stacks, or database drivers. The daemon is intentionally lightweight so it can run alongside any number of agent processes without resource contention.

**Relationship to others.** Depends on `bwoc-core` and `bwoc-signing`. `bwoc-cli` controls it via IPC (ping, status, stop). `bwoc-harness` is the peer that handles actual LLM interaction; `bwoc-agent` handles the control plane.

**How to verify.** `cargo tree -p bwoc-agent | grep -E 'tokio|axum|reqwest'` must be empty.

---

#### `bwoc-harness`

**Purpose.** The self-hosted agentic runtime for `ollama` and `openai-compatible` backends. Provides the full agentic loop: connect to an OpenAI-compatible endpoint, manage a conversation context, execute tools, process the P3 task queue, emit telemetry, enforce tool-auth via the OS keyring, and stream events on `--chat`. The `--chat` event stream is consumed by `bwoc-tui` for the full-screen chat client.

**What lives here.** The agentic loop (`src/loop/`), tool executor (`src/tools/`), provider client (`src/provider/`), task queue (`src/queue/`), tool-auth credential manager (`src/auth/`), telemetry (`src/telemetry/` — optional behind the `otel` feature flag), and the `--chat` SSE event-stream server. On Linux: `landlock` LSM bindings for kernel-enforced filesystem access control.

**What must NOT leak in.** Nothing from `bwoc-harness` should flow back into `bwoc-core`, `bwoc-signing`, or `bwoc-cli`. The harness is a leaf crate in the dependency DAG. The `keyring` dep (platform credential store integration) and `landlock` (Linux security module) live here precisely because this is where the quarantine permits heavy deps.

**Feature flags.** Default build has zero OpenTelemetry dependencies. Enable with `cargo build --features otel` to pull the OTEL crate graph. Never make `otel` a default feature — it adds network-egress capability that users must opt into explicitly.

**Relationship to others.** `bwoc-cli` execs `bwoc-harness` as a subprocess for `bwoc spawn` (ollama/openai-compatible backends) and `bwoc chat --tui`. The harness reads the agent's `config.manifest.json` (parsed via `bwoc-core` types) to resolve the endpoint and model.

**How to verify.** `cargo test -p bwoc-harness` uses `wiremock` for offline provider tests — no real network required in CI.

---

#### `bwoc-deep-memory`

**Purpose.** Tier-2 (semantic, persistent) memory for agents. Speaks the three-verb contract that the harness and CLI invoke: `wake-up` (emit prior context at session start), `search` (retrieve relevant past decisions given a query), and `mine` (persist session learnings at session end). Storage is a local SQLite database; recall is via embeddings fetched from an OpenAI-compatible `/v1/embeddings` endpoint.

**What lives here.** The three command handlers, SQLite schema management (via `rusqlite` with the `bundled` feature — SQLite compiled from source, no system dependency needed), blocking HTTP calls to the embeddings endpoint (`reqwest` blocking client), and the `clap`-based binary entry point.

**What must NOT leak in.** This crate must never be added as a dependency of `bwoc-core`. The `rusqlite bundled` feature compiles a C library; it is explicitly permitted here and prohibited everywhere else.

**Relationship to others.** `bwoc-harness` calls `bwoc-deep-memory` as a subprocess (via `deepMemoryCmd` from the manifest) during the agentic loop. `bwoc memory wake-up`, `bwoc memory t2-search`, and `bwoc memory mine` in the CLI also delegate to it. It is never linked directly by `bwoc-core` or `bwoc-cli`.

**How to verify.** `cargo build -p bwoc-deep-memory` on all CI platforms. The `bundled` SQLite feature means no `libsqlite3-dev` install step is needed in CI.

---

#### `bwoc-a2a`

**Purpose.** Implements the Agent2Agent (A2A) protocol v1.0.0 for BWOC agents. Provides the Agent Card (the public capability advertisement), JSON-RPC message and task handlers, an `axum`-based HTTP listener, and (in phase 4) an outbound client using `reqwest`. Agents expose a `/a2a` HTTP endpoint via this crate; other agents and external systems discover and call them through it.

**What lives here.** A2A type definitions, the Agent Card builder (bridging `bwoc-core` manifest fields to the A2A schema), JSON-RPC 2.0 request/response handling, SSE streaming for task progress, the `axum` router, and the `clap`-based binary entry point. The tokio runtime instantiated here is a `current_thread` runtime to keep the CLI's process synchronous — `bwoc-cli` execs `bwoc-a2a` as a subprocess; tokio/hyper never enter the CLI's dependency tree.

**What must NOT leak in.** HTTP and network deps must stay in `bwoc-a2a`. `bwoc-core` must not gain any awareness of A2A specifics. The protocol core (types, card, RPC parsing) is transport-agnostic and unit-testable without binding a real socket.

**Relationship to others.** `bwoc-a2a` is examinable from the CLI via `bwoc a2a` subcommand (which execs the `bwoc-a2a` binary). It reads `bwoc-core` manifest types to build the Agent Card. It is independent of `bwoc-harness` — the harness handles LLM interaction; A2A handles protocol interop.

**How to verify.** Tests use `tower` to drive the `axum` router without binding a network socket, and `wiremock` for outbound client tests. `cargo test -p bwoc-a2a` is fully offline.

---

#### `bwoc-mqtt`

**Purpose.** MQTT transport for inter-workspace message routing. Two modes: `publish` (send an envelope to a broker topic) and `serve` (subscribe to a topic and deliver arriving envelopes into an agent's `inbox.jsonl`). Enables cross-workspace agent communication without requiring A2A HTTP infrastructure.

**What lives here.** MQTT client (`rumqttc`), envelope serialization, `inbox.jsonl` append logic, and the `clap`-based binary entry point.

**What must NOT leak in.** This crate must not be linked directly by `bwoc-core` or `bwoc-cli`. The CLI execs it as a subprocess.

**Relationship to others.** Conceptually parallel to `bwoc-a2a` — both are transport crates. MQTT suits low-latency broadcast across many agents; A2A suits point-to-point request/response with full protocol semantics. They can coexist in the same workspace.

---

#### `bwoc-tui`

**Purpose.** Shared ratatui rendering primitives for the interactive dashboard and `--chat` full-screen client. Used by both `bwoc-cli` (the `bwoc dashboard` command) and `bwoc-harness` (rendering the `--chat` event stream).

**What lives here.** Widget definitions, the agent-list dashboard, chat message rendering, and `crossterm`-based terminal management. Parses the `chat_proto` event stream emitted by `bwoc-harness --chat`.

**What must NOT leak in.** No async runtime, no HTTP client, no business logic. This is a pure rendering crate.

**Relationship to others.** `bwoc-tui` is the only crate (apart from `bwoc-core`) that is linked directly by `bwoc-cli` in addition to being used by `bwoc-harness`. All other crates with heavy deps are exec'd as subprocesses.

---

## 3. Dependency-Quarantine Principle

The framework enforces a strict quarantine on where heavy dependencies are permitted. This is not a style preference — it is a security and maintainability invariant enforced in PR review.

**The rule in one sentence:** `bwoc-core` accepts only pure-Rust serialization deps. Async, HTTP, crypto, SQLite, OS keyring, and platform-security bindings are allowed exclusively in the crates purpose-built to carry them.

**Why it matters.**

1. **Audit surface.** A framework for running AI agents that executes arbitrary code must have a core that can be read and audited quickly. A lean `bwoc-core` with four deps is auditable; one with a transitive tokio tree is not.
2. **Attack surface.** Each dep is a supply-chain risk. Confining network deps to `bwoc-a2a` and `bwoc-harness` means a compromised HTTP library cannot affect code paths that handle only manifest parsing.
3. **Startup time.** `bwoc-cli` stays synchronous because it never links an async runtime directly. Subcommands that need async exec their binary counterpart. This keeps `bwoc list` and `bwoc check` instant even on slow machines.
4. **Testability.** `bwoc-core` and `bwoc-signing` tests run entirely without network mocks. `bwoc-harness` and `bwoc-a2a` tests use `wiremock` for offline simulation. The boundary is clear.

**The allowed-dep map** (summary):

| What | Allowed in |
|---|---|
| `serde`, `toml`, `thiserror` | All crates |
| `ed25519-dalek`, `rand_core`, `hex` | `bwoc-signing` only |
| `tokio`, `axum`, `reqwest`, `async-stream`, `futures-*` | `bwoc-harness`, `bwoc-a2a`, `bwoc-deep-memory` (blocking reqwest only) |
| `rusqlite` (bundled C) | `bwoc-deep-memory` only |
| `rumqttc` | `bwoc-mqtt` only |
| `keyring` | `bwoc-harness` only |
| `landlock` | `bwoc-harness` (Linux target) only |
| `ratatui`, `crossterm` | `bwoc-tui`, `bwoc-cli` only |
| `fluent-bundle`, `unic-langid`, `include_dir` | `bwoc-cli`, `bwoc-agent` only |
| `interprocess` (named pipe) | `bwoc-cli`, `bwoc-agent` (Windows cfg gate only) |

**How to comply when adding a new dep.** Before adding any dependency:
1. Check which crate you are editing.
2. Consult the allowed-dep map above.
3. If the dep crosses a quarantine boundary, either (a) place it in the appropriate crate and exec it as a subprocess, or (b) raise a design question in the PR so the team can decide the right boundary.
4. Never add a dep to `bwoc-core` without an explicit architectural decision and a note in `notes/`.

**How to verify.** `cargo tree -p bwoc-core` and `cargo tree -p bwoc-signing` must not show tokio, axum, reqwest, rusqlite, rumqttc, keyring, or landlock. The CI matrix runs on all three platforms; a quarantine violation that slips past a reviewer will fail the build on any platform where a dep is unavailable.

---

## 4. Build, Test, and Install

**Prerequisites.** Rust 1.85 or newer (check with `rustup show`). No system SQLite, OpenSSL, or libsodium needed — all C deps use the `bundled` feature or pure-Rust equivalents.

```bash
# Clone
git clone https://github.com/bemindlabs/BWOC-Framework
cd BWOC-Framework

# Build all crates
cargo build

# Run all tests (offline — no network required)
cargo test

# Build in release mode
cargo build --release

# Install the bwoc CLI locally
cargo install --path crates/bwoc-cli

# Build the harness with optional OpenTelemetry export
cargo build -p bwoc-harness --features otel

# Build a single crate
cargo build -p bwoc-core

# Run tests for a single crate
cargo test -p bwoc-signing

# Check formatting (must pass before commit)
cargo fmt --check

# Run clippy (must pass before commit)
cargo clippy -- -D warnings

# Shell completion (optional, install to your shell)
bwoc completion zsh > ~/.zfunc/_bwoc
bwoc completion bash > /etc/bash_completion.d/bwoc
```

**Release profile.** The workspace `Cargo.toml` sets `lto = "thin"`, `codegen-units = 1`, `strip = true` for release builds. This produces small, fast binaries appropriate for distribution.

**Edition 2024.** All crates use `edition = "2024"`. If you are adding a crate, set `edition.workspace = true` — do not hard-code an older edition.

---

## 5. CI Gates

Every PR must pass all four required CI checks before it can merge. These run on the matrix: macOS, Linux, and Windows.

| Check | What it runs | Why it is required |
|---|---|---|
| **fmt** | `cargo fmt --check` | Uniform style; no whitespace debates in review |
| **clippy** | `cargo clippy -- -D warnings` | Catches common correctness issues at zero runtime cost |
| **build** | `cargo build` | Confirms the workspace compiles on all three platforms |
| **test** | `cargo test` | Regression guard; all tests are offline |

**Stale-branch protection.** GitHub's strict branch-protection requires the PR branch to contain the latest `main` commit before merging. A conflict-free but stale branch still fails this gate. Fix it with:

```bash
gh pr update-branch <PR_NUMBER>
```

This rebases or merges `main` into your branch and re-triggers CI. Do not force-push to `main` to work around it.

**Conversation resolution gate.** Every review comment thread must be marked "Resolved" before the PR can merge. If a reviewer opens a thread for a security concern, it blocks the merge until explicitly resolved. This is intentional — see [Rule 8](#rule-8-protected-main-and-pr-discipline) for the security review checklist.

---

## 6. The Nine Hard Rules

These are non-negotiable. Each one has a reason. Understanding the reason makes compliance natural.

---

### Rule 1: Two-Tier Document Format

**What.** Every Markdown file in the repository falls into exactly one of two tiers:

| Tier | Files | Format |
|---|---|---|
| Instructions (Tier 1) | `modules/agent-template/AGENTS.md` and all its backend symlinks (`CLAUDE.md`, `AGY.md`, `CODEX.md`, `KIMI.md`, `OLLAMA.md`, `OPENAI.md`) | Plain Markdown only. No YAML frontmatter, no `[[wikilinks]]`, no `> [!callout]` blocks, no vendor-specific rendering. |
| Documentation (Tier 2) | All other `.md` files | Obsidian Markdown: YAML frontmatter allowed, `[[wikilinks]]` allowed, callouts allowed. Approved callout types only: `abstract`, `tip`, `warning`, `example`, `note`, `danger`. No other types. |

**Why.** `AGENTS.md` is read by every LLM backend — Claude, Kimi, Codex, local Ollama models, and any future backend. Obsidian syntax (`> [!callout]`, `[[link]]`, YAML frontmatter) is an Obsidian-specific rendering convention that many models parse incorrectly or render as literal text, producing confused agent behavior. Plain Markdown is the lowest common denominator every backend handles correctly.

Documentation files (specs, handbooks, philosophy docs) are intended for human readers in Obsidian or GitHub's renderer, where rich syntax improves comprehension.

**How to comply.** Before editing `AGENTS.md` or any of its symlinks, confirm you are in Tier 1 and strip any rich syntax. Before editing any spec or handbook page, confirm you are in Tier 2 and use YAML frontmatter and wikilinks freely. When adding a new callout type, check the approved list first — do not introduce `> [!caution]`, `> [!info]`, or other unsanctioned types.

**How to verify.** `bwoc check <agent-path>` audits `AGENTS.md` for YAML frontmatter, wikilinks, and unsupported callout syntax. For documentation files, open them in Obsidian or inspect the rendered Markdown on GitHub — broken frontmatter or unresolved wikilinks indicate a problem.

---

### Rule 2: Backend Neutrality

**What.** `AGENTS.md` must not contain hardcoded model IDs, vendor names, or tool names. All configurable values use `{{camelCase}}` placeholder syntax (e.g., `{{agentId}}`, `{{primaryModel}}`, `{{fallbackModel}}`). Backend-specific phrasing belongs only in Section 0 "Backend Registration" of `AGENTS.md` or in vendor entry files that symlink to it. Adding a new backend is exactly one command:

```bash
ln -s AGENTS.md <BACKEND>.md
```

No separate per-backend content is created.

**Why.** BWOC's foundational commitment is Samānattatā — equal treatment of all backends. Hardcoding `gpt-4o` or `claude-opus-4` anywhere in `AGENTS.md` makes that particular model a privileged dependency. When the model changes or the vendor raises prices, every agent instance that references it needs a manual update. Placeholders decouple the agent spec from vendor decisions.

**How to comply.** Grep `AGENTS.md` for known model IDs and vendor names before committing. Replace any found with the appropriate `{{placeholder}}`. When your `config.manifest.json` resolves the placeholder to a real value at runtime, that is the correct layer for the hardcoded string.

**How to verify.** `bwoc check <agent-path>` scans for hardcoded model IDs and vendor names and reports violations. Run `bwoc check --all` for a fleet-wide audit. The CI job for agent PRs should include `bwoc check` as a step.

---

### Rule 3: Bilingual Parity

**What.** Every `docs/en/*.en.md` file has a counterpart `docs/th/*.th.md`. When you edit one, you edit the other in the same PR. This applies to both the framework root `docs/` and `modules/agent-template/docs/`.

The following files are **exempt** from TH parity by design:

| File | Reason |
|---|---|
| `README.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, `LICENSE`, `VERSION.md` | Mechanical OSS docs; translation cost outweighs value for short-lived process docs that change with every PR. |
| `crates/*/README.md` | Code-side convention; adopt TH pair only when the framework explicitly targets a Thai developer audience. |
| `notes/YYYY-MM-DD_*.md` | Per-session implementation logs; ephemeral. |
| `.claude/` content | Operator-internal; not a public-contributor surface. |

Doctrine docs that state project direction or principles (e.g., `VISION.md`, `PHILOSOPHY.en.md`) always require a TH pair.

**Why.** BWOC's primary author and a significant portion of its user base are Thai speakers. Keeping doctrine docs bilingual ensures that the Thai-speaking audience can evaluate whether BWOC is for them without relying on machine translation of the canonical documents. Exempting mechanical process docs keeps the maintenance burden honest — these files update on every PR and a forced TH update would create review noise without adding value.

**How to comply.** When opening a PR that touches any `docs/en/*.en.md`, also update `docs/th/*.th.md` in the same commit. The PR checklist in [CONTRIBUTING.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/CONTRIBUTING.md) includes a bilingual parity checkbox. If you are not fluent in Thai, note in the PR that a TH update is needed and request a Thai-speaking reviewer.

**How to verify.** Check that `docs/en/` and `docs/th/` have the same set of base filenames (substituting `.en.md` for `.th.md`). A simple `diff <(ls docs/en/ | sed 's/\.en\.md//')  <(ls docs/th/ | sed 's/\.th\.md//')` should produce no output.

---

### Rule 4: Repo-as-Workspace Inversion

**What.** The framework repo has `.bwoc/`, `agents/`, and `projects/` in its `.gitignore`. User workspaces created by `bwoc init` have the opposite: they track `agents/` and `projects/` and ignore only ephemeral runtime files.

**Why.** The framework repo uses its own `.bwoc/workspace.toml` for end-to-end CLI testing — you can run `bwoc list` from the framework root and see test agents. But those test agents and the `.bwoc/` runtime state must not be committed to the framework repo; they are test fixtures, not source artifacts.

**How to comply.** Never commit `.bwoc/`, `agents/`, or `projects/` directories to the framework repo. If you notice them in a `git status`, verify `.gitignore` is correct and that you have not accidentally checked out a user workspace `.gitignore` template.

**How to verify.** `git check-ignore -v .bwoc agents projects` should show a rule from `.gitignore` for each. The contrast with user workspaces is in `crates/bwoc-cli/src/init.rs::GITIGNORE_TEMPLATE`.

---

### Rule 5: Auto-Version Hook (Expected Working-Tree Change)

**What.** The hook at `.claude/hooks/auto-version.sh` automatically bumps the patch component of `[workspace.package].version` in `Cargo.toml` on every `.rs` or `.toml` write, and bumps `Document-Version` in `VERSION.md` on every `.md` write. After any edit session, these files will appear modified in your working tree.

**Why.** This is not drift — it is intentional. Every edit becomes a uniquely versioned checkpoint. `bwoc --version` during development always reports the exact commit that produced the binary, enabling precise reproduction of agent behavior.

**How to comply.** Do not revert auto-version bumps. Do not fight the hook by manually setting the version to the pre-edit value. If you need a MINOR or MAJOR bump (new capability or breaking change), edit `Cargo.toml` directly for the higher-order component; the hook only manages PATCH. For documentation MINOR/MAJOR, edit the `Document-Version` line in `VERSION.md` directly.

**How to verify.** After any edit, `git diff Cargo.toml` should show a patch increment. `git diff VERSION.md` should show a `Document-Version` increment (for `.md` edits) and a refreshed `Last-Updated` timestamp.

---

### Rule 6: Implementation Logs

**What.** Every significant change gets one note per session in `notes/YYYY-MM-DD_<title>.md`. The note is development-oriented (what changed, why, decisions made, alternatives considered, bugs found) — distinct from `CHANGELOG.md`, which is release-oriented.

A change is "significant" if any of these are true:
- New documentation file added or removed (not a typo fix)
- New code module, crate, hook, skill, workflow, or script added
- A decision affecting future contributors (architecture, naming, versioning, policy)
- A bug fix that changed observable behavior
- Anything that warranted a `CHANGELOG.md` entry beyond a single line

**Note skeleton** (fill only sections that apply):

```markdown
# YYYY-MM-DD — <Title>

<one-paragraph summary>

## What changed
## Decisions
## Alternatives considered
## Bugs surfaced and fixed
## Status / deferred
## Related (links)
```

**Why.** `CHANGELOG.md` captures what shipped. Notes capture why decisions were made — information that is irreplaceable months later when a contributor asks "why does bwoc-core not have HTTP?" or "why is the gitignore inverted?" Notes are searchable context that prevents repeated design discussions.

**How to comply.** Write one note per session, not per file. A session that produces ten files gets one note covering all of them. Daily routine maintenance (formatting fixes, minor doc tweaks) does not need a note. Use `bwoc notes new "<title>"` to scaffold the file with the correct date prefix.

**How to verify.** After a significant PR, confirm a note file exists in `notes/` with today's date. Check the PR diff — if ten new files appear and no note exists, ask the author to add one before merging.

---

### Rule 7: Conventional Commits

**What.** Commit messages follow lightweight [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <short summary>

[optional body — explain *why*, not *what*]
[optional footer — Refs/Closes #issue]
```

Valid types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `style`, `ci`.

**Examples:**
```
feat(bwoc-harness): add P3 task queue with cancellation token
fix(bwoc-signing): correct canonical JSON key ordering for nested objects
docs(harness): add tool-auth section to HARNESS.en.md and HARNESS.th.md
chore(deps): bump reqwest to 0.12.5
```

**Why.** Conventional Commits make `CHANGELOG.md` generation automated, make PR titles parseable by tooling, and communicate intent at a glance. The lightweight form (no strict body required) keeps the barrier low while preserving machine-readability.

**How to comply.** Squash commits before merge (the auto-merge command uses `--squash`). The squash commit message becomes the PR title — make the PR title a valid conventional commit message.

**How to verify.** Read the commit log after merge: `git log --oneline`. Every entry should follow the pattern. PRs with titles like "fix stuff" or "wip" should be renamed before merging.

---

### Rule 8: Protected Main and PR Discipline

**What.** `main` is a protected branch in every BWOC repo, including forks. Changes land only through a PR. The PR is mergeable only when **all** of the following hold simultaneously:

1. All four CI checks pass (fmt, clippy, build, test) on macOS, Linux, and Windows.
2. The branch is up to date with `main` (use `gh pr update-branch <PR>` if stale).
3. Every review conversation is resolved (no open threads).

The only merge command permitted is:

```bash
gh pr merge <PR_NUMBER> --squash --auto
```

This schedules GitHub to merge the moment the gates clear. Do not use `--squash` or `--merge` without `--auto` (that bypasses the CI wait). Do not push directly to `main` under any circumstances, including automation.

**One concern per PR.** A sprint's worth of unrelated changes must not ride in a single mega-PR. Split per feature. Reviewers cannot meaningfully review a 2,000-line diff that touches three unrelated subsystems.

**Cross-repo (fork → upstream) full security review.** PRs arriving from forks carry external code into a security framework. Before merging, reviewers must check:

- **Injection / path-traversal / RCE.** Any code that executes plugins, skills, or scripts, or parses untrusted input, is read for injection vectors.
- **Dep-quarantine integrity.** Confirm `bwoc-core` gained no new deps. Confirm no async/HTTP dep appeared in `bwoc-signing`.
- **Backend neutrality.** Confirm `AGENTS.md` uses no hardcoded model IDs.
- **EN/TH bilingual parity.** Confirm any touched `docs/en/*.en.md` has a matching `docs/th/*.th.md` update.

Security or correctness blockers: request changes (the conversation-resolution gate keeps the PR blocked) rather than merging to fix later.

**Why.** `main` is the release branch. A direct push to `main` that breaks a platform skips CI and ships broken binaries. The PR gate is the only reliable contract between "code compiles on the author's machine" and "code compiles on all three platforms for all users."

**How to verify.** `gh pr status` shows gate states. `gh pr checks <PR>` shows the per-check breakdown. A stale branch shows "Branch is out of date" — fix with `gh pr update-branch`.

---

### Rule 9: Over-Engineering Protection

**What.** Default to NOT adding. Do not proactively add files, hooks, skills, workflows, or specs unless explicitly requested in the current session. Do not chain ahead — finish what is asked, surface what is next, wait for direction. Prefer trimming over expanding when both are reasonable.

The framework's own `VISION.md` names this principle explicitly: "the smaller specification beats the more complete one."

Signals that you may be over-engineering:
- File count grows by more than one per turn without an explicit ask.
- A new doc is created that duplicates content already in another doc.
- A hook or workflow is added that has no current consumer.
- A skill or plugin is scaffolded "just in case."

**Why.** Mattaññutā (right amount) is not a slogan — it is an engineering discipline. Every file added to the repository is a file that must be maintained, kept in bilingual parity, and understood by future contributors. The cost of unnecessary files compounds. A lean codebase is easier to audit, easier to onboard into, and less likely to contain stale contradictory information.

**How to comply.** When tempted to add something "while you're in there," write a note about the idea instead (a `notes/` entry or a PR comment) and let the team decide whether it earns a place in a future session.

**How to verify.** Before opening a PR, count the files added versus the files the request explicitly asked for. If the ratio is more than 2:1, reconsider.

---

## 7. Auto-Version Hook

The hook lives at `.claude/hooks/auto-version.sh` in the framework repo. It is triggered by Claude Code after any file write.

**Behavior:**

- On any `.rs` or `.toml` write: reads `[workspace.package].version` from `Cargo.toml`, increments the PATCH component, writes it back to `Cargo.toml`, and mirrors the new value into the `Software-Version` line of `VERSION.md`.
- On any `.md` write: reads `Document-Version` from `VERSION.md`, increments the PATCH component, writes it back, and refreshes the `Last-Updated` timestamp to the current UTC ISO 8601 time.

**Version namespaces:**

| Namespace | Scheme | Canonical location | Role |
|---|---|---|---|
| Cargo SemVer | `MAJOR.MINOR.PATCH` | `Cargo.toml [workspace.package].version` | Development checkpoint; auto-bumped on every edit |
| Release CalVer | `vYYYY.M.D-<patch>` | Git tag, GitHub Release name | Public release identity; cut manually by maintainers |

These two namespaces are intentionally independent. `bwoc --version` during development reports Cargo SemVer. Release notes and download links use CalVer tags.

**Manual bump rules:**

| Bump type | Action |
|---|---|
| PATCH (auto) | Hook handles it; do nothing |
| MINOR (new backward-compatible capability) | Edit `[workspace.package].version` directly in `Cargo.toml` |
| MAJOR (breaking change to spec, manifest schema, or CLI surface) | Edit `[workspace.package].version` directly in `Cargo.toml`; update `CHANGELOG.md` |
| Document MINOR/MAJOR | Edit `Document-Version` in `VERSION.md` directly |

The hook does not touch MAJOR or MINOR — only PATCH. A `1.0.0` → `1.0.1` bump is automatic; a `1.0.1` → `1.1.0` bump requires a deliberate manual edit.

---

## 8. Docs and Bilingual Workflow

Framework documentation lives under `docs/en/` and `docs/th/`. The canonical list of spec documents (each with `.en.md` and `.th.md` counterparts):

| Document | Purpose |
|---|---|
| [ARCHITECTURE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/ARCHITECTURE.en.md) | System architecture, crate relationships, data flow |
| [DESIGN.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/DESIGN.en.md) | Design rationale and tradeoffs |
| [HARNESS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/HARNESS.en.md) | `bwoc-harness` agentic loop, tools, task queue, tool-auth |
| [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) | Skill system: format, maturity levels, gates |
| [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) | Plugin system: format, workspace registration |
| [RELEASING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/RELEASING.en.md) | Release process: CalVer tags, CI pipeline, Homebrew formula |
| [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) | ed25519 identity proof, key rotation |
| [WORKSPACE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/WORKSPACE.en.md) | Workspace layout, `workspace.toml`, routes.toml |
| [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md) | Naming conventions for files, branches, notes, agents |
| [FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) | Aparihāniya-dhamma 7 fleet-health signals |
| [GLOSSARY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) | Term definitions |
| [FAQ.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) | Frequently asked questions |
| [INCARNATION.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/INCARNATION.en.md) | Agent incarnation walkthrough |

**Editing workflow for a doc pair:**

```bash
# 1. Edit the EN file
$EDITOR docs/en/HARNESS.en.md

# 2. Edit the TH counterpart in the same session
$EDITOR docs/th/HARNESS.th.md

# 3. Commit both together
git add docs/en/HARNESS.en.md docs/th/HARNESS.th.md
git commit -m "docs(harness): document tool-auth credential lifecycle"
```

Never commit only the EN file without the TH pair (unless the file is explicitly exempt per Rule 3).

**Frontmatter requirements** for Tier 2 docs:

```yaml
---
title: "Document Title"
version: "X.Y.Z"
audience: "target reader"
language: en
counterpart: FILENAME.th.md
---
```

The `counterpart` field lets tooling and readers navigate directly to the other language version.

---

## 9. Skill Development

Skills are reusable agent capabilities that live under `modules/skills/<name>/` and are declared in `config.manifest.json`. They follow the format defined in [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) and have a `maturity` level (`L1` through `L7`) in their frontmatter.

### 9.1 Skill CLI Reference

```bash
# Scaffold a new skill from the skill template
bwoc skill init <name>

# Install a skill from a local path, git URL, or tarball URL
# SHA-256 trust gate is enforced — the install will fail if the
# computed hash does not match the declared hash in the manifest.
bwoc skill install <source>

# List installed skills with their maturity and enabled state
bwoc skill list

# Show one skill's manifest and SPEC location in full detail
bwoc skill show <name>

# Enable a skill on the current agent (sets enabled = true in manifest)
bwoc skill enable <name>

# Disable a skill (keeps the entry; sets enabled = false)
bwoc skill disable <name>

# Remove a skill: deletes modules/skills/<name>/ and cleans
# every consuming agent's manifest entry
bwoc skill remove <name>

# Statically verify skills: prints each [gates].verify command
# WITHOUT running it. Exits non-zero if any gate is missing.
bwoc skill verify <name>

# Verify AND execute the gates (gates come from an untrusted manifest —
# only run this after you have read and understood the gate commands)
bwoc skill verify <name> --run-gates
```

### 9.2 Skill Development Conventions

**Frontmatter.** Skill `.md` files use full Obsidian frontmatter with the `domain/<area>` tag and a `maturity` field (`L1` = stub, `L7` = production-hardened). This is Tier 2 format — wikilinks and callouts are allowed.

**The trust gate.** `bwoc skill install` computes the SHA-256 hash of the skill archive and compares it against the declared hash in the manifest. Installation fails if the hashes do not match. This prevents supply-chain substitution of skill archives. The `verify` subcommand prints the `[gates].verify` commands without running them — read them first, then pass `--run-gates` if you trust the source.

**Relationship to agents.** Skills are installed at the workspace level and enabled per-agent. An agent's `config.manifest.json` lists which skills are enabled. `bwoc skill enable` modifies the manifest; `bwoc skill disable` sets `enabled = false` without removing the entry (preserving the declared dependency for future re-enable).

**Maturity levels.** A skill at `L1` (stub) may be checked in to declare intent; it is not expected to be production-safe. PRs that add an `L7` skill must include gate verification evidence. Do not ship `L1` skills as enabled defaults in the template.

---

## 10. Plugin Development

Plugins extend the workspace itself (not individual agents) and live under `modules/plugins/<name>/`. They are declared in `workspace.toml` under `[plugins.<name>]`. Plugin format is defined in [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md).

### 10.1 Plugin CLI Reference

```bash
# Scaffold a new plugin from the plugin template
bwoc plugin init <name>

# Install a plugin from a local path, git URL, or tarball URL
# SHA-256 trust gate enforced (same mechanism as skill install)
bwoc plugin install <source>

# List installed plugins with their enabled state and workspace registration
bwoc plugin list

# Show one plugin's manifest, SPEC location, and workspace registration
bwoc plugin show <name>

# Enable a plugin in workspace.toml (sets [plugins.<name>] enabled = true)
bwoc plugin enable <name>

# Disable (keeps the entry; sets enabled = false)
bwoc plugin disable <name>

# Remove: deletes modules/plugins/<name>/ and removes the
# [plugins.<name>] table from workspace.toml
bwoc plugin remove <name>
```

### 10.2 Plugin vs Skill

| Dimension | Skill | Plugin |
|---|---|---|
| Scope | Per-agent capability | Workspace-level extension |
| Enabled in | `config.manifest.json` per agent | `workspace.toml` for all agents |
| `verify` subcommand | Yes (SHA-256 gate + optional `--run-gates`) | No `verify` in v1 |
| Template | `modules/skill-template/` | `modules/plugin-template/` |
| Location | `modules/skills/<name>/` | `modules/plugins/<name>/` |

Note: plugins do not have a `verify` subcommand in v1. The trust gate still applies at install time (SHA-256 check) but there is no post-install gate execution command. See [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) for the full specification.

---

## 11. Contribution Flow

This section documents every gate a PR must pass. Follow the steps in order.

### 11.1 Fork and Branch

```bash
# Fork on GitHub, then clone your fork
git clone https://github.com/<your-github-handle>/BWOC-Framework
cd BWOC-Framework
git remote add upstream https://github.com/bemindlabs/BWOC-Framework

# Create a topic branch — type follows Conventional Commits vocabulary
git checkout -b feat/<short-name>     # new capability
git checkout -b fix/<short-name>      # bug fix
git checkout -b docs/<short-name>     # documentation
git checkout -b refactor/<short-name> # internal restructure
git checkout -b chore/<short-name>    # dependency bump, CI config
```

No `release/*` or `hotfix/*` branches. Version tags are cut directly on `main`. The branch name format is `type/<short-name>` — the same vocabulary as commit types.

### 11.2 Make Changes

- Keep the diff focused. One concern per PR. If you discover an unrelated issue, fix it in a separate branch.
- After every `.rs` or `.toml` edit, the auto-version hook bumps the patch. Do not revert this.
- After every `.md` edit, `Document-Version` in `VERSION.md` is bumped. Do not revert this.
- For significant changes, write one `notes/YYYY-MM-DD_<title>.md` entry covering the session.

### 11.3 Verify Locally

Run these before pushing — CI will catch them, but catching them locally is faster:

```bash
# Format check
cargo fmt --check

# Lints (warnings treated as errors)
cargo clippy -- -D warnings

# Full build on your platform
cargo build

# All tests (offline, no network)
cargo test

# Backend neutrality audit (if you touched any agent or AGENTS.md)
bwoc check --all
```

### 11.4 Commit

```bash
git add -A
git commit -m "feat(bwoc-harness): add P3 task queue with CancellationToken"
```

Use Conventional Commits format. The commit message will become the squash-merge commit on `main`.

### 11.5 Open a Pull Request

```bash
# Push your branch
git push origin feat/<short-name>

# Open a PR against upstream main
gh pr create \
  --repo bemindlabs/BWOC-Framework \
  --base main \
  --title "feat(bwoc-harness): add P3 task queue with CancellationToken" \
  --body "## What and why
<explain the change and motivation>

## Checklist
- [ ] Branch up to date with main
- [ ] cargo fmt --check passes
- [ ] cargo clippy -- -D warnings passes
- [ ] cargo test passes on my platform
- [ ] Documentation updated (EN + TH if applicable)
- [ ] bwoc check --all passes (if agents touched)
- [ ] No secrets, credentials, or personal paths in the diff
- [ ] Significant change → notes/ entry added

Closes #<issue>"
```

### 11.6 Handle Review Comments

If a reviewer opens a thread:
- Address the concern in a new commit on your branch (do not amend already-pushed commits unless explicitly asked).
- Mark the conversation resolved after your fix is pushed.
- If you disagree, explain in the thread — do not silently leave it open.

The conversation-resolution gate is hard. An unresolved thread blocks the merge regardless of CI state.

### 11.7 Keep the Branch Up to Date

If `main` advances while your PR is open:

```bash
gh pr update-branch <PR_NUMBER>
```

This merges `main` into your branch (or rebases, depending on repo settings) and re-triggers CI. Do not force-push to work around a stale branch — the protection is strict.

Alternatively, rebase manually:

```bash
git fetch upstream
git rebase upstream/main
git push --force-with-lease origin feat/<short-name>
```

### 11.8 Enable Auto-Merge

Once you believe the PR is ready (CI green, all threads resolved, branch up to date):

```bash
gh pr merge <PR_NUMBER> --squash --auto
```

GitHub will merge the moment all gates clear. Do not merge manually with `--squash` or `--merge` without `--auto` — that bypasses the CI wait gate.

### 11.9 After Merge

- Delete the branch: `git push origin --delete feat/<short-name>`.
- Update your local main: `git fetch upstream && git rebase upstream/main`.
- If the PR was significant, verify the `notes/` entry exists.

---

## 12. Releasing

Releases use CalVer tags cut directly on `main`. The CI pipeline (`release.yml`) builds cross-platform binaries and uploads them as GitHub Release assets with SHA-256 sidecars. The full process is documented in [RELEASING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/RELEASING.en.md).

**Quick recipe for maintainers:**

```bash
# Decide the CalVer tag for today's release iteration
# Format: vYYYY.M.D-<patch> where patch starts at 0
git tag v2026.6.6-0

# Push the tag — release.yml takes over from here
git push origin v2026.6.6-0
```

`release.yml` builds the matrix (Linux x86_64, macOS aarch64 + x86_64, Windows x86_64), packages each as `<archive>.{tar.gz,zip}` with a `.sha256` sidecar, and uploads them to the auto-created GitHub Release.

Same-day reissue: `v2026.6.6-1`, `v2026.6.6-2`, etc.

The Homebrew formula update is a separate auto-merged PR opened by the release pipeline — it never pushes directly to `main`.

---

## 13. Where to Look Next

| Question | Where to look |
|---|---|
| How does the agentic loop work? | [HARNESS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/HARNESS.en.md) |
| How do agents sign and verify messages? | [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) |
| How does A2A interop work? | [ARCHITECTURE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/ARCHITECTURE.en.md) |
| How do I write a skill? | [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) |
| How do I write a plugin? | [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) |
| How do I cut a release? | [RELEASING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/RELEASING.en.md) |
| What do all the naming conventions mean? | [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md) |
| What is a fleet health check? | [FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) |
| What does a term mean? | [GLOSSARY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) · [../glossary.en.md](../glossary.en.md) |
| I want to incarnate an agent, not build the framework | [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md) |
| Full PR checklist | [CONTRIBUTING.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/CONTRIBUTING.md) |
| Current version and phase | [VERSION.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) |

## Publishing this site (GitHub Pages)

This handbook ships an MkDocs + Material for MkDocs config (`mkdocs.yml`) and a GitHub Actions workflow (`.github/workflows/pages.yml`).

To publish:

1. Push the handbook to a GitHub repo.
2. In the repo, go to **Settings → Pages** and set the source to **GitHub Actions**.
3. The workflow builds and deploys the site on every push to `main` (and via manual `workflow_dispatch`).

To preview locally:

```bash
pip install -r requirements.txt
mkdocs serve
```
