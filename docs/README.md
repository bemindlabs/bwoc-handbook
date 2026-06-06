# BWOC Handbook

The single entry point for everyone who **uses, builds, operates, or indexes** the **BWOC framework** — a backend-neutral framework for incarnating and orchestrating AI coding agents, driven by one CLI: `bwoc`.

> 🇹🇭 ฉบับภาษาไทย: [`README.th.md`](README.th.md) · Framework repo: [`bemindlabs/BWOC-Framework`](https://github.com/bemindlabs/BWOC-Framework) · v2.24.0 · Canonical docs (EN/TH): [`docs/`](https://github.com/bemindlabs/BWOC-Framework/tree/main/docs)

This handbook is **role-indexed and bilingual**. English is primary (canonical); each page has a Thai counterpart (`*.en.md` ↔ `*.th.md`). Open your role, read one book, follow links into the framework docs only when you need the full treatment.

---

## Find your handbook

| You are… | Read | One-liner |
|---|---|---|
| 🧑‍💻 **End user** — run agents, chat, teams, tasks (no framework code) | [`end-user/HANDBOOK.en.md`](end-user/HANDBOOK.en.md) · [TH](end-user/HANDBOOK.th.md) | Install, init a workspace, drive agents with `bwoc` |
| 🛠️ **Developer** — build on / contribute to the framework (Rust, CLI, harness) | [`developer/HANDBOOK.en.md`](developer/HANDBOOK.en.md) · [TH](developer/HANDBOOK.th.md) | Crate map, build/test, hooks, versioning, release, PR gates |
| 🤖 **Agent author / operator** — incarnate, tune, run agents | [`agents/HANDBOOK.en.md`](agents/HANDBOOK.en.md) · [TH](agents/HANDBOOK.th.md) | Agent layout, the AGENTS.md rule, slots, manifest, the arc, `bwoc check` |
| 🔌 **Backends** — drive & configure agents via each CLI | [`backends/HANDBOOK.en.md`](backends/HANDBOOK.en.md) · [TH](backends/HANDBOOK.th.md) | Claude/Codex/AGY/Kimi/Copilot/Ollama; prompt-driven config after `init` |
| 🔎 **AI search / retrieval agent** — ingest this corpus to answer questions | [`ai-search/HANDBOOK.en.md`](ai-search/HANDBOOK.en.md) · [TH](ai-search/HANDBOOK.th.md) + [`llms.txt`](llms.txt) | Where canonical facts live, how to cite, what not to hallucinate |
| 🕷️ **Web crawler / indexer** | [`crawler/HANDBOOK.en.md`](crawler/HANDBOOK.en.md) · [TH](crawler/HANDBOOK.th.md) + [`robots.txt`](robots.txt) + [`sitemap.md`](sitemap.md) | Crawl policy, freshness signals, what to index vs skip |
| 🧩 **Anyone mapping the family** — every BWOC & `bwoc-*` project | [`ecosystem/HANDBOOK.en.md`](ecosystem/HANDBOOK.en.md) · [TH](ecosystem/HANDBOOK.th.md) | What each project is, its stack, and how it connects to the core |

Term lookup for any reader: [`glossary.en.md`](glossary.en.md) · [TH](glossary.th.md)

---

## What BWOC is, in five sentences

1. **One folder, one agent.** Each agent is a self-contained directory cloned from a template; its identity, mindsets, and skills live in slot folders.
2. **Backend-neutral.** The same agent runs on Claude, Codex, Kimi, or a self-hosted model (Ollama / any OpenAI-compatible endpoint) via `bwoc-harness`. No vendor is favored.
3. **Designed on explicit principles.** A few specialized terms exist, but every one maps to a real engineering concern (lifecycle, memory pruning, inter-agent trust, threat modeling) — **you don't need to learn the terms to use it.** See the glossary when curious.
4. **Persistent, prune-aware memory.** Agents accumulate knowledge across sessions and deliberately drop stale entries (`MEMORY.md` ≤ 200 lines).
5. **Multi-agent safe.** Many agents share one workspace — shared task lists, teams, trust scoring — without collision.

---

## The lifecycle every BWOC object follows

| Phase | Plain name | What happens | CLI |
|---|---|---|---|
| 1 | **Birth** | Identity created, manifest resolved, capabilities declared | `bwoc new`, `bwoc init` |
| 2 | **Live / work** | Agent operates; state and memory evolve under discipline | `bwoc spawn`, `chat`, `run`, `supervise` |
| 3 | **Retire** | Cleanup; branches released, memory pruned, task closed, agent retired | `bwoc retire`, `stop` |

> The framework docs label these three phases with Pali terms (*uppāda · ṭhiti · vaya*). You can ignore the labels and just think "birth / live / retire."

---

## Conventions used across this handbook

- **Handbook vs source of truth.** This handbook *orients and summarizes*. The **source of truth** is the framework repo [`bemindlabs/BWOC-Framework`](https://github.com/bemindlabs/BWOC-Framework): code in `crates/`, docs in `docs/en` + `docs/th`, the spec in `modules/agent-template/AGENTS.md`. On conflict, the repo wins — and that means this handbook has a bug; please fix it.
- **Bilingual, English-primary.** Every page is `*.en.md` (canonical) with a `*.th.md` counterpart kept in parity. Edit both in the same change.
- **Plain language, minimal jargon.** Specialized terms are explained on first use and link to the glossary.
- **Commands** target the `bwoc` CLI. For the live surface run `bwoc help getting-started` or `bwoc <cmd> --help`.
- **Versions move.** Pin claims to a version when it matters; check [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) for current `Software-Version` / `Document-Version`.
- **Links never point into a private workspace.** All references to framework source go to the public GitHub repo, never to a local path like `projects/bwoc-framwork`.

---

## Fast paths

```bash
# Just use it
bwoc init ./my-workspace && cd my-workspace && bwoc list

# Make an agent (template auto-detected from the framework clone in cwd ancestors)
bwoc new sage --role "research assistant" --target agents/agent-sage
bwoc check agents/agent-sage

# Build the framework (from a clone of github.com/bemindlabs/BWOC-Framework)
git clone https://github.com/bemindlabs/BWOC-Framework && cd BWOC-Framework
cargo build && cargo test

# Index this corpus (machine)
cat llms.txt
```
