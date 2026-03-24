# Speed-CLI — Agentic docs

**GitHub Pages (this repo is the site root):** [../index.html](../index.html) — **Settings → Pages → Branch: `main`, folder `/ (root)`**, then open [https://lightspeedfoundation.github.io/speed-skills/](https://lightspeedfoundation.github.io/speed-skills/) (after the first deploy).

Skill-style pages for **GitHub Pages**: YAML frontmatter (when to load), then digestible sections — **mental models**, **tables**, **pitfalls**, **related links**. These docs cover the **core multichain CLI** and **speed-scripts**. They **do not** cover SAI (`speed agent`), Speed Terminal (`speed terminal`), or Launchpad (`speed launch`).

## How agents should use this folder

1. **Match intent** using the `description` in each file’s frontmatter (trigger terms are listed there).
2. **Read one skill end-to-end** before mixing commands — [Orchestration](./orchestration/skill.md) ties phases together.
3. **Deep CLI flags** — always confirm with `speed <cmd> --help`; [Commands](./commands/skill.md) summarizes roles, not every flag.
4. **Upstream script catalog** — [Scripts](./scripts/skill.md) links to **scripts-reference** and **function-builder** (canonical for decimals and shell patterns).

## Skill index

| Section | Skill file | What’s inside (high level) |
|---------|------------|----------------------------|
| [Start](./start/skill.md) | Install, `speed setup`, MCP, `speed skill`, `whoami`, scripting flags, **advanced setup** (Cursor + clone speed-scripts, example cbBTC / trailing-stoploss-any) |
| [Why Speed-CLI](./why/skill.md) | Problems solved, design pillars, what the CLI is *not*, relation to speed-scripts |
| [Protocol](./protocol/skill.md) | Env matrix, chains/explorers, SPEED address, 0x/Squid/oracles, RPC, MCP merge rules, identity/SANS |
| [Commands](./commands/skill.md) | Global flags, JSON contract, grouped subcommands, `doctor` behavior |
| [Scripts](./scripts/skill.md) | Clone [speed-scripts](https://github.com/lightspeedfoundation/speed-scripts/), folder discovery, `*-skill.md`, upstream doc links |
| [Building blocks](./building-blocks/skill.md) | Base vs target, strategy families, regime thinking, hazards, decimals |
| [Orchestration](./orchestration/skill.md) | Chaining JSON, multi-terminal, agent phases, errors, logging |
| [Identity & SANS](./identity-sans/skill.md) | `.speed` on Base, profiles, SANS / OpenSea listings and offers |
| [MCP](./mcp/skill.md) | `speed start`, `mcp-url`, encrypted env, trust model, debugging |
| [Security](./security/skill.md) | Keys, allowances, revoke, MCP vs wallet boundary |
| [Observability](./observability/skill.md) | `doctor`, `history`, `pending`, `xp` |

**GitHub:** [index.html](../index.html) is the styled landing page. Section links point to **rendered** Markdown on GitHub (`blob/main/agentic/.../skill.md` in [speed-skills](https://github.com/lightspeedfoundation/speed-skills)). The same paths are served on Pages as raw `.md` if opened directly on the site.

**Optional later:** per-command pages from `--help`, a fuller chain appendix, public-site risk disclaimers, and pinned install examples (`npm install -g @lightspeed-cli/speed-cli@x.y.z`) — none of these block publishing.
