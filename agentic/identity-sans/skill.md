---
name: agentic-identity-sans
description: >-
  .speed names on Base via speed identity (register, resolve, profile, community)
  and SANS tradeable names on OpenSea via speed sans (listings, buy, list, offers).
  Triggers: register agent name, set profile, buy or sell .speed NFT, OpenSea SANS.
compatibility: >-
  Base chain 8453 for identity/SANS flows; ETH for registration fees; OPENSEA_API_KEY
  recommended for stable OpenSea API usage.
---

# Identity & SANS

## Agent intent

Use this skill when the user cares about **on-chain identity** (`.speed` names, profiles, resolution) or **marketplace** activity (discover, buy, list, offer on **SANS** — Speed Agent Name Service on OpenSea), **not** when they only want swaps or bridges.

| Intent | Primary commands |
|--------|------------------|
| Mint / configure a `.speed` name | `speed identity …` |
| Trade or discover `.speed` as NFTs | `speed sans …` |

## Mental model

- **Identity (`identity`)** — ERC-8004-style **registry + factory + per-name instance** on **Base**. Registering mints an identity; you set **resolve address**, **profile** fields, and optional **community favorite token** rules.
- **SANS (`sans`)** — Same **factory** NFTs exposed through **OpenSea** (collection **`speedcli-sans`**). Flows hit **OpenSea API** (`api.opensea.io`) and on-chain **Seaport**-style listing/bids; **offers use WETH** on Base, not raw native ETH.

Default on-chain addresses can be overridden — see **Environment**.

## Environment

- **Signing wallet** — same as other mutating CLI commands (configure via **`speed setup`**).
- **`SPEED_REGISTRY_ADDRESS`** — override SpeedRegistry (default in CLI for public Base registry).
- **`SPEED_FACTORY_ADDRESS`** — override factory used for **both** identity resolution and SANS NFT operations.
- **`OPENSEA_API_KEY`** — sent as **`X-API-KEY`** on OpenSea requests; improves reliability for **`sans listings`**, listing, offers, etc.

## Chain scope

- **Identity** subcommands default to **Base (`8453`)**; registry/factory are deployed for that ecosystem.
- **SANS** is **Base-only** in implementation (`BASE_CHAIN_ID = 8453`): listings, buy, list, unlist, transfer, offers target this chain.

## Identity — subcommands (overview)

| Area | Commands | Notes |
|------|----------|--------|
| **Registration** | `identity register <name>` | Name must end with **`.speed`**; **≥ 3 characters** before `.speed`. **~0.0025 ETH** registration fee (on-chain constant). |
| **Resolution** | `identity set-resolve <name> <address>` | Points the identity at a wallet/contract. |
| **Profile** | `identity profile get`, `identity profile set` | Username, bio, URLs, picture URL; string limits enforced on-chain (see CLI). |
| **Favorite token** | `identity set-favorite-token` | Owner’s favorite token for the identity. |
| **Community** | `identity community set-fee`, `identity community set` | Optional fee so others can set **community** favorite token. |

Use **`speed identity --help`** and **`speed identity <sub> --help`** for full flags and JSON shapes.

**Errors:** Registry may return structured hints (e.g. identity contract not set, invalid name prefix) — CLI maps some custom errors to clear messages.

## SANS — subcommands (overview)

| Area | Commands | Notes |
|------|----------|--------|
| **Discovery** | `sans listings` | OpenSea listings; **`--json`** for machine output; **`--limit`** capped (e.g. up to 200). |
| **Inventory** | `sans owned` | `.speed` names you own on Base (OpenSea API + factory). |
| **Buy** | `sans buy <nameOrTokenId>` | Purchase a listed name (or token id). |
| **Sell** | `sans list`, `sans unlist` | Seaport listing; may require **NFT approval** for OpenSea conduit. |
| **Transfer** | `sans transfer` | NFT transfer; typically **unlist** first if listed. |
| **Offers** | `sans offers`, `sans offer`, `sans accept-offer`, `sans cancel-offer` | **WETH** on Base; CLI may wrap ETH → WETH when needed. |

Collection: **`speedcli-sans`** — [OpenSea collection](https://opensea.io/collection/speedcli-sans).

Always pass **`--json`** for agent parsing where supported.

## JSON & automation

- Prefer **`speed --json`** with subcommands that support it.
- After mutating txs, capture **`txHash`** / explorer links from JSON when returned.

## Pitfalls

| Issue | Mitigation |
|-------|------------|
| Name not ending in `.speed` | CLI rejects with `INVALID_NAME`. |
| Wrong chain expectations | Identity/SANS flows are **Base-centric**; don’t assume L2-agnostic behavior without reading `--help`. |
| OpenSea rate limits / empty listings | Set **`OPENSEA_API_KEY`**; retry with backoff. |
| Offers vs native ETH | Offers are **WETH**-denominated on Base for Seaport flows. |

## Related

- [Protocol](../protocol/skill.md) (env, explorers)
- [Security](../security/skill.md) (keys, approvals before listing)
- [Commands](../commands/skill.md) (full command list)
- [Start](../start/skill.md)
- [Index](../README.md)
