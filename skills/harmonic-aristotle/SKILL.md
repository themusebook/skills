# Harmonic Aristotle Formal Verification

## Know what Aristotle is, and what it is not

- Treat Aristotle as Harmonic's hosted formal-verification service. It compiles a Rust program into a Lean model and returns, per stated property, proved, refuted with a runnable counterexample, or unresolved, together with an explicit verification boundary stating what was reached and what was assumed.

- Keep Aristotle and Certora as separate vendors with separate surfaces. Certora is driven through the certoraRun CLI with hand-written CVLR specs and a CERTORAKEY, and bills in prover minutes; Aristotle is a hosted upload service driven through its own CLI, bills in credits, and reasons over a Lean model Harmonic builds. For the Certora route read the certora-solana-fv skill; for the full cold-start walkthrough of Aristotle read the Harmonic onboarding guide.

- Verify Rust only. Aristotle says nothing about Circom circuits, Groth16 soundness, trusted setups, or ceremonies, and it performs no compute-unit, termination, or liveness reasoning. Keep those claims with the audits, the ceremony, and measurement.

- Expect proofs about a neighbouring build. Aristotle's documented compiler is a single rustc 1.97.0-nightly, while Solana programs here build with platform-tools rustc 1.95 under the Anza fork, so a verdict covers the source under Harmonic's compiler semantics rather than the exact deployed artefact.

- Model the syscall boundary honestly. Harmonic's docs list modelled Solana syscalls as sysvars, SHA-256, program-derived addresses, curve-point validation, and CPI. sol_poseidon and the alt_bn128 pairing path are not listed, so state them as contracts in the spec (a deterministic hash function; a verifier oracle accepting only valid proofs) and let the boundary statement carry them as assumptions, never as proved facts.

- Hold the Manifest case study as the calibration for what a good run looks like. Aristotle's Rust support launched on 2026-11-16 with a Solana Foundation partnership, and for Manifest's Solana order book it produced 28 proved results, found 2 real bugs that were then fixed, and proved a swap progress property with every error classified by origin.

## Connect and install the CLI

- Start from Harmonic's official installation docs, the canonical setup source: https://aristotle.harmonic.fun/docs/installation (the "MCP & CLI" page: per-client MCP setup, CLI install, and the login flow). The docs' primary CLI install is `uv tool install "aristotlelib>=3"` (global, no venv to create or activate); installing into a venv with pip is their documented alternative, and is also how you get the library. The pinned route below is our hardened variant of that alternative; where it differs from the docs' defaults it differs deliberately, and says why.

