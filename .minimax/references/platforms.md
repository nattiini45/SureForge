# Platform notes

Local host schema and tool behavior rechecked: **2026-10-22**. These notes are capability guidance,
not a claim that SureForge has been behaviorally tested on every host. Tool names, context
inheritance, discovery paths, and permission controls can change. **Inspect the current host schema
before use** — `mavis({ command: "agent list" })`, `mavis({ command: "mcp list" })`, and the tool schemas in the session.

## Portable core

SureForge uses standard `name`, `description`, `license`, and string-valued `metadata` frontmatter.
It does not set experimental `allowed-tools`, a model, automatic subagent execution, hooks, or
platform-specific permissions. The installed package contains instructions and templates, not a
runtime gatekeeper.

The distribution source is this skill directory. An installation directory is a separate concern.
Copy the complete skill directory, including its license and referenced resources; copying only
SKILL.md is incomplete. Do not edit global configuration or overwrite an existing skill to conduct
a smoke test.

## Capability matrix — MiniMax Code

| Host | Documented capability and cautions | SureForge adaptation | Behavioral status |
| --- | --- | --- | --- |
| MiniMax Code Desktop (this runtime) | Skills load from `~/.minimax/skills/<name>/SKILL.md` (global) and from an installed plugin's `skills/<name>/SKILL.md` (catalog name `studio-skills:<name>`). Tools: `read`/`write`/`edit`/`grep`/`glob`, `bash`, `task`, `skill`, `mavis`, `ask_user`. | Install the complete directory globally. Refer to the skill by its catalog name in instructions. Use `task` children for the independent reviewer. | Require run-specific behavioral evidence; discovery is a narrower check than behavior. |
| MiniMax Code — Windows shell | `bash` runs **PowerShell 5.1** (`C:\WINDOWS\System32\WindowsPowerShell\v1.0\powershell.exe`). No `&&`, no `||`, no ternary operator. `Set-Content -Encoding UTF8` writes a BOM. | Use separate statements and `$LASTEXITCODE`. Write parseable files with `[System.IO.File]::WriteAllText`. Read files with `read`, not `Get-Content`, when the bytes matter. | Verified on this host on 2026-10-22. |
| MiniMax Code — `task` delegation | A child agent receives **no parent conversation**. Roles: `mavis`, `explore` (read-only), `worker`, `verifier` (reviews a deliverable; no project-file changes). The runtime enforces a per-session child concurrency limit and answers `SUBAGENT_CONCURRENCY_LIMIT`. A child is not a sandbox: it still has shell access. | Use `agent_name: "verifier"` for the independent reviewer and `"worker"` for bounded implementation. Wait for a slot and retry instead of fanning out. Disclose the role contract as a contract, not an isolation boundary. | Verified on this host on 2026-10-22. |
| MiniMax Code — `ask_user` | Questionnaire mode, **1–4 steps**, each with **2–4 options**; a recommended option can be marked; free text is always available. Ordinary questionnaires wait for an explicit reply. In an **active Goal**, a timeout can auto-adopt the recommended option unless `requiresExplicitResponse: true` is set. | Batch independent questions, sequence dependent ones, mark a recommendation. Treat a skipped, declined, or timed-out answer as `unanswered`, never as approval — and set `requiresExplicitResponse: true` for any decision that must not be auto-adopted. | Verified on this host on 2026-10-22 (tool schema). |
| MiniMax Code — `mavis` | A **native tool, not a program** — there is no `mavis` CLI on this host, so do not shell out to it. `mavis({ command: "session delete", args: { session_id: "<sessionId>" } })` removes a session; `mavis({ command: "session update", args: { session_id, archived: true } })` archives it. | Use `mavis` to inspect the roster before dispatching, and to replace a bot's session. | Verified on this host on 2026-10-22 (tool schema + the built-in `mavis` skill reference). |
| MiniMax Bot plugin (team bots) | Bots exchange work through `message_bot` / `dispatch_tasks` / `post_group`; results return to the source chat; `get_tasks` polls a batch. The plugin writes a question card only when the Host reports `pendingQuestion`, so a bot turn that ends on `ask_user` can leave the user with no way to answer it. | Ask from a session where the card is confirmed to render; answer bots through the card's option buttons. If a bot stops accepting turns, replace its session with the `mavis` tool. | Partly verified on this host on 2026-10-22 — see the note below for what was observed and what was not isolated. |
| GitHub | `gh` CLI, authenticated as the repository owner; `gh api` for remote blob and tree verification. | Read-only for research; use a branch + pull request for changes. Never push to the default branch. | Verified on this host on 2026-10-22. |
| Gitea (`git.pawcommit.com`) | MCP tools for repos, branches, contents, issues and pull requests. | Read-only for research; use `create_branch` + `create_or_update_file` + `pull_request_write` for changes. Never push to the default branch. | Verified on this host on 2026-10-22. |

