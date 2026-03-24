---
name: agentic-security
description: >-
  Speed-CLI security: local signing wallet vs MCP (API keys only), allowance approve revoke,
  operational hygiene, Base builder code. Triggers: key safety, revoke approvals,
  least privilege, compromised key response.
compatibility: >-
  Local filesystem access for CLI config; no custodial recovery if the signing key is lost.
---

# Security

## Agent intent

Use this skill when the user asks about **keeping keys safe**, **revoking token access**, **what MCP can and cannot see**, or **hygiene after scripted swaps**. This is about **risk and controls**, not feature discovery.

## Trust boundaries

| Asset | Role | MCP / remote |
|-------|------|--------------|
| **Signing key** | From **`speed setup`** and normal env load (see [Protocol](../protocol/skill.md)) | **Never** applied from MCP — server env merge skips signing-key names. |
| **API keys** (0x, Alchemy, Squid, OpenSea) | Local setup + optional **MCP** injection | May come from **MCP** `get_speed_env_vars` after **`speed start`**. |
| **MCP URL** | Persisted after **`speed start`** | Not a secret; identifies which server to call. |

**Implication:** Treat the **MCP server operator** as trusted for **API keys**, not for **wallet custody**. Anyone who holds the **signing key** can move funds.

## ERC-20 allowances

Many flows use **0x AllowanceHolder** or protocol spenders. The CLI may **approve** automatically during **`speed swap`**; for manual or debugging flows:

| Command | Use |
|---------|-----|
| **`speed allowance`** | Read current allowance for **`--token`** + **`--spender`**. |
| **`speed approve`** | Set allowance; **`max`** for unlimited (convenience vs blast radius). |
| **`speed revoke`** | Set allowance to **0** after operations (hygiene). |

**Agent guidance:** After a scripted batch, prefer **revoke** or **finite approve** if the user wants minimal lingering approvals — confirm spender address from **`speed swap --dry-run`** / docs for their CLI version.

## Configuration files

- **Setup secrets** — must **never** be committed; add to **`.gitignore`** any project-local file that mirrors secrets.
- **Cwd `.env`** can override global keys for most vars — be explicit which directory you run `speed` from in CI.

## Key handling practices

| Practice | Why |
|----------|-----|
| **Funded hot wallet** | CLI usage is typically a **hot** key; limit balance for operational risk. |
| **Separate keys** | Use different keys for bots vs savings (not enforced by CLI — operational). |
| **Rotate if leaked** | Compromised signing key ⇒ move assets to a new wallet; **revoke** old token approvals if the wallet is still used. |
| **`setup --skip`** | Can **generate** a new random wallet if none exists — fund before mainnet use. |

## Base Builder Code

On **Base**, the CLI may attach **Builder Code** to transactions for attribution (see project README). This is **not** a security control; it affects **fee attribution**, not custody.

## MCP-specific security

- **Encrypted env** (RSA + AES) reduces exposure in transit vs plain JSON — requires **`speed_mcp_key.pem`**.
- **Plain env** from MCP is still **TLS-dependent** on HTTPS URLs; do not use untrusted networks without VPN judgment.

## What this document is not

- Not legal or compliance advice.
- Not an audit of 0x, Squid, or OpenSea — those are external dependencies.

## Related

- [MCP](../mcp/skill.md) (trust model, encrypted env)
- [Protocol](../protocol/skill.md) (env loading)
- [Start](../start/skill.md) (setup)
- [Commands](../commands/skill.md) (`approve`, `revoke`, `allowance`)
- [Index](../README.md)
