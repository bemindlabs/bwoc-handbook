# BWOC Handbook — Sitemap

This file is the human-readable and machine-parseable corpus inventory for the BWOC Handbook.
It is intended for use by web crawlers, documentation indexers, and RAG ingestion pipelines as a crawl-queue seed list.

Paths are **site-root-relative** (e.g. `/end-user/HANDBOOK.en.md`).
Operators: substitute your deployed host to produce absolute URLs.

For allow/deny crawl rules see [`robots.txt`](robots.txt).
For AI/RAG ingestion hints see [`ai-search/HANDBOOK.en.md`](ai-search/HANDBOOK.en.md).
For the full freshness-signal spec see [`crawler/HANDBOOK.en.md`](crawler/HANDBOOK.en.md).

---

## Handbook corpus — by role

### Entry points

| Path | Language | Canonical? | One-line description |
|---|---|---|---|
| `/README.md` | English | Yes | Role-routing entry point for all readers; introduces the five handbook roles, the lifecycle model, and fast-path commands. |
| `/README.th.md` | Thai | Parity of `/README.md` | Thai-language entry point; identical structure to the English version; maintained in parity. |

**Summary — `/README.md`:** The single entry point for the entire handbook. Presents a routing table (end-user, developer, agent author, AI-search agent, crawler), explains the BWOC framework in five sentences, describes the three-phase lifecycle (birth / live / retire), and states the cross-cutting conventions (bilingual-English-primary, no local path links, version pinning). High authority; changes infrequently.

**Summary — `/README.th.md`:** Exact Thai-language counterpart of `README.md`. Routing table and all sections are kept in parity. Index as `hreflang="th"` with canonical `README.md`.

---

### End-user handbook

| Path | Language | Canonical? | One-line description |
|---|---|---|---|
| `/end-user/HANDBOOK.en.md` | English | Yes | Install, workspace setup, and all CLI interactions for users who run agents without touching framework code. |
| `/end-user/HANDBOOK.th.md` | Thai | Parity | Thai-language parity of the end-user handbook. |

**Summary — `/end-user/HANDBOOK.en.md`:** Covers the complete end-user journey: Rust/CLI installation and `bwoc doctor`, workspace creation, four ways to drive an agent (`chat`, `run`, `spawn`, `send`), backend overrides, inbox/log/memory/health commands, team (Sangha) creation and shared task-list management, the agent lifecycle commands (`new`, `stop`, `start`, `retire`, `check`), fleet overview (`dashboard`, `sessions`, `fleet`, `peer`), document scaffolding (`notes`, `retro`, `research`), update and shell completion, and a troubleshooting table. Cross-references the agents and developer handbooks.

**Summary — `/end-user/HANDBOOK.th.md`:** Thai-language parity of `/end-user/HANDBOOK.en.md`. All ten sections are maintained in parity. Index as `hreflang="th"` with canonical `/end-user/HANDBOOK.en.md`.

---

### Developer handbook

| Path | Language | Canonical? | One-line description |
|---|---|---|---|
| `/developer/HANDBOOK.en.md` | English | Yes | Crate map, build/test procedures, hooks, versioning policy, release recipe, and PR gates for framework contributors. |
| `/developer/HANDBOOK.th.md` | Thai | Parity | Thai-language parity of the developer handbook. |

**Summary — `/developer/HANDBOOK.en.md`:** Guides contributors who work on the Rust framework itself. Covers the three-crate workspace (`bwoc-core`, `bwoc-cli`, `bwoc-agent`, `bwoc-deep-memory`), build and test commands, the auto-version hook that stamps `VERSION.md` on every edit, the dual versioning namespace (Cargo SemVer for development checkpoints; CalVer tags for public releases), the protected-branch PR discipline (four CI gates, auto-merge, one-concern-per-PR), bilingual parity rules, and implementation note conventions in `notes/`.

**Summary — `/developer/HANDBOOK.th.md`:** Thai-language parity of `/developer/HANDBOOK.en.md`. Index as `hreflang="th"` with canonical `/developer/HANDBOOK.en.md`.

---

### Agent author / operator handbook

| Path | Language | Canonical? | One-line description |
|---|---|---|---|
| `/agents/HANDBOOK.en.md` | English | Yes | Agent directory layout, the AGENTS.md rule, slot system, manifest schema, backend-neutrality validation, and the full agent arc. |
| `/agents/HANDBOOK.th.md` | Thai | Parity | Thai-language parity of the agent handbook. |

**Summary — `/agents/HANDBOOK.en.md`:** The definitive guide for incarnating, tuning, and operating BWOC agents. Explains the `agents/agent-<name>/` directory structure, the five slot directories (`persona/`, `mindsets/`, `skills/`, `interconnect/`, `memories/`), the AGENTS.md symlink pattern (all backends link to one file), `config.manifest.json` schema, the `{{camelCase}}` placeholder rule for backend neutrality, `bwoc check` audit, the `MEMORY.md` 200-line discipline, team membership via `.bwoc/teams/<team>.toml`, and the lifecycle CLI commands. Also covers Obsidian-format slot files vs. plain-Markdown instruction files.