File installation and discovery can be tested without exercising model behavior. Keep those results
separate from this column. No platform receives a behavioral compatibility badge based only on
documentation or copied files.

## Questions

Use the actual tool exposed by the host, not a guessed function call. On this runtime that is
`ask_user`: 1–4 steps, 2–4 options each. A skipped question remains unanswered in every interface.

**Bot-session hazard — observed 2026-10-22; the trigger was not isolated.**

The plugin writes the question card only when the Host reports `pendingQuestion`
(`engine.mjs`). If a bot turn ends on `ask_user` and no card reaches the chat, the user has no
way to answer, and the bot can stop accepting turns: one observed run produced 13 consecutive
`AGENT_UNAVAILABLE` failures with no successful turn in between, until the
session was replaced.

What is **not** established: the missing card was not proven to be the cause. The plugin treats
`AGENT_UNAVAILABLE` as transient ("The Agent is temporarily unavailable. Try again shortly"); the
8 s + 20 s retry budget is a **turn-retry backoff**, not an admission timeout; and two other
`ask_user`-without-card turns were followed by successful turns in the same session. Treat the
causal chain as a hypothesis and read the message store before concluding.

When a bot stops accepting turns:

1. Read the bot's transcript for a message with a `question` property, and check the runtime's
   `questionnaire_requests` rows for a `pending` entry.
2. Replace the session: `mavis({ command: "session delete", args: { session_id: "<sessionId>" } })`.
   The plugin's own `SESSION_NOT_FOUND` self-heal then clears the session binding and starts a
   fresh session with the bot's standing brief re-injected. (Archiving was tried first in the
   observed run and did not restore the session; that is one observation, not a proof that
   archiving never works.)
3. If the bot's questions must be answerable, ask from a session where the card is confirmed to
   render, and answer through the card's option buttons.

There is **no `mavis` CLI** on this host — it is a native tool. Do not shell out to it.

## Review isolation checklist

Before counting an independent reviewer, establish:

1. Whether conversation history, summaries, project rules, memories, skills, or author reports are
   inherited. A `task` child inherits none of the parent's transcript, but it does inherit the
   host's project instructions and the skill it is told to load.
2. Whether the reviewed snapshot is stable and accessible with its latest contract and necessary
   materials.
3. Whether write, network/provider, and delegation permissions fit the approved scope.
4. Whether the reviewer can report actual methods and coverage, not merely a verdict.
5. Whether the provider/data destination and resource use are authorized.

If any property is unknown, disclose it. Do not launch a second CLI to bypass a host restriction on
subagents. A handoff to a separately authorized human or fresh session is the fallback for required
independent review.

## Installation and discovery on this host

- **Global skill:** copy the complete directory to `~/.minimax/skills/<name>/` (on Windows:
  `$env:USERPROFILE\.minimax\skills\<name>\`). The skill then appears in the catalog under its
  bare name (`sureforge`).
- **Plugin skill:** copy the directory into the plugin's `skills\<area>-<name>\` folder, list it in
  `.minimax-plugin\plugin.json`, and bump the manifest version. The skill then appears as
  `studio-skills:<name>`. The studio plugin's `tools\install-local.ps1` mirrors the package into
  the MiniMax data directory; `tools/` itself is not copied.
- MiniMax Code rescans local plugins automatically. A new skill still needs a **new session** to be
  visible to that session's catalog.

Verify discovery separately from behavior: the catalog entry proves the package was found, not
that the skill changed any output.

## Sources

- [Agent Skills specification](https://agentskills.io/specification)
- [Da7-Tech/SureForge](https://github.com/Da7-Tech/SureForge) — the upstream this port is derived
  from, at commit `dadf5110621739143063da5e4cbb33790d48c95d`.

The upstream's own platform notes covered Devin CLI, Claude Code, Codex, Cursor, and Hermes. This
file replaces that matrix for MiniMax Code; the upstream matrix remains valid for its own hosts.
