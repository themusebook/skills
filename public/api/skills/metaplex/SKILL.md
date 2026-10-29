---
name: metaplex
description: Official Metaplex Skill for Solana — a SKILL.md router with progressive-disclosure reference files giving AI coding agents accurate knowledge of Metaplex programs, CLI commands, and SDK patterns. Use when building with the Metaplex Agent Registry, Genesis token launches, Core NFTs and plugins, Token Metadata, Bubblegum compressed NFTs, or Core Candy Machine drops, via the mplx CLI, the Umi SDK, or (Token Metadata only) the Kit SDK (@solana/kit). Hub entry: install the official skill with `npx skills add metaplex` for the full reference files.
user-invocable: true
compatibility: Requires Node.js and npm (for `npx skills add metaplex`)
metadata:
  author: Metaplex (Metaplex Global Ltd)
  version: 1.0.0
  source: https://developers.metaplex.com
  documentation: https://developers.metaplex.com
---

# Metaplex — Official Skill (Hub Entry)

This is a hub entry / pointer for the **official Metaplex Skill** by Metaplex
(Metaplex Global Ltd). It summarizes what the official skill covers; it does
not reproduce its reference files. Install the official skill (below) for the
full router and reference content.

## Summary

The official Metaplex Skill is an Agent Skill: a `SKILL.md` router plus
progressive-disclosure reference files that give AI coding agents accurate
knowledge of Metaplex programs, CLI commands, and SDK patterns — so agents
load the right program knowledge for a task instead of guessing at accounts,
instructions, or SDK calls.

## Install

```bash
npx skills add metaplex
```

Requires Node.js / npm.

## Programs

| Program | Purpose | CLI | Umi | Kit |
| --- | --- | --- | --- | --- |
| Agent Registry | On-chain agent identity, wallets, and execution delegation | Yes | Yes | — |
| Genesis | Token launches via launchpool or bonding curve, with Raydium graduation | Yes | Yes | — |
| Core | Next-generation NFTs: plugins and royalty enforcement | Yes | Yes | — |
| Token Metadata | Fungible tokens, NFTs, pNFTs, and editions | Yes | Yes | Yes |
| Bubblegum | Compressed NFTs (cNFTs) via Merkle trees | Yes | Yes | — |
| Core Candy Machine | NFT drops with configurable guards | Yes | Yes | — |

## Operations summary

- **mplx CLI** (`@metaplex-foundation/cli`) — supported for all six programs:
  Agent Registry, Genesis, Core, Token Metadata, Bubblegum, and Core Candy
  Machine.
- **Umi SDK** — supported for all six programs.
- **Kit SDK** (`@solana/kit`) — supported for Token Metadata only.

## Compatible agents

Works with Agent-Skills-format agents, including Claude Code, Cursor,
GitHub Copilot, Codex, and Windsurf.

## Progressive disclosure

The official skill is structured as a router: the top-level `SKILL.md`
directs the agent to per-program reference files, which are loaded only when
a task needs them. This hub entry is only a summary — install the official
skill to get the router and its reference files; do not rely on this page as
a substitute for them.

## Links & attribution

- Official skill by **Metaplex (Metaplex Global Ltd)**.
- Documentation: https://developers.metaplex.com
- Install: `npx skills add metaplex`
