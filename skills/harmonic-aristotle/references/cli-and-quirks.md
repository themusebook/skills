# Aristotle CLI commands and this VM's quirks

Field record from the Dark Protocol pilot of 2026-11-18, run with aristotlelib 3.4.0. Every command below was executed in that form unless marked as documented-only. Read the onboarding guide for the walkthrough; this file is the command sheet.

## Install

- Canonical source: Harmonic's installation docs, https://aristotle.harmonic.fun/docs/installation (the "MCP & CLI" page), with Harmonic's agent-setup guide at https://aristotle.harmonic.fun/agent-setup.md. The route below is our pinned, hardened variant of what those pages document; install facts confirmed against them on 2026-11-25.

- Create the venv under the home directory and pin the version:

  `python3 -m venv ~/workspace/aristotle-cli/venv`

  `~/workspace/aristotle-cli/venv/bin/pip install aristotlelib==3.4.0`

- Verify with `~/workspace/aristotle-cli/venv/bin/aristotle --version`, which prints `aristotlelib 3.4.0`.

- Harmonic's documented primary install is `uv tool install "aristotlelib>=3"` (global, no venv; the installation page's one-liner also installs uv if missing and logs you in, and re-running it upgrades). The venv install above is the docs' documented alternative — "also how you get the library" — with the specifier `pip install "aristotlelib>=3"`. Prefer our pinned venv anyway: the pilot's calibration is tied to 3.4.0 (still the current PyPI release as of 2026-11-25), and a newer library is a deliberate upgrade, not an accident of timing. The docs pin no patch version; their contract is the 3.x line.

- Never recreate the pilot's original location. The first venv lived at /aristotle-scratch/venv on the root overlay and was lost in the VM replacement of 2026-11-24. Older records, including READINESS.md in the harmonic-grant repository, name the scratch path; the home path in this file supersedes it, because the home directory survives replacement and the overlay does not.

## The NO_PROXY rule (every command, no exceptions)

- Prefix every invocation:

  `env NO_PROXY=localhost,127.0.0.1 no_proxy=localhost,127.0.0.1 ~/workspace/aristotle-cli/venv/bin/aristotle <command>`

- The cause is local, not Harmonic's: this VM's NO_PROXY value contains an IPv6 range that aristotlelib's HTTP client cannot parse, so commands fail against an otherwise healthy service. Sanitize the pair rather than debugging the endpoint.

## Login and credentials

- Log in with stdin closed:

  `env NO_PROXY=localhost,127.0.0.1 no_proxy=localhost,127.0.0.1 ~/workspace/aristotle-cli/venv/bin/aristotle login < /dev/null`

- The `< /dev/null` is load-bearing: with stdin at EOF the CLI takes the browser device-code path, printing an approval-page address and a code. Without it the CLI can wait on an interactive path that never completes in an agent shell.

- A human approves the code in a browser. On first login that human also creates the Aristotle account and chooses the organization the machine key belongs to; both are the dev's acts, never the agent's.

- The approval code expires after 10 minutes (docs, confirmed 2026-11-25). For agent-driven flows the docs give `aristotle login --json`, which emits {"verification_uri", "user_code", "expires_in"} for the human to approve in a browser; the key goes straight to the CLI and never passes through the conversation. `aristotle login --no-browser` is the variant when no browser can open locally.

- For CI/scripts the docs name an environment-variable route: `ARISTOTLE_API_KEY` overrides a stored key, and a key can be piped in (`printf '%s\n' "$KEY" | aristotle login`) or passed to a single command. We do not use these routes operationally — the stored device-code key is our route — but know they exist before declaring a login impossible.

- The CLI stores a machine-named API key in ~/.config/aristotle/credentials.json (0600). The pilot key was named "aristotle CLI on htch-runtime" and billed to the dev's personal organization. Refer to the file by name and location only; never open, print, copy, or commit it. Revocation lives in the web UI under API keys.

- The credentials file survived the 2026-11-24 VM replacement because it sits under the home directory. After reinstalling the CLI, expect the stored key to be picked up; if the CLI reports logged-out, the device-code login above is the recovery, and it needs the dev.

## Balance

- `aristotle balance` prints the credit position with Available, Free, Purchased, and Granted lines. The pilot's post-run read was: Available 0 Credits (Free 0, Purchased 0), with Granted (active grants) listing 500 Credits — the signup grant fully consumed, its expiry shown as 2026-12-18.

- Run balance before and after every submission and record both numbers in the project record. A balance read is free; a surprise drain is not.

## Dry run and submit

- Dry run (sends nothing):

  `aristotle submit "Verify the properties stated in dark-protocol_spec.md" --project-dir <staging-folder> --dry-run`

  The pilot's dry run printed: "Dry run: would upload 10 files (49.2 KB)." Require the count and size to match the staged file list exactly.

