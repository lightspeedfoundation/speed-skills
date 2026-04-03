---
name: agentic-start
description: >-
  Bootstrap Speed-CLI: npm global install, speed setup --skip, MCP via speed start,
  speed skill for OpenClaw/Cursor, whoami; advanced path: Cursor, clone speed-scripts,
  trailing-stoploss-any + cbBTC example. Triggers: first install, new machine, MCP, editor skill.
---

# Start

## When to use

- Installing or recovering the CLI on a new machine or CI image.
- Connecting to hosted MCP so API keys can be supplied remotely (encrypted).
- Copying the bundled editor skill into a project (`speed skill`).
- Verifying the wallet with `speed whoami` after setup.

## Mental model

1. **Install** the `speed` binary (global `npm` or `npx`).
2. **`speed setup`** creates the CLI’s persistent config; the CLI loads it automatically — you do not need to manage key paths by hand for normal use.
3. **Optional MCP** (`speed start`) fetches **API** keys from a server; it does **not** replace the local signing wallet.
4. **Automation** always uses `--json` and usually `-y` so scripts do not hang on prompts.

## Prerequisites

| Need | Why |
|------|-----|
| **Node.js + npm** | Package is `@lightspeed-cli/speed-cli`; commands are `speed …`. |
| **Network** | First `npm install`, MCP fetch, 0x/Squid/RPC calls. |
| **Shell** | PowerShell, bash, or zsh — any environment that can run Node and set env vars. |

Git is optional unless you clone `speed-scripts` or this repo.

## Install the CLI

**Global (typical for agents and daily use):**

```bash
npm install -g @lightspeed-cli/speed-cli
```

**One-off (CI or pinned version):**

```bash
npx @lightspeed-cli/speed-cli whoami
```

Confirm: `speed --version` matches what you expect.

## Persistent config (overview)

The CLI keeps its data under **`~/.speed`** (override with **`SPEED_CONFIG_DIR`** if needed): setup secrets, optional MCP URL file, optional PEM for encrypted MCP responses, and **`speed config`** preferences. You rarely need to touch these paths — use **`speed setup`**, **`speed start`**, and **`speed config`** instead.

## Environment loading (order)

Load order for most keys: **global setup file** → **current directory `.env`**. Later overrides earlier where applicable.

**Quirk:** keys starting with a digit (notably **`0X_API_KEY`**) are applied manually by this CLI so they still work with typical dotenv loaders.

## `speed setup`

| Mode | Behavior |
|------|----------|
| **Interactive** | `speed setup` — prompts; use when humans are at the keyboard. |
| **`--skip`** | No prompts. Ensures config dir exists. If no wallet is configured, **generates a new random wallet** and saves it via setup. |
| **Inline** | `--key`, `--alchemy`, `--squid`, `--ox`, `--opensea` — non-interactive fill. |

**Important:** `speed setup` with **`--json` is rejected** — omit `--json` for this command.

After setup, run **`speed whoami`** and **`speed doctor`** to validate.

## `speed start` (MCP)

Connects to a **Streamable HTTP** MCP server at **`{baseUrl}/mcp`** (POST `initialize` → session → **`get_speed_env_vars`**). On **success**:

- Persists the MCP base URL and ensures a client PEM exists for encrypted env delivery when used.
- Merges **API** keys into the process environment.

**Trust:** the server **never** supplies the signing wallet — only local setup does.

Example (production):

```bash
speed start mcp.ispeed.pro
```

If MCP is down or TLS is wrong, the command **fails** and nothing useful is persisted — fix URL/network and retry.

## `speed skill`

Installs the bundled **OpenClaw/Cursor** skill from the package (`openclaw/skills/speed-token/SKILL.md`) into your project.

| Flag | Effect |
|------|--------|
| `--path` | Print path to bundled `SKILL.md` only (JSON: `{ path }`). |
| `-d / --dir` | Target directory under cwd (default `openclaw`). |
| `--file` | Filename (default `skill.md`). |
| `-f / --force` | Overwrite existing file. |

Default write target: `./openclaw/skill.md` (unless `--dir` / `--file` change it).

## Minimal bootstrap sequence

```bash
npm install -g @lightspeed-cli/speed-cli
speed setup --skip
speed start mcp.ispeed.pro
speed skill
speed whoami
```

1. CLI on `PATH`.
2. Setup complete (wallet + optional API keys).
3. MCP URL saved; remote API keys merged if the server provides them.
4. Project-level skill file for the IDE agent.
5. **`whoami`** confirms the active address.

## Advanced setup (Cursor + full speed-scripts)

Use this path when you want **Cursor** and the **entire** [speed-scripts](https://github.com/lightspeedfoundation/speed-scripts) tree — **clone** the repo locally.

1. **Install Cursor** — you can skip payment and use the free tier.
2. **Open a terminal** in your project and run the same bootstrap as above:

```bash
npm install -g @lightspeed-cli/speed-cli
speed setup --skip
speed start mcp.ispeed.pro
speed skill
speed whoami
```

After this you have a full CLI setup and your **public wallet address** from `whoami`.

3. **Clone the script library** (full repo):

```bash
git clone https://github.com/lightspeedfoundation/speed-scripts/
```

4. **Example trade (trailing stop on cbBTC, Base):** Open the clone in Cursor and attach the **`scripts/`** folder for the agent (e.g. **`@/scripts`** in Cursor so it can execute against that tree). Run the **`trailing-stoploss-any`** bot with **cbBTC** at **`0xcbB7C0000aB88B473b1f5aFd9ef808440eed33Bf`** on Base. Follow that folder’s **`*-skill.md`** for parameters, decimals, and step order; confirm flags with `speed … --help` where needed.

## Scripting defaults

Place **global flags before the subcommand**:

```bash
speed --json --yes swap -c base --sell eth --buy speed -a 0.001
```

| Flag | Role |
|------|------|
| `--json` | Single JSON object on stdout for successes; `{ error, code }` on failures. |
| `-y` / `--yes` | Skip interactive confirmations (`swap`, `bridge`, etc.). |

Stderr may still contain spinners or progress — **parse stdout only** for JSON.

## Pitfalls

| Issue | Mitigation |
|-------|------------|
| `setup --skip` created an unfunded wallet | Fund the printed address before swaps/bridges. |
| MCP fails immediately | Check URL, TLS, firewall; use `SPEED_MCP_DEBUG=1` for verbose client logs (if supported). |
| No API keys after MCP | Server may return empty env; ensure local API keys are set if required. |
| Wrong `speed` binary | `which speed` / `Get-Command speed` — stale global install vs repo `node dist/cli.js`. |

## Related

- [Why Speed-CLI](../why/skill.md) · [Protocol](../protocol/skill.md) · [Commands](../commands/skill.md) · [Orchestration](../orchestration/skill.md)
- [Scripts](../scripts/skill.md) (clone speed-scripts) · [Function builder](../function-builder/skill.md) (CLI reference)
- [Index](../README.md)
