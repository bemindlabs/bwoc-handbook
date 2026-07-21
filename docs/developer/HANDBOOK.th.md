---
title: "คู่มือนักพัฒนา BWOC"
version: "1.0.0"
audience: "ผู้พัฒนาและผู้ร่วมพัฒนา framework (Rust)"
language: th
counterpart: HANDBOOK.en.md
---

> ฉบับภาษาอังกฤษ: [HANDBOOK.en.md](HANDBOOK.en.md) — Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) — v2.33.0 · Rust 1.85+

# คู่มือนักพัฒนา BWOC

คู่มือนี้เขียนสำหรับผู้ที่พัฒนาต่อยอดหรือมีส่วนร่วมพัฒนา BWOC Framework โดยตรง ได้แก่ ผู้เขียน crate, ผู้ร่วมพัฒนา CLI, ผู้พัฒนา harness และทุกคนที่ส่ง PR ขึ้น upstream หากต้องการเพียงสร้างและรัน agent ดูที่ [[../agents/HANDBOOK.th.md]] แทน

---

## สารบัญ

1. [สองขอบเขตที่ต้องรู้](#1-สองขอบเขตที่ต้องรู้)
2. [แผนผัง Crate](#2-แผนผัง-crate)
3. [หลักการ Dependency Quarantine](#3-หลักการ-dependency-quarantine)
4. [Build, Test, และ Install](#4-build-test-และ-install)
5. [ด่าน CI](#5-ด่าน-ci)
6. [กฎเหล็กเก้าข้อ](#6-กฎเหล็กเก้าข้อ)
7. [Hook Auto-Version](#7-hook-auto-version)
8. [เอกสารและ Workflow สองภาษา](#8-เอกสารและ-workflow-สองภาษา)
9. [การพัฒนา Skill](#9-การพัฒนา-skill)
10. [การพัฒนา Plugin](#10-การพัฒนา-plugin)
11. [ขั้นตอนการ Contribute](#11-ขั้นตอนการ-contribute)
12. [การ Release](#12-การ-release)
13. [ลำดับถัดไปที่ควรอ่าน](#13-ลำดับถัดไปที่ควรอ่าน)

---

## 1. สองขอบเขตที่ต้องรู้

เมื่อเปิด framework repo จะพบพื้นที่ทำงานสองส่วนที่แตกต่างกัน การสับสนสองส่วนนี้มักนำไปสู่ความผิดพลาด

### 1.1 Framework Root

ทุกอย่างในระดับบนสุดของ [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework):

- `crates/` — Rust workspace (สิบ crate ตามที่อธิบายด้านล่าง)
- `docs/en/` และ `docs/th/` — เอกสาร specification สองภาษา ได้แก่ `ARCHITECTURE`, `DESIGN`, `HARNESS`, `SKILLS`, `PLUGINS`, `RELEASING`, `SIGNING`, `WORKSPACE`, `NAMING`, `FLEET-GOVERNANCE`, `GLOSSARY`, `FAQ`, `INCARNATION` แต่ละฉบับมีทั้ง `.en.md` และ `.th.md`
- OSS docs ระดับ root: `README.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `SECURITY.md`, `LICENSE`, `VERSION.md` — ภาษาอังกฤษเท่านั้น (ดู [กฎข้อ 3](#กฎข้อ-3-bilingual-parity))
- `scripts/install.sh`, `scripts/bump-version.sh` — scripts สำหรับ release pipeline
- `.claude/hooks/auto-version.sh` — hook สำหรับ bump patch อัตโนมัติ (ดู [หัวข้อ 7](#7-hook-auto-version))
- `modules/` — agent-template, plugin-template, skill-template และ `plugins/`, `skills/` ที่ build ไว้แล้ว

การแก้ไขในส่วนนี้ต้องผ่านขั้นตอน PR และ CI gate ทั้งหมดตามที่อธิบายใน [หัวข้อ 11](#11-ขั้นตอนการ-contribute)

### 1.2 Agent Template (`modules/agent-template/`)

artifact ที่ `bwoc new` ใช้ copy เมื่อสร้าง agent ใหม่ มีโครงสร้างดังนี้:

- `AGENTS.md` — แหล่งความจริงเดียวสำหรับทุก backend โดย backend symlinks (`CLAUDE.md`, `AGY.md`, `CODEX.md`, `KIMI.md`, `OLLAMA.md`, `OPENAI.md`) ทั้งหมดชี้มาที่นี่
- Slot directories: `persona/`, `mindsets/`, `skills/`, `interconnect/`, `memories/`
- `docs/` (มีคู่ EN/TH: `OVERVIEW`, `PHILOSOPHY`, `PRD`, `SELF-IMPROVEMENT`, `SRS`, `THREAT-MODEL`)
- `scripts/check-agent-neutrality.sh` และ `scripts/incarnate.sh` — รันจากภายใน template directory เท่านั้น ไม่ใช่จาก framework root

`AGENTS.md` ของ template คือ specification ที่ incarnated agent อ่านขณะ runtime ส่วน `CLAUDE.md` ของ framework root ใช้สำหรับ AI editor ที่ทำงานบน framework เอง อย่าสับสนสองไฟล์นี้

> **ข้อควรระวัง: Repo ทำหน้าที่เป็น Workspace ด้วย.** Framework repo ใช้งานตัวเองเป็น BWOC workspace (`.bwoc/workspace.toml` ที่ root) สำหรับทดสอบ CLI แบบ end-to-end ดังนั้น `.gitignore` ของมันจะ exclude `.bwoc/`, `agents/`, และ `projects/` ทั้งหมด — **ตรงกันข้าม** กับสิ่งที่ `bwoc init` เขียนให้ user workspace ความแตกต่างนี้อยู่ใน `crates/bwoc-cli/src/init.rs::GITIGNORE_TEMPLATE`

---

## 2. แผนผัง Crate

workspace มีสิบ crate ซึ่งความสัมพันธ์ dependency ของมันบังคับใช้หลัก quarantine ที่อธิบายใน [หัวข้อ 3](#3-หลักการ-dependency-quarantine) โดย `[workspace.package].version` ปัจจุบันคือ `2.33.0`

### 2.1 ตารางอ้างอิงด่วน

| Crate | Binary | จุดประสงค์ | Deps ที่อนุญาต (นอก workspace) | ห้ามมีเด็ดขาด |
|---|---|---|---|---|
| `bwoc-core` | — | Shared types: manifest, identity, lifecycle phases | `serde`, `serde_json`, `toml`, `thiserror` | async, HTTP, crypto, SQLite, network ทุกชนิด |
| `bwoc-signing` | — | ed25519 primitives สำหรับ agent identity proof | `ed25519-dalek`, `rand_core`, `hex`, `serde*`, `thiserror` | async, HTTP, heavy runtime |
| `bwoc-cli` | `bwoc` | CLI สำหรับผู้ใช้ output สองภาษา TH/EN | `bwoc-core`, `bwoc-signing`, `bwoc-tui`, `clap`, `ratatui`, `crossterm`, `fluent-bundle`, `unic-langid`, `sha2`, `include_dir`, `libc`; `interprocess` (Windows เท่านั้น) | `tokio`/`axum`/`reqwest` โดยตรง; ให้ exec binary หนักเป็น subprocess แทน |
| `bwoc-agent` | `bwoc-agent` | Runtime ขนาดเล็กที่ส่งพร้อมกับแต่ละ agent; daemon `--serve` ผ่าน Unix socket + Windows named pipe; clients ping/status/stop | `bwoc-core`, `bwoc-signing`, `fluent-bundle`, `unic-langid`, `ctrlc`, `serde_json`; `interprocess` (Windows เท่านั้น) | Heavy async runtime ระดับ library |
| `bwoc-harness` | `bwoc-harness` | Self-hosted agentic runtime: OpenAI-compatible provider, agentic loop, core tools, P3 task queue, telemetry, tool-auth, event stream `--chat` | `tokio` (rt-multi-thread), `reqwest`, `keyring`, `landlock` (Linux), optional `opentelemetry` (feature `otel`) | ห้ามอยู่ใน `bwoc-core`; CLI exec เป็น subprocess |
| `bwoc-deep-memory` | `bwoc-deep-memory` | Tier-2 memory ผ่าน SQLite local; semantic recall ด้วย embeddings; พูดสัญญา `wake-up`/`search`/`mine` | `rusqlite` (bundled), `reqwest` (blocking), `clap`, `serde*` | ห้ามอยู่ใน `bwoc-core` |
| `bwoc-a2a` | `bwoc-a2a` | A2A protocol v1.0.0 interop; Agent Card, JSON-RPC handlers; axum HTTP listener; outbound client | `axum`, `tokio`, `reqwest`, `async-stream`, `futures-core`, `clap` | ห้ามอยู่ใน `bwoc-core`; CLI exec เป็น subprocess |
| `bwoc-mqtt` | `bwoc-mqtt` | MQTT transport สำหรับ inter-workspace routing: publish envelope หรือ subscribe และส่งเข้า `inbox.jsonl` | `rumqttc`, `serde_json`, `clap`, `thiserror` | ห้ามอยู่ใน `bwoc-core` |
| `bwoc-tui` | — | ratatui dashboard แบบ interactive และ `--chat` client; shared rendering primitives | `ratatui`, `crossterm`, `serde_json` | Heavy async runtime |

### 2.2 รายละเอียดแต่ละ Crate

#### `bwoc-core`

**จุดประสงค์.** แหล่ง shared types เดียวสำหรับทั้ง framework: schema ของ `config.manifest.json`, struct ของ agent identity, สามเฟสวงจรชีวิต (*uppāda · ṭhiti · vaya*) และ error enumeration ทุก crate อื่นที่ต้องการเข้าใจ "agent คืออะไร" import จาก `bwoc-core`

**สิ่งที่อยู่ที่นี่.** Pure data types, serialization/deserialization (ผ่าน `serde`), validation logic และ error types แบบ `thiserror` เท่านั้น

**สิ่งที่ห้ามเข้ามาเด็ดขาด.** async executor, HTTP client, cryptographic operation, database driver, network socket หรือความสามารถเฉพาะ OS ใดๆ กฎนี้เด็ดขาด: `bwoc-core` ต้องสามารถ import ได้ในสภาพแวดล้อม `no_std` ในอนาคตโดยไม่แก้ไข และต้องตรวจสอบ audit ได้ภายในไม่กี่นาที การเพิ่ม `tokio` หรือ `reqwest` ที่นี่ แม้เป็น optional feature ก็ละเมิด quarantine และจะถูก reject ใน review

**ความสัมพันธ์กับ crate อื่น.** `bwoc-cli`, `bwoc-agent`, `bwoc-harness`, `bwoc-a2a`, `bwoc-mqtt`, `bwoc-tui` และ `bwoc-deep-memory` ทั้งหมด depend on `bwoc-core` มันคือ crate เดียวที่เป็นฐานรากแท้จริง ส่วน crate อื่นเป็น optional ในการ deploy

**วิธีตรวจสอบ.** `cargo tree -p bwoc-core` ต้องแสดงเฉพาะ `serde`, `serde_json`, `toml` และ `thiserror` (พร้อม transitive pure-Rust deps ของมัน) หาก async หรือ network dep ปรากฏที่นี่ถือเป็น quarantine violation — เปิด PR review comment แบบ blocking

---

#### `bwoc-signing`

**จุดประสงค์.** สร้าง ed25519 keypair, เซ็น message และตรวจสอบ signature สำหรับ agent identity proof (feature set HV2-4) `bwoc-cli` ใช้เพื่อเซ็น outbound messages และ `bwoc-agent` ใช้ตรวจสอบ inbound messages

**สิ่งที่อยู่ที่นี่.** keypair และ signing ด้วย `ed25519-dalek`, entropy ด้วย `rand_core`, encoding key material ด้วย `hex` และ canonical JSON serialization (sorted keys ผ่าน `BTreeMap`) ของ fields ที่ถูกเซ็น

**สิ่งที่ห้ามเข้ามา.** async runtime, HTTP client หรือ tokio dep ใดๆ crate นี้ synchronous โดยตั้งใจเพื่อให้ link ได้จากทั้ง CLI (synchronous) และ agent daemon โดยไม่ดึง async executor ตัวที่สองเข้ามา

**ความสัมพันธ์กับ crate อื่น.** `bwoc-cli` เรียก `bwoc trust --keygen` เพื่อสร้าง keypair ผ่าน crate นี้ `bwoc-agent` เรียก verify path บนทุก inbound envelope `bwoc-core` ไม่ depend on `bwoc-signing` — trust layer อยู่เหนือ type layer

**วิธีตรวจสอบ.** `cargo tree -p bwoc-signing` ต้องไม่มี node tokio, axum หรือ reqwest

---

#### `bwoc-cli`

**จุดประสงค์.** binary `bwoc` ที่ user และ operator ใช้งานประจำวัน: `bwoc new`, `bwoc check`, `bwoc spawn`, `bwoc chat`, `bwoc status`, `bwoc trust`, `bwoc skill`, `bwoc plugin` และอีกประมาณสามสิบ subcommand output แปลภาษาผ่าน `fluent-bundle` (TH และ EN bundles embed ตอน compile ด้วย `include_dir`) ส่ง pre-built สำหรับ macOS (aarch64 + x86_64), Linux (x86_64) และ Windows (x86_64) ในทุก release

**สิ่งที่อยู่ที่นี่.** CLI argument parsing (`clap`), command dispatch, workspace resolution logic, agent registry read/write (`agents.toml`), `bwoc check` neutrality audit, `bwoc init` workspace scaffold (รวม constant `GITIGNORE_TEMPLATE`), `bwoc trust --keygen` keypair generation และ TUI dashboard (ผ่าน `bwoc-tui`) บน Windows: `interprocess` สำหรับ named-pipe IPC กับ `bwoc-agent --serve`

**สิ่งที่ห้ามเข้ามา.** import `tokio` หรือ `axum` โดยตรง เมื่อ subcommand ต้องการ async capability (A2A, harness, MQTT) CLI exec binary คู่ที่เกี่ยวข้องเป็น subprocess (`bwoc-a2a`, `bwoc-harness`, `bwoc-mqtt`) วิธีนี้ทำให้ dependency tree ของ CLI synchronous, startup เร็ว และ binary มีขนาดเล็ก subprocess pattern อยู่ใน `crates/bwoc-cli/src/spawn.rs`

**ความสัมพันธ์กับ crate อื่น.** `bwoc-cli` depend on `bwoc-core`, `bwoc-signing` และ `bwoc-tui` exec `bwoc-agent`, `bwoc-harness`, `bwoc-a2a` และ `bwoc-mqtt` เป็น subprocess ไม่ link ตรงเลย

**วิธีตรวจสอบ.** `cargo tree -p bwoc-cli | grep tokio` ต้องคืนค่าว่าง (ยกเว้น transitive ของ `interprocess` Windows ซึ่งยอมรับได้เพราะอยู่ใน cfg gate แยก) รัน `cargo build -p bwoc-cli` บนทั้งสามแพลตฟอร์มใน CI

---

#### `bwoc-agent`

**จุดประสงค์.** runtime ขนาดเล็กที่ส่งพร้อมกับแต่ละ incarnated agent รันเป็น `bwoc-agent --serve` เพื่อเริ่ม daemon ที่รับฟังบน Unix socket (macOS/Linux) หรือ Windows named pipe จากนั้นตอบสนองต่อ message `PING`, `STATUS` และ `STOP` จาก CLI รวมถึงพิมพ์ liveness banner จาก `config.manifest.json` เมื่อเริ่มต้น

**สิ่งที่อยู่ที่นี่.** daemon entry point, IPC message dispatch, liveness banner rendering จาก manifest, signal handling (`ctrlc`), signature verification ของ inbound envelopes (ผ่าน `bwoc-signing`) และ Fluent localization strings บน Windows: `interprocess` named-pipe server และ client

**สิ่งที่ห้ามเข้ามา.** heavy async frameworks, HTTP stacks หรือ database drivers daemon นี้ lightweight โดยตั้งใจเพื่อให้รันพร้อมกับ agent processes จำนวนมากโดยไม่แย่ง resource

**ความสัมพันธ์กับ crate อื่น.** depend on `bwoc-core` และ `bwoc-signing` `bwoc-cli` ควบคุมผ่าน IPC (ping, status, stop) `bwoc-harness` คือ peer ที่จัดการ LLM interaction จริงๆ; `bwoc-agent` จัดการ control plane

**วิธีตรวจสอบ.** `cargo tree -p bwoc-agent | grep -E 'tokio|axum|reqwest'` ต้องว่าง

---

#### `bwoc-harness`

**จุดประสงค์.** self-hosted agentic runtime สำหรับ backend `ollama` และ `openai-compatible` มี: connect กับ OpenAI-compatible endpoint, จัดการ conversation context, execute tools, process P3 task queue, emit telemetry, บังคับใช้ tool-auth ผ่าน OS keyring และ stream events บน `--chat` event stream ของ `--chat` ถูกใช้โดย `bwoc-tui` สำหรับ full-screen chat client

**สิ่งที่อยู่ที่นี่.** agentic loop (`src/loop/`), tool executor (`src/tools/`), provider client (`src/provider/`), task queue (`src/queue/`), tool-auth credential manager (`src/auth/`), telemetry (`src/telemetry/` — optional ด้วย feature `otel`) และ `--chat` SSE event-stream server บน Linux: `landlock` LSM bindings สำหรับ kernel-enforced filesystem access control

**สิ่งที่ห้ามเข้ามา.** ไม่มีอะไรจาก `bwoc-harness` ที่ควรไหลกลับไปยัง `bwoc-core`, `bwoc-signing` หรือ `bwoc-cli` harness คือ leaf crate ใน dependency DAG `keyring` dep (platform credential store) และ `landlock` (Linux security module) อยู่ที่นี่เพราะนี่คือที่ที่ quarantine อนุญาตให้มี heavy deps

**Feature flags.** Default build มีศูนย์ OpenTelemetry dependencies เปิดใช้ด้วย `cargo build --features otel` ห้ามทำให้ `otel` เป็น default feature เด็ดขาด

**ความสัมพันธ์กับ crate อื่น.** `bwoc-cli` exec `bwoc-harness` เป็น subprocess สำหรับ `bwoc spawn` (backend ollama/openai-compatible) และ `bwoc chat --tui` harness อ่าน `config.manifest.json` ของ agent (parse ผ่าน types ของ `bwoc-core`) เพื่อ resolve endpoint และ model

**วิธีตรวจสอบ.** `cargo test -p bwoc-harness` ใช้ `wiremock` สำหรับ offline provider tests ไม่ต้องใช้ network จริงใน CI

---

#### `bwoc-deep-memory`

**จุดประสงค์.** Tier-2 (semantic, persistent) memory สำหรับ agents พูดสัญญาสามกริยาที่ harness และ CLI invoke: `wake-up` (emit prior context เมื่อเริ่ม session), `search` (retrieve ข้อมูลอดีตที่เกี่ยวข้องตาม query) และ `mine` (persist session learnings เมื่อสิ้นสุด session) storage เป็น SQLite local; recall ผ่าน embeddings ที่ดึงจาก OpenAI-compatible `/v1/embeddings` endpoint

**สิ่งที่อยู่ที่นี่.** command handlers สามตัว, SQLite schema management (ผ่าน `rusqlite` feature `bundled` — SQLite compile จาก source ไม่มี system dependency), blocking HTTP calls ไปยัง embeddings endpoint (`reqwest` blocking client) และ binary entry point แบบ `clap`

**สิ่งที่ห้ามเข้ามา.** crate นี้ต้องไม่เพิ่มเป็น dependency ของ `bwoc-core` เด็ดขาด feature `bundled` ของ `rusqlite` compile C library; อนุญาตเฉพาะที่นี่และห้ามทุกที่อื่น

**ความสัมพันธ์กับ crate อื่น.** `bwoc-harness` เรียก `bwoc-deep-memory` เป็น subprocess (ผ่าน `deepMemoryCmd` จาก manifest) ระหว่าง agentic loop `bwoc memory wake-up`, `bwoc memory t2-search` และ `bwoc memory mine` ใน CLI ก็ delegate มาที่มันเช่นกัน มันไม่ถูก link โดยตรงจาก `bwoc-core` หรือ `bwoc-cli`

**วิธีตรวจสอบ.** `cargo build -p bwoc-deep-memory` บนทุกแพลตฟอร์มใน CI feature `bundled` ของ SQLite หมายความว่าไม่ต้องมีขั้นตอน `libsqlite3-dev` ใน CI

---

#### `bwoc-a2a`

**จุดประสงค์.** implement A2A (Agent2Agent) protocol v1.0.0 สำหรับ BWOC agents มี: Agent Card (การโฆษณา capability สาธารณะ), JSON-RPC message และ task handlers, axum HTTP listener และ (ใน phase 4) outbound client ด้วย `reqwest` agents expose HTTP endpoint `/a2a` ผ่าน crate นี้; agent อื่นและระบบภายนอกค้นพบและเรียกผ่านมัน

**สิ่งที่อยู่ที่นี่.** type definitions ของ A2A, Agent Card builder (bridge manifest fields ของ `bwoc-core` ไปยัง A2A schema), JSON-RPC 2.0 request/response handling, SSE streaming สำหรับ task progress, `axum` router และ binary entry point แบบ `clap` tokio runtime ที่ instantiate ที่นี่เป็น `current_thread` runtime เพื่อให้ CLI process synchronous

**สิ่งที่ห้ามเข้ามา.** HTTP และ network deps ต้องอยู่ใน `bwoc-a2a` `bwoc-core` ต้องไม่รับรู้ A2A specifics ส่วน protocol core (types, card, RPC parsing) เป็น transport-agnostic และ unit-testable โดยไม่ต้อง bind real socket

**ความสัมพันธ์กับ crate อื่น.** `bwoc-a2a` เข้าถึงได้จาก CLI ผ่าน subcommand `bwoc a2a` (ซึ่ง exec binary `bwoc-a2a`) อ่าน manifest types ของ `bwoc-core` เพื่อสร้าง Agent Card เป็นอิสระจาก `bwoc-harness` — harness จัดการ LLM interaction; A2A จัดการ protocol interop

**วิธีตรวจสอบ.** tests ใช้ `tower` เพื่อขับ `axum` router โดยไม่ต้อง bind network socket จริง และ `wiremock` สำหรับ outbound client tests `cargo test -p bwoc-a2a` offline ทั้งหมด

---

#### `bwoc-mqtt`

**จุดประสงค์.** MQTT transport สำหรับ inter-workspace message routing สองโหมด: `publish` (ส่ง envelope ไปยัง broker topic) และ `serve` (subscribe กับ topic และส่ง envelopes ที่มาถึงเข้า `inbox.jsonl` ของ agent) ทำให้ agent สื่อสารข้าม workspace ได้โดยไม่ต้องมี A2A HTTP infrastructure

**สิ่งที่อยู่ที่นี่.** MQTT client (`rumqttc`), envelope serialization, append logic ของ `inbox.jsonl` และ binary entry point แบบ `clap`

**สิ่งที่ห้ามเข้ามา.** crate นี้ต้องไม่ถูก link โดยตรงจาก `bwoc-core` หรือ `bwoc-cli` CLI exec เป็น subprocess

**ความสัมพันธ์กับ crate อื่น.** parallel กับ `bwoc-a2a` ในแง่ conceptual ทั้งคู่คือ transport crates MQTT เหมาะกับ low-latency broadcast ไปยัง agents หลายตัว; A2A เหมาะกับ point-to-point request/response ที่มี protocol semantics เต็มรูปแบบ สองอันนี้อยู่ร่วม workspace ได้

---

#### `bwoc-tui`

**จุดประสงค์.** shared ratatui rendering primitives สำหรับ interactive dashboard และ `--chat` full-screen client ใช้ทั้งใน `bwoc-cli` (command `bwoc dashboard`) และ `bwoc-harness` (render `--chat` event stream)

**สิ่งที่อยู่ที่นี่.** widget definitions, agent-list dashboard, chat message rendering และ terminal management แบบ `crossterm` parse `chat_proto` event stream ที่ emit โดย `bwoc-harness --chat`

**สิ่งที่ห้ามเข้ามา.** async runtime, HTTP client หรือ business logic ใดๆ นี่คือ pure rendering crate

**ความสัมพันธ์กับ crate อื่น.** `bwoc-tui` คือ crate เดียว (นอกจาก `bwoc-core`) ที่ `bwoc-cli` link โดยตรงพร้อมกับที่ `bwoc-harness` ใช้ด้วย crate อื่นทั้งหมดที่มี heavy deps จะถูก exec เป็น subprocess

---

## 3. หลักการ Dependency Quarantine

framework บังคับใช้ quarantine เข้มงวดว่า heavy dependencies ได้รับอนุญาตที่ไหน นี่ไม่ใช่ preference ด้านสไตล์ — มันคือ invariant ด้านความปลอดภัยและการบำรุงรักษาที่บังคับใช้ใน PR review

**กฎในประโยคเดียว:** `bwoc-core` ยอมรับเฉพาะ pure-Rust serialization deps เท่านั้น async, HTTP, crypto, SQLite, OS keyring และ platform-security bindings อนุญาตเฉพาะใน crates ที่ออกแบบมารองรับโดยเฉพาะ

**ทำไมจึงสำคัญ**

1. **Audit surface.** framework สำหรับรัน AI agents ที่ execute arbitrary code ต้องมี core ที่อ่านและตรวจสอบได้เร็ว `bwoc-core` ที่มีสี่ deps ตรวจสอบได้; อันที่มี transitive tokio tree ทำไม่ได้
2. **Attack surface.** แต่ละ dep คือ supply-chain risk การจำกัด network deps ไว้ที่ `bwoc-a2a` และ `bwoc-harness` หมายความว่า HTTP library ที่ถูก compromise ไม่สามารถกระทบ code paths ที่จัดการเฉพาะ manifest parsing
3. **Startup time.** `bwoc-cli` synchronous เพราะไม่ link async runtime โดยตรง subcommands ที่ต้องการ async exec binary คู่ วิธีนี้ทำให้ `bwoc list` และ `bwoc check` ทันทีแม้บน machine ที่ช้า
4. **Testability.** `bwoc-core` และ `bwoc-signing` tests รันโดยไม่ต้องมี network mocks เลย `bwoc-harness` และ `bwoc-a2a` tests ใช้ `wiremock` สำหรับ offline simulation ขอบเขตชัดเจน

**แผนที่ deps ที่อนุญาต** (สรุป):

| สิ่งที่ | อนุญาตใน |
|---|---|
| `serde`, `toml`, `thiserror` | ทุก crate |
| `ed25519-dalek`, `rand_core`, `hex` | `bwoc-signing` เท่านั้น |
| `tokio`, `axum`, `reqwest`, `async-stream`, `futures-*` | `bwoc-harness`, `bwoc-a2a`, `bwoc-deep-memory` (blocking reqwest เท่านั้น) |
| `rusqlite` (bundled C) | `bwoc-deep-memory` เท่านั้น |
| `rumqttc` | `bwoc-mqtt` เท่านั้น |
| `keyring` | `bwoc-harness` เท่านั้น |
| `landlock` | `bwoc-harness` (Linux target) เท่านั้น |
| `ratatui`, `crossterm` | `bwoc-tui`, `bwoc-cli` เท่านั้น |
| `fluent-bundle`, `unic-langid`, `include_dir` | `bwoc-cli`, `bwoc-agent` เท่านั้น |
| `interprocess` (named pipe) | `bwoc-cli`, `bwoc-agent` (Windows cfg gate เท่านั้น) |

**วิธีปฏิบัติตามเมื่อเพิ่ม dep ใหม่.** ก่อนเพิ่ม dependency ใดๆ:
1. ตรวจสอบว่ากำลังแก้ไข crate ใด
2. ตรวจสอบแผนที่ deps ที่อนุญาตด้านบน
3. หาก dep ข้ามขอบเขต quarantine ให้เลือก (a) วางไว้ใน crate ที่เหมาะสมและ exec เป็น subprocess หรือ (b) ยกคำถามใน PR เพื่อให้ทีมตัดสินใจขอบเขตที่ถูกต้อง
4. ห้ามเพิ่ม dep ให้ `bwoc-core` โดยไม่มี architectural decision ชัดเจนและ note ใน `notes/`

**วิธีตรวจสอบ.** `cargo tree -p bwoc-core` และ `cargo tree -p bwoc-signing` ต้องไม่แสดง tokio, axum, reqwest, rusqlite, rumqttc, keyring หรือ landlock CI matrix รันบนสามแพลตฟอร์ม; quarantine violation ที่หลุดผ่าน reviewer จะ fail build บนแพลตฟอร์มใดก็ตามที่ dep ไม่มี

---

## 4. Build, Test, และ Install

**Prerequisites.** Rust 1.85 หรือใหม่กว่า (ตรวจสอบด้วย `rustup show`) ไม่ต้องมี system SQLite, OpenSSL หรือ libsodium — C deps ทั้งหมดใช้ feature `bundled` หรือ pure-Rust equivalents

```bash
# Clone
git clone https://github.com/bemindlabs/BWOC-Framework
cd BWOC-Framework

# Build ทุก crate
cargo build

# รัน tests ทั้งหมด (offline — ไม่ต้องใช้ network)
cargo test

# Build ใน release mode
cargo build --release

# Install bwoc CLI ในเครื่อง
cargo install --path crates/bwoc-cli

# Build harness พร้อม optional OpenTelemetry export
cargo build -p bwoc-harness --features otel

# Build crate เดียว
cargo build -p bwoc-core

# รัน tests สำหรับ crate เดียว
cargo test -p bwoc-signing

# ตรวจสอบ formatting (ต้องผ่านก่อน commit)
cargo fmt --check

# รัน clippy (ต้องผ่านก่อน commit)
cargo clippy -- -D warnings

# Shell completion (optional, ติดตั้งใน shell ของคุณ)
bwoc completion zsh > ~/.zfunc/_bwoc
bwoc completion bash > /etc/bash_completion.d/bwoc
```

**Release profile.** `Cargo.toml` ของ workspace ตั้งค่า `lto = "thin"`, `codegen-units = 1`, `strip = true` สำหรับ release builds ทำให้ได้ binary ขนาดเล็ก เร็ว เหมาะสำหรับ distribution

**Edition 2024.** ทุก crate ใช้ `edition = "2024"` หากเพิ่ม crate ใหม่ ให้ตั้ง `edition.workspace = true` ห้าม hardcode edition รุ่นเก่า

---

## 5. ด่าน CI

ทุก PR ต้องผ่าน CI checks สี่รายการที่จำเป็นทั้งหมดก่อนจึงจะ merge ได้ โดยรันบน matrix: macOS, Linux และ Windows

| Check | สิ่งที่รัน | ทำไมจึงจำเป็น |
|---|---|---|
| **fmt** | `cargo fmt --check` | style สม่ำเสมอ; ไม่มีการถกเถียงเรื่อง whitespace ใน review |
| **clippy** | `cargo clippy -- -D warnings` | ตรวจจับปัญหาความถูกต้องทั่วไปโดยไม่มี runtime cost |
| **build** | `cargo build` | ยืนยันว่า workspace compile ได้บนทั้งสามแพลตฟอร์ม |
| **test** | `cargo test` | regression guard; tests ทั้งหมด offline |

**Stale-branch protection.** strict branch-protection ของ GitHub กำหนดให้ PR branch ต้องมี commit ล่าสุดของ `main` ก่อน merge branch ที่ conflict-free แต่ stale ก็ยังผ่านด่านนี้ไม่ได้ แก้ด้วย:

```bash
gh pr update-branch <PR_NUMBER>
```

สิ่งนี้ rebase หรือ merge `main` เข้า branch ของคุณและ re-trigger CI อย่า force-push ไปที่ `main` เพื่อหลีกเลี่ยง

**Conversation resolution gate.** ทุก review comment thread ต้องถูก mark "Resolved" ก่อน PR จะ merge ได้ หาก reviewer เปิด thread สำหรับ security concern มันจะ block merge จนกว่าจะแก้ไขโดยเจตนา — ดู [กฎข้อ 8](#กฎข้อ-8-protected-main-และ-pr-discipline) สำหรับ security review checklist

---

## 6. กฎเหล็กเก้าข้อ

กฎเหล่านี้ไม่มีการต่อรอง แต่ละข้อมีเหตุผล การเข้าใจเหตุผลทำให้การปฏิบัติตามเป็นเรื่องธรรมชาติ

---

### กฎข้อ 1: Two-Tier Document Format

**อะไร.** ไฟล์ Markdown ทุกไฟล์ใน repository แบ่งเป็นสองระดับ:

| ระดับ | ไฟล์ | Format |
|---|---|---|
| Instructions (Tier 1) | `modules/agent-template/AGENTS.md` และ backend symlinks ทั้งหมด (`CLAUDE.md`, `AGY.md`, `CODEX.md`, `KIMI.md`, `OLLAMA.md`, `OPENAI.md`) | Plain Markdown เท่านั้น ห้าม YAML frontmatter, `[[wikilinks]]`, `> [!callout]` blocks หรือ vendor-specific rendering |
| Documentation (Tier 2) | ไฟล์ `.md` อื่นทั้งหมด | Obsidian Markdown: YAML frontmatter, `[[wikilinks]]`, callouts อนุญาต ประเภท callout ที่อนุมัติเท่านั้น: `abstract`, `tip`, `warning`, `example`, `note`, `danger` ห้ามประเภทอื่น |

**ทำไม.** `AGENTS.md` ถูกอ่านโดย LLM backend ทุกตัว — Claude, Kimi, Codex, Ollama local models และ backend ในอนาคต syntax ของ Obsidian (`> [!callout]`, `[[link]]`, YAML frontmatter) เป็น convention ของ Obsidian โดยเฉพาะที่ model หลายตัว parse ผิดหรือ render เป็น literal text ทำให้ agent ทำงานสับสน Plain Markdown คือ common denominator ต่ำสุดที่ทุก backend จัดการได้ถูกต้อง

**วิธีปฏิบัติตาม.** ก่อนแก้ไข `AGENTS.md` หรือ symlinks ยืนยันว่าอยู่ใน Tier 1 และลบ rich syntax ออก ก่อนแก้ไขหน้า spec หรือ handbook ยืนยันว่าอยู่ใน Tier 2 และใช้ YAML frontmatter กับ wikilinks ได้เลย เมื่อเพิ่ม callout type ใหม่ตรวจสอบ approved list ก่อน

**วิธีตรวจสอบ.** `bwoc check <agent-path>` ตรวจสอบ `AGENTS.md` สำหรับ YAML frontmatter, wikilinks และ unsupported callout syntax สำหรับ documentation files เปิดใน Obsidian หรือดู rendered Markdown บน GitHub

---

### กฎข้อ 2: Backend Neutrality

**อะไร.** `AGENTS.md` ต้องไม่มี model IDs, vendor names หรือ tool names แบบ hardcoded ค่าที่ configure ได้ทั้งหมดใช้ syntax placeholder `{{camelCase}}` (เช่น `{{agentId}}`, `{{primaryModel}}`, `{{fallbackModel}}`) การเพิ่ม backend ใหม่คือคำสั่งเดียว:

```bash
ln -s AGENTS.md <BACKEND>.md
```

ห้ามสร้าง per-backend content แยกต่างหาก

**ทำไม.** ความผูกพันพื้นฐานของ BWOC คือ Samānattatā — การปฏิบัติต่อทุก backend อย่างเท่าเทียม การ hardcode `gpt-4o` หรือ `claude-opus-4` ใน `AGENTS.md` ทำให้ model นั้นกลายเป็น privileged dependency เมื่อ model เปลี่ยนหรือ vendor ขึ้นราคา ทุก agent instance ที่อ้างถึงมันต้องอัปเดตด้วยมือ Placeholders แยก agent spec ออกจากการตัดสินใจของ vendor

**วิธีปฏิบัติตาม.** Grep `AGENTS.md` สำหรับ model IDs และ vendor names ที่รู้จักก่อน commit แทนที่ที่พบด้วย `{{placeholder}}` ที่เหมาะสม เมื่อ `config.manifest.json` resolve placeholder ไปเป็น real value ขณะ runtime นั่นคือ layer ที่ถูกต้องสำหรับ string แบบ hardcoded

**วิธีตรวจสอบ.** `bwoc check <agent-path>` scan สำหรับ model IDs และ vendor names แบบ hardcoded รัน `bwoc check --all` สำหรับ fleet-wide audit

---

### กฎข้อ 3: Bilingual Parity

**อะไร.** ทุกไฟล์ `docs/en/*.en.md` มี counterpart `docs/th/*.th.md` เมื่อแก้ไขอันหนึ่งต้องแก้ไขอีกอันใน PR เดียวกัน ใช้ทั้งกับ `docs/` ของ framework root และ `modules/agent-template/docs/`

ไฟล์ต่อไปนี้ **ยกเว้น** จาก TH parity โดยการออกแบบ:

| ไฟล์ | เหตุผล |
|---|---|
| `README.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, `LICENSE`, `VERSION.md` | Mechanical OSS docs; ต้นทุนการแปลสูงกว่าคุณค่าสำหรับ process docs ที่อัปเดตทุก PR |
| `crates/*/README.md` | Code-side convention; ใช้ TH pair เมื่อ framework target Thai developer audience อย่างชัดเจน |
| `notes/YYYY-MM-DD_*.md` | implementation logs ต่อ session; ephemeral |
| `.claude/` content | Operator-internal; ไม่ใช่ public-contributor surface |

Doctrine docs ที่ระบุทิศทางหรือหลักการของ project (เช่น `VISION.md`, `PHILOSOPHY.en.md`) ต้องมี TH pair เสมอ

**ทำไม.** ผู้เขียนหลักและส่วนสำคัญของฐานผู้ใช้ BWOC พูดภาษาไทย การรักษา doctrine docs สองภาษาให้ Thai-speaking audience ประเมิน BWOC ได้โดยไม่พึ่ง machine translation ของเอกสาร canonical การยกเว้น mechanical process docs รักษา maintenance burden ให้สมเหตุสมผล

**วิธีปฏิบัติตาม.** เมื่อเปิด PR ที่แตะ `docs/en/*.en.md` ใดๆ ให้อัปเดต `docs/th/*.th.md` ด้วยใน commit เดียวกัน PR checklist ใน [CONTRIBUTING.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/CONTRIBUTING.md) มี checkbox bilingual parity

**วิธีตรวจสอบ.** ตรวจสอบว่า `docs/en/` และ `docs/th/` มี base filenames ชุดเดียวกัน คำสั่งง่ายๆ `diff <(ls docs/en/ | sed 's/\.en\.md//') <(ls docs/th/ | sed 's/\.th\.md//')` ต้องไม่มี output

---

### กฎข้อ 4: Repo-as-Workspace Inversion

**อะไร.** Framework repo มี `.bwoc/`, `agents/` และ `projects/` ใน `.gitignore` User workspaces ที่สร้างด้วย `bwoc init` ตรงกันข้าม: track `agents/` และ `projects/` และ ignore เฉพาะ ephemeral runtime files

**ทำไม.** Framework repo ใช้ `.bwoc/workspace.toml` ของตัวเองสำหรับ CLI testing แบบ end-to-end แต่ test agents เหล่านั้นและ `.bwoc/` runtime state ต้องไม่ commit เข้า framework repo พวกมันคือ test fixtures ไม่ใช่ source artifacts

**วิธีปฏิบัติตาม.** ห้าม commit `.bwoc/`, `agents/` หรือ `projects/` directories ลงใน framework repo หากพบใน `git status` ตรวจสอบว่า `.gitignore` ถูกต้อง

**วิธีตรวจสอบ.** `git check-ignore -v .bwoc agents projects` ต้องแสดง rule จาก `.gitignore` สำหรับแต่ละอัน ความแตกต่างกับ user workspaces อยู่ใน `crates/bwoc-cli/src/init.rs::GITIGNORE_TEMPLATE`

---

### กฎข้อ 5: Auto-Version Hook (การเปลี่ยนแปลง Working-Tree ที่คาดหวัง)

**อะไร.** hook ที่ `.claude/hooks/auto-version.sh` bump patch component ของ `[workspace.package].version` ใน `Cargo.toml` อัตโนมัติบนทุก write ของ `.rs` หรือ `.toml` และ bump `Document-Version` ใน `VERSION.md` บนทุก write ของ `.md` หลัง edit session ใดๆ ไฟล์เหล่านี้จะปรากฏ modified ใน working tree ของคุณ

**ทำไม.** นี่ไม่ใช่ drift — มันตั้งใจ ทุก edit กลายเป็น versioned checkpoint ที่ไม่ซ้ำกัน `bwoc --version` ระหว่าง development รายงาน commit ที่แน่นอนที่สร้าง binary ทำให้ reproduce behavior ของ agent ได้แม่นยำ

**วิธีปฏิบัติตาม.** ห้าม revert auto-version bumps ห้ามต่อสู้กับ hook โดยตั้งค่า version เป็นค่าก่อน edit ด้วยมือ หากต้องการ MINOR หรือ MAJOR bump ให้แก้ไข `Cargo.toml` โดยตรงสำหรับ higher-order component; hook จัดการแค่ PATCH เท่านั้น

**วิธีตรวจสอบ.** หลัง edit ใดๆ `git diff Cargo.toml` ต้องแสดง patch increment `git diff VERSION.md` ต้องแสดง `Document-Version` increment (สำหรับ `.md` edits) และ `Last-Updated` timestamp ที่ refresh

---

### กฎข้อ 6: Implementation Logs

**อะไร.** ทุกการเปลี่ยนแปลงที่ "significant" ได้หนึ่ง note ต่อ session ใน `notes/YYYY-MM-DD_<title>.md` note เป็น development-oriented (อะไรเปลี่ยน, ทำไม, การตัดสินใจที่ทำ, ทางเลือกที่พิจารณา, bugs ที่พบ) — แยกจาก `CHANGELOG.md` ซึ่งเป็น release-oriented

การเปลี่ยนแปลงถือเป็น "significant" หากเข้าข่ายข้อใดข้อหนึ่ง:
- เพิ่มหรือลบ documentation file ใหม่ (ไม่ใช่ typo fix)
- เพิ่ม code module, crate, hook, skill, workflow หรือ script ใหม่
- การตัดสินใจที่กระทบ contributors ในอนาคต (architecture, naming, versioning, policy)
- bug fix ที่เปลี่ยน observable behavior
- สิ่งใดก็ตามที่ warranted `CHANGELOG.md` entry เกินกว่าหนึ่งบรรทัด

**โครงร่าง note** (กรอกเฉพาะ sections ที่ใช้):

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

**ทำไม.** `CHANGELOG.md` บันทึกสิ่งที่ shipped Notes บันทึกว่าทำไมจึงตัดสินใจเช่นนั้น — ข้อมูลที่แทนที่ไม่ได้หลายเดือนต่อมา เมื่อ contributor ถามว่า "ทำไม bwoc-core ถึงไม่มี HTTP?" หรือ "ทำไม gitignore ถึงกลับทิศ?"

**วิธีปฏิบัติตาม.** เขียน note หนึ่ง note ต่อ session ไม่ใช่ต่อไฟล์ session ที่สร้างสิบไฟล์ได้ note เดียวครอบคลุมทั้งหมด maintenance ประจำวัน (formatting fixes, minor doc tweaks) ไม่ต้องมี note ใช้ `bwoc notes new "<title>"` เพื่อ scaffold ไฟล์ด้วย date prefix ที่ถูกต้อง

**วิธีตรวจสอบ.** หลัง PR ที่ significant ยืนยันว่า note file มีอยู่ใน `notes/` ด้วยวันที่วันนี้ ตรวจสอบ PR diff — หากสิบไฟล์ใหม่ปรากฏแต่ไม่มี note ขอให้ author เพิ่มก่อน merge

---

### กฎข้อ 7: Conventional Commits

**อะไร.** Commit messages ใช้ format [Conventional Commits](https://www.conventionalcommits.org/) แบบ lightweight:

```
<type>(<scope>): <short summary>

[optional body — อธิบาย *ทำไม* ไม่ใช่ *อะไร*]
[optional footer — Refs/Closes #issue]
```

Types ที่ valid: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `style`, `ci`

**ตัวอย่าง:**
```
feat(bwoc-harness): เพิ่ม P3 task queue พร้อม cancellation token
fix(bwoc-signing): แก้ canonical JSON key ordering สำหรับ nested objects
docs(harness): เพิ่มส่วน tool-auth ใน HARNESS.en.md และ HARNESS.th.md
chore(deps): bump reqwest เป็น 0.12.5
```

**ทำไม.** Conventional Commits ทำให้การสร้าง `CHANGELOG.md` อัตโนมัติได้ ทำให้ PR titles parse ได้ด้วย tooling และสื่อสาร intent ได้ทันที

**วิธีปฏิบัติตาม.** Squash commits ก่อน merge (auto-merge command ใช้ `--squash`) squash commit message กลายเป็น PR title — ทำ PR title ให้เป็น valid conventional commit message

**วิธีตรวจสอบ.** อ่าน commit log หลัง merge: `git log --oneline` ทุก entry ต้องตามรูปแบบ PRs ที่มีชื่อเช่น "fix stuff" หรือ "wip" ควรเปลี่ยนชื่อก่อน merge

---

### กฎข้อ 8: Protected Main และ PR Discipline

**อะไร.** `main` เป็น protected branch ในทุก BWOC repo รวมถึง forks การเปลี่ยนแปลง land ผ่าน PR เท่านั้น PR mergeable ได้ก็ต่อเมื่อ **ทุกข้อ** ต่อไปนี้ hold พร้อมกัน:

1. CI checks สี่รายการผ่านทั้งหมด (fmt, clippy, build, test) บน macOS, Linux และ Windows
2. branch up to date กับ `main` (ใช้ `gh pr update-branch <PR>` หาก stale)
3. ทุก review conversation resolved (ไม่มี thread เปิดค้างอยู่)

คำสั่ง merge ที่อนุญาตเพียงอย่างเดียวคือ:

```bash
gh pr merge <PR_NUMBER> --squash --auto
```

สิ่งนี้ schedule ให้ GitHub merge ทันทีที่ gates clear ห้ามใช้ `--squash` หรือ `--merge` โดยไม่มี `--auto` ห้าม push ไปยัง `main` โดยตรงไม่ว่ากรณีใดรวมถึง automation

**หนึ่ง concern ต่อ PR.** การเปลี่ยนแปลงที่ไม่เกี่ยวข้องกันหลายอย่างต้องไม่รวมกันใน PR เดียว แยกตาม feature Reviewers ไม่สามารถ review diff 2,000 บรรทัดที่แตะสามระบบย่อยที่ไม่เกี่ยวข้องกันได้อย่างมีความหมาย

**Cross-repo (fork → upstream) full security review.** PRs ที่มาจาก forks นำ external code เข้าสู่ security framework ก่อน merge reviewers ต้องตรวจสอบ:

- **Injection / path-traversal / RCE.** code ใดๆ ที่ execute plugins, skills หรือ scripts หรือ parse untrusted input ถูกอ่านหา injection vectors
- **Dep-quarantine integrity.** ยืนยัน `bwoc-core` ไม่ได้รับ deps ใหม่ ยืนยัน async/HTTP dep ไม่ปรากฏใน `bwoc-signing`
- **Backend neutrality.** ยืนยัน `AGENTS.md` ไม่มี model IDs แบบ hardcoded
- **EN/TH bilingual parity.** ยืนยัน `docs/en/*.en.md` ที่ถูกแตะมีการอัปเดต `docs/th/*.th.md` ที่ตรงกัน

Security หรือ correctness blockers: request changes (conversation-resolution gate ทำให้ PR blocked) แทนที่จะ merge "ค่อยแก้ทีหลัง"

**ทำไม.** `main` คือ release branch การ push โดยตรงไปยัง `main` ที่ทำลาย platform ข้าม CI และ ship binary ที่เสียหาย PR gate คือ contract เดียวที่เชื่อถือได้ระหว่าง "code compile บนเครื่อง author" กับ "code compile บนสามแพลตฟอร์มสำหรับทุกผู้ใช้"

**วิธีตรวจสอบ.** `gh pr status` แสดงสถานะ gate `gh pr checks <PR>` แสดง breakdown ต่อ check branch ที่ stale แสดง "Branch is out of date" — แก้ด้วย `gh pr update-branch`

---

### กฎข้อ 9: Over-Engineering Protection

**อะไร.** Default ไปที่ NOT adding ห้ามเพิ่มไฟล์, hooks, skills, workflows หรือ specs โดย proactive นอกจากจะมีการร้องขอชัดเจนใน session ปัจจุบัน ห้าม chain ahead — ทำสิ่งที่ขอให้เสร็จ, บอกสิ่งถัดไป, รอทิศทาง เลือก trim มากกว่า expand เมื่อทั้งสองสมเหตุสมผล

`VISION.md` ของ framework เองระบุหลักการนี้ชัดเจน: "the smaller specification beats the more complete one"

สัญญาณที่คุณอาจ over-engineer:
- จำนวนไฟล์เพิ่มขึ้นมากกว่าหนึ่งต่อ turn โดยไม่มีการร้องขอชัดเจน
- doc ใหม่ถูกสร้างที่ duplicate content ที่มีอยู่แล้วใน doc อื่น
- hook หรือ workflow ถูกเพิ่มที่ไม่มี consumer ปัจจุบัน
- skill หรือ plugin ถูก scaffold "just in case"

**ทำไม.** Mattaññutā (ความพอดี) ไม่ใช่แค่คำขวัญ — มันคือ engineering discipline ทุกไฟล์ที่เพิ่มลงใน repository คือไฟล์ที่ต้อง maintain, รักษา bilingual parity และต้องเข้าใจโดย contributors ในอนาคต ต้นทุนของไฟล์ที่ไม่จำเป็น compound ขึ้นเรื่อยๆ

**วิธีปฏิบัติตาม.** เมื่อล่อแหลมจะเพิ่มบางอย่าง "ขณะที่อยู่ตรงนั้นแล้ว" ให้เขียน note เกี่ยวกับ idea แทน (entry ใน `notes/` หรือ PR comment) และให้ทีมตัดสินใจว่ามันควรได้ที่ใน future session

**วิธีตรวจสอบ.** ก่อนเปิด PR นับไฟล์ที่เพิ่มเทียบกับไฟล์ที่การร้องขอระบุชัดเจน หาก ratio มากกว่า 2:1 ให้พิจารณาใหม่

---

## 7. Hook Auto-Version

hook อยู่ที่ `.claude/hooks/auto-version.sh` ใน framework repo ถูก trigger โดย Claude Code หลัง file write ใดๆ

**พฤติกรรม:**

- บน write ของ `.rs` หรือ `.toml` ใดๆ: อ่าน `[workspace.package].version` จาก `Cargo.toml`, increment PATCH component, เขียนกลับไปยัง `Cargo.toml` และ mirror ค่าใหม่ไปยัง `Software-Version` line ของ `VERSION.md`
- บน write ของ `.md` ใดๆ: อ่าน `Document-Version` จาก `VERSION.md`, increment PATCH component, เขียนกลับ และ refresh `Last-Updated` timestamp เป็น UTC ISO 8601 ปัจจุบัน

**Version namespaces:**

| Namespace | Scheme | Canonical location | Role |
|---|---|---|---|
| Cargo SemVer | `MAJOR.MINOR.PATCH` | `Cargo.toml [workspace.package].version` | Development checkpoint; auto-bump บนทุก edit |
| Release CalVer | `vYYYY.M.D-<patch>` | Git tag, GitHub Release name | Public release identity; cut ด้วยมือโดย maintainers |

สอง namespaces นี้เป็นอิสระจากกันโดยตั้งใจ `bwoc --version` ระหว่าง development รายงาน Cargo SemVer release notes และ download links ใช้ CalVer tags

**กฎ manual bump:**

| Bump type | Action |
|---|---|
| PATCH (auto) | Hook จัดการ; ไม่ต้องทำอะไร |
| MINOR (capability ใหม่แบบ backward-compatible) | แก้ไข `[workspace.package].version` โดยตรงใน `Cargo.toml` |
| MAJOR (breaking change ต่อ spec, manifest schema หรือ CLI surface) | แก้ไข `[workspace.package].version` โดยตรงใน `Cargo.toml`; อัปเดต `CHANGELOG.md` |
| Document MINOR/MAJOR | แก้ไข `Document-Version` ใน `VERSION.md` โดยตรง |

---

## 8. เอกสารและ Workflow สองภาษา

เอกสาร framework อยู่ใน `docs/en/` และ `docs/th/` รายการ spec documents canonical (แต่ละฉบับมี counterpart `.en.md` และ `.th.md`):

| เอกสาร | จุดประสงค์ |
|---|---|
| [ARCHITECTURE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/ARCHITECTURE.en.md) | System architecture, crate relationships, data flow |
| [DESIGN.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/DESIGN.en.md) | Design rationale และ tradeoffs |
| [HARNESS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/HARNESS.en.md) | `bwoc-harness` agentic loop, tools, task queue, tool-auth |
| [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) | Skill system: format, maturity levels, gates |
| [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) | Plugin system: format, workspace registration |
| [RELEASING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/RELEASING.en.md) | Release process: CalVer tags, CI pipeline, Homebrew formula |
| [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) | ed25519 identity proof, key rotation |
| [WORKSPACE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/WORKSPACE.en.md) | Workspace layout, `workspace.toml`, routes.toml |
| [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md) | Naming conventions สำหรับไฟล์, branches, notes, agents |
| [FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) | Aparihāniya-dhamma 7 fleet-health signals |
| [GLOSSARY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) | คำจำกัดความ |
| [FAQ.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) | คำถามที่พบบ่อย |
| [INCARNATION.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/INCARNATION.en.md) | Agent incarnation walkthrough |

**Editing workflow สำหรับ doc pair:**

```bash
# 1. แก้ไขไฟล์ EN
$EDITOR docs/en/HARNESS.en.md

# 2. แก้ไข TH counterpart ใน session เดียวกัน
$EDITOR docs/th/HARNESS.th.md

# 3. Commit ทั้งสองพร้อมกัน
git add docs/en/HARNESS.en.md docs/th/HARNESS.th.md
git commit -m "docs(harness): document tool-auth credential lifecycle"
```

ห้าม commit เฉพาะไฟล์ EN โดยไม่มี TH pair (เว้นแต่ไฟล์นั้นได้รับการยกเว้นชัดเจนตามกฎข้อ 3)

**Frontmatter requirements** สำหรับ Tier 2 docs:

```yaml
---
title: "ชื่อเอกสาร"
version: "X.Y.Z"
audience: "กลุ่มผู้อ่านเป้าหมาย"
language: th
counterpart: FILENAME.en.md
---
```

---

## 9. การพัฒนา Skill

Skills คือ agent capabilities ที่ใช้ซ้ำได้ อยู่ใน `modules/skills/<name>/` และประกาศใน `config.manifest.json` ตาม format ที่กำหนดใน [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) และมี `maturity` level (`L1` ถึง `L7`) ใน frontmatter

### 9.1 Skill CLI Reference

```bash
# Scaffold skill ใหม่จาก skill template
bwoc skill init <name>

# Install skill จาก local path, git URL หรือ tarball URL
# SHA-256 trust gate บังคับใช้ — install จะล้มเหลวหาก hash ไม่ตรงกัน
bwoc skill install <source>

# List installed skills พร้อม maturity และ enabled state
bwoc skill list

# แสดง manifest และ SPEC location ของ skill หนึ่งอย่างละเอียด
bwoc skill show <name>

# Enable skill บน agent ปัจจุบัน (ตั้ง enabled = true ใน manifest)
bwoc skill enable <name>

# Disable skill (เก็บ entry ไว้; ตั้ง enabled = false)
bwoc skill disable <name>

# Remove skill: ลบ modules/skills/<name>/ และ clean
# manifest entry ของ agent ทุกตัวที่ใช้มัน
bwoc skill remove <name>

# ตรวจสอบ skill แบบ static: พิมพ์แต่ละคำสั่ง [gates].verify
# โดยไม่รัน exit non-zero หาก gate ใดขาดหายไป
bwoc skill verify <name>

# ตรวจสอบ AND execute gates
# (gates มาจาก manifest ที่ไม่น่าเชื่อถือ —
# อ่านและเข้าใจคำสั่ง gate ก่อนรันเสมอ)
bwoc skill verify <name> --run-gates
```

### 9.2 Conventions การพัฒนา Skill

**Frontmatter.** ไฟล์ `.md` ของ skill ใช้ Obsidian frontmatter แบบเต็มพร้อม tag `domain/<area>` และ field `maturity` (`L1` = stub, `L7` = production-hardened) นี่คือ Tier 2 format — wikilinks และ callouts อนุญาต

**Trust gate.** `bwoc skill install` compute SHA-256 hash ของ skill archive และเปรียบเทียบกับ hash ที่ประกาศใน manifest หาก hash ไม่ตรงกัน installation ล้มเหลว ป้องกัน supply-chain substitution ของ skill archives `verify` subcommand พิมพ์คำสั่ง `[gates].verify` โดยไม่รัน อ่านก่อน จากนั้นส่ง `--run-gates` หากเชื่อถือ source

**ความสัมพันธ์กับ agents.** Skills ถูก install ที่ workspace level และ enable ต่อ agent `config.manifest.json` ของ agent แสดงว่า skills ใด enabled `bwoc skill enable` แก้ไข manifest; `bwoc skill disable` ตั้ง `enabled = false` โดยไม่ลบ entry

**Maturity levels.** skill ที่ `L1` (stub) อาจ check in เพื่อประกาศ intent; ไม่คาดว่าจะ production-safe PRs ที่เพิ่ม `L7` skill ต้องมีหลักฐาน gate verification ห้าม ship `L1` skills เป็น enabled defaults ใน template

---

## 10. การพัฒนา Plugin

Plugins ขยาย workspace เอง (ไม่ใช่ individual agents) อยู่ใน `modules/plugins/<name>/` ประกาศใน `workspace.toml` ใต้ `[plugins.<name>]` Plugin format กำหนดใน [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md)

### 10.1 Plugin CLI Reference

```bash
# Scaffold plugin ใหม่จาก plugin template
bwoc plugin init <name>

# Install plugin จาก local path, git URL หรือ tarball URL
# SHA-256 trust gate บังคับใช้ (mechanism เดียวกับ skill install)
bwoc plugin install <source>

# List installed plugins พร้อม enabled state และ workspace registration
bwoc plugin list

# แสดง manifest, SPEC location และ workspace registration ของ plugin หนึ่ง
bwoc plugin show <name>

# Enable plugin ใน workspace.toml (ตั้ง [plugins.<name>] enabled = true)
bwoc plugin enable <name>

# Disable (เก็บ entry ไว้; ตั้ง enabled = false)
bwoc plugin disable <name>

# Remove: ลบ modules/plugins/<name>/ และลบ
# [plugins.<name>] table ออกจาก workspace.toml
bwoc plugin remove <name>
```

### 10.2 Plugin เทียบกับ Skill

| มิติ | Skill | Plugin |
|---|---|---|
| Scope | Per-agent capability | Workspace-level extension |
| Enable ใน | `config.manifest.json` ต่อ agent | `workspace.toml` สำหรับ agents ทั้งหมด |
| `verify` subcommand | ใช่ (SHA-256 gate + optional `--run-gates`) | ไม่มี `verify` ใน v1 |
| Template | `modules/skill-template/` | `modules/plugin-template/` |
| Location | `modules/skills/<name>/` | `modules/plugins/<name>/` |

หมายเหตุ: plugins ไม่มี `verify` subcommand ใน v1 trust gate ยังคง apply ตอน install time (SHA-256 check) แต่ไม่มีคำสั่ง gate execution หลัง install ดู [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) สำหรับ specification เต็ม

---

## 11. ขั้นตอนการ Contribute

หัวข้อนี้อธิบายทุก gate ที่ PR ต้องผ่าน ทำตามขั้นตอนตามลำดับ

### 11.1 Fork และสร้าง Branch

```bash
# Fork บน GitHub แล้ว clone fork ของคุณ
git clone https://github.com/<your-github-handle>/BWOC-Framework
cd BWOC-Framework
git remote add upstream https://github.com/bemindlabs/BWOC-Framework

# สร้าง topic branch — type ใช้ vocabulary เดียวกับ commit types
git checkout -b feat/<short-name>     # capability ใหม่
git checkout -b fix/<short-name>      # bug fix
git checkout -b docs/<short-name>     # documentation
git checkout -b refactor/<short-name> # internal restructure
git checkout -b chore/<short-name>    # dependency bump, CI config
```

ไม่มี `release/*` หรือ `hotfix/*` branches version tags cut โดยตรงบน `main` ชื่อ branch format คือ `type/<short-name>` — vocabulary เดียวกับ commit types

### 11.2 ทำการเปลี่ยนแปลง

- รักษา diff ให้ focused หนึ่ง concern ต่อ PR หากพบปัญหาที่ไม่เกี่ยวข้อง แก้ใน branch แยก
- หลัง edit ทุก `.rs` หรือ `.toml` hook auto-version bump patch ห้าม revert
- หลัง edit ทุก `.md` `Document-Version` ใน `VERSION.md` ถูก bump ห้าม revert
- สำหรับการเปลี่ยนแปลงที่ significant เขียน `notes/YYYY-MM-DD_<title>.md` หนึ่ง entry ครอบคลุม session

### 11.3 ตรวจสอบในเครื่อง

รันสิ่งเหล่านี้ก่อน push — CI จะ catch แต่ catch ในเครื่องเร็วกว่า:

```bash
# ตรวจสอบ format
cargo fmt --check

# Lints (warnings treated as errors)
cargo clippy -- -D warnings

# Full build บนแพลตฟอร์มของคุณ
cargo build

# tests ทั้งหมด (offline, ไม่ต้องใช้ network)
cargo test

# Backend neutrality audit (หากแตะ agent หรือ AGENTS.md ใดๆ)
bwoc check --all
```

### 11.4 Commit

```bash
git add -A
git commit -m "feat(bwoc-harness): add P3 task queue with CancellationToken"
```

ใช้ Conventional Commits format commit message จะกลายเป็น squash-merge commit บน `main`

### 11.5 เปิด Pull Request

```bash
# Push branch ของคุณ
git push origin feat/<short-name>

# เปิด PR กับ upstream main
gh pr create \
  --repo bemindlabs/BWOC-Framework \
  --base main \
  --title "feat(bwoc-harness): add P3 task queue with CancellationToken" \
  --body "## อะไรและทำไม
<อธิบายการเปลี่ยนแปลงและแรงจูงใจ>

## Checklist
- [ ] Branch up to date กับ main
- [ ] cargo fmt --check ผ่าน
- [ ] cargo clippy -- -D warnings ผ่าน
- [ ] cargo test ผ่านบนแพลตฟอร์มของฉัน
- [ ] Documentation อัปเดตแล้ว (EN + TH ถ้าเกี่ยวข้อง)
- [ ] bwoc check --all ผ่าน (ถ้าแตะ agents)
- [ ] ไม่มี secrets, credentials หรือ personal paths ใน diff
- [ ] การเปลี่ยนแปลงที่ significant → เพิ่ม notes/ entry แล้ว

Closes #<issue>"
```

### 11.6 จัดการ Review Comments

หาก reviewer เปิด thread:
- แก้ไข concern ใน commit ใหม่บน branch ของคุณ (ห้าม amend commits ที่ push แล้วนอกจากจะถูกขอชัดเจน)
- Mark conversation resolved หลังจาก fix ของคุณ push แล้ว
- หากไม่เห็นด้วย อธิบายใน thread ห้ามปล่อยทิ้งไว้เปิดค้าง

conversation-resolution gate เด็ดขาด thread ที่ยังเปิดค้าง block merge โดยไม่คำนึงถึงสถานะ CI

### 11.7 รักษา Branch ให้ Up to Date

หาก `main` เดินหน้าขณะที่ PR ของคุณยังเปิดอยู่:

```bash
gh pr update-branch <PR_NUMBER>
```

สิ่งนี้ merge `main` เข้า branch ของคุณ (หรือ rebase ตาม repo settings) และ re-trigger CI ห้าม force-push เพื่อหลีกเลี่ยง stale branch — protection เด็ดขาด

หรือ rebase ด้วยมือ:

```bash
git fetch upstream
git rebase upstream/main
git push --force-with-lease origin feat/<short-name>
```

### 11.8 Enable Auto-Merge

เมื่อเชื่อว่า PR พร้อมแล้ว (CI green, threads resolved ทั้งหมด, branch up to date):

```bash
gh pr merge <PR_NUMBER> --squash --auto
```

GitHub จะ merge ทันทีที่ gates clear ห้าม merge ด้วยมือด้วย `--squash` หรือ `--merge` โดยไม่มี `--auto`

### 11.9 หลัง Merge

- ลบ branch: `git push origin --delete feat/<short-name>`
- อัปเดต local main: `git fetch upstream && git rebase upstream/main`
- หาก PR significant ยืนยันว่า `notes/` entry มีอยู่

---

## 12. การ Release

Releases ใช้ CalVer tags cut โดยตรงบน `main` CI pipeline (`release.yml`) build cross-platform binaries และ upload เป็น GitHub Release assets พร้อม SHA-256 sidecars กระบวนการเต็มอยู่ใน [RELEASING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/RELEASING.en.md)

**Recipe ด่วนสำหรับ maintainers:**

```bash
# กำหนด CalVer tag สำหรับ release iteration วันนี้
# Format: vYYYY.M.D-<patch> โดย patch เริ่มที่ 0
git tag v2026.7.20-0

# Push tag — release.yml รับช่วงต่อจากนี้
git push origin v2026.7.20-0
```

`release.yml` build matrix (Linux x86_64, macOS aarch64 + x86_64, Windows x86_64), package แต่ละอันเป็น `<archive>.{tar.gz,zip}` พร้อม `.sha256` sidecar และ upload ไปยัง GitHub Release ที่สร้างอัตโนมัติ

Release ซ้ำวันเดียวกัน: `v2026.6.6-1`, `v2026.6.6-2`, เป็นต้น

การอัปเดต Homebrew formula คือ PR auto-merge แยกที่ release pipeline เปิด — ไม่ push โดยตรงไปยัง `main` เลย

---

## 13. ลำดับถัดไปที่ควรอ่าน

| คำถาม | ดูที่ |
|---|---|
| Agentic loop ทำงานอย่างไร? | [HARNESS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/HARNESS.en.md) |
| Agents เซ็นและตรวจสอบ messages อย่างไร? | [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) |
| A2A interop ทำงานอย่างไร? | [ARCHITECTURE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/ARCHITECTURE.en.md) |
| ฉันจะเขียน skill ได้อย่างไร? | [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) |
| ฉันจะเขียน plugin ได้อย่างไร? | [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) |
| ฉันจะ cut release ได้อย่างไร? | [RELEASING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/RELEASING.en.md) |
| Naming conventions ทั้งหมดหมายความว่าอะไร? | [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md) |
| Fleet health check คืออะไร? | [FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) |
| คำนี้หมายความว่าอะไร? | [GLOSSARY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) · [../glossary.en.md](../glossary.en.md) |
| ฉันต้องการสร้าง agent ไม่ใช่ build framework | [../agents/HANDBOOK.th.md](../agents/HANDBOOK.th.md) |
| PR checklist เต็ม | [CONTRIBUTING.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/CONTRIBUTING.md) |
| version และ phase ปัจจุบัน | [VERSION.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) |

## เผยแพร่เว็บไซต์นี้ (GitHub Pages)

handbook ชุดนี้มาพร้อมไฟล์ตั้งค่า MkDocs + Material for MkDocs (`mkdocs.yml`) และ GitHub Actions workflow (`.github/workflows/pages.yml`) ให้เรียบร้อย

วิธีเผยแพร่:

1. push handbook ขึ้น GitHub repo
2. ไปที่ **Settings → Pages** ของ repo แล้วตั้งค่า source เป็น **GitHub Actions**
3. workflow จะ build และ deploy เว็บไซต์ทุกครั้งที่ push เข้า `main` (และสั่งเองได้ผ่าน `workflow_dispatch`)

พรีวิวบนเครื่อง:

```bash
pip install -r requirements.txt
mkdocs serve
```
