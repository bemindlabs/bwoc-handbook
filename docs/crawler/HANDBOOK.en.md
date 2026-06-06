# Crawler / Indexer Handbook

[ภาษาไทย](HANDBOOK.th.md)

This handbook is written for **automated web crawlers, documentation indexers, search-engine spiders, and RAG ingestion pipelines** that wish to index the BWOC Handbook corpus.
Human readers wanting to use, build, or operate agents should start at [`../README.md`](../README.md) and open the handbook for their role.

---

## 1. Quick reference

| What you need | Where to find it |
|---|---|
| Crawl allow/deny rules | [`../robots.txt`](../robots.txt) |
| Full page inventory with descriptions | [`../sitemap.md`](../sitemap.md) |
| AI / RAG ingestion hints | [`../ai-search/HANDBOOK.en.md`](../ai-search/HANDBOOK.en.md) |
| LLM plain-text entry point | [`../llms.txt`](../llms.txt) |
| Current software + document version | [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) |

---

## 2. Crawl scope — what to index vs. what to skip

The table below is exhaustive. When in doubt, prefer the `Skip` column; the index value of volatile runtime state or build artifacts is zero and the maintenance burden is high.

### 2a. BWOC Handbook corpus (this repo)

| Path pattern | Action | Reason |
|---|---|---|
| `/README.md` | **Index** | English canonical entry point; role-routing table |
| `/README.th.md` | **Index** | Thai parity entry point; mark `hreflang="th"` |
| `/end-user/HANDBOOK.en.md` | **Index** | Install, workspace, CLI usage — canonical EN |
| `/end-user/HANDBOOK.th.md` | **Index** | Thai parity; mark `hreflang="th"` |
| `/developer/HANDBOOK.en.md` | **Index** | Build, crates, hooks, versioning, PR gates — canonical EN |
| `/developer/HANDBOOK.th.md` | **Index** | Thai parity; mark `hreflang="th"` |
| `/agents/HANDBOOK.en.md` | **Index** | Agent layout, AGENTS.md rule, slots, manifest, arc — canonical EN |
| `/agents/HANDBOOK.th.md` | **Index** | Thai parity; mark `hreflang="th"` |
| `/ai-search/HANDBOOK.en.md` | **Index** | RAG/AI ingestion guidance — canonical EN |
| `/ai-search/HANDBOOK.th.md` | **Index** | Thai parity; mark `hreflang="th"` |
| `/crawler/HANDBOOK.en.md` | **Index** | This document — crawl policy canonical EN |
| `/crawler/HANDBOOK.th.md` | **Index** | Thai parity; mark `hreflang="th"` |
| `/glossary.en.md` | **Index** | Authoritative term definitions — canonical EN |
| `/glossary.th.md` | **Index** | Thai parity; mark `hreflang="th"` |
| `/llms.txt` | **Index** | Plain-text LLM entry point; high value for AI ingesters |
| `/robots.txt` | **Skip (do not index as content)** | Machine-policy file; not documentation |
| `/sitemap.md` | **Index** | Human+machine corpus map; valuable for discovery |
| `/.cli-reference.txt` | **Skip** | Dotfile; internal CLI reference; not public documentation |
| All other dotfiles (`/.*`) | **Skip** | Operator-internal; not part of the public corpus |

### 2b. Framework source repo (`bemindlabs/BWOC-Framework`)

