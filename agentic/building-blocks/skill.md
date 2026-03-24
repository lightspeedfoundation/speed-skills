---
name: agentic-building-blocks
description: >-
  Trading Lego: compose speed quote/swap/balance with speed-scripts strategies;
  base vs target token, regime map, script interaction hazards, decimals, dry-run.
  Triggers: design strategy, pick bot type, avoid double entries.
---

# Building blocks

## Metaphor: CLI = Lego bricks

| Brick (CLI) | Typical use |
|-------------|-------------|
| **`quote`** | Measure execution price / output size without spending gas (beyond RPC). |
| **`swap`** | Change inventory: enter or exit a position leg. |
| **`balance`** | Decide **how much** you can sell; check wrapped native. |
| **`price` / oracle** | Context for P/L or triggers that compare to external reference. |
| **`bridge` / `status`** | Move SPEED across chains and track settlement. |
| **`gas`** | Refuel native for fees or unwrap WETH/WPOL/WBNB. |
| **`dca` / `volume`** | Built-in **time-based** or **pattern** activity without custom scripts. |

**speed-scripts** stacks these bricks into **loops** (poll → decide → act) with parameters tuned per strategy (stops, targets, rungs, timeouts).

## Roles: base token vs target token

| Role | Meaning | CLI pattern |
|------|---------|-------------|
| **Base** | Unit of account for spend / receive / P/L (often **`speed` in v2 bots**) | Buying target: `--sell <base> --buy <target>`; exiting: `--sell <target> --buy <base>` |
| **Target** | The asset you trade around (any address or alias) | Passed as the non-base leg |

**Rule:** In **generic** automation, pass **`--sell` and `--buy` explicitly** — do not rely on implicit CLI defaults for “what am I swapping.”

## Strategy families (from scripts-reference)

The upstream **scripts-reference** groups bots by **intent**. Use it as the menu when choosing or explaining a strategy:

| Family | Examples (names vary) | Idea |
|--------|------------------------|------|
| **Entry + managed exit** | trailing-stop, limit-order, bracket, ladder-sell, hybrid-exit | Enter, then manage exit (trail, fixed target, OCO, staged sells, hybrid). |
| **Conditional entry** | momentum, mean-revert, crash-buy, compression-buy | Wait for signal, then enter; exit often uses trails or combined rules. |
| **Accumulation** | ladder-buy, grid-trade, value-average | Build or rebalance position over time / levels. |
| **Execution quality** | twap-buy, twap-sell | Slice large orders across time (impact reduction). |
| **Watchers** | market-watch | **Quotes only** — no swaps; monitoring and baselines. |

See the live table and one-line behaviors in **[scripts-reference.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/scripts-reference.md)**.

## Regime map (mental shortcut)

Upstream provides a **regime → script** table (trending, ranging, crash, etc.). Use it to **shortlist** bots — then read the folder’s **`*-skill.md`** for exact parameters.

## Interaction hazards (compose, don’t collide)

scripts-reference warns: **two bots on the same token** can **double-enter** the same move (e.g. crash-buy + mean-revert both firing on a crash). That is not necessarily a bug — it must be **intentional** (size, mutual exclusion, or staggered deployment).

Agents should:

1. **Name** which bots may run concurrently on the same **chain + token**.
2. **Document** acceptable overlap or enforce **one strategy per token**.

## Decimals and amounts

- JSON fields like **`buyAmount`** / **`sellAmount`** are **raw integer strings** in the token’s smallest units.
- Before passing amounts back to **`-a`**, convert using **token decimals** (18 for many tokens; **8** for cbBTC/WBTC-class; **6** for USDC/USDT-class).
- **Never** emit scientific notation (e.g. `2.9e-5`) into CLI amount strings — use fixed decimal formatting with the correct precision (function-builder spells this out for PowerShell and bash).

## Validation before capital

| Step | Tool |
|------|------|
| Syntax / intent | Read **`*-skill.md`** |
| Economic check | **`quote`**, **`swap --dry-run`**, **`estimate`** |
| Inventory | **`balance --json`** |
| Live small size | Reduce **`-a`** until venue min and gas make sense |

## Related

- [Scripts](../scripts/skill.md) (clone + upstream links)
- [Protocol](../protocol/skill.md) (chains, tokens)
- [Orchestration](../orchestration/skill.md) (execution discipline)
- [Index](../README.md)
