# Platform notes

Local host schema and tool behavior rechecked: **2026-10-22**. These notes are capability guidance,
not a claim that SureForge has been behaviorally tested on every host. Tool names, context
inheritance, discovery paths, and permission controls can change. **Inspect the current host schema
before use** — `mavis agent list`, `mavis mcp list`, and the tool schemas in the session.

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
| MiniMax Code — `ask_user` | Questionnaire mode, **1–4 steps**, each with **2–4 options**; a recommended option can be marked; free text is always available. Ordinary questionnaires wait for an explicit reply. | Batch independent questions, sequence dependent ones, mark a recommendation. Treat a skipped, declined, or timed-out answer as `unanswered`, never as approval. | Verified on this host on 2026-10-22. |
| MiniMax Code — `mavis` | Manages agents, sessions, MCP servers. `session delete` removes a session; `session update --archived` does not stop the runtime from resolving it. | Use `mavis` to inspect the roster before dispatching, and to repair a wedged bot session. | Verified on this host on 2026-10-22. |
| MiniMax Bot plugin (team bots) | Bots exchange work through `message_bot` / `dispatch_tasks` / `post_group`; results return to the source chat; `get_tasks` polls a batch. A turn that ends on `ask_user` can wedge the bot's session permanently if the question card is not written into the chat. | Ask from a session where the card is confirmed to render; answer bots through the card's option buttons. Repair a wedged session with `mavis session delete <session id>`. | Verified on this host on 2026-10-22 (see the wedge case below). |
| GitHub | `gh` CLI, authenticated as the repository owner; `gh api` for remote blob and tree verification. | Read-only for research; use a branch + pull request for changes. Never push to the default branch. | Verified on this host on 2026-10-22. |
| Gitea (`git.pawcommit.com`) | MCP tools for repos, branches, contents, issues and pull requests. | Read-only for research; use `create_branch` + `create_or_update_file` + `pull_request_write` for changes. Never push to the default branch. | Verified on this host on 2026-10-22. |

File installation and discovery can be tested without exercising model behavior. Keep those results
separate from this column. No platform receives a behavioral compatibility badge based only on
documentation or copied files.

## Questions

Use the actual tool exposed by the host, not a guessed function call. On this runtime that is
`ask_user`: 1–4 steps, 2–4 options each. A skipped question remains unanswered in every interface.

**Bot-session hazard.** In the MiniMax Bot plugin a bot turn that ends in `ask_user` can leave a
`questionnaire_requests` row pending **with no question card in the transcript**. That session is
then dead: every later turn fails with `AGENT_UNAVAILABLE` after a fixed ~28 s admission timeout,
and messages queued from other bots pile up undelivered. The discriminator is the card, not the
pending row — a pending row whose card renders is answerable. Repair: `mavis session delete
<session id>`. Archiving is **not** enough: the runtime still resolves an archived session. The
plugin's own `SESSION_NOT_FOUND` self-heal then rebinds the bot to a fresh session and re-injects
its standing brief.

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

- **Global skill:** copy the complete directory to `C:\Users\Caspar\.minimax\skills\<name>\`. The
  skill then appears in the catalog under its bare name (`sureforge`).
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