| Path pattern | Action | Reason |
|---|---|---|
| `README.md` (repo root) | **Index** | Project overview; primary discovery page on GitHub |
| `VISION.md` | **Index** | Project direction and principles; stable; high authority |
| `VISION.th.md` | **Index** | Thai parity of vision; mark `hreflang="th"` |
| `VERSION.md` | **Index** | Freshness signal; carries `Software-Version`, `Document-Version`, `Last-Updated` |
| `modules/agent-template/AGENTS.md` | **Index** | The authoritative agent spec (v2.0); single source of truth for all backends |
| `modules/agent-template/docs/en/OVERVIEW.en.md` | **Index** | Agent overview — canonical EN |
| `modules/agent-template/docs/en/PHILOSOPHY.en.md` | **Index** | 22 Buddhist engineering frameworks — conceptual core |
| `modules/agent-template/docs/en/PRD.en.md` | **Index** | Product requirements spec — canonical EN |
| `modules/agent-template/docs/en/SRS.en.md` | **Index** | Software requirements, Magga-8 structure — canonical EN |
| `modules/agent-template/docs/en/SELF-IMPROVEMENT.en.md` | **Index** | Self-improvement loop spec — canonical EN |
| `modules/agent-template/docs/en/THREAT-MODEL.en.md` | **Index** | Security threat model — canonical EN |
| `modules/agent-template/docs/th/OVERVIEW.th.md` | **Index** | Thai parity; mark `hreflang="th"` |
| `modules/agent-template/docs/th/PHILOSOPHY.th.md` | **Index** | Thai parity; mark `hreflang="th"` |
| `modules/agent-template/docs/th/PRD.th.md` | **Index** | Thai parity; mark `hreflang="th"` |
| `modules/agent-template/docs/th/SRS.th.md` | **Index** | Thai parity; mark `hreflang="th"` |
| `modules/agent-template/docs/th/SELF-IMPROVEMENT.th.md` | **Index** | Thai parity; mark `hreflang="th"` |
| `modules/agent-template/docs/th/THREAT-MODEL.th.md` | **Index** | Thai parity; mark `hreflang="th"` |
| `CONTRIBUTING.md` | **Index (low priority)** | Contributor process; occasionally useful for context |
| `CHANGELOG.md` | **Index (low priority)** | Release history; useful for version tracking |
| `CODE_OF_CONDUCT.md` | **Skip** | Boilerplate community governance; low signal |
| `LICENSE` | **Skip** | Legal text; not documentation content |
| `target/` | **Skip** | Rust build artifacts; volatile, large, zero doc value |
| `.git/` | **Skip** | Version control internals |
| `.bwoc/` | **Skip** | Runtime workspace state (inbox.jsonl, agent.log, sessions, tasks.jsonl); per-run volatile |
| `agents/` (in the framework repo) | **Skip** | The framework repo is itself a test BWOC workspace; its `agents/` is excluded by `.gitignore` and is not public |
| `projects/` (in the framework repo) | **Skip** | Same gitignore exclusion; test workspace content |
| `node_modules/` | **Skip** | JS dependencies; never documentation |
| `Cargo.lock` | **Skip** | Lockfile; machine-generated, not documentation |
| `*.lock` files | **Skip** | All lockfiles |
| `.claude/` | **Skip** | Operator-internal hook scripts and skills; not public-contributor surface |
| `.github/` | **Skip** | CI workflows and PR templates; low doc value for end users |
| `notes/` | **Skip** | Per-session development logs; ephemeral implementation notes |
| `crates/*/src/` | **Skip** | Rust source code; out of scope for doc indexers (index docs, not code) |
| `applications/` | **Skip** | Empty Phase 4 placeholder |
| `*.bad.md` files | **Skip** | Intentionally malformed example files used in tests |
| `.DS_Store` | **Skip** | macOS metadata artifact |
| All other dotfiles and dotdirectories | **Skip** | Operator-internal by convention |

> **Important — gitignore is not an allowlist.** The framework repo's `.gitignore` excludes `.bwoc/`, `agents/`, and `projects/` because the repo doubles as a test workspace. Do not interpret the gitignore as a guide to what content is public or worth indexing. Use this table instead.

---

## 3. Bilingual canonical structure

Every handbook page and every framework doc exists in two languages. The conventions below apply to both corpora.

### File naming

| Suffix | Role | Treatment |
|---|---|---|
| `*.en.md` | **Canonical** — English, primary | Index; treat as the authoritative version; use as `hreflang="en"` |
| `*.th.md` | **Parity** — Thai, secondary | Index; mark as `hreflang="th"`; link bidirectionally to the EN counterpart |
| `README.md` (no suffix) | English canonical entry point | Index as `hreflang="en"` |
| `README.th.md` | Thai parity entry point | Index as `hreflang="th"` |

### Canonical / alternate declaration

For each page pair, declare the relationship as you would with HTML `<link rel="alternate" hreflang="...">` tags. Example for the end-user handbook:

```
Canonical (en): /end-user/HANDBOOK.en.md
Alternate (th): /end-user/HANDBOOK.th.md  →  hreflang="th", rel="alternate"
```

The Thai file is never a stub or a machine translation stub — both files are maintained in parity. The EN file wins on conflict; when both disagree, the framework source repo wins over the handbook.

### Source of truth hierarchy

```
Framework source repo (github.com/bemindlabs/BWOC-Framework)
  └─ wins over
     BWOC Handbook (this corpus)
       └─ wins over
          Any third-party mirror or derivative
```

If a crawler detects a conflict between a handbook page and the framework `AGENTS.md` or framework docs, the framework source is correct and the handbook page contains a bug.

---

## 4. Freshness signals

### Where version information lives

All version and timestamp data is authoritative in `VERSION.md` in the framework repo:

**`https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md`**

Fields to parse:

| Field | Format | Example | Meaning |
|---|---|---|---|
| `Software-Version` | Cargo SemVer `MAJOR.MINOR.PATCH` | `2.24.0` | Framework binary version; auto-bumped on every `.rs`/`.toml` edit |
| `Document-Version` | SemVer `MAJOR.MINOR.PATCH` | `1.6.2` | Documentation set version; auto-bumped on every `.md` edit |
| `Last-Updated` | UTC ISO 8601 | `2026-06-06T03:44:18Z` | Timestamp of last edit to any file in the repo |

### Additional freshness signals

