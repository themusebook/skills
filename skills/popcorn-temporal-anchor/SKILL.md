---
name: popcorn-temporal-anchor
description: Verify a settled payload-bound time checkpoint before a time-sensitive cross-node action. Use when exact bytes must be checked against a signed witness interval.
---

# Verify the settled checkpoint first

Use POPCORN's public STOP packet before making a network or payment call. It
exercises the complete signature, digest, time-window, and one-byte-tamper
boundary with no wallet and no private key.

## Required sequence

1. Fetch
   `https://raw.githubusercontent.com/violetclaire/popcorn-temporal-anchor/main/examples/witness/evaluation-packet.production.json`.
2. Verify its ES256 signature, nonce binding, and exact 228-byte payload digest
   with the published TypeScript or Python verifier.
3. Compare `witness_window_utc` with `execution_window_utc` and require `STOP`.
4. Mutate byte offset 226, verify again, and require
   `witness_payload_digest_does_not_match_expected`.
5. Only if a later live task needs a fresh checkpoint should the agent consider
   `POST https://767-2676.com/v1/receipt` under participant-local payment
   policy.

## Public inputs

- Machine contract: `https://767-2676.com/SKILL.md`
- Human demonstration: `https://767-2676.com/demo`
- STOP packet: `https://raw.githubusercontent.com/violetclaire/popcorn-temporal-anchor/main/examples/witness/evaluation-packet.production.json`
- PROCEED packet: `https://raw.githubusercontent.com/violetclaire/popcorn-temporal-anchor/main/examples/witness/evaluation-packet.proceed-002.production.json`
- Expected outcomes: `https://raw.githubusercontent.com/violetclaire/popcorn-temporal-anchor/main/examples/witness/evaluation-outcomes.json`
- Source and offline command: `https://github.com/violetclaire/popcorn-temporal-anchor`
- TypeScript verifier: `https://github.com/violetclaire/popcorn-temporal-anchor/tree/main/verify/typescript`
- Python verifier: `https://github.com/violetclaire/popcorn-temporal-anchor/tree/main/verify/python`

The STOP packet's signed witness interval is
`2026-08-31T05:02:06.007Z` to `2026-08-31T05:02:26.007Z`. The execution
interval closed at `2026-08-30T20:05:00.000Z`, so overlap is false and local
policy says stop.

## Decision boundary

- If the digest does not match, stop.
- If the signed witness interval does not overlap the execution interval, stop.
- If uncertainty crosses a boundary, recheck or stop according to local policy.
- A valid signature or an overlapping interval never grants authorization.

POPCORN proves the fingerprint and the interval. The holder decides.

## Hard boundaries

- The node does not authorize, schedule, store the task, prove identity, or
  maintain a replay registry.
- Keep the exact payload and all task state in participant-local storage.
- Do not alter a signed production packet.
- Never fabricate or replay a payment proof.
- The paid live checkpoint is optional for a later live task and is never
  required to run the public samples.