**Summary — `/agents/HANDBOOK.th.md`:** Thai-language parity of `/agents/HANDBOOK.en.md`. Index as `hreflang="th"` with canonical `/agents/HANDBOOK.en.md`.

---

### AI search / RAG handbook

| Path | Language | Canonical? | One-line description |
|---|---|---|---|
| `/ai-search/HANDBOOK.en.md` | English | Yes | Chunking hints, citation format, authoritative-source hierarchy, and hallucination warnings for AI/RAG pipelines ingesting this corpus. |
| `/ai-search/HANDBOOK.th.md` | Thai | Parity | Thai-language parity of the AI-search handbook. |

**Summary — `/ai-search/HANDBOOK.en.md`:** Written for LLM-based retrieval agents and RAG pipelines rather than traditional web spiders. Provides a curated fetch order optimized for LLM context windows, a canonical source-of-truth hierarchy (framework repo beats handbook), specific hallucination warnings (e.g., do not invent CLI flags; always cite `VERSION.md` for version claims), recommended citation format, and guidance on which pages to weight most heavily for question-answering tasks.

**Summary — `/ai-search/HANDBOOK.th.md`:** Thai-language parity of `/ai-search/HANDBOOK.en.md`. Index as `hreflang="th"` with canonical `/ai-search/HANDBOOK.en.md`.

---

### Crawler / indexer handbook

| Path | Language | Canonical? | One-line description |
|---|---|---|---|
| `/crawler/HANDBOOK.en.md` | English | Yes | Crawl scope table, bilingual canonical structure, freshness signals, polite-crawl guidelines, and pointers to `robots.txt` and `sitemap.md`. |
| `/crawler/HANDBOOK.th.md` | Thai | Parity | Thai-language parity of the crawler handbook. |

**Summary — `/crawler/HANDBOOK.en.md`:** Authoritative crawl policy document. Contains an exhaustive two-column index-vs-skip table covering every path pattern in both the handbook corpus and the framework repo, the bilingual hreflang declaration scheme, the freshness-signal spec (where `Software-Version`, `Document-Version`, and `Last-Updated` live and how the auto-version hook maintains them), recommended re-crawl cadences, polite-crawl requirements (honor `robots.txt`, `Crawl-delay: 10`, conditional GET, User-agent identification), and the linking policy (public GitHub URLs for framework source; relative paths within the handbook).

**Summary — `/crawler/HANDBOOK.th.md`:** Thai-language parity of `/crawler/HANDBOOK.en.md`. Index as `hreflang="th"` with canonical `/crawler/HANDBOOK.en.md`.

---

### Ecosystem / projects handbook

| Path | Language | Canonical? | One-line description |
|---|---|---|---|
| `/ecosystem/HANDBOOK.en.md` | English | Yes | Maps every BWOC and related `bwoc-*` project — core framework, desktop/menu-bar apps, and device targets — with each project's stack and how it connects to the core. |
| `/ecosystem/HANDBOOK.th.md` | Thai | Parity | Thai-language parity of the ecosystem handbook. |

**Summary — `/ecosystem/HANDBOOK.en.md`:** A single page mapping the BWOC family: the core framework + `bwoc` CLI, bwoc-chat (egui desktop chat), bwoc-devices (portable Rust device core + ESP32 firmware), bwoc-devices-app / BWOC Monitor (Tauri macOS dashboard), bwoc-llm-pm / LLM Provider Monitor (macOS menu-bar), bwoc-mcc (SwiftUI menu-bar control center), and this handbook. Each entry gives the stack and how it connects to the CLI/shared protocol. Index as canonical; links to public GitHub repos only.

**Summary — `/ecosystem/HANDBOOK.th.md`:** Thai-language parity of `/ecosystem/HANDBOOK.en.md`. Index as `hreflang="th"` with canonical `/ecosystem/HANDBOOK.en.md`.

---

### Glossary

| Path | Language | Canonical? | One-line description |
|---|---|---|---|
| `/glossary.en.md` | English | Yes | Authoritative definitions of all specialized BWOC terms; linked from every handbook page on first use. |
| `/glossary.th.md` | Thai | Parity | Thai-language parity of the glossary. |

**Summary — `/glossary.en.md`:** The single reference for BWOC vocabulary. Defines terms from both the engineering layer (workspace, agent, slot, manifest, backend-neutrality, trust level, Sangha, harness) and the philosophy layer (uppāda, ṭhiti, vaya, yoniso manasikāra, mattaññutā, anattā, samānattatā, sīlasāmaññatā). Every definition includes a plain-English explanation and a cross-reference to the framework concept it encodes. Stable; changes rarely.

**Summary — `/glossary.th.md`:** Thai-language parity of `/glossary.en.md`. Term entries are in parity; all definitions are maintained together. Index as `hreflang="th"` with canonical `/glossary.en.md`.

---

### Ancillary files

| Path | Language | Canonical? | One-line description |
|---|---|---|---|
| `/llms.txt` | English | Yes | Plain-text LLM entry point; compact corpus summary optimized for LLM context-window ingestion. |
| `/sitemap.md` | English | Yes | This file — human+machine corpus map with descriptions and canonical framework doc links. |

