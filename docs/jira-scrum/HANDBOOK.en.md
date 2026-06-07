# Jira & Scrum with BWOC

> Language toggle: **English (canonical)** | [ภาษาไทย](HANDBOOK.th.md)
>
> This chapter covers the two pieces that together give your BWOC agents native Jira integration and scrum vocabulary: the `jira-cloud-rest` plugin and the `scrum-via-jira` skill. Read the plugin section to understand the integration layer; read the skill section to see what your agent can actually do with it.

---

## Table of Contents

1. [The two pieces and how they fit together](#1-the-two-pieces-and-how-they-fit-together)
2. [The jira-cloud-rest plugin — the integration layer](#2-the-jira-cloud-rest-plugin--the-integration-layer)
   - [2.1 Install and enable](#21-install-and-enable)
   - [2.2 Configuration — the project key](#22-configuration--the-project-key)
   - [2.3 Credentials — BWOC_JIRA_* and secrets.toml](#23-credentials--bwoc_jira_-and-secretstoml)
   - [2.4 Verbs the plugin exposes](#24-verbs-the-plugin-exposes)
   - [2.5 Bidirectional sync and gated transitions](#25-bidirectional-sync-and-gated-transitions)
3. [The scrum-via-jira skill — the agent vocabulary](#3-the-scrum-via-jira-skill--the-agent-vocabulary)
   - [3.1 Enable the skill on an agent](#31-enable-the-skill-on-an-agent)
   - [3.2 The six scrum verbs](#32-the-six-scrum-verbs)
4. [The kind-dependency model — swap the adapter without touching the skill](#4-the-kind-dependency-model--swap-the-adapter-without-touching-the-skill)
5. [Sprint workflow example](#5-sprint-workflow-example)
6. [See also](#6-see-also)

---

## 1. The two pieces and how they fit together

Running a Jira-backed scrum workflow in BWOC takes two modules working at different levels:

| Module | Type | Layer | Who turns it on |
|---|---|---|---|
| `jira-cloud-rest` | Plugin (`kind = "jira"`) | Framework integration — owns REST, auth, JQL, rate limits, the sync ledger | Workspace operator in `workspace.toml` |
| `scrum-via-jira` | Skill (maturity L1) | Agent capability — owns scrum semantics (sprints, stories, backlog) | Agent author in `config.manifest.json` |

The plugin is the HTTP adapter. It knows Atlassian Cloud REST v3, how to authenticate, how to scope JQL to your project, how to handle `429` backoff, and how to write the sync ledger at `.scrum/jira-sync.json`. It does not know anything about scrum.

The skill is the scrum vocabulary layer. It knows what "open a sprint" or "transition a story" means, and it delegates the mechanical part — the actual HTTP call — to whatever `jira`-kind plugin is enabled in the workspace. It does not hold credentials and does not touch the sync ledger directly.

The dependency is one-way: the skill calls the plugin's verbs; the plugin has no knowledge of the skill. A `jira`-kind plugin works perfectly well with no skill present — the operator can drive everything through the `bwoc jira` CLI directly.

---

## 2. The jira-cloud-rest plugin — the integration layer

`jira-cloud-rest` is the reference implementation of the `jira` plugin kind. It is the framework's first write-capable plugin: unlike audit or reporting plugins that only read, this one reads **and** writes an external system of record — Jira Cloud — which is why writes are gated behind operator confirmation.

Source: [modules/plugins/jira-cloud-rest](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/jira-cloud-rest)

### 2.1 Install and enable

If the plugin is not already installed under `modules/plugins/jira-cloud-rest/` in your workspace, install it:

```bash
bwoc plugin install https://github.com/bemindlabs/BWOC-Framework.git#main
# or from a local path if you have the framework cloned:
bwoc plugin install ./projects/bwoc-framwork/modules/plugins/jira-cloud-rest/
```

Then enable it by adding a block to your `workspace.toml`:

```toml
[plugins.jira-cloud-rest]
enabled = true
project = "MYPROJ"   # your Jira project key — the only config field
```

Verify the manifest passes the neutrality and schema checks:

```bash
bwoc check --all
```

### 2.2 Configuration — the project key

The only declared configuration key is `project` — the Jira project key that every read is scoped to (e.g. `"BWOC"`, `"MYTEAM"`). This is not optional. Every JQL query the plugin issues is automatically bounded to this project; any query that tries to escape it is rejected or augmented.

There is intentionally nothing else in config. Credentials are not config — they belong in the environment or a gitignored secrets file (see below).

### 2.3 Credentials — BWOC_JIRA_* and secrets.toml

The plugin authenticates with Atlassian Cloud using HTTP Basic: your account email as the username and an API token as the password, against your site's base URL.

Three environment variables carry these inputs. The plugin reads them at runtime — the first hit wins:

| Variable | What it carries |
|---|---|
| `BWOC_JIRA_EMAIL` | Your Atlassian account email |
| `BWOC_JIRA_TOKEN` | Your Atlassian API token — this is the secret |
| `BWOC_JIRA_BASE_URL` | Your site URL, e.g. `https://myteam.atlassian.net` |

If environment variables are not set, the plugin falls back to `.bwoc/secrets.toml` — a gitignored, `chmod 600` file in your workspace root. Set it up like this:

```toml
# .bwoc/secrets.toml — never commit this file
[jira]
email    = "you@example.com"
token    = "your-api-token"
base_url = "https://myteam.atlassian.net"
```

The token is passed directly to `curl -u` and is never echoed, never written to any output file, and never included in JSON responses. If any of the three inputs is missing, the plugin fails fast with a clear diagnostic naming which variable is absent — it never proceeds with incomplete auth.

To generate an API token, go to id.atlassian.com > Security > API tokens.

### 2.4 Verbs the plugin exposes

The `bwoc jira` CLI dispatches three live verbs to the plugin adapter:

| Verb | Direction | What it does | Gate |
|---|---|---|---|
| `query` | read | Executes project-scoped JQL against `/rest/api/3/search/jql`; returns paginated issues in the normative Issue Mapping Schema | none — reads are free |
| `transition` | write | Reads the issue's current status, then posts to `/transitions` to move it to the target status; idempotent (already-at-target is a no-op success) | operator confirmation required |
| `sync` | read/write | Reads the sync ledger and reconciles it with Jira; `--dry-run` shows the planned changes without applying them | apply is gated |

Two additional verbs (`link`, `unlink`, `status`) are offline — they touch only the local sync ledger and never reach the adapter.

Result sets from `query` are capped at 100 items per page. The response includes a `next_page_token` and `is_last` flag for future paging.

### 2.5 Bidirectional sync and gated transitions

The `jira` kind is the first BWOC plugin kind that writes to an external system. This earns it two structural controls:

**Operator-confirmation gate.** Every write verb (`transition`, `sync` apply) requires explicit confirmation at the `bwoc jira` CLI before the adapter is invoked. The gate shows the exact effect — the target issue, the current status, the target status — before acting. Default is No. A headless agent must pass `--yes` explicitly, and only when the operator has authorized that specific action.

**Sync ledger.** The plugin maintains `.scrum/jira-sync.json`, a mapping between your local scrum stories and Jira issues. Each entry records the `issue_key`, `project`, `summary`, `status`, and a `last_synced` watermark. The watermark drives per-field last-writer-wins conflict detection. A `401`/`403` from Jira is surfaced as "rotate your token" — it never creates a spurious sync conflict.

Sync in v0.1.0 establishes the contract, auth, and read path; the full bidirectional resolution engine is deferred to a later release. The envelope shape is final so the CLI contract is stable now.

---

## 3. The scrum-via-jira skill — the agent vocabulary

`scrum-via-jira` is the framework's first skill-on-plugin skill. It gives an agent six higher-level scrum operations that each delegate their underlying HTTP work to a `jira`-kind plugin via `bwoc jira` verbs. The skill owns scrum semantics; the plugin owns the integration.

Source: [modules/skills/scrum-via-jira](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/skills/scrum-via-jira)

### 3.1 Enable the skill on an agent

Add the skill to the agent's `config.manifest.json`:

```json
{
  "skills": {
    "framework": [
      { "name": "scrum-via-jira", "version": ">=0.1.0", "enabled": true }
    ]
  }
}
```

At spawn, the framework checks that an enabled `jira`-kind plugin exists in the workspace. If no `jira`-kind plugin is enabled, spawn fails immediately with a diagnostic naming the missing kind. The agent is never partially wired.

Run the verify gate before spawn to catch this earlier:

```bash
bwoc skill verify scrum-via-jira
```

### 3.2 The six scrum verbs

The skill exposes six operations. Two are read-only and carry no gate; four are writes that inherit the plugin's operator-confirmation gate.

| Verb | Scrum intent | Plugin verbs used | Direction | Gate |
|---|---|---|---|---|
| `propose-sprint` | Gather candidate backlog stories and emit a proposed sprint composition | `bwoc jira query` | read | none |
| `open-sprint` | Activate the agreed sprint; assign its stories | `bwoc jira link` then `bwoc jira sync` | write | operator confirmation |
| `transition-story` | Move one story across the scrum lifecycle (`backlog → in_progress → done`) | `bwoc jira transition` | write | operator confirmation |
| `sync-backlog` | Reconcile the backlog with Jira; pull external changes and push local ones | `bwoc jira sync` (`--dry-run` previews first) | read/write | apply is gated |
| `close-sprint` | Finalize a sprint; transition remaining stories and record the close | `bwoc jira transition` + `bwoc jira sync` | write | operator confirmation |
| `list-active-sprints` | List open sprints for the configured project | `bwoc jira query` | read | none |

The skill adds no second gate on top of the plugin's gate, and it removes none. Write verbs carry exactly one confirmation point — at the `bwoc jira` CLI boundary.

The skill never touches `.scrum/jira-sync.json` directly and never holds a credential. It reaches the sync ledger only transitively through the plugin. Credentials resolve inside the plugin from `BWOC_JIRA_*` env or `.bwoc/secrets.toml`, never through the skill.

At `init`, the skill resolves the `jira`-kind plugin dependency and caches the dispatch handle. No REST calls are made at init — the agent is never half-wired and init is always fast. Teardown is a no-op; the skill holds no external state.

---

## 4. The kind-dependency model — swap the adapter without touching the skill

`scrum-via-jira` declares its dependency in `manifest.toml` as:

```toml
[contract]
requires         = []
requires_plugins = ["jira"]
```

The value `"jira"` is a plugin **kind**, not a plugin name. The skill does not know about `jira-cloud-rest` specifically. It depends on any enabled plugin whose manifest declares `kind = "jira"`.

This means:

- Today the workspace runs `jira-cloud-rest` (Atlassian Cloud REST v3).
- Tomorrow someone ships `jira-server-rest` for self-hosted Jira Server — or a `jira-mock` for testing.
- Swapping the adapter is a workspace config change: disable the old plugin, enable the new one, no changes to the skill or to any agent that uses it.

The separation is enforced structurally. `requires_plugins` names kinds (from the [Plugin Kinds](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) enum); `requires` names skill names. A bare `"jira"` cannot be ambiguous — the two fields have different namespaces. `bwoc check` validates at manifest-read time that every value in `requires_plugins` is a valid kind enum, without requiring the plugin to be enabled yet. Enablement is checked at agent spawn.

The full model — why a dedicated field beats overloading `requires` — is in [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md).

---

## 5. Sprint workflow example

This example shows an agent running a single sprint from proposal through close. The operator confirms each write.

**Before you start**, ensure:

1. `jira-cloud-rest` is enabled in `workspace.toml` with your `project` key set.
2. `BWOC_JIRA_EMAIL`, `BWOC_JIRA_TOKEN`, and `BWOC_JIRA_BASE_URL` are in the environment (or in `.bwoc/secrets.toml`).
3. `scrum-via-jira` is enabled in the agent's `config.manifest.json`.
4. `bwoc skill verify scrum-via-jira` exits 0.

```bash
# 1. Check what sprints are already running
bwoc jira query --jql "sprint in openSprints()"

# 2. Propose a sprint — the agent reads the backlog and drafts a composition
#    (propose-sprint → bwoc jira query, read-only, no gate)
bwoc skill invoke scrum-via-jira propose-sprint

# 3. Open the sprint — assigns the selected stories to the sprint
#    (open-sprint → bwoc jira link + sync, write, gated)
bwoc skill invoke scrum-via-jira open-sprint --stories MYPROJ-10,MYPROJ-11,MYPROJ-12

# 4. During the sprint — move a story to in-progress
#    (transition-story → bwoc jira transition, write, gated)
bwoc skill invoke scrum-via-jira transition-story --story MYPROJ-10 --to in_progress

# 5. Pull in any status changes made directly in Jira by teammates
#    (sync-backlog → bwoc jira sync --dry-run first, then apply, gated)
bwoc skill invoke scrum-via-jira sync-backlog

# 6. Close the sprint when all stories are done
#    (close-sprint → bwoc jira transition + sync, write, gated)
bwoc skill invoke scrum-via-jira close-sprint
```

At steps 3, 4, 5 (apply), and 6, the `bwoc jira` CLI shows the exact planned change and prompts `y/N` before touching Jira. In a headless agent flow, the operator pre-authorizes these steps with `--yes` in the invocation context.

**If a transition is already at the target status** (e.g., the story was moved in Jira before `transition-story` ran), the adapter detects this and returns a no-op success — no duplicate transition, no error.

**If Jira returns a `429 Too Many Requests`**, the adapter honors the `Retry-After` header and retries up to four times before surfacing a retryable error. All write verbs are idempotent, so retrying after a transient failure is safe.

---

## 6. See also

**Framework source (public GitHub):**

- [jira-cloud-rest plugin](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/jira-cloud-rest) — manifest, SPEC, and `jira.sh` entry point
- [scrum-via-jira skill](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/skills/scrum-via-jira) — manifest and SPEC
- [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) — the `jira` kind row, the Issue Mapping Schema, the write-verb operator-confirm gate contract
- [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) — the skill-on-plugin dependency model; `requires_plugins`

**Sibling handbook chapters:**

- [../slots/HANDBOOK.en.md](../slots/HANDBOOK.en.md) — how to add and configure skills in an agent's slot files
- [../glossary.en.md](../glossary.en.md) — term lookup including plugin, skill, kind, maturity

---

*This handbook is maintained alongside the framework. When behavior described here diverges from the [framework repo](https://github.com/bemindlabs/BWOC-Framework), the repo is correct — please open a fix.*
