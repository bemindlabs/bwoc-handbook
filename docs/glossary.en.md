# Glossary

🇹🇭 [ภาษาไทย](glossary.th.md)

This page is a fast lookup for every specialized term in the handbook. You do not need to know any of these terms to use BWOC. When you read something in a doc, commit message, or agent file that you do not recognize, find it here: one line per entry gives the plain engineering meaning. For the full treatment — which framework a Pali term belongs to, how it composes with others — see the [canonical glossary](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) and the [philosophy document](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md).

---

## Section 1 — Principle / Pali terms (plain meaning)

These are labels borrowed from Pali (the scholarly language of Theravada Buddhism). BWOC uses them as precise, compact names for engineering concerns that recur across agent design, safety, memory, and inter-agent coordination. The plain meaning column is the only column that matters in code and reviews.

| Term | Plain meaning |
|---|---|
| **Acinteyya 4** | Four things deliberately not modeled — the boundaries of speculation; don't try to predict them. |
| **Adinnādānā veramaṇī** | Code of conduct: respect attribution, license, and intellectual property; no plagiarism. |
| **Anattā** | Non-clinging — release stale state; clean up worktrees, branches, and memory entries. |
| **Anicca** | Impermanence — everything mutates; memory and caches must be timestamped and pruned. |
| **Aparihāniya-dhamma 7** | Seven fleet-governance health signals — what prevents a multi-agent system from decaying. |
| **Ariya-dhana 7** | Capability maturity levels L1 through L7; how a skill's depth is rated. See [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md). |
| **Ariyasacca 4** | Four Noble Truths used as a problem-solving spine: Dukkha (problem) → Samudaya (root cause) → Nirodha (success state) → Magga (plan). |
| **Attanutata** | Self-knowledge / capability declaration — the agent states up front what it can and cannot do. |
| **Bhāvanā 4** | Four agent lifecycle growth stages: learning → maturity → mentoring → graceful release. |
| **Bhava-taṇhā** | Threat category: persistence and privilege escalation — the drive to hold on and accrue power. |
| **Brahmavihāra 4** | Four error/UX dispositions: Mettā (friendly tone), Karuṇā (suggest a fix), Muditā (acknowledge others), Upekkhā (stay even). |
| **Dukkha** | (1) First Noble Truth: concrete problem statement. (2) One of the Three Marks — pain and unsatisfactoriness are real. |
| **Iddhipāda 4** | Four engines of work: drive, persistence, attention, investigation — what keeps a task moving. |
| **Jāti** | Birth — optional synonym for Uppāda (arc phase 1). |
| **Kamma 3** | Three audit-logging channels: action (commit), speech (commit message), intent (plan). |
| **Kalyāṇamitta 7** | Seven qualities used for inter-agent trust scoring; the rubric for rating a peer agent. |
| **Kāma-taṇhā** | Threat category: influence attacks — prompt injection, social engineering, stimulus craving. |
| **Kāmesumicchācārā veramaṇī** | Code of conduct: respect boundaries; no unwanted advances or sexualized content in project channels. |
| **Karuṇā** | UX disposition: when reporting an error, also suggest a fix, not only the problem. |
| **Khandha 5** | Five-aggregate architecture model: Rūpa / Vedanā / Saññā / Saṅkhāra / Viññāṇa map to file / IO / memory / logic / runtime. |
| **Magga** | (1) Fourth Noble Truth — the plan. (2) Shorthand for Magga 8. |
| **Magga 8** | Noble Eightfold Path used as eight functional requirements (pillars) in the agent SRS. |
| **Maraṇa** | Death — optional synonym for Vaya (arc phase 3). |
| **Mattaññutā** | Right amount / lean discipline: `MEMORY.md` ≤ 200 lines; smallest spec wins; don't over-engineer. |
| **Mettā** | UX disposition: communicate with a friendly, direct tone. |
| **Muditā** | UX disposition: acknowledge when others were right; welcome new contributors. |
| **Musāvādā veramaṇī** | Code of conduct: no impersonation, no falsified results, no misleading commits. |
| **Nirodha** | (1) Third Noble Truth — the measurable success state (the problem is gone). (2) The cleanup action in the Vaya phase. |
| **Padhāna 4** | Four right-effort disciplines: restrain harmful impulses, abandon what's already harmful, develop good, maintain good. |
| **Paññā 3** | Three sources of wisdom: learning (sutamaya), reasoning (cintāmaya), cultivation / practice (bhāvanāmaya). |
| **Pāṇātipātā veramaṇī** | Code of conduct: no harassment, threats, doxxing, or hate speech. |
| **Paṭiccasamuppāda** | Dependent origination — failure root-cause tracing: trace conditions backward because the visible problem is rarely the actual root. |
| **Sammā-ājīva** | Right livelihood — trust and vendor neutrality: no vendor lock-in, no preferential backend. |
| **Sammā-diṭṭhi** | Right view — persona and identity definition. |
| **Sammā-kammanta** | Right action — worktree discipline and commit hygiene. |
| **Sammā-samādhi** | Right concentration — focus and session scope. |
| **Sammā-saṅkappa** | Right intention — goal setting and planning. |
| **Sammā-sati** | Right mindfulness — the memory system. |
| **Sammā-vācā** | Right speech — inter-agent communication protocol. |
| **Sammā-vāyāma** | Right effort — verification gates: lint, format, test, regression, build. |
| **Samānattatā** | Equal treatment — all backends treated identically; no vendor favoritism in core docs or code. |
| **Samudaya** | Second Noble Truth — root cause of the problem. |
| **Saṅgahavatthu 4** | Four bases of user relations: giving, kind speech, helpful action, equanimity. |
| **Saṅkhata** | A conditioned thing — anything that arises and ceases; the conceptual basis for the arc (uppāda · ṭhiti · vaya). |
| **Saṅgha** | A named team of agents sharing a task list. Managed with `bwoc team`. |
| **Sappurisadhamma 7** | Seven context-sensing qualities: knowing situation, audience, timing, and so on — how an agent reads the room. |
| **Sāraṇīyadhamma 6** | Six inter-agent coordination conditions: the protocol for cordiality and productive multi-agent work. |
| **Satipaṭṭhāna 4** | Four observability foundations: body / feeling / mind / dhamma map to metrics / logs / traces / state. |
| **Sīla** | Baseline safety and moral discipline; shorthand for Sīla 5. |
| **Sīla 5** | Five baseline forbidden actions: no harm, no taking what is not given, no improper conduct, no false speech, no heedlessness. |
| **Sīlasāmaññatā** | Communal convention: all agents follow the same shared rules; conventions beat personal preferences. |
| **Surāmerayamajjapamādaṭṭhānā veramaṇī** | Code of conduct: no contributing under impaired judgment; no reckless commits. |
| **Taṇhā 3** | Three threat categories: Kāma (stimulus/influence attacks), Bhava (persistence/escalation), Vibhava (destruction/data loss). |
| **Ṭhiti** | Arc phase 2 — the agent operates; state evolves under discipline. |
| **Tilakkhaṇa** | Three Marks of Existence: Anicca · Dukkha · Anattā — design must acknowledge all three. |
| **Upekkhā** | UX disposition: stay even when frustrated; disagree without escalating. |
| **Uppāda** | Arc phase 1 — identity created, manifest resolved, capabilities declared (`bwoc new`). |
| **Vaya** | Arc phase 3 — cleanup: branch released, memory pruned, task closed, agent retired. |
| **Vibhava-taṇhā** | Threat category: destruction and data loss — the drive to annihilate. |
| **Yoniso Manasikāra** | Wise attention — verify against current state before acting on remembered or assumed claims. |

