# BWOC + Google Workspace Handbook

🇹🇭 [ภาษาไทย](HANDBOOK.th.md)

Connect a BWOC workspace to Google Workspace — Drive, Gmail, and Calendar — through the built-in `gws` plugin family.
This chapter covers what the plugins give you, how to install and enable them, the available operations, and how credentials are handled safely.

Term lookup: [`../glossary.en.md`](../glossary.en.md) · Plugin source: [bemindlabs/BWOC-Framework — modules/plugins/gws](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/gws) · Plugin spec: [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md)

---

## Table of Contents

1. [What the gws plugins give you](#1-what-the-gws-plugins-give-you)
2. [Install and enable](#2-install-and-enable)
3. [Credentials and secrets](#3-credentials-and-secrets)
4. [Operations reference](#4-operations-reference)
5. [Usage example](#5-usage-example)
6. [Security and OAuth scopes](#6-security-and-oauth-scopes)
7. [See also](#7-see-also)

---

## 1. What the gws plugins give you

The `gws` family is a set of four framework plugins — all of `kind = "gws"` — that let your agents and the `bwoc gws` CLI read Google Workspace data without writing any credential handling themselves.

| Plugin | What it covers |
|---|---|
| `gws-auth` | OAuth2 credential foundation: token resolution, Bearer header, automatic refresh, and the `status` verb that reports auth state without ever printing the token. All sibling plugins source their auth from here. |
| `gws-drive` | Google Drive: list files and read a single file's metadata. |
| `gws-gmail` | Gmail: search threads, show one thread, and list labels. |
| `gws-calendar` | Google Calendar: list calendars and list events on a calendar. |

All four are **read-mostly**. No verb sends mail, creates an event, or uploads a file. Write operations are explicitly deferred to future releases; any future write verb will require an explicit operator confirmation before acting.

The `gws` kind is separate from the `gcloud` family. `gcloud` reaches Google Cloud Platform infrastructure through the local `gcloud` CLI with service-account credentials. `gws` reaches Workspace productivity apps (Drive, Gmail, Calendar) through the Workspace REST APIs with OAuth2 user-consent scopes. They are different authentication families with different surfaces.

---

## 2. Install and enable

### 2.1 Install the plugins

The `gws` plugins ship as part of the framework source. Install each one by pointing `bwoc plugin install` at the local path inside your framework clone, or from the public Git URL:

```bash
# From a local framework clone at ./bwoc-framework
bwoc plugin install ./bwoc-framework/modules/plugins/gws/gws-auth
bwoc plugin install ./bwoc-framework/modules/plugins/gws/gws-drive
bwoc plugin install ./bwoc-framework/modules/plugins/gws/gws-gmail
bwoc plugin install ./bwoc-framework/modules/plugins/gws/gws-calendar
```

After install, each plugin sits under `modules/plugins/gws-auth/`, `modules/plugins/gws-drive/`, and so on. Install does not enable automatically — that is a deliberate two-step.

### 2.2 Enable in workspace.toml

Add an entry for each plugin you want active. `gws-auth` must be enabled whenever any service plugin is enabled, because the service plugins source their OAuth helpers from it.

```toml
# workspace.toml

[plugins.gws-auth]
enabled = true

[plugins.gws-drive]
enabled = true

[plugins.gws-gmail]
enabled = true

[plugins.gws-calendar]
enabled = true
```

The `enabled` key is the only workspace-level config field any `gws` plugin uses. There is no `[config.schema]` for these plugins — credential resolution is environment-driven, not config-driven (see [Section 3](#3-credentials-and-secrets)).

You can also toggle with the CLI:

```bash
bwoc plugin enable gws-auth
bwoc plugin enable gws-drive
```

To check what is installed and enabled:

```bash
bwoc plugin list --kind gws
```

### 2.3 Runtime dependencies

Each service plugin requires `jq` and `curl` on your `PATH`. `gws-auth` alone requires only `jq`. The plugins check for these at invocation time and exit with a clear message if either is missing.

---

## 3. Credentials and secrets

### 3.1 How the token reaches the plugin

The `gws-auth` plugin resolves the OAuth2 access token from two sources, checked in this order:

1. **`BWOC_GWS_TOKEN` environment variable** — set the token directly in the environment. This is the preferred path for CI and short-lived contexts. No metadata (scopes, expiry, account) is available from an env token.
2. **Token file** at `<workspace>/.bwoc/secrets/gws-token.json` — a JSON file that holds the access token and optionally a refresh token, expiry, scopes, and account information. The file must be `chmod 600` (owner-readable only); the plugin refuses a group- or world-readable file.

The token file format:

```json
{
  "access_token": "ya29.YOUR_TOKEN_HERE",
  "refresh_token": "1//YOUR_REFRESH_TOKEN",
  "client_id": "YOUR_CLIENT_ID.apps.googleusercontent.com",
  "client_secret": "YOUR_CLIENT_SECRET",
  "expiry": "2026-06-08T12:00:00Z",
  "scopes": [
    "https://www.googleapis.com/auth/drive.readonly",
    "https://www.googleapis.com/auth/gmail.readonly",
    "https://www.googleapis.com/auth/calendar.readonly"
  ],
  "account": "you@example.com"
}
```

Only `access_token` is required. When `refresh_token`, `client_id`, and `client_secret` are all present and the token has expired, `gws-auth` performs the refresh automatically and rewrites the file in place — you do not need to rotate the token manually on expiry.

The `.bwoc/secrets/` directory is gitignored at the workspace level. Never commit the token file.

### 3.2 Obtaining a token

Token acquisition — the OAuth consent flow — is an operator action outside BWOC. The typical approach:

1. Create an OAuth 2.0 Client ID in the Google Cloud Console (application type: Desktop).
2. Run the consent flow using a tool such as `oauth2l` or a short script using the Google Auth Library. Request only the scopes you need (see [Section 6](#6-security-and-oauth-scopes)).
3. Write the resulting `access_token` (and optionally the refresh trio) into `.bwoc/secrets/gws-token.json` with `chmod 600`.

Or, set `BWOC_GWS_TOKEN` to a short-lived access token for a one-off run.

### 3.3 Checking auth state

```bash
bwoc gws status
```

This calls the `gws-auth` `status` verb. It reports whether a token is present, which source it came from, the granted scopes, the account, and whether the token is expired or refreshable — but it **never** prints the token value.

Example output:

```json
{
  "ok": true,
  "plugin": "gws-auth",
  "operation": "status",
  "active_source": "secrets-file",
  "has_token": true,
  "account": "you@example.com",
  "scopes": [
    "https://www.googleapis.com/auth/drive.readonly"
  ],
  "expiry": "2026-06-08T12:00:00Z",
  "expired": false,
  "refreshable": true
}
```

---

## 4. Operations reference

### 4.1 gws-auth

| Verb | What it does |
|---|---|
| `status` | Reports token presence, active source, granted scopes, account, expiry, and whether a refresh is possible. Read-only. Never prints the token value. |

### 4.2 gws-drive

| Verb | What it does |
|---|---|
| `list` | Lists files in Drive. Accepts an optional `query` (Drive search syntax) and `max` (default 100). Paginates automatically. |
| `get` | Returns metadata for one file by `file_id`. Never downloads content. |

Each result is projected into the normative Drive file shape: `file_id`, `name`, `mime_type`, `modified_time`, and optionally `owners` and `web_view_link`.

### 4.3 gws-gmail

| Verb | Aliases | What it does |
|---|---|---|
| `search` | `threads` | Searches Gmail threads. Accepts an optional `query` (Gmail search syntax, e.g. `from:me is:unread`) and `max` (default 100). Enriches each result with subject, sender, and timestamp. |
| `show` | `message`, `messages` | Returns metadata for one thread by `thread_id`. |
| `labels` | — | Lists all labels the account has (system and user-created). |

Each thread is projected into the normative Gmail thread shape: `thread_id`, `subject`, `from`, `last_message_time`, and optionally `snippet` and `labels`.

### 4.4 gws-calendar

| Verb | Aliases | What it does |
|---|---|---|
| `calendars` | `list` | Lists the calendars the token can see. Returns `calendar_id`, `summary`, and optionally `primary` and `access_role`. |
| `events` | — | Lists events on a calendar. Accepts `calendar_id` (default `primary`) and `max` (default 100). Paginates; expands recurring events. |

Each event is projected into the normative Calendar event shape: `event_id`, `calendar_id`, `summary`, `start`, `end`, and optionally `attendees_count`. All-day events carry a date; timed events carry a datetime.

### 4.5 Error codes (all gws plugins)

All `gws` plugins share the same exit codes:

| Code | Meaning |
|---|---|
| `0` | Success — one JSON object on stdout. |
| `1` | Missing dependency (`jq` or `curl` not on PATH). |
| `2` | Usage error — unknown operation, missing required field (e.g. `file_id`, `thread_id`), or no token found. |
| `3` | Auth or scope error — HTTP 401 (token invalid/expired) or 403 (token lacks the required scope). |
| `4` | Rate limited — HTTP 429 after four backoff attempts. |
| `5` | Not found — HTTP 404. |
| `6` | Transport or unexpected HTTP error. |

---

## 5. Usage example

The following example assumes `gws-auth` and `gws-drive` are enabled and a valid token is in place.

Check auth state first:

```bash
bwoc gws status
```

List the 10 most recently modified Google Docs in Drive:

```bash
echo '{"operation":"list","query":"mimeType='\''application/vnd.google-apps.document'\''","max":10}' \
  | bwoc gws drive
```

Or with the operation passed via environment:

```bash
BWOC_GWS_OPERATION=list bwoc gws drive \
  <<< '{"query":"mimeType='\''application/vnd.google-apps.document'\''","max":10}'
```

Fetch metadata for a specific file:

```bash
echo '{"operation":"get","file_id":"1AbC_dEfGhIjKlMnOpQrStUvWxYz"}' | bwoc gws drive
```

Search Gmail for unread messages from a specific sender:

```bash
echo '{"operation":"search","query":"from:alice@example.com is:unread","max":5}' | bwoc gws gmail
```

List upcoming events on the primary calendar:

```bash
echo '{"operation":"events","calendar_id":"primary","max":10}' | bwoc gws calendar
```

All verbs emit a single JSON object on stdout. On error, a plain-text diagnostic goes to stderr and the process exits with a non-zero code.

---

## 6. Security and OAuth scopes

**Scopes are per-service and consent-bound.** A token granted `drive.readonly` cannot read Gmail or Calendar. Each plugin requires exactly one scope:

| Plugin | Required scope |
|---|---|
| `gws-drive` | `https://www.googleapis.com/auth/drive.readonly` |
| `gws-gmail` | `https://www.googleapis.com/auth/gmail.readonly` |
| `gws-calendar` | `https://www.googleapis.com/auth/calendar.readonly` |

Request only the scopes your agents actually need. If a call hits a service whose scope is absent from the token, the plugin surfaces `token lacks the required scope for <service>` rather than a bare HTTP 403, so it is clear which scope to add.

**The token is never logged or printed.** The `gws-auth` helpers pass the access token only as an `Authorization: Bearer` header on outbound curl calls. It never appears in any plugin output, JSON envelope, or log. The `status` verb reports metadata (source, scopes, account, expiry) — not the value.

**The token file must be owner-only.** The plugin refuses to read `.bwoc/secrets/gws-token.json` if it is group- or world-readable. Run `chmod 600 .bwoc/secrets/gws-token.json` after creating or updating the file.

**The `.bwoc/secrets/` directory is gitignored.** Never commit the token file. The `auth.toml` in the plugin source declares the shape of the contract (field names, env var name, file path) but contains no values; `bwoc check` rejects any `auth.toml` that contains a non-empty token value.

**Automatic token refresh.** When the token file carries an expiry in the past and the refresh trio is present (`refresh_token` + `client_id` + `client_secret`), `gws-auth` refreshes the token automatically before each request and rewrites the file atomically (via a `chmod 600` temp file + `mv`). If a refresh is needed but not possible (env token, or missing refresh trio), the request proceeds with the stale token and any resulting HTTP 401 is surfaced with a clear "re-authorize" message.

---

## 7. See also

- [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) — full plugin spec: kinds, manifest format, lifecycle hooks, loading, and the Workspace Resource Schema the `gws` plugins emit against.
- [gws plugin source](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/gws) — `manifest.toml`, `SPEC.md`, and entry scripts for all four plugins.
- [`../backends/HANDBOOK.en.md`](../backends/HANDBOOK.en.md) — backend configuration; relevant if you are running agents that will call `bwoc gws` verbs.
- [`../security/HANDBOOK.en.md`](../security/HANDBOOK.en.md) — workspace threat model and the secret-store rules that govern `.bwoc/secrets/`.
- [`../glossary.en.md`](../glossary.en.md) — term lookup.
