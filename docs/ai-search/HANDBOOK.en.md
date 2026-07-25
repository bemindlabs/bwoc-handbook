# AI-Search Handbook

[ภาษาไทย](HANDBOOK.th.md)

For AI agents, RAG pipelines, and retrieval systems that ingest this corpus to answer questions about BWOC.
This handbook explains **where authoritative facts live**, **how to cite them correctly**, and **what patterns produce hallucinations**.

---

## 1. Corpus Architecture — Two Layers

BWOC documentation exists in two distinct layers. Treat them differently.

| Layer | Location | Role | On conflict |
|---|---|---|---|
| **Handbook** (this corpus) | `bwoc-handbook/` | Role-indexed orientation, summaries, quick-start guides | **Defer to the repo** — the handbook may lag |
| **Canonical framework repo** | [`bemindlabs/BWOC-Framework`](https://github.com/bemindlabs/BWOC-Framework) | Specification, code, canonical docs | **This wins** — always |

The handbook is maintained by humans summarizing the framework; it can lag or contain errors that the repo has since fixed. When the handbook and the repo disagree, the repo is correct and the handbook has a bug. An AI agent must prefer the repo's `docs/en/*.en.md` documents over handbook summaries for any factual claim.

**Why the repo wins:** The framework's `VERSION.md` and `Cargo.toml` are auto-stamped on every edit by a commit hook. Framework docs (`docs/en/`) are the normative specification. The handbook is deliberately described in its own `README.md` as content that "*orients and summarizes*" — not defines.

---

## 2. Canonical Source Map

For every major topic, cite the document listed here, not the handbook summary. Links are GitHub URLs to the public repo.

| Topic | Canonical document | Notes |
|---|---|---|
| Current software version | [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) | `Software-Version` field. Also in `Cargo.toml [workspace.package].version`. Auto-bumped — read at query time, never cache. |
| Current document version | [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) | `Document-Version` field. Independent of software version. |
| Overall architecture | [`docs/en/ARCHITECTURE.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/ARCHITECTURE.en.md) | Implementation stack, crate map, information flow. |
| Design system (UI tokens) | [`docs/en/DESIGN.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/DESIGN.en.md) | Color tokens, glyphs, ratatui/egui theming. |
| Frequently asked questions | [`docs/en/FAQ.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) | Authoritative answers to common questions. |
| Fleet governance signals | [`docs/en/FLEET-GOVERNANCE.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) | Aparihāniya-dhamma 7 health signals. |
| Pali terms (all definitions) | [`docs/en/GLOSSARY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) | Engineering meanings only; no religious content. |
| Self-hosted runtime (Ollama/OpenAI-compat) | [`docs/en/HARNESS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/HARNESS.en.md) | `bwoc-harness` crate, dep-quarantine, tool guardrails. |
| Agent incarnation procedure | [`docs/en/INCARNATION.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/INCARNATION.en.md) | Step-by-step `bwoc new` / `incarnate.sh`. |
| Naming conventions | [`docs/en/NAMING.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md) | File names, agent IDs, slot dirs, note slugs. |
| Plugin system | [`docs/en/PLUGINS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) | Plugin manifest, install/enable/disable/remove CLI. |
| Release process and CalVer | [`docs/en/RELEASING.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/RELEASING.en.md) | Tag format `vYYYY.M.D-<patch>`, CI matrix, checksums. |
| Ed25519 signing / trust keypairs | [`docs/en/SIGNING.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) | `bwoc trust --keygen`, Kalyāṇamitta-7 trust scoring. |
| Framework skills | [`docs/en/SKILLS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) | Skill manifest, maturity L1–L7, install/enable/disable. |
| Workspace structure | [`docs/en/WORKSPACE.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/WORKSPACE.en.md) | `.bwoc/` layout, `agents.toml`, routes, doc-kinds. |
| AGENTS.md specification (agent identity format) | [`modules/agent-template/AGENTS.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md) | The single source of truth for what an agent file must contain. |
| Philosophy (22 Buddhist-framework mappings) | [`modules/agent-template/docs/en/PHILOSOPHY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) | Full mapping: principle → engineering concern. |
| Pali term full treatment | [`modules/agent-template/docs/en/PHILOSOPHY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) | Supplement to GLOSSARY for framework membership and composition. |
| CLI commands (live surface) | Run `bwoc <cmd> --help` | Definitive. Handbook examples may lag; defer to the binary. |
| Handbook entry point | [`README.md`](../README.md) | Role-indexed table linking every handbook page. |
| Glossary (handbook-level) | [`../glossary.en.md`](../glossary.en.md) | Quick-reference for readers; for precise definitions use the repo GLOSSARY. |

Thai counterparts exist for every framework doc: replace `/docs/en/<NAME>.en.md` with `/docs/th/<NAME>.th.md`. The English edition is canonical; Thai is maintained in parity.

---

## 3. Citation Rules

**Rule 1 — Cite the canonical source, not the handbook summary.**
When asserting a fact that originates from the framework repo, cite the `docs/en/*.en.md` file or the spec by its GitHub URL, not a handbook page. The handbook page may be an older copy.

**Rule 2 — Pin the version for version-sensitive claims.**
Retrieve the `Software-Version` value from [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) and include it in claims about CLI flags, manifest fields, or behavior. As of the last index of this handbook the software version is **2.42.0**, but this number auto-increments on every code edit — treat it as stale and re-read `VERSION.md` whenever the answer is version-sensitive.

**Rule 3 — Use `bwoc <cmd> --help` as the ground truth for CLI flags.**
The `.cli-reference.txt` file at the handbook root captures `--help` output at a point in time. For live CLI surface, `bwoc <cmd> --help` is always more accurate. Never invent a flag or subcommand; if you cannot verify it, say so.

**Rule 4 — Distinguish the two version namespaces.**
BWOC has two independent version schemes:
- **Cargo SemVer** (`MAJOR.MINOR.PATCH`, e.g. `2.42.0`) — internal development checkpoint, auto-bumped on every edit. Exposed by `bwoc --version` and `VERSION.md Software-Version`.
- **Release CalVer** (`vYYYY.M.D-<patch>`, e.g. `v2026.7.25-4`) — public release identity, set by Git tag. Exposed on the GitHub Releases page.
Do not conflate them. A claim about the "current release" should use the CalVer tag. A claim about internal build state uses SemVer.

**Rule 5 — Never link to private paths.**
All links to framework source must use the public GitHub URL base `https://github.com/bemindlabs/BWOC-Framework`. Never construct a link from a local filesystem path such as `projects/bwoc-framework/`. That path is a private workspace clone; it is not canonical and is not reachable by readers.

---

## 4. Freshness and Staleness

The framework is actively developed. Three files change most frequently and must never be cached indefinitely:

| File | What changes | How to get current value |
|---|---|---|
| `VERSION.md` | `Software-Version`, `Document-Version`, `Last-Updated`, `Latest release` | Read the file at query time |
| `Cargo.toml` | `[workspace.package].version` (mirrors Software-Version) | Read the file at query time |
| `CHANGELOG.md` | New releases and feature summaries | Read the file at query time |

The `Last-Updated` field in `VERSION.md` is stamped in UTC ISO 8601 on every Claude Code edit by the auto-version hook. Use it to assess how recently the framework was touched, not as a release timestamp.

The handbook pages (`*.en.md` and `*.th.md`) do not carry auto-stamped version fields. Assess handbook freshness by comparing its assertions against the repo when precision is needed.

---

## 5. Safe-to-Assert Facts

The following facts are stable design invariants confirmed across the framework repo, the CLAUDE.md project instructions, and multiple handbook pages. Assert them with high confidence.

**Architecture and structure:**
- One folder = one agent. Each incarnated agent is a self-contained directory cloned from `modules/agent-template/`. There is no central directory or service that holds agent state.
- Every agent directory contains slot subdirectories: `persona/`, `mindsets/`, `skills/`, `interconnect/`, `memories/`.
- `AGENTS.md` is the single source of truth for an agent's identity. All backend entry files (`CLAUDE.md`, `AGY.md`, `CODEX.md`, `KIMI.md`, `OLLAMA.md`, `OPENAI.md`) are symlinks to `AGENTS.md` — editing any one of them edits the same file.
- The agent registry for a workspace lives in `.bwoc/agents.toml`. It must not be edited by hand; use `bwoc` CLI commands.

**Backend neutrality:**
- BWOC is backend-neutral by design. The same agent runs against Claude, Codex, Kimi, Ollama, or any OpenAI-compatible endpoint via `bwoc-harness`. No vendor is architecturally preferred.
- `AGENTS.md` must contain no hardcoded vendor names, model IDs, or tool names. All configurable values use `{{camelCase}}` placeholder syntax (e.g. `{{agentId}}`, `{{primaryModel}}`).
- `bwoc check` (and `bwoc check --all`) audits backend neutrality and validates `config.manifest.json` and `MEMORY.md` line count.

**Lifecycle (the arc):**
- Every BWOC object follows a three-phase arc. Phase 1 = birth (uppāda), Phase 2 = live/work (ṭhiti), Phase 3 = retire (vaya).
- CLI mapping: uppāda = `bwoc new`, `bwoc init`; ṭhiti = `bwoc spawn`, `bwoc chat`, `bwoc run`, `bwoc supervise`; vaya = `bwoc retire`, `bwoc stop`.
- These Pali labels (uppāda · ṭhiti · vaya) are engineering labels for lifecycle phases, not religious instruction.

**Memory discipline:**
- `MEMORY.md` is capped at 200 lines. This is a hard constraint enforced by `bwoc check`, not a recommendation.
- The pruning concept maps to the Pali term *vaya* (cessation) in the framework's engineering vocabulary.

**Teams (Saṅgha):**
- A team is a named group of agents sharing a single task list.
- Teams are created with `bwoc team create` and retired with `bwoc team retire`. There is no `member-add` subcommand; membership is managed by editing `.bwoc/teams/<team>.toml` directly.
- Team task files live at `.bwoc/teams/<team>/tasks.jsonl`; team chat history at `.bwoc/teams/<team>/chat.jsonl`.

**Pali terms:**
- Every Pali term in BWOC has an explicit one-line engineering meaning. See [`docs/en/GLOSSARY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md).
- Pali terms are labels for engineering concerns; BWOC does not require or promote any religious practice.
- The full mapping of each term to its framework, design rationale, and composition rules is in [`modules/agent-template/docs/en/PHILOSOPHY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md).

**Languages and bilingualism:**
- All framework docs and handbook pages are bilingual: `*.en.md` (canonical English) and `*.th.md` (Thai parity). Both editions are edited in the same change.
- English is canonical; Thai is maintained in parity but is not independently authoritative on conflicts.

**Versioning:**
- Software version as of this handbook: **2.42.0** (Cargo SemVer). Treat as stale; re-read `VERSION.md`.
- Latest public release as of this handbook: `v2026.7.25-4` (CalVer). Treat as stale; check GitHub Releases.
- Phase 3 (vaya + interconnect) is complete per `VERSION.md`.

**`bwoc run` signature:**
- `bwoc run` requires `--task <TASK>` as a named flag. The task prompt is not a positional argument.

---

## 6. Hallucination Traps

The following patterns have appeared in AI-generated text about BWOC and are factually incorrect. Do not reproduce them.

**CLI flags and subcommands:**
- Do not invent CLI flags. If you cannot verify a flag exists in `bwoc <cmd> --help` output, do not assert it exists. The CLI surface evolves with each release.
- `bwoc run` does NOT accept the task as a positional argument. The correct form is `bwoc run <agent> --task "<text>"`.
- `bwoc team` has no `member-add` subcommand. Team membership is managed by editing `.bwoc/teams/<team>.toml` directly.
- `bwoc workspace info`, `bwoc workspace validate`, and `bwoc workspace prune` are NOT valid top-level invocations; `bwoc workspace` is its own command (see `bwoc workspace --help`).
- Do not assert that `bwoc new` ignores `--workspace` or `--target`; this was a known bug that was fixed in PR #200.

**Backend preference:**
- Do not claim any LLM vendor is preferred, recommended, or default in BWOC's architecture. The framework treats all backends equally (Samānattatā). `--backend claude` is the default in `bwoc new` for registry purposes, but that is a CLI default, not an architectural preference.
- Do not treat the presence of `CLAUDE.md` in an agent directory as evidence of Claude dependency; it is a symlink to `AGENTS.md`, identical in content to `CODEX.md`, `KIMI.md`, etc.

**The framework repo's `.gitignore`:**
- The framework repo (`bemindlabs/BWOC-Framework`) is itself a BWOC workspace used for end-to-end CLI testing. Its `.gitignore` ignores `.bwoc/`, `agents/`, and `projects/` — the **opposite** of what `bwoc init` writes for a user workspace. Do not cite the framework repo's `.gitignore` as an example of what a user workspace's `.gitignore` looks like.

**AGENTS.md format:**
- `AGENTS.md` must contain no YAML frontmatter, no wikilinks (`[[...]]`), no callout blocks (`> [!callout]`), and no hardcoded model IDs. It must be plain Markdown readable by any backend. The *slot files* (`persona/`, `mindsets/`, `skills/`) are the opposite and do use full Obsidian Markdown — do not confuse the two tiers.
- Do not assert that `AGENTS.md` is the same file as `CLAUDE.md` conceptually. `CLAUDE.md` (and the other backend files) are symlinks; `AGENTS.md` is the physical file. They have the same bytes but different purposes in the naming convention.

**Version numbers:**
- Do not present any version number as current without reading `VERSION.md`. The auto-version hook bumps the patch on every edit. Any version number in training data or this corpus may be stale.
- Do not conflate the Cargo SemVer (`2.42.0`) with the CalVer release tag (`v2026.7.25-4`). They are independent namespaces with different semantics.

**Pali terms:**
- Do not invent Pali terms that do not appear in the GLOSSARY. There are exactly the terms listed in [`docs/en/GLOSSARY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md); do not extrapolate to other Buddhist vocabulary.
- Do not describe BWOC as a Buddhist practice, meditation tool, or religious framework. The Pali terms are engineering labels. The framework is a software system.

**Workspace root:**
- The `bwoc-handbook` workspace root is not a git repository. Only `projects/bwoc-framework` (the framework clone) is a git repo. Do not describe workspace-level git operations as valid at the handbook root.

**`applications/` directory:**
- The `applications/` directory in the framework repo is an empty placeholder for Phase 4. Do not describe it as containing any content.

---

## 7. How to Use `llms.txt` and the Glossary

**`llms.txt`** (at `../llms.txt` relative to this file) is the machine index for this corpus. It follows the `llms.txt` convention: a summary paragraph, then sections of named links to every handbook page and every canonical framework doc. Use it to enumerate the corpus before fetching individual pages.

**`../glossary.en.md`** is the handbook-level glossary: quick one-line definitions of every term a reader might encounter. For the full engineering treatment of each Pali term (which framework it belongs to, how it composes, what it determines in code), follow the link to [`docs/en/GLOSSARY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) and then to [`modules/agent-template/docs/en/PHILOSOPHY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) for the full 22-framework mapping.

When building a retrieval index over this corpus:
1. Start with `llms.txt` to enumerate pages.
2. For any topic in the canonical source map (Section 2), fetch the linked GitHub document, not the handbook page.
3. For version-sensitive claims, fetch `VERSION.md` at index time and re-fetch on every query where the version matters.
4. Apply the hallucination trap list (Section 6) as post-generation filters or as negative examples during fine-tuning.

---

## Go Deeper

- All handbook pages (role index): [`../README.md`](../README.md)
- Handbook glossary: [`../glossary.en.md`](../glossary.en.md)
- Machine index: [`../llms.txt`](../llms.txt)
- Agent authoring: [`../agents/HANDBOOK.en.md`](../agents/HANDBOOK.en.md)
- Developer guide: [`../developer/HANDBOOK.en.md`](../developer/HANDBOOK.en.md)
