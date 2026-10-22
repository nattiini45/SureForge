# NOTICE — sureforge (MiniMax Code port)

This skill is a **port** of [Da7-Tech/SureForge](https://github.com/Da7-Tech/SureForge).

- Upstream: `Da7-Tech/SureForge`
- Pinned commit: `dadf5110621739143063da5e4cbb33790d48c95d`
- Upstream path: `skills/sureforge/`
- Upstream version: `1.0.0`
- Upstream license: MIT — see `LICENSE` in this directory (reproduced verbatim)
- Port: PawCommit Studio
- Port version: `1.0.0-mmx.1`
- Port date: 2026-10-22

Upstream copyright notice, unchanged:

> MIT License — Copyright (c) 2026 Da7-Tech

The upstream `skills/sureforge/LICENSE` is reproduced verbatim in this directory.

## What was changed

The upstream workflow, its five rules, its three tiers, its four phases, its READY / REPAIR /
BLOCKED gate semantics, the three-review-rounds-per-gate limit, the coverage-denominator
requirement, and the separation of production from review are **unchanged**. The port changes how
the workflow is expressed for the MiniMax Code runtime and for the PawCommit Studio team.

### `SKILL.md`

- Added a "Host primitives used here" table mapping each abstract host capability to its concrete
  MiniMax Code tool (`ask_user`, `task` with `agent_name`, `read`/`write`/`edit`/`grep`/`glob`,
  `bash`, `mavis`, `skill`, and the MiniMax Bot plugin tools).
- Added a "Studio team operating parameters" section: concurrency cap 2, one writer per file set,
  artifact-first handover with a ≤10-line chat relay, branch + pull request for repository changes,
  and the security-swarm base-2 gate. These parameters are authoritative for the studio team and
  override this document's defaults.
- The question-tool paragraph now names `ask_user` and its real limits, and carries the
  MiniMax Bot session-wedge hazard plus its repair (`mavis session delete`).
- The Plan phase now notes the concurrency cap; the Deliver phase points at the result artifact.
- Frontmatter `metadata` gained `upstream` and `port`; `version` became `1.0.0-mmx.1`.

### `references/research.md`

- The question tool is named (`ask_user`, 1–4 steps, 2–4 options each), with the recommendation
  and free-text rules.
- Added the MiniMax Bot session-wedge hazard, the "answer through the card's option buttons" rule,
  and the `mavis session delete` repair.
- The research-tool paragraph now names `grep`, `glob`, and `web_search`.

### `references/plan.md`

- Added a repository-changes row to the coverage denominator (changed files, branch and pull
  request, CI on the exact head commit, merge-base diff — not the local working tree).
- Parallel work now explicitly references the concurrency cap.

### `references/execute.md`

- Added a "Running checks on this host" subsection: PowerShell 5.1 (no `&&`, no `||`, no ternary,
  `$LASTEXITCODE`), `[System.IO.File]::WriteAllText` instead of `Set-Content -Encoding UTF8`
  (which writes a BOM), reading with `read` rather than `Get-Content`, and "evidence is the pasted
  output and exit code, not the fact that the command ran".

### `references/deliver.md`

- Added "Hand over as an artifact, not as prose": the result file carries the verdict, commit SHA,
  changed paths, the acceptance-check command with its exit code and the pasted real output, and an
  explicit "Unverified" section; the chat handover is a pointer to it.
- The candidate for a remote repository is the head commit of the branch, verified at the remote
  SHA; the local working tree is not the candidate.

### `references/review-protocol.md`

- Added "How isolation is obtained on this host": a `task` child with `agent_name: "verifier"`
  receives no parent transcript; the role contract forbids file changes and further agents but is
  not a sandbox; `task_append` is not a steering channel; the owner's session must not be handed
  over; there is no per-child network kill-switch.
- The evidence table and every gate rule are otherwise unchanged.

### `references/verification-catalog.md`

- Added a "Running checks on this host" subsection: PowerShell exit-code discipline, enumerating
  scope with `grep`/`glob` instead of assuming counts, and verifying the remote blob at the exact
  SHA rather than the local working tree.

### `references/platforms.md`

- **Rewritten.** The upstream matrix (Devin CLI, Claude Code, Codex, Cursor, Hermes) is replaced by
  a MiniMax Code matrix: skill discovery paths, the Windows PowerShell 5.1 shell, `task` delegation
  and the concurrency limit, the `ask_user` schema, the `mavis` tool, the MiniMax Bot plugin, and
  GitHub (`gh`) / Gitea (MCP) repository access. The upstream matrix remains valid for its own
  hosts.
- The bot-session wedge hazard is documented with its cause, its discriminator, and its repair.

### `assets/task-ledger.md`

- Added a "Repository state" block: remote, branch, head SHA, pull request, checks on the head
  commit, and the base-2 security-swarm scan with its findings and their disposition.
- The acceptance-mapping row now records the owned paths of each task in flight.

### `assets/reviewer-brief.md`

- Added a "Context you should expect" section describing the `task`-child dispatch, the absence of
  the author's transcript, and the honest statement that the role contract is not a sandbox.

### `assets/critic-brief.md`

- Unchanged apart from surrounding attribution.

### Unchanged, byte-for-byte

- `LICENSE` (upstream MIT text, verbatim)
- `assets/coverage-ledger.csv` (the header row is unchanged)
