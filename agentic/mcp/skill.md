---
name: agentic-mcp
description: >-
  Speed-CLI MCP integration: speed start, persisted MCP URL, Streamable HTTP …/mcp,
  get_speed_env_vars (plain or RSA+AES encrypted), speed_mcp_key.pem, merge rules,
  SPEED_MCP_DEBUG. Triggers: MCP connection failed, remote API keys, encrypted env.
compatibility: >-
  Network access to MCP host; HTTPS for remote hosts; Node crypto for PEM key and
  decryption. CLI skips MCP preload when first argv is speed start.
---

# MCP (Model Context Protocol)

## Agent intent

Use this skill when the user needs **hosted or remote API keys** without checking them into git, when **`speed start` fails**, or when debugging **why 0x/Alchemy/Squid keys appear after connect**. This is **not** the same as running `speed swap` — it is **how optional secrets get into the process** before other commands run.

## What MCP does here

1. **`speed start [url]`** connects to a **Streamable HTTP** MCP server at **`{baseUrl}/mcp`** (POST `initialize` → session → `notifications/initialized` → **`tools/call`** **`get_speed_env_vars`**).
2. On success, the CLI persists the MCP base URL and strips legacy **`SPEED_MCP_URL=`** from the setup secrets file so the URL is not stored next to other secrets.
3. Fetched env vars are merged into **`process.env`** for API keys (0x, Alchemy, Squid, OpenSea, etc.).

## Trust model (critical)

| Rule | Detail |
|------|--------|
| **Wallet stays local** | Signing-key material from MCP payloads is **never** applied. The wallet always comes from **`speed setup`** (and normal env load order). |
| **Non-empty only** | Empty strings in the payload are skipped. |
| **`SPEED_MCP_DEBUG`** | Not applied from server (avoid remote toggling debug). |

After merge, **`0X_API_KEY`** is mirrored to **`OX_API_KEY`** when set.

Current List of hosted mcp:

speed start https://speed-mcp-production.up.railway.app

(more to come soon)

## Files on disk

| Path | Role |
|------|------|
| **MCP URL file** | Persisted base URL after successful **`speed start`**. |
| **`speed_mcp_key.pem`** | Client **RSA key** for MCP encryption; created/ensured during **`speed start`**. Not your chain signing key. |

If the server returns **encrypted** env but this file is missing or unreadable, fetch fails with an explicit error.

## URL normalization

- **Local** hosts (`127.0.0.1`, `localhost`, `0.0.0.0`): default **`http://`** if no scheme.
- **Remote** hosts: default **`https://`** if no scheme.
- **`speed start`** uses **`ensureHttpsMcpUrl`** for what gets **persisted** (TLS for hosted URLs).

## When env is loaded

- **`cli.ts`** loads MCP env **after** `loadEnv()` when a speed MCP URL is configured — **except** the first subcommand is **`start`**, so **`speed start <url>`** does not use a stale persisted URL and block your new URL.
- **`loadMcpEnvAndMerge`** **caches** env per process URL; same URL reuses cache.

## Encrypted vs plain response

| Mode | When |
|------|------|
| **Plain** | Server returns JSON with **`{ "env": { "KEY": "value" } }`** (shape may be wrapped in MCP `content[0].text`). |
| **Encrypted** | Server returns **`encrypted_key_b64`**, **`nonce_b64`**, **`ciphertext_b64`**; client decrypts with **`speed_mcp_key.pem`**. |

If a key file exists but **public key cannot be derived**, client may fall back to requesting **without** encryption (see debug logs).

## Debugging

Set **`SPEED_MCP_DEBUG=1`** or **`true`** for stderr diagnostics (key paths, whether public key was sent, applied env keys, presence of Alchemy/0x keys).

## Failure modes (symptoms → checks)

| Symptom | Check |
|---------|--------|
| **`404`** at `…/mcp` | Server not running or wrong path; confirm hosted URL uses **`https://`** and **`…/mcp`** endpoint. |
| **`initialize` failed / session** | Old CLI double-POST or wrong protocol — error text may suggest HTTPS-only and single POST to **`/mcp`**. |
| **Encrypted env but no key** | Ensure **`speed_mcp_key.pem`** exists and is valid PEM (see MCP URL / config dir). |
| **MCP loaded but no API keys** | Server returned empty env; configure API keys locally via **`speed setup`** as fallback. **`doctor`** may show MCP URL with sparse API keys. |

## One-line bootstrap (reminder)

```bash
speed start https://<your-mcp-host>
```

Use production URL only if you trust that operator; **you still control the wallet locally.**

## Related

- [Start](../start/skill.md) (bootstrap sequence)
- [Protocol](../protocol/skill.md) (env matrix)
- [Security](../security/skill.md) (trust boundaries)
- [Observability](../observability/skill.md) (`doctor` MCP checks)
- [Index](../README.md)
