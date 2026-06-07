# Chat Connectors — Telegram & Discord

🇹🇭 [ภาษาไทย](HANDBOOK.th.md)

For people who want to talk to a BWOC agent directly from Telegram or Discord, without opening a terminal. This chapter covers how `bwoc-connect` works, how to configure secrets, how to run and supervise the connector, and what the security boundaries are.

Term lookup: [`../glossary.en.md`](../glossary.en.md) · Crate source: [bwoc-connect](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-connect) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

## Table of Contents

1. [Why a Chat Connector?](#1-why-a-chat-connector)
2. [Supported Platforms](#2-supported-platforms)
   - [2.1 Telegram (DM and group)](#21-telegram-dm-and-group)
   - [2.2 Discord (DM and server channel)](#22-discord-dm-and-server-channel)
3. [Setup](#3-setup)
   - [3.1 Create a bot and get a token](#31-create-a-bot-and-get-a-token)
   - [3.2 Write the connector config](#32-write-the-connector-config)
   - [3.3 Store the token securely](#33-store-the-token-securely)
   - [3.4 Run the connector](#34-run-the-connector)
   - [3.5 Daemon supervision via bwoc-agent --serve](#35-daemon-supervision-via-bwoc-agent---serve)
4. [How Messages Flow](#4-how-messages-flow)
   - [4.1 DM path](#41-dm-path)
   - [4.2 Group / server-channel path](#42-group--server-channel-path)
5. [Security](#5-security)
6. [Limitations and Version Note](#6-limitations-and-version-note)
7. [See Also](#7-see-also)

---

## 1. Why a Chat Connector?

An agent running under a vendor CLI or `bwoc-harness` is normally reached from a terminal: you type `bwoc chat`, get a TUI, and exchange messages there. That is efficient for a developer but awkward for anyone else on the team who does not live in a terminal.

`bwoc-connect` is the bridge that moves the agent's conversation surface into a chat app that everyone already has open. You get the same BWOC agent — same memory, same skills, same tool access — reachable from a Telegram DM or a Discord server channel. No extra interface to learn; no terminal required for the end user.

The connector is a separate binary, not part of `bwoc-cli` or `bwoc-agent`. Network dependencies (reqwest, tokio-tungstenite) are quarantined there so the core framework stays lean.

---

## 2. Supported Platforms

### 2.1 Telegram (DM and group)

Telegram is the primary platform. The connector uses the **Bot API long-poll** (`getUpdates` with a 25-second timeout): it sends a request, waits for new messages, processes them, advances the offset, and repeats — no webhook, no inbound port required.

**DMs.** A user messages the bot directly. Each DM conversation is isolated: the connector holds one agent session per `chat_id` so your conversation thread is never mixed with another user's.

**Groups and supergroups.** An agent can join a group and participate in a Saṅgha team chat. Group behaviour depends on `mention_only` in the config:

- **Mention-gated (default, `mention_only = true`).** A message that @mentions the bot starts a real agent turn; the agent reads the room's peer context (other people's messages) and replies. A message that does not mention the bot is silently logged to the team's `chat.jsonl` as peer context for the agent's next turn — no reply is sent.
- **Open (`mention_only = false`).** Every message from an allow-listed sender triggers an agent turn.

The connector resolves the bot's `@username` at startup (via `getMe`) so it can detect mentions accurately. Channels (broadcast-only) and anonymous posts are ignored.

### 2.2 Discord (DM and server channel)

Discord does not offer a long-poll API. Instead the connector connects to the **Discord Gateway websocket** (v10), authenticates with IDENTIFY, and sends periodic heartbeats. A background task owns the connection, pushes incoming `MESSAGE_CREATE` events into a queue, and reconnects automatically after any disconnect.

**Intents required on your bot application:** `GUILD_MESSAGES`, `DIRECT_MESSAGES`, `MESSAGE_CONTENT` (privileged).

**DMs.** Works the same as Telegram DMs — one isolated agent session per channel.

**Server channels (guilds).** Behave the same as Telegram groups: `mention_only = true` by default, mention detection uses the structured `mentions[]` array in the Discord payload (more robust than substring matching). Bot-authored messages are automatically ignored to prevent reply loops.

The routing and allow-list logic is identical for both platforms; only the transport layer differs.

---

## 3. Setup

### 3.1 Create a bot and get a token

**Telegram.** Open [@BotFather](https://t.me/BotFather), send `/newbot`, follow the prompts, copy the token (`123456789:ABCdef...`).

**Discord.** Open the [Discord Developer Portal](https://discord.com/developers/applications), create an application, add a Bot, enable the `MESSAGE CONTENT` privileged intent, and copy the bot token. Invite the bot to your server with the `bot` scope and the `Send Messages` + `Read Message History` permissions.

### 3.2 Write the connector config

Create `connectors/<platform>.toml` inside the agent directory (e.g. `agents/agent-sage/connectors/telegram.toml`).

Minimal config (DM only):

```toml
enabled = true

# Platform user ids allowed to reach this agent.
# Empty list = nobody. This is closed by default — add your own id.
allow_from = [123456789]
```

Config with group/server-channel support:

```toml
enabled = true
allow_from = [123456789, 987654321]

[group]
# Saṅgha team whose shared chat.jsonl backs this group room.
team = "my-team"
# Reply only when @mentioned (default true). Set false to reply to every message.
mention_only = true
```

The `team` value must be a plain token (letters, digits, `-`, `_` only). It maps to `.bwoc/teams/<team>/chat.jsonl` in the workspace root.

Both `telegram.toml` and `discord.toml` use this same config shape.

### 3.3 Store the token securely

**macOS and Windows.** The connector reads the token from the OS native keyring first. The keyring service is `bwoc/<platform>` and the account name is the agent directory's basename.

Example (macOS, Telegram, agent named `agent-sage`):

```sh
security add-generic-password -a agent-sage -s bwoc/telegram -w "YOUR_TOKEN"
```

**Linux and fallback on all platforms.** Set the environment variable before running the connector:

```sh
export TELEGRAM_BOT_TOKEN="YOUR_TOKEN"
# or for Discord:
export DISCORD_BOT_TOKEN="YOUR_TOKEN"
```

If neither the keyring entry nor the environment variable is present, `bwoc-connect` exits with a clear error message naming the expected variable.

### 3.4 Run the connector

```sh
bwoc-connect telegram --agent agents/agent-sage
# or
bwoc-connect discord --agent agents/agent-sage
```

The connector logs to stderr: which platform, which agent, the allow list, and (if configured) the team chat log path. The main loop runs until interrupted.

For manual smoke-testing, `--max-polls N` limits the polling loop to N iterations and then exits cleanly. This flag is not for production use.

### 3.5 Daemon supervision via bwoc-agent --serve

Running `bwoc-connect` as a long-lived foreground process is fine for development. For production, use `bwoc-agent --serve` from the agent directory:

```sh
bwoc-agent --serve --workdir agents/agent-sage
```

The `ConnectorSupervisor` inside `bwoc-agent` detects which connector config is enabled, locates the `bwoc-connect` binary (sibling of the current process, then PATH), spawns it, and keeps it alive:

- If `bwoc-connect` exits for any reason, it is respawned after a 5-second backoff to prevent a crash-loop from spinning hot.
- When `bwoc-agent --serve` shuts down, it kills the connector child and writes a `stopped` status marker.
- The health state (`running`, `exited`, `stopped`) is written atomically to `<agent>/.bwoc/connector.status` so `bwoc status` can surface it.

If `bwoc-connect` is not on PATH and is not a sibling binary, `bwoc-agent --serve` logs a warning and runs without the connector — it does not abort.

---

## 4. How Messages Flow

### 4.1 DM path

```
Platform (Telegram/Discord)
        |
        |  [poll / gateway push]
        v
  bwoc-connect bridge
        |
        |  allow-list check (from_user_id in allow_from?)
        |    rejected → silent drop
        v
  per-chat AgentSession (bwoc-harness --chat subprocess)
        |
        |  ChatInput::User written to harness stdin
        |  ChatEvent stream read from harness stdout
        |    PermissionRequest → auto-denied (remote user cannot approve tools)
        |    ChatEvent::Message → reply text
        v
  Platform send (reply to same chat_id)
```

One `bwoc-harness --chat` subprocess is spawned per `chat_id` on the first message and reused for the lifetime of the conversation. If the harness exits mid-session, the connector drops the dead session and respawns a fresh one on the next message from that chat.

### 4.2 Group / server-channel path

Incoming group messages go through the same allow-list check first.

After that, the connector checks `mentions_bot`:

- **Mentioned (or `mention_only = false`).** The message is the user's turn. A `--team-chat` session (one per group `chat_id`) is spawned or reused, the message is delivered as a `ChatInput::User`, and the agent's reply is sent back to the group.
- **Not mentioned (`mention_only = true` default).** The message is appended to the team's `chat.jsonl` as peer context (`TeamChatMessage` with `from: "tg:<user_id>"` or `from: "dc:<user_id>"`). No harness session is created; no reply is sent. The agent will see this context the next time it is addressed.

The team `chat.jsonl` path is resolved by walking up from the agent directory until a `.bwoc/workspace.toml` marker is found, then joining `.bwoc/teams/<team>/chat.jsonl`. The directory is created automatically if it does not exist.

---

## 5. Security

**Closed by default.** An empty `allow_from` list means nobody can reach the agent, even if the bot is publicly listed on Telegram or added to a Discord server. You must explicitly list numeric platform user ids to open access. This is not configurable away — it is the design.

**Tool calls cannot be approved remotely.** The harness session runs in non-TTY mode. Any `PermissionRequest` event from the agent is automatically denied. A person chatting over Telegram or Discord can never authorize a destructive tool call.

**Token storage.** On macOS and Windows, the token lives in the native keyring (Keychain / Windows Credential Manager), not in a config file. On Linux it is an environment variable; do not hardcode it in shell scripts that are checked into source control.

**Group allow-list.** In a group or server channel, the `allow_from` check applies to the individual sender's user id, not to the group itself. Someone who is not on the list cannot reach the agent even if they are a member of the group — their messages are silently dropped (not even logged as peer context).

---

## 6. Limitations and Version Note

- `bwoc-connect` handles **one enabled connector per agent** in the current daemon supervision model (`bwoc-agent --serve` supervises the first enabled config it finds).
- **Media (images, files, voice messages) are not handled.** Only text messages are relayed. Non-text updates are silently skipped.
- **Slash commands** are not implemented. The bot responds to plain text messages only.
- **Multi-group binding** (one agent bridged to several group rooms at once) is not supported yet.
- **Discord gateway integration** is not verified against a live bot in CI; the live edge is eyeball-reviewed. Treat it as beta.
- The `bwoc-connect` crate version tracks the framework `Software-Version`. Check [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) for the current release.

---

## 7. See Also

- [Self-Hosting with bwoc-harness](../harness/HANDBOOK.en.md) — how the harness subprocess works and what `chat_proto` is
- [Cross-Workspace & Protocols](../cross-workspace/HANDBOOK.en.md) — peer protocol, MQTT, multi-machine agents
- [Fleet Operations](../fleet-ops/HANDBOOK.en.md) — running and supervising many agents with `bwoc-agent --serve`
- [Security & the Tianting Team](../security/HANDBOOK.en.md) — threat model, Sīla 5, signed trust
- [Glossary](../glossary.en.md) — Saṅgha, chat_proto, allow-list, and other terms
- Crate source: [bwoc-connect](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-connect)
