---
name: agentic-observability
description: >-
  Observe Speed-CLI health and activity: speed doctor, history, pending, xp.
  Triggers: diagnose env, list recent txs, track bridges, gamification stats, JSON status.
compatibility: >-
  ALCHEMY_API_KEY strongly recommended for history and pending mempool views; doctor
  checks RPC and oracles per supported chain.
---

# Observability

## Agent intent

Use this skill when the user wants **health checks**, **recent on-chain activity**, **in-flight bridges / mempool**, or **local XP stats** — **read-only or status** flows, not swapping.

| Goal | Command |
|------|---------|
| “Is my setup OK?” | **`speed doctor`** |
| “What did my wallet do recently?” | **`speed history`** |
| “Any open bridges or pending txs?” | **`speed pending`** |
| “Bot / usage progression?” | **`speed xp`** |

## `speed doctor`

**Purpose:** One-shot validation of configuration and connectivity.

**Checks include (non-exhaustive):**

- Signing wallet parseable → address.
- Presence flags for **`0X_API_KEY`**, **`SQUID_INTEGRATOR_ID`**, **`ALCHEMY_API_KEY`**, **`OPENSEA_API_KEY`**.
- **MCP:** URL if set; whether encrypted key file exists.
- **RPC:** `getBlockNumber` per **supported mainnet** chain ID in the CLI.
- **Oracle:** price/oracle read per supported chain.
- **SPEED balance** on the chain selected by **`-c`** (default from config / `8453`).

**Output:** Human table or **`--json`** → **`{ ok: boolean, checks: { ... } }`**.

**Use when:** After setup, after changing API keys or config, or when swaps fail mysteriously (RPC/oracle).

## `speed history`

**Purpose:** Recent transactions for the wallet via **Alchemy**-backed transfer APIs.

| Flag | Notes |
|------|--------|
| **`-c` / `--chain`** | Single chain; **omit** to query **all** supported chains. |
| **`-n` / `--limit`** | Per chain (max **100** enforced in CLI). |

**Requires:** **`ALCHEMY_API_KEY`** for the implementation to work as designed; without it, behavior may degrade or empty (confirm current CLI message).

**JSON:** Structured **`chains`** with **`transactions`** entries (hashes, timestamps, categories, etc.) — use **`--json`** for agents.

## `speed pending`

**Purpose:** Two concerns in one command:

1. **Bridges** — Reads **locally stored** pending bridge records and optionally polls **Squid** with **`SQUID_INTEGRATOR_ID`** + stored **`requestId`** / **`txHash`** / **`quoteId`**.
2. **Pending txs** — Mempool-style list via Alchemy helper **per chain** (optional **`-c`** filter).

| Flag | Notes |
|------|--------|
| **`--no-bridges`** | Skip bridge section. |
| **`--no-txs`** | Skip pending tx list. |
| **`--clear-done`** | Remove completed/failed bridges from **local** stored list. |

**Requires:** **`SQUID_INTEGRATOR_ID`** for live Squid status on stored bridges; Alchemy for pending tx list quality.

## `speed xp`

**Purpose:** **Local** gamification state (level, streak, per-action stats) — **not** an on-chain leaderboard unless specified elsewhere.

| Flag | Notes |
|------|--------|
| **`--reset`** | Clears XP state (destructive). |
| **`--no-title`** | Omit decorative title in human mode. |

**JSON:** **`totalXP`**, **`level`**, **`streak`**, **`progress`**, **`stats`**, **`recentHistory`**, etc.

**Use when:** Users want “how much have I used the CLI” narratives or bots report progress.

## Cross-command observability tips

| Tip | Detail |
|-----|--------|
| **Always `--json`** | For agents parsing `doctor`, `history`, `pending`, `xp`. |
| **Alchemy** | Single key unlocks richer **history** / **pending**; **doctor** flags presence. |
| **Bridges** | Pair **`pending`** with **`speed status --request-id`** from [Commands](../commands/skill.md) when you need authoritative settlement. |

## Pitfalls

| Issue | Mitigation |
|-------|------------|
| Empty **history** / **pending** | Set **`ALCHEMY_API_KEY`**; verify address is correct (`whoami`). |
| Bridge list stale | **`--clear-done`** after reconciling; ensure **`SQUID_INTEGRATOR_ID`** matches bridge. |
| **`doctor` ok but swap fails** | Doctor checks presence of keys, not 0x quote success — use **`quote`** / **`swap --dry-run`**. |

## Related

- [Commands](../commands/skill.md) (`status`, `balance`, `whoami`)
- [Protocol](../protocol/skill.md) (Alchemy, chains)
- [Orchestration](../orchestration/skill.md) (monitor phase)
- [Index](../README.md)