**Summary — `/llms.txt`:** A compact, plain-text file following the `llms.txt` convention. Provides a structured summary of the entire handbook corpus in a format optimized for direct ingestion into LLM context windows. Lists the most authoritative pages in priority order, states the source-of-truth hierarchy, and provides the key framework URLs. AI pipelines should fetch this file first before deciding which detailed pages to retrieve.

**Summary — `/sitemap.md`:** This file. Provides a grouped, role-organized inventory of every indexable handbook page and every canonical framework doc, with site-root-relative paths, language tags, canonical flags, one-line descriptions, and one-paragraph summaries. Intended as the crawl-queue seed list for web indexers and as a quick navigation aid for human readers.

---

## Canonical framework docs

These files are hosted in the public `bemindlabs/BWOC-Framework` repository. They are the authoritative sources for framework behavior; the handbook summarizes and links to them. Index directly from GitHub.

> Freshness signal for all framework docs: check `Last-Updated` in
> `https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md`
> before re-fetching. Current: Software-Version `2.24.0`, Document-Version `1.6.2`, Last-Updated `2026-06-06T03:44:18Z`.

| URL | Language | One-line description |
|---|---|---|
| [`README.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/README.md) | English | Framework project overview; what BWOC is, the 22 Buddhist engineering frameworks, crate map, status table, and quick-start commands. |
| [`VISION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VISION.md) | English | Project direction, long-term goals, and the hard-tradeoff principles that govern framework design decisions. |
| [`VISION.th.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VISION.th.md) | Thai | Thai-language parity of `VISION.md`; `hreflang="th"`. |
| [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) | English | Freshness beacon: carries `Software-Version` (Cargo SemVer), `Document-Version`, `Last-Updated` (UTC ISO 8601), current phase, and versioning policy details. |
| [`modules/agent-template/AGENTS.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md) | English | The authoritative agent base profile spec (v2.0); single source of truth for all LLM backends; all backend files (`CLAUDE.md`, `AGY.md`, `CODEX.md`, `KIMI.md`) are symlinks to this file. |
| [`modules/agent-template/docs/en/OVERVIEW.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/OVERVIEW.en.md) | English | Conceptual overview of what a BWOC agent is, its slot structure, and how the template instantiation model works. |
| [`modules/agent-template/docs/th/OVERVIEW.th.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/th/OVERVIEW.th.md) | Thai | Thai-language parity of `OVERVIEW.en.md`; `hreflang="th"`. |
| [`modules/agent-template/docs/en/PHILOSOPHY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) | English | The 22 Buddhist engineering frameworks that ground every structural decision in the codebase; the conceptual core of BWOC. |
| [`modules/agent-template/docs/th/PHILOSOPHY.th.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/th/PHILOSOPHY.th.md) | Thai | Thai-language parity of `PHILOSOPHY.en.md`; `hreflang="th"`. |
| [`modules/agent-template/docs/en/PRD.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PRD.en.md) | English | Product requirements document for the agent template; defines the problem space, target users, and feature requirements. |
| [`modules/agent-template/docs/th/PRD.th.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/th/PRD.th.md) | Thai | Thai-language parity of `PRD.en.md`; `hreflang="th"`. |
| [`modules/agent-template/docs/en/SRS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/SRS.en.md) | English | Software requirements specification structured around the Magga eightfold model; the engineering contract for agent behavior. |
| [`modules/agent-template/docs/th/SRS.th.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/th/SRS.th.md) | Thai | Thai-language parity of `SRS.en.md`; `hreflang="th"`. |
| [`modules/agent-template/docs/en/SELF-IMPROVEMENT.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/SELF-IMPROVEMENT.en.md) | English | Specification of the agent self-improvement loop: how agents accumulate, evaluate, and prune knowledge across sessions. |
| [`modules/agent-template/docs/th/SELF-IMPROVEMENT.th.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/th/SELF-IMPROVEMENT.th.md) | Thai | Thai-language parity of `SELF-IMPROVEMENT.en.md`; `hreflang="th"`. |
| [`modules/agent-template/docs/en/THREAT-MODEL.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/THREAT-MODEL.en.md) | English | Security threat model for incarnated agents: trust tiers (T0-T3), attack surface analysis, and mitigations. |
| [`modules/agent-template/docs/th/THREAT-MODEL.th.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/th/THREAT-MODEL.th.md) | Thai | Thai-language parity of `THREAT-MODEL.en.md`; `hreflang="th"`. |

---

## Notes for operators

- **Set the real host.** Uncomment and update the `Sitemap:` line in `robots.txt` before deploying.
- **Site-root-relative paths** in this file assume the handbook is served from the domain root. If served under a subpath (e.g. `https://example.com/handbook/`), prepend that subpath to all `/` paths listed here.
- **Framework doc URLs** are permanent GitHub blob links and require no operator adjustment.
- **Freshness:** re-check `VERSION.md` at the framework repo before deciding whether to re-index framework docs. Re-check git commit dates on individual handbook files before deciding whether to re-index handbook pages.
