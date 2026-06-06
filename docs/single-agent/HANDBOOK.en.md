# Single-Agent Workspace

🇹🇭 [ภาษาไทย](HANDBOOK.th.md)

Not every project needs a whole fleet. BWOC has a **single-agent** mode: a workspace built around **one** agent instead of the multi-agent fleet default. This chapter covers when to use it, how to create it, and how it relates to the fleet.

> Terminology: BWOC agents always live inside a **workspace**. There is no machine-wide "global agent" concept — a one-agent setup is a **single-agent workspace**. If you want one agent reachable from other workspaces/machines, that is the [cross-workspace](../cross-workspace/HANDBOOK.en.md) story (peers / A2A), not this one.

## When to use single-agent

| Use single-agent when… | Use the fleet (default) when… |
|---|---|
| One focused assistant for one project/repo | Many agents with distinct roles |
| Simple setup, no teams/coordination | You need teams, shared task lists, peer review |
| Learning BWOC / a quick experiment | A security team, pipelines, division of labour |
| CI / read-only inspection of a workspace | Ongoing multi-agent operations |

## Create it

```bash
bwoc init ./my-ws --single-agent     # one-agent workspace (profile: single-agent)
cd my-ws
```

What `--single-agent` changes vs the fleet default:

- The workspace **profile** is recorded as `single-agent` (the default is `fleet`).
- The `agents/` directory is scaffolded with **single-agent-oriented guidance** instead of the fleet README.
- Everything else is a normal workspace: `.bwoc/workspace.toml`, the `agents.toml` registry, memory, etc.

> For CI / inspection-only workspaces that never spawn a daemon, add `--no-runtime` (omits the daemon-ephemeral `.gitignore` patterns; `bwoc check` still passes). It composes with `--single-agent`.

## Add and drive your one agent

```bash
bwoc new sage --role "project assistant" --target agents/agent-sage
bwoc check agents/agent-sage
```

Then use the full single-agent toolset — no team/peer needed:

```bash
bwoc chat agent-sage                     # interactive
bwoc run  agent-sage --task "สรุปงานวันนี้"   # headless
bwoc spawn agent-sage --backend ollama   # pick any backend
bwoc memory put note "..."; bwoc memory search "..."   # workspace memory
bwoc status ; bwoc log agent-sage -f      # observe
```

Authoring its identity (persona / mindsets / skills) and memory works exactly as in the [Agents](../agents/HANDBOOK.en.md) and [Persona·Mindsets·Skills](../slots/HANDBOOK.en.md) chapters — a single-agent workspace just has one of them.

## Relationship to the fleet

A single-agent workspace is **the same on-disk structure** as a fleet workspace — the flag mainly sets the profile and the scaffolded guidance. That means:

- **Grow later:** add more agents with `bwoc new` whenever the work outgrows one agent; teams (`bwoc team`) and shared tasks (`bwoc task`) are available the moment you have more than one.
- **Stay lean:** if one agent is enough, keep it — *Mattaññutā* (right amount): don't add a fleet you don't need.
- **Share across machines:** to expose your one agent to other workspaces, use [`bwoc peer`](../cross-workspace/HANDBOOK.en.md) and the A2A protocol — that's federation, not a "global" install.

## See also

- [Quickstart](../quickstart/HANDBOOK.en.md) · [End-user](../end-user/HANDBOOK.en.md) · [Agents](../agents/HANDBOOK.en.md) · [Cross-workspace](../cross-workspace/HANDBOOK.en.md)
- Canonical: [`WORKSPACE.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/WORKSPACE.en.md) · [`INCARNATION.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/INCARNATION.en.md)