- Submit (the disclosure and the spend; dev approval for that project first):

  `aristotle submit "<property prompt>. The spec file is at the project root; its stated contracts are part of the properties." --project-dir <staging-folder> --name <project-name>`

- Add `--wait` to follow the run in the terminal. Add `--no-questions` only on the dev's direction; it disables clarifying questions for the run.

- Compressed uploads are capped at 100 MB. From a git checkout the CLI excludes credentials (.env, keys, .ssh), .git, and gitignored files, but the hand-staged bundle plus dry run is the verified route.

## Questions

- `aristotle questions` lists a run's clarifying questions; they also appear in the web app. Harmonic's docs give a roughly 15-minute answer window, but the pilot's question stayed pending and answerable-in-principle across roughly two hours of checks while the run continued under its stated default. Treat the window as soft, and the default stated inside the question as what an unanswered question costs.

- The pilot question, verbatim in the project record, was ID 901ef2c1-5612-59bf-b5e8-ae9cf1fdde0e, asked 2026-11-18T16:56:17Z: whether external crate bodies (ark-ff, ark-bn254, groth16-solana helpers, anchor-lang paths) should also be verified or treated as stated contracts, with "supplied code only" as the running default. It was never answered and the run ended before any answer could reach Aristotle.

- Surface questions verbatim with their IDs. Never answer one. Deduplicate by ID when watching, recording reported IDs so the same question is never re-reported as new.

## Status, show, download, continue, cancel

- `aristotle show --json` returns the run's property counts (proved, refuted, in_progress, unresolved) and per-property failure reasons. The pilot's terminal read was proved 0, refuted 0, in_progress 0, unresolved 7, every property FAILED with failure_reason OUT_OF_BUDGET.

- Terminal credit stops carry the CLI's own wording: "This run stopped because your organization does not have enough Credits left. Buy credits, then send a follow-up." and "Task stopped: your organization ran out of credits." Quote them in the record rather than paraphrasing a stop into a verdict.

- `aristotle download <project-id>` saves `<project-id>-aristotle.tar.gz`. The pilot's archive was 56,017 bytes and contained only the original project files plus SepLogic Lean scaffolding (SepLogic/Public.lean, Pure.lean, PublicGeneric.lean) — no summary, proof, or boundary report, because the run produced none. A thin download after a stopped run is expected, not corruption.

- `aristotle continue <project-id>` starts a NEW billed run on the project. It is the resume path after credits are bought and the trap this whole file exists to prevent firing casually: never run it speculatively, never to "check" anything, and only on the dev's explicit go with credits confirmed by a balance read first.

- A queued task can be cancelled through the CLI on the dev's order. The orderbook pilot's task was cancelled while QUEUED (never started); its project and uploaded files remained intact, and restarting it would be a continue — a new billed run.

- More of the current command surface (docs/PyPI command table, confirmed 2026-11-25; untested here beyond the pilot's set): `aristotle properties <project>` and `aristotle property <project-id> <name>` read the Properties table from the terminal (terminal labels read REFUTED with an EVIDENCE column; `(unchecked)` marks results from early-ended runs); `aristotle tasks` lists a project's tasks; `aristotle list` lists projects; `rename`, `archive`/`unarchive`, and `delete <project>` manage projects (delete is permanent; a project with a running task can't be archived or deleted). Scripting contract: exit 5 means no key or a rejected key (`aristotle list --limit 1 --json` is the docs' access check), exit 10 means a `--non-interactive` wait stopped because questions are pending, exit 6 covers rate-limiting and insufficient credits to start a run.

## Watch pattern

- The pilot used a roughly 10-minute polling watch over status and questions, reporting terminal transitions and each new question ID once. Remove or disable the watch when the run reaches a terminal state; the Dark watch removed itself at the run's end, and the orderbook watch was disabled, not removed, when its run was paused.

## MCP surface (documented, untested)

- Harmonic runs a hosted MCP server at https://aristotle.harmonic.fun/mcp with OAuth sign-in against the same account — no key to copy. Per-client setup (Claude Code, Claude Desktop, claude.ai, ChatGPT, Codex, Cursor, VS Code) is on the installation page: https://aristotle.harmonic.fun/docs/installation. Documented tools: get_credit_balance, create_upload, start_run, get_run, wait_for_run, answer_question, send_message, list_properties, get_property, continue_run, list_projects, list_runs, create_download, cancel_run, rename_project, delete_project, send_feedback. MCP billing behaviour per the docs: the first run asks which organization pays and that confirmation holds for 90 days (prompt-only new projects ask each time; every run asks while org credits are low).

- Nobody here has run it. It carries the same uploaded-code disclosure as the CLI, so it inherits the same per-project approval gate, and the same rule against answering questions for the dev — the MCP `answer_question` tool changes nothing about whose decision a scope answer is.