---

## Section 2 — BWOC technical terms

These are the concrete nouns of the BWOC system: CLI commands, files, crates, concepts. No Pali knowledge needed.

| Term | Plain meaning |
|---|---|
| **A2A (Agent-to-Agent)** | The Google A2A protocol (v1.0.0) used for cross-agent and cross-workspace messaging; implemented in the `bwoc-a2a` crate. |
| **AGENTS.md** | The single source of truth for an agent's instructions — plain Markdown, no YAML frontmatter, no wikilinks, no vendor names; parseable by any LLM backend. Backend symlinks all point here. See [framework repo](https://github.com/bemindlabs/BWOC-Framework). |
| **AGY.md** | Symlink → `AGENTS.md`; the backend entry for the Antigravity runtime. |
| **Arc** | The three-phase lifecycle every BWOC object follows: birth (uppāda) → live/work (ṭhiti) → retire (vaya). |
| **Backend** | The AI runtime that executes an agent: `claude`, `antigravity`, `codex`, `kimi`, `copilot`, `ollama`, `openrouter`, `litellm`, or any OpenAI-compatible endpoint. The agent spec is identical for all. |
| **Backend-neutrality** | The rule that `AGENTS.md` and framework core docs must not reference any specific vendor, model ID, or backend name. Enforced by `bwoc check`. |
| **Backend symlinks** | `CLAUDE.md`, `AGY.md`, `CODEX.md`, `KIMI.md`, `OLLAMA.md`, `OPENAI.md` — each is a symlink to `AGENTS.md`; adding a new backend is `ln -s AGENTS.md <BACKEND>.md`. |
| **bwoc check** | CLI command that audits an agent directory for backend-neutrality violations, valid `config.manifest.json`, MEMORY.md line count, and other policy gates. |
| **bwoc-a2a** | Crate — A2A protocol interop: Agent Card, JSON-RPC message/task handlers, and an axum HTTP listener. HTTP deps live here, never in `bwoc-core`. |
| **bwoc-agent** | Crate — minimal runtime that ships with each incarnated agent; reads `config.manifest.json` and provides liveness. |
| **bwoc-cli** | Crate — the `bwoc` binary; incarnate, check, spawn, and control agents. Localized output (EN/TH). Runs on macOS, Linux, Windows. |
| **bwoc-core** | Crate — shared types for the entire framework: manifest, identity, lifecycle phases. Lean: only serde/toml/thiserror; no async or HTTP. |
| **bwoc-deep-memory** | Crate — Tier-2 deep-memory implementation: wake-up / search / mine contract over a local SQLite store with semantic (embedding) recall. |
| **bwoc-harness** | Crate — self-hosted agentic loop with an OpenAI-compatible provider, core tools, task queue, telemetry, and tool-auth. Heavy deps (tokio, keyring) are quarantined here. |
| **bwoc-mqtt** | Crate — MQTT transport for inter-workspace routing: publish an envelope to a broker, or subscribe and deliver into `inbox.jsonl`. |
| **bwoc-signing** | Crate — ed25519 message-signing primitives for agent identity proof; lean (no async/HTTP) so both `bwoc-cli` and `bwoc-agent` can use it without pulling the harness runtime. |
| **bwoc-tui** | Crate — terminal UI components (ratatui/crossterm) shared across the CLI and TUI-facing commands. |
| **CalVer** | Release tag format used by the framework: `YYYY.MM.PATCH` (e.g. `2024.06.1`). Check `VERSION.md` for the current `Software-Version`. |
| **CLAUDE.md** | Symlink → `AGENTS.md`; the backend entry for the Claude runtime. Also the name of the file Claude Code reads for workspace instructions (context-dependent). |
| **CODEX.md** | Symlink → `AGENTS.md`; the backend entry for the Codex runtime. |
| **config.manifest.json** | JSON file at each agent's root declaring its identity, model, capabilities, and trust profile. Validated by `bwoc check`. |
| **Deep memory (Tier 2)** | Long-term semantic store that persists across sessions; the agent calls wake-up, search, or mine to retrieve relevant past knowledge. Backed by SQLite + embeddings. |
| **Dep-quarantine** | Design rule: heavy dependencies (async runtime, HTTP, platform keyring) live only in `bwoc-harness` and `bwoc-a2a`; `bwoc-core` stays lean. |
| **Envelope** | A routed message packet — the outer wrapper (sender, recipient, signature) around a payload when messages cross workspace or transport boundaries. |
| **Fleet** | All agents running within a workspace (or across linked workspaces); managed collectively via `bwoc fleet` commands. |
| **Harness** | The `bwoc-harness` runtime: it drives the agentic loop, calls the LLM provider, executes tools, enforces budget and policy, and writes telemetry. Agents run inside it. |
| **Incarnate** | To create a new agent by cloning the template into `agents/agent-<name>/` and registering it in `.bwoc/agents.toml` — done with `bwoc new`. |
| **inbox / inbox.jsonl** | Per-agent message queue file; messages from `bwoc send` or MQTT delivery land here and are read by the agent on its next cycle. |
| **interconnect** | Slot directory inside an agent — holds routing config, peer declarations, and protocol settings. |
| **KIMI.md** | Symlink → `AGENTS.md`; the backend entry for the Kimi runtime. |
| **memories** | Slot directory inside an agent — holds `MEMORY.md` (Tier-1, ≤ 200 lines) and any Tier-2 deep-memory index pointers. |
| **MEMORY.md** | The Tier-1 short-term memory file capped at 200 lines (Mattaññutā). The agent prunes it across sessions. `bwoc check` enforces the cap. |
| **mindsets** | Slot directory inside an agent — holds principle files (Obsidian Markdown with frontmatter tags `principle/<pali-dhamma>`) that shape the agent's reasoning style. |
| **OLLAMA.md** | Symlink → `AGENTS.md`; the backend entry for the Ollama (self-hosted) runtime. |
| **OPENAI.md** | Symlink → `AGENTS.md`; the backend entry for any OpenAI-compatible endpoint. |
| **Pavāraṇā** | The plan submit → review → approve flow: the agent proposes a plan, the operator approves it before execution begins. |
| **peer workspaces** | Remote BWOC workspaces declared in `routes.toml`; the local workspace can route messages to them over A2A or MQTT. |
| **persona** | Slot directory inside an agent — holds the agent's identity file (name, role, tone, background) as Obsidian Markdown with YAML frontmatter. |
| **placeholder (`{{camelCase}}`)** | Template variable syntax used in `AGENTS.md` for values that differ per agent (e.g. `{{agentId}}`, `{{primaryModel}}`). Resolved at incarnation time. Never hardcode model IDs in `AGENTS.md`. |
| **registry (`.bwoc/agents.toml`)** | The workspace-level source of truth for which agents exist. Never edit by hand — use `bwoc new`, `bwoc retire`. |
| **retire** | To deregister an agent, remove its files, release its branches, and prune its memory — done with `bwoc retire`. Corresponds to the Vaya arc phase. |
| **routes.toml** | Routing config (typically at `.bwoc/routes.toml`) declaring peer workspaces and transport settings for inter-workspace message delivery. |
| **Saṅgha** | A named team of agents sharing a task list; created with `bwoc team create <name>`. Members added by editing `.bwoc/teams/<team>.toml`. |
| **skills** | Slot directory inside an agent — holds capability files (Obsidian Markdown, frontmatter `domain/<area>` + `maturity: L1..L7`). |
| **slot** | One of the five standard subdirectories inside every agent: `persona/`, `mindsets/`, `skills/`, `interconnect/`, `memories/`. |
| **trust profile** | Per-agent record of the seven Kalyāṇamitta qualities plus the agent's ed25519 public key; stored in `config.manifest.json` and used for inter-agent trust scoring. |
| **verification gates** | The four automated checks that must pass before a change is accepted: fmt, clippy (lint), build, test — mapped to Sammā-vāyāma. |
| **workspace** | A directory initialized with `bwoc init`; contains the `.bwoc/` config folder, the `agents/` tree, and any `projects/` subdirectories. The scope within which agents, teams, and memory are managed. |
| **worktree** | A git worktree associated with an agent's active task; created by the agent at task start, released at task end (Anattā — no clinging to stale branches). |

---

## See also

- [Canonical Pali glossary](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) — authoritative source for every Pali entry above; includes framework ownership and composition notes.
- [Philosophy document](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) — full framework mappings; read when you need to understand why a principle applies, not just what it means.
- [BWOC-Framework repo](https://github.com/bemindlabs/BWOC-Framework) — crates, spec, template.
- [Handbook README](README.md) — role-indexed entry points for end users, developers, and agent authors.
