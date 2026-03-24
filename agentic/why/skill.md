---
name: agentic-why-speed-cli
description: >-
  Why Speed-CLI exists for agents: JSON-first CLI, multichain swaps and bridges,
  config persistence, MCP for remote API keys, composable with speed-scripts. Triggers:
  product rationale, compare to raw web3, automation design.
---

# Why Speed-CLI

## Problem being solved

Agents and scripts need **repeatable, inspectable** ways to move tokens, bridge, check balances, and manage on-chain identity — without building a custom wallet stack each time. Speed-CLI bundles:

- **Swaps** via **0x** (quotes + execution with allowance handling).
- **Bridges** via **Squid** (cross-chain SPEED with status polling).
- **Prices** via **Lightspeed oracles** on supported chains.
- **Optional Alchemy** for richer RPC, `history`, and `pending`.
- **.speed identities** on Base and **SANS** marketplace flows on OpenSea.

All behind one **`speed`** binary with stable flags and **`--json`** output.

## Design pillars

### 1. Agent-native I/O

- **`--json`**: stdout is a **single JSON** object (or structured payload per command). Agents grep / parse one line starting with `{` where needed.
- **Errors**: in JSON mode, failures are still JSON: `{ "error": "...", "code": "SOME_CODE" }` — check **`error`** before reading success fields.
- **`-y` / `--yes`**: non-interactive execution for `swap`, `bridge`, `approve`, etc.

### 2. Secrets and config separation

- **Setup** stores secrets for the CLI; **config** stores non-secret defaults (`speed config`).
- **Optional MCP** injects **API** keys into the process — not the signing wallet.
- Keep project repos free of secrets; use **`speed setup`** / hosting env, not checked-in key files.

### 3. Composable primitives

Low-level commands are **Lego bricks**: `quote` → `swap`, `bridge` → `status`, `balance` → decide size. Higher-level strategies belong in **[speed-scripts](https://github.com/lightspeedfoundation/speed-scripts)** (poll loops, exits, regime logic) — see [Scripts](../scripts/skill.md) and [Building blocks](../building-blocks/skill.md).

### 4. Multichain with one UX

Chain is **` -c <id|name>`** (e.g. `base`, `8453`, `arbitrum`). Defaults favor **Base (8453)** unless `speed config set default-chain` says otherwise.

## What this CLI is not

| Not | Instead |
|-----|---------|
| A custodial service | The tool signs with your local wallet from setup — non-custodial. |
| A trading venue | Execution goes through 0x / Squid / chain RPCs subject to their rules. |
| A full strategy engine | Use **speed-scripts** or your own orchestration for bots. |

## Who benefits

| User | Benefit |
|------|---------|
| **Coding agents** | Predictable flags + JSON + error codes. |
| **Humans** | `speed doctor`, progressive setup, explorer links in output. |
| **Script authors** | Same commands from PowerShell, bash, or CI. |

## Relationship to speed-scripts

| Layer | Responsibility |
|-------|----------------|
| **speed-cli** | Atomic operations, one-shot flows, config, identity, SANS helpers. |
| **speed-scripts** | Multi-step bots, polling, risk/exit parameters documented per folder **`*-skill.md`**. Get the library by cloning [speed-scripts](https://github.com/lightspeedfoundation/speed-scripts/). |

## Related

- [Start](../start/skill.md) · [Protocol](../protocol/skill.md) · [Commands](../commands/skill.md)
- [Building blocks](../building-blocks/skill.md) · [Orchestration](../orchestration/skill.md)
- [Index](../README.md)
