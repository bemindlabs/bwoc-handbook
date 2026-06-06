---
title: Connecting Workspaces & Agents Across Machines
nav_order: 12
---

# Connecting Workspaces & Agents Across Machines

> Thai version: [HANDBOOK.th.md](HANDBOOK.th.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · Glossary: [../glossary.en.md](../glossary.en.md) · Agents handbook: [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md) · Security handbook: [../security/HANDBOOK.en.md](../security/HANDBOOK.en.md)

**Source of truth.** The canonical specification lives in the framework repo. On any conflict, the repo wins and this handbook has a bug. Relevant crates: [`bwoc-a2a`](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-a2a) · [`bwoc-mqtt`](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-mqtt) · docs: [`A2A.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/A2A.en.md) · [`SIGNING.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) · [`PEERS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PEERS.en.md)

---

## Table of Contents

1. [Why Cross-Workspace?](#1-why-cross-workspace)
2. [The Trust Foundation — Signed Envelopes](#2-the-trust-foundation--signed-envelopes)
3. [Peers and the `bwoc peer` Command](#3-peers-and-the-bwoc-peer-command)
   - [3.1 Declaring a Peer in routes.toml](#31-declaring-a-peer-in-routestoml)
   - [3.2 bwoc peer list](#32-bwoc-peer-list)
   - [3.3 bwoc peer status](#33-bwoc-peer-status)
   - [3.4 bwoc peer learn](#34-bwoc-peer-learn)
   - [3.5 bwoc peer feedback](#35-bwoc-peer-feedback)
4. [A2A Protocol — Agent2Agent Interop](#4-a2a-protocol--agent2agent-interop)
   - [4.1 What bwoc-a2a Gives You](#41-what-bwoc-a2a-gives-you)
   - [4.2 Agent Card](#42-agent-card)
   - [4.3 JSON-RPC Message and Task Handlers](#43-json-rpc-message-and-task-handlers)
   - [4.4 The HTTP Listener](#44-the-http-listener)
   - [4.5 Dependency Quarantine](#45-dependency-quarantine)
5. [MQTT Transport](#5-mqtt-transport)
   - [5.1 Publishing an Envelope](#51-publishing-an-envelope)
   - [5.2 Subscribing — Delivering to inbox.jsonl](#52-subscribing--delivering-to-inboxjsonl)
6. [Which Transport When?](#6-which-transport-when)
7. [See Also](#7-see-also)

---

## 1. Why Cross-Workspace?

A single workspace is enough for most teams. But some real problems only appear when agents from different machines — or different organizations — need to coordinate:

- An agent in one workspace must see the task list or shared knowledge of a specialist agent that lives elsewhere.
- A pipeline on a remote machine produces output that a local agent should consume.
- Two teams want to exchange verified feedback on each other's agents without merging their workspaces.
- A fully automated deployment means no human can pass messages manually between services.

BWOC solves all four cases with a single unified idea: **signed envelopes routed over one of three transports**. The transports differ in latency, topology, and setup cost. The trust model is identical across all of them.

---

## 2. The Trust Foundation — Signed Envelopes

Every cross-workspace message in BWOC is a **signed envelope**: a JSON structure that carries a payload, the sender's public key, and an [ed25519](https://ed25519.cr.yp.to/) signature over the payload. A recipient workspace never processes a cross-workspace message before verifying that signature.

Generate a keypair for your workspace:

```bash
bwoc trust --keygen
```

This writes the private key into the workspace credential store and records the public key in `.bwoc/identity.toml`. Share the public key with any workspace you want to communicate with; they add it to their peer declaration (see [Section 3.1](#31-declaring-a-peer-in-routestoml)).

The framework's trust scoring for inter-agent relationships uses the **Kalyanamitta-7** model — seven qualities that determine whether a peer agent is considered a "good friend" (trustworthy collaborator). Cross-workspace trust is one of those signals. The full rubric is in the [security handbook](../security/HANDBOOK.en.md) and [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md).

> **Rule.** If a message arrives without a valid signature, or if the signing key is not in the recipient's peer allowlist, the envelope is dropped silently and logged. No exception.

---

## 3. Peers and the `bwoc peer` Command

A **peer** is a read-only cross-workspace view. You declare that a remote workspace exists; your workspace can then observe its agents, tasks, and shared documents — but it cannot write into that workspace directly. Writing happens through the signed feedback mechanism ([Section 3.5](#35-bwoc-peer-feedback)) or through A2A/MQTT.

Peer declarations live in `.bwoc/routes.toml` in your workspace root.

### 3.1 Declaring a Peer in routes.toml

```toml
# .bwoc/routes.toml

[[peer]]
key        = "oracle"                        # short local alias you use in bwoc commands
name       = "Oracle Workspace"              # human label
url        = "https://oracle.example.com"   # HTTP base URL of the peer's A2A listener
# OR for local / LAN:
# path     = "/Users/shared/oracle-workspace"
pubkey     = "ed25519:<base64-encoded-public-key>"
allowlist  = true    # whether to honour their shared.toml doc allowlist
```

The `key` is the alias you use in every `bwoc peer` command. The `pubkey` must match the key the peer workspace published with `bwoc trust --keygen`. The framework validates this at connection time.

### 3.2 bwoc peer list

```bash
bwoc peer list
```

Lists every peer declared in this workspace's `routes.toml`, with their key, name, URL or path, and reachability status (reachable / unreachable / unknown).

```
KEY       NAME                STATUS
oracle    Oracle Workspace    reachable
staging   Staging Fleet       unreachable (last seen 2h ago)
```

### 3.3 bwoc peer status

```bash
bwoc peer status oracle
```

Shows the live state of that peer workspace: which agents are registered, which teams exist, and which tasks are currently open on each team's shared task list. This is a **read-only snapshot** — you cannot modify anything through this command.

```
Peer: oracle  (Oracle Workspace)
Agents:        agent-wenchang, agent-guanyin, agent-nezha
Teams:         research (3 tasks open), ops (1 task open)
Last verified: 2026-06-07T10:04:33Z
```

### 3.4 bwoc peer learn

```bash
bwoc peer learn oracle           # list documents the peer has shared
bwoc peer learn oracle DESIGN    # view a specific shared document by name
```

A peer workspace controls exactly which documents it shares by listing them in its own `.bwoc/interconnect/shared.toml`. Your workspace can only read what is on that allowlist — the peer's private files are never exposed.

`bwoc peer learn` without a document name prints the allowlist: document name, one-line description, and last-modified date. Supply a document name to fetch and display the full text in your terminal.

This is how knowledge spreads between organizations without merging repositories: the originating workspace decides the boundary; your workspace pulls at will.

### 3.5 bwoc peer feedback

```bash
bwoc peer feedback oracle agent-wenchang \
  --score 4 \
  --comment "Excellent root-cause analysis on the deploy failure."
```

Sends a `kind: feedback` signed envelope to the target peer workspace, addressed to a specific agent's inbox. The envelope is signed with your workspace's private key. The receiving workspace verifies the signature against your declared public key before accepting it; if verification fails, the envelope is dropped.

Once accepted, the feedback lands in the target agent's `memories/inbox.jsonl`. The agent's operator can review, curate, or promote it as part of the [self-improvement cycle](../self-improvement/HANDBOOK.en.md).

> **Why the signature matters here.** Unsigned or unverifiable feedback is trivially spoofed. The ed25519 signature ensures the receiving agent knows exactly which workspace sent the rating and cannot be misled by an impersonator.

---

## 4. A2A Protocol — Agent2Agent Interop

Peers give you a read-only, human-initiated view across workspaces. A2A is the **machine-to-machine protocol** for agents that need to exchange tasks and messages programmatically, including agents from entirely different organizations running different software.

The `bwoc-a2a` crate implements the [Agent2Agent protocol](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-a2a), pinned to **A2A spec v1.0.0**.

### 4.1 What bwoc-a2a Gives You

| Capability | What it does |
|---|---|
| **Agent Card** | Machine-readable identity document your agent serves at a known URL so peers can discover capabilities |
| **JSON-RPC handlers** | Receive `message/send` and `task/send` calls; dispatch to the right agent; return results |
| **HTTP listener** | An [axum](https://github.com/tokio-rs/axum)-based server that wires all of the above together |
| **Envelope signing/verification** | Every request and response is a signed BWOC envelope |

### 4.2 Agent Card

An Agent Card is a JSON document served at a stable URL (by convention `/.well-known/agent.json` on your A2A listener). It describes:

- The agent's name and version
- The A2A spec version it implements (`"1.0.0"`)
- The capabilities it accepts (message types, task schemas)
- The public key used to verify its signatures

A remote agent fetches your Agent Card to decide whether it can interoperate with you before sending a single message. You do not need to configure this manually — `bwoc-a2a` generates the card from the agent's `config.manifest.json` and the workspace identity.

### 4.3 JSON-RPC Message and Task Handlers

A2A communication is [JSON-RPC 2.0](https://www.jsonrpc.org/specification). The two core method families are:

**`message/send`** — fire-and-forget or request-reply messages. Used when an agent wants to inform another of an event, share a result, or ask a question without a persistent task context.

**`task/send`** — creates or updates a task in the receiving agent's context. The receiver can accept, reject, or return partial results. Used for longer-running coordination where state must persist across turns.

Both methods carry a signed BWOC envelope as their `params` payload. The handler verifies the signature before doing any work.

### 4.4 The HTTP Listener

The `bwoc-a2a` axum listener exposes:

```
GET  /.well-known/agent.json   → Agent Card
POST /a2a                      → JSON-RPC endpoint (message/send, task/send, ...)
```

Start it from the CLI (the exact subcommand is documented in [`A2A.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/A2A.en.md)):

