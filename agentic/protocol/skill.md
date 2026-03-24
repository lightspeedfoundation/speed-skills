---
name: agentic-protocol
description: >-
  Speed-CLI system protocol: on-disk layout, full env var matrix, chains and explorers,
  SPEED token, oracles, 0x swaps, Squid bridges, MCP merge (signing keys never from server),
  RPC Alchemy vs public, identity registry. Triggers: env troubleshooting, architecture.
---

# Protocol

## System shape (layers)

```
Local signing wallet (from speed setup)
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│  speed CLI  ──►  JSON-RPC (Alchemy or public fallback)     │
│       │              │                                       │
│       ├──────────────┼──► 0x API (swap / quote / gas path)   │
│       ├──────────────┼──► Squid API (bridge quotes / status) │
│       └──────────────┴──► On-chain: oracles, ERC-20, routers │
└──────────────────────────────────────────────────────────┘
Optional: MCP ──► merges API keys into process.env (not signing keys)
```

## On-disk layout (`~/.speed` or `SPEED_CONFIG_DIR`)

| File | Contents |
|------|----------|
| **`.env`** | Signing key + API keys from **`speed setup`**; optional identity overrides — see command help / env matrix below. |
| **`config.json`** | `defaultChain`, `defaultSlippage`, `outputFormat` — readable via `speed config get`. |
| **`mcp-url`** | Single-line MCP base URL (no query); used after successful `speed start`. |
| **`speed_mcp_key.pem`** | Client key for RSA + AES decryption of MCP env payloads. |

**Legacy migration:** old `mcpUrl` inside `config.json` or `SPEED_MCP_URL=` in `.env` is migrated to `mcp-url` on load.

## Environment variables (matrix)

| Variable | Required for | If missing |
|----------|--------------|------------|
| **`PRIVATE_KEY`** | Any signing: `swap`, `bridge`, `send`, `approve`, `identity`, most `sans` | Commands fail at wallet init |
| **`0X_API_KEY`** (or `OX_API_KEY`) | `quote`, `swap`, `gas` (swap path), `volume`, `dca`, `estimate` (swap) | 0x API errors |
| **`SQUID_INTEGRATOR_ID`** | `bridge`, `status` with bridge request id | Squid integration errors |
| **`ALCHEMY_API_KEY`** | Best RPC; **`history`**, **`pending`** as implemented | Public RPC fallback; history/pending may be limited or empty |
| **`OPENSEA_API_KEY`** | Stable `sans listings`, listing, some offer flows | Rate limits or failures on OpenSea-heavy commands |
| **`SPEED_REGISTRY_ADDRESS`**, **`SPEED_FACTORY_ADDRESS`** | Override defaults for `identity` | Uses packaged Base defaults from docs / `.env.example` |

**MCP-supplied keys:** When `speed start` succeeds, the server may inject API keys (e.g. Alchemy, 0x). **Signing keys from MCP are never applied** — the wallet always comes from local setup.

## Supported mainnet chains (CLI swap/bridge surface)

Chain IDs **1, 8453, 10, 42161, 137, 56** are first-class for resolution (`resolveChainId`). Names like `ethereum`, `base`, `op`, `arbitrum`, `polygon`, `bnb`/`bsc` map to these IDs.

| ID | Common names | Native | Wrapped (gas display) | Explorer (tx links) |
|----|----------------|--------|------------------------|----------------------|
| 1 | ethereum, eth, mainnet | ETH | WETH | etherscan.io |
| 8453 | base | ETH | WETH | basescan.org |
| 10 | optimism, op | ETH | WETH | optimistic.etherscan.io |
| 42161 | arbitrum, arb | ETH | WETH | arbiscan.io |
| 137 | polygon, matic | MATIC | WPOL | polygonscan.com |
| 56 | bnb, bsc | BNB | WBNB | bscscan.com |

**Default chain:** **8453 (Base)** if not overridden by `-c` or `speed config set default-chain`.

## SPEED token

- **Address (canonical):** `0xB01CF1bE9568f09449382a47Cd5bF58e2A9D5922` — alias **`speed`** on CLI flags.
- Same address is used across supported chains where SPEED is deployed.

## Native ETH sentinel (0x)

**`eth`**, **`ether`**, **`native`** resolve to **`0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`** — 0x’s native-asset sentinel for sell/buy legs.

## Oracles (`speed price`)

Lightspeed **oracle contracts** per chain (see codebase `ORACLE_ADDRESSES`) expose reads such as SPEED/ETH, SPEED/USD, ETH/USD. `doctor` probes oracle reads per supported chain when checking health.

## Swaps (0x)

- HTTP API base: **`https://api.0x.org`** (see `ZEROX_API_BASE`).
- Flow: **`speed quote`** (preview) → **`speed swap`** (approve if needed + fill).
- Slippage: default from **`speed config`** `default-slippage` when applicable.

## Bridges (Squid)

- API base: **`https://v2.api.squidrouter.com`** (`SQUID_BASE_URL`).
- **`speed bridge`** submits the transaction; response includes **`requestId`** (and often **tx hash**).
- **`speed status --request-id …`** polls Squid; combine with **`--tx`** for chain confirmation when needed.

## RPC selection

- If **`ALCHEMY_API_KEY`** is set, RPC URLs use Alchemy hostnames per chain (`ALCHEMY_CHAIN_PREFIX` mapping).
- Otherwise **public fallbacks** (e.g. `mainnet.base.org` for Base) from `PUBLIC_RPC_FALLBACKS`.
- Optional **`RPC_URL`** in project docs may override Base in some setups — check `.env.example` / README for your release.

## MCP (detailed behavior)

1. **`speed start <url>`** normalizes URL, fetches env via MCP tool **`get_speed_env_vars`**.
2. Supports **plain** `{ "env": { ... } }` or **encrypted** payloads decrypted with `speed_mcp_key.pem`.
3. **Merged keys** apply to `process.env` except signing-key env names and `SPEED_MCP_DEBUG` (see implementation).
4. **`0X_API_KEY`** mirrors to **`OX_API_KEY`** when set.
5. On every CLI run (except when the first argv is `start`), if an MCP URL is configured, **`loadMcpEnvAndMerge`** may run **before** subcommands — so API keys can arrive without manual export.

## Identity protocol (Base)

- **`speed identity`** targets **chain 8453** for registry/factory unless env overrides addresses.
- Registering a `.speed` name costs on-chain ETH (see command help for current economics).

## SANS / OpenSea

- **`speed sans`** uses OpenSea API for listings, offers, and discovery where implemented.
- **`OPENSEA_API_KEY`** improves reliability and rate limits.

## Related

- [Start](../start/skill.md) (bootstrap + MCP) · [Commands](../commands/skill.md) (invoke surface)
- [MCP](../mcp/skill.md) · [Identity & SANS](../identity-sans/skill.md) · [Security](../security/skill.md) · [Observability](../observability/skill.md)
- [Building blocks](../building-blocks/skill.md) · [Orchestration](../orchestration/skill.md)
- [Index](../README.md)