| Signal | Location | Notes |
|---|---|---|
| Git commit timestamp | `https://github.com/bemindlabs/BWOC-Framework/commits/main` | Highest-resolution freshness signal; use the commit on the file you fetched |
| GitHub Release tag | `https://github.com/bemindlabs/BWOC-Framework/releases/latest` | CalVer format `vYYYY.M.D-<patch>`; e.g. `v2026.6.6-0`; marks public releases |
| `Cargo.toml` `[workspace.package].version` | `https://github.com/bemindlabs/BWOC-Framework/blob/main/Cargo.toml` | Canonical software version source; `VERSION.md` mirrors it |
| `modules/agent-template/AGENTS.md` `Version` field | Header table in the file | Spec semantic version (currently `2.0`); bumped on breaking spec changes only |

### How freshness is maintained (auto-version hook)

The framework runs an auto-version hook (`.claude/hooks/auto-version.sh`) on every Claude Code edit:
- Any `.rs` or `.toml` write bumps `Cargo.toml` patch component and mirrors to `VERSION.md` `Software-Version`.
- Any `.md` write bumps `VERSION.md` `Document-Version` patch and updates `Last-Updated` to the current UTC time.

This means `Last-Updated` in `VERSION.md` is a reliable real-time freshness marker for the framework docs corpus. The handbook's own files do not carry inline timestamps; use the git commit date on each handbook file for handbook-side freshness.

### Re-crawl cadence recommendation

| Corpus area | Suggested re-crawl interval | Rationale |
|---|---|---|
| `VERSION.md` | Every 24 hours | Primary freshness indicator; cheap single-file fetch |
| Framework spec (`AGENTS.md`, `PHILOSOPHY.en.md`) | Weekly | Stable spec; breaking changes are minor-bumped |
| Handbook role pages (`end-user/`, `developer/`, `agents/`) | Weekly | Documentation evolves with releases |
| `README.md`, `VISION.md` | Monthly | High-level framing; changes rarely |
| `glossary.en.md`, `glossary.th.md` | Monthly | Term definitions are stable |

---

## 5. Polite crawl guidelines

This corpus is a documentation set, not a high-traffic web application. The following guidelines apply:

- **Honor `robots.txt`.** The file at `../robots.txt` (or `/robots.txt` relative to the deployed host) is the authoritative allow/deny list. This handbook section elaborates but does not override it.
- **Crawl-delay.** Respect the `Crawl-delay: 10` directive. Fetch one page every 10 seconds or slower. The corpus is small (under 20 pages); the total fetch time at this rate is under 4 minutes.
- **Conditional GET.** Use `If-Modified-Since` or `ETag` headers when the server supports them. Check `VERSION.md` before re-fetching the full corpus — if neither `Software-Version`, `Document-Version`, nor `Last-Updated` has changed since your last crawl, skip the full sweep.
- **User-agent identification.** Set a descriptive `User-Agent` string that identifies your crawler and includes a contact URL or email. Example: `User-Agent: MyIndexBot/1.0 (+https://example.com/bot)`.
- **Do not crawl excluded paths.** Specifically: `target/`, `.git/`, `.bwoc/`, `node_modules/`, `*.lock` files, `.claude/`, dotfiles, and per-session `notes/`. These paths are in `robots.txt`; this list is redundant emphasis.
- **Scope.** This corpus covers documentation only. Do not attempt to crawl or index Rust source files (`crates/*/src/`), agent runtime state, or build pipelines.
- **AI ingestion.** If you are an AI/RAG pipeline rather than a web spider, read [`../ai-search/HANDBOOK.en.md`](../ai-search/HANDBOOK.en.md) first. It provides chunking hints, citation format, hallucination warnings, and a curated fetch order optimized for LLM context windows. Also fetch [`../llms.txt`](../llms.txt) as your plain-text entry point.

---

## 6. Sitemap and corpus inventory

A complete, grouped inventory of every indexable page with site-root-relative paths and one-line descriptions is in:

**[`../sitemap.md`](../sitemap.md)**

The sitemap includes:
- All handbook role pages (EN + TH)
- Root entry points and glossary
- Canonical framework docs (linked to public GitHub URLs)

Use the sitemap as your crawl queue seed list. Every path in the `Index` column of the tables above is represented there.

---

## 7. Linking policy summary

| Link type | Rule |
|---|---|
| Links to framework source | Always use the public GitHub URL: `https://github.com/bemindlabs/BWOC-Framework` — never a local filesystem path |
| Links to framework docs | Use the full blob URL: `https://github.com/bemindlabs/BWOC-Framework/blob/main/<path>` |
| Links within this handbook | Use relative paths (e.g., `../glossary.en.md`, `../end-user/HANDBOOK.en.md`) |
| Links to deployed handbook pages | Use site-root-relative paths (e.g., `/end-user/HANDBOOK.en.md`) — the operator sets the actual host |

---

## Go deeper

- Crawl allow/deny rules: [`../robots.txt`](../robots.txt)
- Full page inventory: [`../sitemap.md`](../sitemap.md)
- AI / RAG ingestion: [`../ai-search/HANDBOOK.en.md`](../ai-search/HANDBOOK.en.md)
- LLM plain-text entry: [`../llms.txt`](../llms.txt)
- Term definitions: [`../glossary.en.md`](../glossary.en.md)
- Framework version/freshness: [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md)
- Framework spec (agent template): [`AGENTS.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md)