```bash
bwoc a2a serve --agent agents/agent-wenchang --port 8080
```

Put a TLS-terminating proxy (nginx, Caddy, Cloudflare) in front for production. The listener itself speaks plain HTTP; TLS is a network-layer concern.

### 4.5 Dependency Quarantine

`bwoc-a2a` is deliberately isolated from `bwoc-core`. The rule is: **all HTTP and network dependencies live in `bwoc-a2a`, never in `bwoc-core`**. This keeps the core crate lean and prevents network-stack transitive dependencies from leaking into workloads that do not need them.

If you are writing a workspace tool that only needs local agent operations, depend only on `bwoc-core`. Pull in `bwoc-a2a` only when you need A2A HTTP capability.

---

## 5. MQTT Transport

MQTT is the right choice when agents communicate over a shared message broker rather than point-to-point HTTP. It fits:

- IoT or embedded fleet scenarios where devices publish events to a broker
- Low-latency local networks where a Mosquitto broker is already running
- Fan-out patterns where one sender needs many receivers

The `bwoc-mqtt` crate handles both directions. See the crate reference at [`bwoc-mqtt`](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-mqtt).

### 5.1 Publishing an Envelope

```bash
bwoc mqtt publish \
  --broker mqtt://192.168.1.113:1883 \
  --topic  bwoc/oracle/agent-wenchang/inbox \
  --agent  agents/agent-guanyin \
  --payload '{"kind":"message","body":"Deploy complete."}'
```

