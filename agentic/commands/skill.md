---
name: agentic-commands
description: >-
  Full command map for Speed-CLI (excluding agent/SAI, terminal, launch): global flags,
  JSON error contract, grouped subcommands with roles and key flags, doctor checks.
  Triggers: which speed subcommand to run, automation reference.
---

# Commands

## Scope

This skill summarizes subcommands that appear in **`speed --help`** for **`@lightspeed-cli/speed-cli`**. Prefer **`speed <cmd> --help`** for exhaustive flags.

**Out of scope here (see CLI / product docs elsewhere):** **`speed terminal`** (Chrome extension pairing).

**Names you may see in older docs but often missing from current npm builds:** **`speed agent`**, **`speed memory`**, **`speed launch`**, **`speed scripts`** — if the CLI prints `unknown command`, that build does not ship them; rely on **`speed --help`** and the tables below.

## Global program options

These apply **before** the subcommand:

| Flag | Purpose |
|------|---------|
| `--version` / `-V` | CLI version string. |
| `--help` / `-h` | Help for `speed` or nested command. |
| `--json` | Machine-readable stdout; errors as `{ error, code }`. |
| `-y` / `--yes` | Skip confirmations where supported (`swap`, `bridge`, etc.). |

**Convention:** `speed --json --yes <subcommand> …`

## JSON output contract

| Outcome | Shape |
|---------|--------|
| **Success** | One JSON **object** (per command; fields vary). |
| **Failure** | `{ "error": "<message>", "code": "<UPPER_SNAKE>" }` — **always check `error` before reading success fields.** |

Stderr may contain spinners or hints; **reliable parsing uses stdout** (first line matching `^\{` is a common pattern in scripts).

## Discovering flags

Every subcommand supports **`speed <cmd> --help`**. Prefer help for exhaustive flags; this skill summarizes **roles** and **agent-critical** notes.

---

## Wallet & environment

| Command | Role | Agent notes |
|---------|------|-------------|
| **`whoami`** | Prints the wallet’s **public address** (`0x…`). | **Primary way for agents** to learn which account the CLI signs as; output has **no private key material**. No network required. |
| **`setup`** | Writes the CLI’s setup secrets (interactive). | **`--json` invalid**; `--skip` can mint a new wallet if none. |
| **`doctor`** | Health snapshot: same **public address** as `whoami`, plus API flags, **RPC per chain**, **oracle per chain**, SPEED balance on chosen `-c`. | Use `--json` for `{ ok, checks }` structure. |
| **`config`** | `get` / `set` for `default-chain`, `default-slippage`, `output-format`. | No secrets stored here. |

---

## Liquidity & execution

| Command | Role | Agent notes |
|---------|------|-------------|
| **`balance`** | Native + wrapped + SPEED + optional `-t` tokens. | Omit `-c` for **all** configured chains; repeat `-t` for extra ERC-20s. |
| **`price`** | Oracle: SPEED/native/USD style metrics. | Chain-specific; use `--json` for floats in JSON. |
| **`quote`** | 0x preview: **`--sell`**, **`--buy`**, **`-a`** sell amount (human units). | No tx; raw amounts in JSON are **integer strings** in token decimals. |
| **`swap`** | Quote → confirm → approve → execute. | **`--dry-run`** = quote only; **`-y`** for scripts; **`--sell`/`--buy`** explicit in generic tools. |
| **`estimate`** | Gas / cost estimate for swap or **`--bridge`**. | Pass **`-a`** for more accurate swap gas when applicable. |
| **`gas`** | Get native for gas: default **unwrap** wrapped native; **`--token`** for selling other assets via 0x. | Selling native ETH: leave headroom for gas fees. |
| **`send`** | Transfer **SPEED** ERC-20 to `-t` address. | Not arbitrary token in all versions — confirm `--help`. |

---

## Approvals

| Command | Role |
|---------|------|
| **`allowance`** | Read `allowance` for `--token` + `--spender`. |
| **`approve`** | Set allowance; **`-a max`** for infinite. |
| **`revoke`** | Set allowance to **0**. |

Use for manual or debugging flows; **`swap`** often handles allowance automatically.

---

## Cross-chain

| Command | Role | Agent notes |
|---------|------|-------------|
| **`bridge`** | Bridge SPEED **from** / **to** chains; **`-a`** amount. | Save **`requestId`** + tx hash from JSON for **`status`**. |
| **`status`** | Confirm tx on chain **or** Squid status with **`--request-id`**. | Bridge flows need integrator id in env. |
| **`pending`** | Pending bridges list + mempool-style pending txs (when Alchemy available). | **`--clear-done`** etc. — see `--help`. |

---

## Automation & activity

| Command | Role | Agent notes |
|---------|------|-------------|
| **`dca`** | Repeated buys with native for **`--token`** on **`--interval`**; **`--count`** optional cap. | **`--dry-run`** available; mind min notional (0x / venue limits). |
| **`volume`** | Randomized buy/sell sequence for activity patterns. | Many tunables (`--ops`, delays, sell frequency); **`--dry-run`**. |

---

## Observability & XP

| Command | Role | Agent notes |
|---------|------|-------------|
| **`history`** | Recent txs via Alchemy. | Needs **`ALCHEMY_API_KEY`** for full behavior. |
| **`xp`** | Local XP / level / streak / stats. | No chain tx; gamification only. |

---

## Identity & marketplace (Base / OpenSea)

| Command | Role |
|---------|------|
| **`identity`** | Subcommands: `register`, `set-resolve`, `profile`, `community`, etc. — **.speed** names on Base. |
| **`sans`** | Subcommands: `listings`, `buy`, `list`, `unlist`, `transfer`, offers — **SANS** NFTs on OpenSea. |

Requires correct **`OPENSEA_API_KEY`** for API-heavy flows.

---

## Agent tooling (non-SAI)

| Command | Role |
|---------|------|
| **`start`** | MCP connect + persist URL + merge API env. |
| **`skill`** | Copy bundled Cursor/OpenClaw **`SKILL.md`** into project (`openclaw/skill.md` by default). |
| *(not a subcommand)* | **Speed-scripts bots** are **not** invoked via `speed scripts` — current CLI builds **do not** register a `scripts` command (it will error as unknown). Get bots from **[speed-scripts](https://github.com/lightspeedfoundation/speed-scripts/)** (**clone** the repo, run scripts under **`scripts/<folder>/`**, follow each **`*-skill.md`**). See [Scripts](../scripts/skill.md). |

---

## `speed doctor` (what it checks)

Useful for agents to interpret failures. **Public address only:** use **`speed whoami`** when you just need the `0x` address; **`doctor`** repeats that address while running broader checks.

- Wallet public address (same as **`whoami`**)
- Presence flags for **`0X_API_KEY`**, **`SQUID_INTEGRATOR_ID`**, **`ALCHEMY_API_KEY`**, **`OPENSEA_API_KEY`** (boolean presence in checks).
- **MCP URL** and whether encrypted key file exists (if MCP configured).
- **RPC** `getBlockNumber` per supported chain ID.
- **Oracle** read per chain.
- **`SPEED_BALANCE_<chain>`** formatted balance on default `-c` chain (string) or failure.

JSON: `{ "ok": boolean, "checks": { ... } }`.

## Related

- [Protocol](../protocol/skill.md) (env, chains, MCP)
- [Function builder](../function-builder/skill.md) (full CLI reference)
- [Start](../start/skill.md) · [Scripts](../scripts/skill.md) · [Orchestration](../orchestration/skill.md)
- [Index](../README.md)
