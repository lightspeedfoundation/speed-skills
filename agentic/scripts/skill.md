---
name: agentic-scripts
description: >-
  Speed-Scripts GitHub repo: clone lightspeedfoundation/speed-scripts, folder-per-bot layout,
  *-skill.md as source of truth. Triggers: bot scripts, upstream docs, build wrappers.
---

# Speed-Scripts

## What it is

**[speed-scripts](https://github.com/lightspeedfoundation/speed-scripts)** is the **upstream library of advanced automation** built on top of `speed`: trading bots, watchers, TWAP tools, and shell/PowerShell wrappers. Each **function** is a **folder** containing:

- A **runnable script** (e.g. `.ps1`, `.sh`).
- A **`*-skill.md`** file: YAML frontmatter + parameters + **ordered steps** (do not reorder when automating).

## How to get it (agents + humans)

```bash
git clone https://github.com/lightspeedfoundation/speed-scripts/
```

1. **Discover bots** by **folder name** under **`scripts/`** (each folder is one advanced function).
2. Open that folder’s **`*-skill.md`** — authoritative for **flags**, **defaults**, and **step order**.
3. Run the script with the **shell your OS uses** (PowerShell on Windows is common for `.ps1`).

**CLI note:** There is no `speed scripts` subcommand in current releases — do not rely on it; bots always come from the **cloned repo** above.

## Function skills ([speed-scripts](https://github.com/lightspeedfoundation/speed-scripts/) on GitHub)

Each folder under `scripts/<name>/` has a **`*-skill.md`** — source of truth for parameters and **ordered flow**. Summaries below; details always live in the linked file.

### Scripts tree (`scripts/`)

> **When to use:** First stop for repo layout, discovery rules, and the available-functions table.  
> **What it does:** Describes the `scripts/` tree, how to list folders, run scripts, and how agents should read skill files.  
> **Skill:** [scripts/README.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/README.md)

### Function builder (authoring, `scripts/`)

> **When to use:** Writing, extending, or debugging automation that calls `speed` (flags, JSON, decimals, shell quirks).  
> **What it does:** Meta-reference for authors — full CLI vocabulary and patterns; not a trading strategy script.  
> **Skill (speed-skills mirror):** [agentic/function-builder/skill.md](../function-builder/skill.md) — same content as upstream **[function-builder-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/function-builder-skill.md)**.

### Bracket order (`bracket-any`)

> **When to use:** You want a defined take-profit **and** stop-loss at entry (OCO-style discipline) and to walk away.  
> **What it does:** After the buy, monitors TP and SL; exits on whichever level hits first.  
> **Skill:** [bracket-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/bracket-any/bracket-skill.md)

### Compression buy (`compression-buy-any`)

> **When to use:** You want entry **after** price coils into a tight range, then breaks out (compression → expansion).  
> **What it does:** Arms when rolling range is tight enough, buys on expansion above the window, then runs a trailing stop like `momentum-any`.  
> **Skill:** [compression-buy-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/compression-buy-any/compression-buy-skill.md)

### Crash buy (`crash-buy-any`)

> **When to use:** You want to buy into sharp, fast drops (velocity / capitulation) rather than slow drifts.  
> **What it does:** Detects velocity events vs a rolling baseline and buys when the crash threshold fires.  
> **Skill:** [crash-buy-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/crash-buy-any/crash-buy-skill.md)

### Grid trade (`grid-trade-any`)

> **When to use:** You expect price to **oscillate in a range** and want to harvest volatility without a directional bet.  
> **What it does:** Places a ladder of buys below spot; cycles through grid logic as price moves (see skill for fill / exit rules).  
> **Skill:** [grid-trade-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/grid-trade-any/grid-trade-skill.md)

### Hybrid exit (`hybrid-exit-any`)

> **When to use:** You want **partial profit-taking** at a fixed target **and** a **trailing stop** on the remainder (plus hard stop).  
> **What it does:** Phase A: hit `TakePct`, sell `ExitFraction%`. Phase C: trail the rest with `TrailPct` from peak.  
> **Skill:** [hybrid-exit-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/hybrid-exit-any/hybrid-exit-skill.md)

### Ladder buy (`ladder-buy-any`)

> **When to use:** You want **staged buys** on successive dips with fixed base per rung (optional trail after N rungs).  
> **What it does:** Waits for each rung’s trigger and accumulates; optional trailing exit per parameters.  
> **Skill:** [ladder-buy-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/ladder-buy-any/ladder-buy-skill.md)

### Ladder sell (`ladder-sell-any`)

> **When to use:** You want to **scale out** as price climbs through predefined rungs instead of one exit.  
> **What it does:** Sells a fraction of the position at each higher target (structured de-risking).  
> **Skill:** [ladder-sell-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/ladder-sell-any/ladder-sell-skill.md)

### Limit order (`limit-order-any`)

> **When to use:** You want a **single profit target** on a buy-and-hold leg (no stop-loss leg like bracket).  
> **What it does:** Buys once, polls until full-position base return ≥ target gain %, else **forced sell** at max iterations.  
> **Skill:** [limit-order-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/limit-order-any/limit-order-skill.md)

### Market watch (`market-watch-any`)

> **When to use:** You want **live monitoring** in base-token terms **without** swaps or position changes.  
> **What it does:** Quote-only: builds a reference size, polls `Token → Base`, prints value, **% vs baseline**, session **H/L**.  
> **Skill:** [market-watch-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/market-watch-any/market-watch-skill.md)

### Mean revert (`mean-revert-any`)

> **When to use:** You want to **buy weakness** vs a rolling mean (dip entry), not breakouts.  
> **What it does:** Enters when price is sufficiently **below** the mean threshold; opposite philosophy to `momentum-any`.  
> **Skill:** [mean-revert-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/mean-revert-any/mean-revert-skill.md)

### Momentum (`momentum-any`)

> **When to use:** You want to buy **breakouts** (new highs in the window), not dips.  
> **What it does:** Enters when price clears the window high by `BreakoutPct`, then runs a trailing stop.  
> **Skill:** [momentum-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/momentum-any/momentum-skill.md)

### Trailing stop-loss (`trailing-stoploss-any`)

> **When to use:** You want to **ride upside** but exit when base return falls **TrailPct%** from the **running peak**.  
> **What it does:** Tracks peak → floor after entry; sells full position on trail break or max iterations.  
> **Skill:** [trailing-stop-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/trailing-stoploss-any/trailing-stop-skill.md)

### TWAP buy (`twap-buy-any`)

> **When to use:** You want to **slice a known buy** over time to reduce timing risk and impact (execution, not direction prediction).  
> **What it does:** Splits the buy into **N** equal slices on an interval schedule.  
> **Skill:** [twap-buy-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/twap-buy-any/twap-buy-skill.md)

### TWAP sell (`twap-sell-any`)

> **When to use:** You want to **exit a large position** in slices on a schedule with logged average exit stats.  
> **What it does:** Sells **N** equal portions of the position each interval regardless of price path.  
> **Skill:** [twap-sell-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/twap-sell-any/twap-sell-skill.md)

### Value average (`value-average-any`)

> **When to use:** You want **scheduled accumulation** toward a **growing target portfolio value** (systematic VA).  
> **What it does:** Each interval, compares position value to the value path and buys the deficit (or sells surplus if configured).  
> **Skill:** [value-average-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/value-average-any/value-average-skill.md)

## Relationship to `speed-cli`

| Piece | Role |
|-------|------|
| **`speed`** | Atomic CLI: `quote`, `swap`, `balance`, … |
| **speed-scripts folder** | Composed loops, polling, exits, strategy parameters. |

**Note:** Some metadata may reference SAI **`script.*`** families — that linkage is optional; scripts remain usable by humans and shell agents without SAI.

## Upstream documentation (read order)

| Doc | URL | Contents |
|-----|-----|----------|
| **Repo** | [github.com/lightspeedfoundation/speed-scripts](https://github.com/lightspeedfoundation/speed-scripts) | Layout rules, “Available scripts” table. |
| **scripts-reference.md** | [blob/main/scripts/scripts-reference.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/scripts-reference.md) | **Full bot catalog**: trailing stop, limit-order, bracket, ladder, hybrid, momentum, mean-revert, crash-buy, compression, grid, TWAP, market-watch, etc.; **regime map**; **interaction warnings** (double entry). |
| **scripts/README.md** | [blob/main/scripts/README.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/README.md) | Human + agent discovery: folder names, skill files, how to run. |
| **Function builder** | **[agentic/function-builder/skill.md](../function-builder/skill.md)** (mirror) · [upstream function-builder-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/function-builder-skill.md) | **Complete CLI vocabulary** for script authors: global flags, env, chain table, **every `speed` subcommand** with JSON examples, **decimal handling**, **PowerShell/Bash patterns**, **pitfalls** (`$Args`, scientific notation, etc.). |

For **strategy taxonomy** and **which bot fits which market**, start with **scripts-reference**; for **how to call `speed` correctly**, read **[Function builder](../function-builder/skill.md)** (mirrored from upstream).

## Conventions (from upstream)

- **v2-style bots** default **base token** to **`speed`** but accept **`-BaseToken` / `--base-token`** for other quote currencies.
- **Generic wrappers** should pass **`--sell` and `--buy` explicitly** so behavior does not depend on CLI defaults.
- **Same-token guard:** if base alias equals target, scripts may temporarily use **`eth`** on one leg to avoid a no-op pair — mirror upstream helpers.

## Operational notes

| Topic | Guidance |
|-------|----------|
| **Version** | `git` tracks **branch**; pin by fork/tag if you need reproducibility. |
| **Secrets** | Scripts rely on the same environment as **`speed`** (configure with **`speed setup`** / MCP as usual). |
| **OS** | Mix of `.ps1` and `.sh`; run the file type your environment supports. |

## Related

- [Function builder](../function-builder/skill.md) (full `speed` vocabulary for script authors)
- [Building blocks](../building-blocks/skill.md) (how strategies compose)
- [Commands](../commands/skill.md) (underlying CLI)
- [Orchestration](../orchestration/skill.md) (multi-bot, parsing)
- [Index](../README.md)