- Install the pinned CLI into a venv under the home directory at ~/workspace/aristotle-cli, with python3 -m venv followed by pip install of aristotlelib==3.4.0. Confirm aristotle --version prints aristotlelib 3.4.0 before first use, matching the version the pilot ran on. (Checked 2026-11-25: PyPI's current release is still 3.4.0, dated 2026-11-18; the docs pin no patch version — their contract is the 3.x line, and the version check prints `aristotlelib 3.` plus the rest.)

- Never install into root-overlay scratch such as /aristotle-scratch. The pilot's venv lived there and was lost in the VM replacement of 2026-11-24, while everything under the home directory survived. The home venv path supersedes the old scratch path wherever older records name it.

- Prefix every aristotle invocation on this VM with env NO_PROXY=localhost,127.0.0.1 no_proxy=localhost,127.0.0.1. The HTTP client inside aristotlelib chokes on this VM's NO_PROXY value, which carries an IPv6 range, and commands fail without the sanitised pair.

- Log in with aristotle login redirected from /dev/null. The stdin EOF makes the CLI take the browser device-code path: it prints an approval-page address and a code, a human approves it in a browser, and the human creates the account and chooses which organization the machine key belongs to on first login.

- Treat the stored credential as a name and a location only. Login writes a machine-named API key to ~/.config/aristotle/credentials.json with 0600 permissions; never open it, print it, copy it, or commit it. The key is revocable in the web UI under API keys, and the pilot key was billed to the dev's personal organization.

- Note the MCP alternative without treating it as a shortcut. Harmonic runs a hosted MCP server at https://aristotle.harmonic.fun/mcp; per-client setup is on the installation page (https://aristotle.harmonic.fun/docs/installation). It signs in with OAuth against the same account — there is no key to copy — and exposes the full workflow as tools (get_credit_balance, create_upload, start_run, get_run, wait_for_run, answer_question, send_message, list_properties, get_property, continue_run, list_projects, list_runs, create_download, cancel_run, rename_project, delete_project, send_feedback). It operates on the same uploaded code, so it changes the transport, not the disclosure decision, and it inherits the per-project approval gate in full. This route is documented by Harmonic and untested by us.

## Respect the credit discipline

- Read credits as the billing unit, and treat the dollar conversion as unverified. Consumption is metered while a run works, and Harmonic publishes no per-proof or per-run price list. FLAG (checked against the docs 2026-11-25): no current Harmonic page states a credit-to-dollar rate; the CLI reference says balance amounts use the billing page's configured Credit unit and precision, and `aristotle balance` prints the conversion for the organization. Pilot-era planning treated one credit as approximately one US dollar, and every dollar figure in this skill and its references rests on that assumption — keep them labelled as estimates, and read the live conversion from a balance check before proposing spend.

- Check the balance with aristotle balance before and after every run. The output separates Available from Free, Purchased, and Granted lines, and watching it drain during the pilot is how the burn rate became known.

- Never buy credits, and never run aristotle continue unasked. A continue starts a brand new billed run on the same project rather than resuming the old one for free, and after an out-of-credits stop it is the exact command that spends purchased credits. Both wait for the dev's explicit go.

- Remember the billing consent shape from Harmonic's docs. The first run asks which organization pays, and runs billed to that organization then start unasked for 90 days unless credits run low, so an approved submission is not a one-off authorisation for later submissions.

- Plan against the pilot calibration before proposing any spend. The 500-credit signup grant, which carried an expiry date, was fully consumed by one ~1,760-line Anchor program with seven properties in about two hours with zero properties closed. Finishing those seven is estimated at $1,000 to $2,000. Read references/costs-and-credits.md for the full dated record.

## Prepare the project and the bundle

- Write the spec in Aristotle's style before touching the CLI. State each property in plain English in terms of the program's real functions, types, and fields, place the spec file at the crate root next to Cargo.toml, and state the external contracts up front, naming any extra assumption such as hash collision resistance exactly where a property needs it.

- Survey the program first and let the properties map to claims a reader can audit, the same discipline as the Certora route. A property that names real check order, real state fields, and real error variants is the kind Aristotle can prove or refute usefully.

- Stage the upload as an explicit hand-built file list in its own folder. The pilot bundle was 10 files and 49,396 bytes: the crate's Cargo.toml, the workspace Cargo.lock, the spec, and seven source files, with target, .git, node_modules, examples, and every other tree excluded by construction rather than by filter.

- Run the secret scan on the staged list before anything is packed or uploaded. Scan filenames for keypair, secret, .env, id_rsa, .pem, deploy, and mnemonic, and scan contents for PEM blocks, deploy-keys paths, mnemonic and seed phrase text, PRIVATE KEY markers, and api_key assignments. Stop on any hit that is not a benign word in a comment, and never let deploy keys or any keypair material enter the staging folder at all.

- Reconfirm the upload set with a dry run. aristotle submit with --dry-run sends nothing and prints the exact file count and size it would upload; require that line to match the staged list exactly before the real submission is even proposed.

- Respect the upload shape. Compressed uploads are capped at 100 MB, and while the CLI keeps credentials, .git, and gitignored files out of uploads from a git checkout, the hand-built and dry-run-verified bundle is the route with a known-exact contents list.

## Pass the disclosure gate and submit

- Obtain the dev's explicit approval for each project before any upload. Uploading is a third-party disclosure of source that may not be public yet, and one project's approval never covers another: the Dark Protocol pilot was approved on 2026-11-18, the orderbook pilot was approved separately the same day, and the dev later ordered the orderbook run paused and it was cancelled through the CLI while still queued.

- Submit exactly once per approval. Use aristotle submit with the property prompt, --project-dir pointing at the staged folder, and --name for the project, adding --wait only when following in the terminal is wanted. Record the project ID and task ID in the project's record file as soon as the submission returns.

- Expect the run to take minutes to days and to bill while it works. A queued task can still be cancelled through the CLI on the dev's order; a running task that exhausts credits stops on its own and reports the stop as a credit problem, not as a verdict.

## Run the questions protocol

- Watch for clarifying questions during a run, in the web app or through aristotle questions. Harmonic's docs describe a roughly 15-minute answer window, but the pilot showed the window is soft: the one question asked stayed pending across many checks while Aristotle continued under the default it had stated in the question itself.

- Surface every question to the dev verbatim, with its question ID, and never answer one for him. Scope answers change what the run proves and what it costs, and they are his decisions. The pilot's scope question, whether external crate bodies should be verified or treated as stated contracts, went unanswered and the run proceeded on supplied code only; record that default as part of the run's boundary.

- Leave questions enabled unless the dev directs otherwise. Passing --no-questions at submission disables them, which trades away the chance to correct the run's scope mid-flight.

## Monitor, download, and report verdicts

- Monitor with aristotle show --json and read its property counts: proved, refuted, in progress, and unresolved, plus the per-property failure reason. A lightweight watch that polls status and new question IDs on an interval, deduplicating questions it has already reported, is the pattern the pilot used.

- Download the result archive with aristotle download and the project ID when the run ends. Record the per-property outcomes exactly as returned, and record the verification boundary verbatim when the run produces one.

- Treat an out-of-budget or out-of-credits stop as no verdicts at all. The pilot ended OUT_OF_BUDGET with all seven properties unresolved, zero proved, zero refuted, and no counterexamples, and that state is not a pass, not a partial pass, and not evidence about the program in either direction. Its download contained only the original project files and Lean scaffolding, because the run produced no summary, proof, or boundary statement.

- Treat a refutation as a finding. Record the counterexample verbatim and act on it; never weaken or reword a property to make a counterexample go away.

- Publish by the standing rule. When a run completes, publish its per-property outcomes exactly as stated on the project page and in the repository, with the claim ceiling of formally verified under stated assumptions. When a run fails, do not auto-update any public page; record the outcome in the project record and report it to the dev, whose call the public wording is.

- Keep a record file beside the spec, in the pilot README's shape: status, project and task IDs, timestamps, balance before and after, the bundle hash and file list, the secret-scan result, the dry-run line, every question with its ID and disposition, and the per-property outcomes. The next agent inherits the run from that file, not from chat.

## Follow the grant-day queue when funded work resumes

- Read READINESS.md in the private Solizardking/harmonic-grant repository before proposing any funded run. It carries the ordered queue (the Dark continuation first, by property value), the budget table and floor plan, and the first-hour environment checklist, and it supersedes any ordering improvised in chat.

- Surface the pilot's unanswered scope question before the first continuation run, since its answer may change the run's scoping. It remains the dev's to answer.

- Keep the two prover tracks separate in every plan and report. The Aristotle queue spends credits; the Certora queues in the certora-solana-fv skill spend none, and neither track's blockers unblock the other.

## Available connectors

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.

- If nothing here fits, look on the web for a maintained API or MCP server and set it up with [Custom Connectors and MCPs](/opt/muse/skills/custom-connector-setup/SKILL.md), or build a custom connector.

## Websites

- [Aristotle installation docs](https://aristotle.harmonic.fun/docs/installation) for the canonical setup page ("MCP & CLI"): per-client MCP setup, CLI install and login, upload exclusions, and billing behaviour. The docs root is https://aristotle.harmonic.fun/docs (overview, properties tracking, FAQ, the Rust verification page, and the SDK reference at /docs/api), and Harmonic's own agent-setup guide is https://aristotle.harmonic.fun/agent-setup.md.

- [Aristotle](https://aristotle.harmonic.fun/) for the web app: account creation, organization and API key management, question answering, and run pages. The pilot project lives at https://aristotle.harmonic.fun/projects/4d9bdb7f-7176-492d-9e31-219601fc0f30 as the worked example of a stopped run's record.

- [Harmonic](https://www.harmonic.fun/) for the vendor and the Aristotle product pages, including the docs the known-limits list came from: modelled syscalls, the single supported compiler, and run behaviour.

- [Manifest case study](https://www.harmonic.fun/aristotle/case-studies/manifest/) for the published Solana worked example: 28 proved results, 2 real bugs found and fixed, and a proved progress property with full error classification.

- [Dark Protocol Aristotle pilot record](~/workspace/zolana-transplant-merge/dark-protocol/aristotle/README.md) on disk for the complete pilot: the seven properties in full, the bundle listing and hash, the secret scan, the dry-run line, the question, and the OUT_OF_BUDGET outcome. The spec sits beside it as dark-protocol_spec.md, and the unsent pre-flight questions as HARMONIC-QUESTIONS.draft.md.

- [Certora formal verification skill](~/workspace/custom-skills/local-certora-solana-fv/SKILL.md) on disk for the other prover track: CVLR suites, the certoraRun CLI, and the hosted Certora engines. Survey and verdict discipline transfer between the tracks; tooling, billing, and blockers do not.
