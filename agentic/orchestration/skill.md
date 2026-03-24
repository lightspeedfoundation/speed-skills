---
name: agentic-orchestration
description: >-
  Run Speed-CLI and speed-scripts: JSON parsing discipline, error handling, multi-terminal
  bots, agent planning phases (recon/paper/execute/monitor), idempotency and logging.
  Triggers: automate strategy, coordinate scripts, operational playbook.
---

# Orchestration

## Principles

1. **Skills are recipes** — For each speed-scripts folder, the **`*-skill.md`** defines order; **do not reorder** documented steps unless you are explicitly patching the strategy.
2. **One JSON line** — Prefer extracting the first stdout line that looks like `{...}` when mixing with stderr.
3. **Fail closed** — If JSON contains **`error`**, stop the pipeline and surface **`code`** when present.
4. **Idempotency** — Swaps and bridges are **not** idempotent; replan after partial failure (new nonce, new quote).

## Single command execution

```bash
speed --json --yes <subcommand> [flags]
```

- Parse **stdout** only for JSON.
- On exit code **non-zero**, still expect JSON in **`--json`** mode for many failures.

**Checklist per invocation:**

| Step | Action |
|------|--------|
| 1 | Build full argv with explicit chain **`-c`** if not relying on default. |
| 2 | Run with **`--json`** (and **`-y`** if mutating). |
| 3 | Parse JSON; if **`error`** → branch to recovery. |
| 4 | Persist **`txHash`**, **`requestId`**, **`explorerLink`** from success objects when present. |

## Chaining commands (shell)

Safe chaining requires **capturing and parsing** each JSON payload:

- **PowerShell:** avoid naming variables **`$Args`** (automatic variable); use named parameters in functions.
- **Bash:** `2>&1` often needed; extract with `grep -m1 '^{'` or `jq` if available.

See **[function-builder-skill.md](https://github.com/lightspeedfoundation/speed-scripts/blob/main/scripts/function-builder-skill.md)** for copy-paste templates.

## Multi-terminal / multi-process

| Pattern | When |
|---------|------|
| **One terminal per bot** | Isolated logs, independent restart, different cwd or `SPEED_CONFIG_DIR` for separate wallets. |
| **One terminal, sequential** | Simpler for agents that run steps in strict order. |
| **Process manager** | systemd, PM2 (for long-running watchers), or CI for scheduled runs. |

**Do not** assume two CLIs share cwd env unless you exported variables in **both** sessions.

## Multi-bot coordination

| Concern | Mitigation |
|---------|------------|
| **Double entry** | See [Building blocks](../building-blocks/skill.md); document which strategies may overlap. |
| **Nonce / gas** | Same wallet cannot parallelize conflicting txs without careful nonce management — serialize or use separate keys. |
| **Rate limits** | 0x / Alchemy / OpenSea — backoff and jitter on polling scripts. |

## Agent workflow (recommended phases)

### Phase A — Reconnaissance (read-only)

| Command | Purpose |
|---------|---------|
| `whoami` | Confirm address. |
| `doctor --json` | Environment and connectivity snapshot. |
| `balance --json` | Inventory across chains/tokens. |
| `price --json` | Oracle context (if strategy needs it). |

### Phase B — Plan

- Resolve **chain**, **base token**, **target token**, **size**, **slippage** (from `config` or flags).
- If using **speed-scripts**, read **`scripts/<name>/*-skill.md`** and list **required parameters**.

### Phase C — Paper / preflight

| Command | Purpose |
|---------|---------|
| `quote` | Validate route and amounts. |
| `swap --dry-run` or `estimate` | Confirm gas and economics without sending (where supported). |

### Phase D — Execute

- `swap -y`, `bridge -y`, `approve -y`, etc.
- Store outputs for audit (tx hash, request id).

### Phase E — Monitor

| Command | Purpose |
|---------|---------|
| `status` | Confirm inclusion; bridge status with **`--request-id`**. |
| `pending` / `history` | Track open operations (Alchemy-dependent). |

### Phase F — Strategy loops (scripts)

- Run installed **`.ps1` / `.sh`** as documented; **do not** shortcut poll intervals or exit conditions without understanding the skill file.

## Error handling matrix

| Signal | Meaning | Typical next step |
|--------|---------|-------------------|
| `{ error, code: INSUFFICIENT_FUNDS }` | Balance too low | `balance`, reduce `-a`, bridge in SPEED or native |
| `{ error, code: INVALID_CHAIN }` | Bad `-c` | Use `doctor` / Protocol chain table |
| MCP / doctor RPC failures | RPC or network | Check `ALCHEMY_API_KEY`, firewall, RPC status |
| Non-zero exit + no JSON | Rare (crash) | Read stderr; rerun without `--json` for human error |

## Logging for operators

- Timestamp each **decision** (quote result hash or key fields).
- Persist **tx hashes** and **bridge request IDs** to a local log file or ticket system.
- Never log signing material or full API keys.

## Related

- [Start](../start/skill.md) · [Commands](../commands/skill.md) · [Scripts](../scripts/skill.md)
- [Building blocks](../building-blocks/skill.md)
- [Index](../README.md)
