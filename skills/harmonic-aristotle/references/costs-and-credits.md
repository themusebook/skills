# Aristotle costs and credits — the dated calibration record

Everything here is an observation from 2026-11-18 or a labelled estimate built on one. Harmonic publishes no per-proof pricing, so treat any number older than the pilot as a planning input, not a quote, and re-derive the burn rate from the balance after the next run before planning further spend.

## The credit model

- One credit was treated as approximately one US dollar in pilot-era planning. FLAG (docs re-checked 2026-11-25): no current Harmonic page — docs, PyPI, or product page — states a credit-to-dollar rate. The CLI reference says `aristotle balance` amounts use the billing page's configured Credit unit and precision and that the command prints the conversion, so the rate is an organization billing setting, not a published constant. Every dollar figure in this file (including the $1,000–$2,000 completion estimate) rests on the ≈$1 assumption; re-read the conversion from a live balance check before treating any of them as money.

- Billing is consumption-based: credits drain while a run works, not per property closed. Proved, refuted, and unresolved properties all consume; the pilot paid full price for seven unresolved ones.

- Sign-up included 500 free credits. The pilot's grant was listed as an active grant of 500 Credits with an expiry of 2026-12-18 — granted credits can expire, so a grant's date is part of its value. (External framing conflicts, unresolved by the docs: Harmonic's own site banner says "500 free credits", while third-party launch coverage described "$500 in credits, valid for 30 days". The docs state neither a dollar value nor a standard validity; the account read above is the only figure we treat as fact.)

- The first run asks which organization pays. Per Harmonic's docs, runs billed to that organization then start unasked for 90 days unless credits run low. An approval to submit once is therefore not a standing approval, and this skill's per-project gate applies to every submission regardless.

- One public report recorded at pilot time described roughly $500 of credit burning on a single large C memory-safety job. It is an anecdote from Harmonic's docs, not a measurement of ours, and it points the same way as the pilot: hundreds of credits per substantial run is normal.

## The pilot burn (the one hard measurement)

- Project: Dark Protocol, an Anchor 0.32.1 shielded-pool program, roughly 1,760 lines of Rust in the program crate, seven properties (no UB/no panics, lamport conservation, withdraw gating, deposit binding, authority impotence, withdraw progress with error classification, Merkle root/frontier integrity).

- Submitted 2026-11-18T16:52:08Z on the 500-credit grant, 0 purchased. Task started 16:52:10Z, last updated 18:51:09Z — about 1 hour 59 minutes of work, during which Aristotle built its Lean model of the program.

- Outcome: OUT_OF_BUDGET. All seven properties unresolved, 0 proved, 0 refuted, no counterexamples, no verification boundary statement produced. The model exists; the fuel did not.

- Balance reads during the shared-grant afternoon (EST, 2026-11-18; the grant was shared with the orderbook pilot's submission, which cost 3 credits at submit):

  - ~11:52 — 500 available at the Dark submission.
  - ~12:54 — 322 available (319 after the orderbook submit).
  - ~13:00 — 266 available.
  - ~13:10 — 162 available.
  - ~13:39 — 0 available; grant fully consumed.

- Reading of the curve: the drain is back-loaded and fast once the model is built and the prover is working — roughly 160 credits in the ten minutes between the last two mid-run reads. Do not extrapolate an hourly rate from the quiet first hour; budget from the total, not the slope.

## The completion estimate

- The grant application and READINESS.md estimate finishing all seven Dark properties at $1,000 to $2,000 in credits, derived from the pilot's burn: the model-building cost is already sunk into the existing project, and a continuation re-enters with the Lean model in place.

- READINESS.md allocates that range per property (deposit binding and conservation first, withdraw progress the expected priciest) with a floor plan: at $1,000, fund in queue order and hold the tail rather than spreading thin; below $1,000, deposit binding plus conservation only. Read the table there before proposing any funded run; do not re-derive allocations in chat.

- The orderbook pilot is a separate, paused project (submitted 2026-11-18, cancelled while queued on the dev's order). Restarting it is a new billed run via continue and needs its own approval and its own budget line; it is not inside the Dark grant estimate.

## What is not known

- No per-proof, per-property, or volume pricing has been published or quoted to us. Whether refuted and unresolved properties price differently from proved ones was asked in the unsent pre-flight questions and never answered.

- Whether a continuation re-bills any of the model-building work, or prices differently from a fresh project, is unobserved — the pilot never continued. The first continuation's balance delta is the next calibration point; record it.

- Grant mechanics for the Foundation-route application (how credits attach to the account, any expiry) are unknown until an approval arrives, per READINESS.md §0.8.
