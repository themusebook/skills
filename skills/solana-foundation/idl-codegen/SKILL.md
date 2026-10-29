---
name: solana-dev-idl-codegen
description: Generate type-safe program clients from IDLs using Codama, eliminating hand-maintained serializers across languages.
license: MIT
metadata:
  author: Solana Foundation
  version: "2.7.0"
  source: https://github.com/solana-foundation/solana-dev-skill
  category: Tooling
---

# IDL & Client Code Generation

> **Solana Foundation skill — mirrored in the Musebook skills hub.**
> This entry reproduces, verbatim, the `idl-codegen.md` reference of the
> foundation-maintained **solana-dev** skill (v2.7.0, MIT) from
> https://github.com/solana-foundation/solana-dev-skill. It is one of the 16 topic
> references the solana-dev router discloses progressively; for the full
> skill (router + all references + nested guides), install the upstream
> original: `npx skills add https://github.com/solana-foundation/solana-dev-skill` — also mirrored in this hub as the
> `solana-dev` entry. Maintained by the Solana Foundation; Musebook does
> not author or maintain this content. Relative links in the body point
> at the upstream skill's file layout and resolve in the full install.

# IDLs + client generation (Codama / Shank)

## Goal
Never hand-maintain multiple program clients by manually re-implementing serializers.
Prefer an IDL-driven, code-generated workflow.

## Codama (preferred)
- Use Codama as the "single program description format" to generate:
  - TypeScript clients (including Kit-friendly output)
  - Rust clients (when available/needed)
  - documentation artifacts

## Anchor → Codama
If the program is Anchor:
1) Produce Anchor IDL from the build
2) Convert Anchor IDL to Codama nodes (nodes-from-anchor)
3) Render a Kit-native TypeScript client (codama renderers)

## Native Rust → Shank → Codama
If the program is native:
1) Use Shank macros to extract a Shank IDL from annotated Rust
2) Convert Shank IDL to Codama
3) Generate clients via Codama renderers

## Repository structure recommendation
- `programs/<name>/` (program source)
- `idl/<name>.json` (Anchor/Shank IDL)
- `codama/<name>.json` (Codama IDL)
- `clients/ts/<name>/` (generated TS client)
- `clients/rust/<name>/` (generated Rust client)

## Generation guardrails
- Codegen outputs should be checked into git if:
  - you need deterministic builds
  - you want users to consume the client without running codegen
- Otherwise, keep codegen in CI and publish artifacts.

## "Do not do this"
- Do not write IDLs by hand unless you have no alternative.
- Do not hand-write Borsh layouts for programs you own; use the IDL/codegen pipeline.
