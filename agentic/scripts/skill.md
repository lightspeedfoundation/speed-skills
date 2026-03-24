---
name: agentic-scripts
description: >-
  Speed-Scripts GitHub repo: folder-per-bot layout, speed scripts install via degit,
  SPEED_SCRIPTS_PATH, *-skill.md as source of truth, links to scripts-reference,
  README, function-builder. Triggers: install bot, read upstream docs, build wrappers.
---

# Speed-Scripts

## What it is

**[speed-scripts](https://github.com/lightspeedfoundation/speed-scripts)** is the **upstream library of advanced automation** built on top of `speed`: trading bots, watchers, TWAP tools, and shell/PowerShell wrappers. Each **function** is a **folder** containing:

- A **runnable script** (e.g. `.ps1`, `.sh`).
- A **`*-skill.md`** file: YAML frontmatter + parameters + **ordered steps** (do not reorder when automating).

The CLI ships **`speed scripts install <folder-name>`** to copy one folder from that repo into your machine.

## Relationship to `speed-cli`

| Piece | Role |
|-------|------|
| **`speed`** | Atomic CLI: `quote`, `swap`, `balance`, … |
| **speed-scripts folder** | Composed loops, polling, exits, strategy parameters. |
| **`speed scripts install X`** | Downloads **`lightspeedfoundation/speed-scripts/X`** via **`npx degit`** into a local directory. |

**Note:** In this repo, `speed scripts` metadata may mention SAI **`script.*`** families — that linkage is optional; scripts remain usable by humans and shell agents without SAI.

## Install mechanics

```bash
speed scripts install limit-order-any
```

| Detail | Behavior |
|--------|----------|
| **Tooling** | Spawns **`npx --yes degit`** — requires **network** and **`npx`** on PATH. |
| **Target directory** | **`SPEED_SCRIPTS_PATH`** env var if set, else **`./scripts`** under **current working directory**. |
| **Collision** | If `scripts/<name>` already exists → **error** (remove or pick another name). |
| **JSON mode** | With **`--json`**, prints `{ installed, path, source }` on success. |

## After install

1. `cd` into **`scripts/<name>/`** (or your `SPEED_SCRIPTS_PATH/<name>/`).
2. Open **`*-skill.md`** — authoritative for **flags**, **defaults**, and **step order**.
3. Run the script with the **shell your OS uses** (PowerShell on Windows is common for `.ps1`).

## Upstream documentation (read order)

| Doc | URL | Contents |
|-----|-----|----------|
| **Repo** | [github.com/lightspeedfoundation/speed-scripts](https://github.com/lightspeedfoundation/speed-scripts) | Layout rules, “Available scripts” table. |
| **scripts-reference.md** | [blob/main/scripts/scripts-reference.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/scripts-reference.md) | **Full bot catalog**: trailing stop, limit-order, bracket, ladder, hybrid, momentum, mean-revert, crash-buy, compression, grid, TWAP, market-watch, etc.; **regime map**; **interaction warnings** (double entry). |
| **scripts/README.md** | [blob/main/scripts/README.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/README.md) | Human + agent discovery: folder names, skill files, how to run. |
| **function-builder-skill.md** | [blob/main/scripts/function-builder-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/function-builder-skill.md) | **Complete CLI vocabulary** for script authors: global flags, env, chain table, **every `speed` subcommand** with JSON examples, **decimal handling**, **PowerShell/Bash patterns**, **pitfalls** (`$Args`, scientific notation, etc.). |

For **strategy taxonomy** and **which bot fits which market**, start with **scripts-reference**; for **how to call `speed` correctly**, use **function-builder**.

## Conventions (from upstream)

- **v2-style bots** default **base token** to **`speed`** but accept **`-BaseToken` / `--base-token`** for other quote currencies.
- **Generic wrappers** should pass **`--sell` and `--buy` explicitly** so behavior does not depend on CLI defaults.
- **Same-token guard:** if base alias equals target, scripts may temporarily use **`eth`** on one leg to avoid a no-op pair — mirror upstream helpers.

## Operational notes

| Topic | Guidance |
|-------|----------|
| **Version** | `degit` pulls **default branch** from GitHub; pin by fork/tag if you need reproducibility. |
| **Secrets** | Scripts rely on the same environment as **`speed`** (configure with **`speed setup`** / MCP as usual). |
| **OS** | Mix of `.ps1` and `.sh`; run the file type your environment supports. |

## Related

- [Building blocks](../building-blocks/skill.md) (how strategies compose)
- [Commands](../commands/skill.md) (underlying CLI)
- [Orchestration](../orchestration/skill.md) (multi-bot, parsing)
- [Index](../README.md)