The CLI signs the envelope with the sending agent's key, then publishes it to the specified topic. The receiving broker holds the message until a subscriber consumes it.

Topics follow the convention `bwoc/<workspace-key>/<agent-name>/<slot>` — you can subscribe at any level of that hierarchy.

### 5.2 Subscribing — Delivering to inbox.jsonl

```bash
bwoc mqtt subscribe \
  --broker mqtt://192.168.1.113:1883 \
  --topic  bwoc/my-workspace/agent-wenchang/inbox \
  --agent  agents/agent-wenchang
```

The subscriber runs continuously. Each arriving message is verified (signature check) and, if valid, appended to the target agent's `memories/inbox.jsonl`. Invalid or unverifiable envelopes are logged and dropped.

Run the subscriber as a background service (systemd unit, launchd plist, or a tmux session) on the machine that hosts the agents.

---

## 6. Which Transport When?

| Situation | Recommended approach |
|---|---|
| You want to observe a peer workspace's agents and tasks without automating anything | `bwoc peer status` / `bwoc peer learn` |
| You want to give rated feedback to a peer agent | `bwoc peer feedback` |
| Two agents on different machines need to exchange tasks programmatically | A2A (`bwoc-a2a`, JSON-RPC over HTTP) |
| You need a stable, discoverable endpoint so external systems can find your agent | A2A — serve the Agent Card at `/.well-known/agent.json` |
| Agents communicate via a shared broker (IoT fleet, LAN, fan-out) | MQTT (`bwoc-mqtt`) |
| High-volume, low-latency, or many-to-many messaging | MQTT over a dedicated broker |
| Simple async notification from a remote script into an agent's inbox | MQTT publish (one command, no persistent listener needed on the sender side) |

The peer commands and A2A are complementary: `bwoc peer` is a human-facing CLI for inspection and feedback; A2A is a machine-to-machine wire protocol for programmatic coordination. MQTT replaces A2A when broker topology is preferred over direct HTTP.

> **All three share the same trust model.** Signed ed25519 envelopes, verified before acceptance. The transport changes; the security guarantee does not.

---

## 7. See Also

- [Security handbook](../security/HANDBOOK.en.md) — key generation, Kalyanamitta-7 trust scoring, threat model
- [Agents handbook](../agents/HANDBOOK.en.md) — agent layout, `interconnect/`, inbox
- [Self-improvement handbook](../self-improvement/HANDBOOK.en.md) — how feedback in `inbox.jsonl` is curated and promoted
- [Glossary](../glossary.en.md) — Kalyanamitta 7, Sammā-vācā, Saṅgha
- Framework source: [`bwoc-a2a` crate](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-a2a) · [`bwoc-mqtt` crate](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-mqtt)
- Framework docs: [`A2A.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/A2A.en.md) · [`SIGNING.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) · [`PEERS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PEERS.en.md)
