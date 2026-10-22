# sureforge (MiniMax Code port)

A quality-control workflow for complex, multi-step work: research → clarification → plan →
execute → deliver, with a READY / REPAIR / BLOCKED gate at every phase and independent review
before a delivery is called done.

Port of [Da7-Tech/SureForge](https://github.com/Da7-Tech/SureForge) (MIT) to MiniMax Code native
primitives. See [NOTICE.md](NOTICE.md) for the full list of changes and the upstream pin.

## Layout

```
SKILL.md                     the workflow, the five rules, the tiers, the four phases, the gate rules
references/research.md       phase 1 - understand, research, three perspectives, ask, alternatives
references/plan.md           phase 2 - criterion mapping, coverage denominator, budget, stop conditions
references/execute.md        phase 3 - controlled change path, discovery handling, finding triage
references/deliver.md        phase 4 - freeze the candidate, inspect scope, exercise the recipient path
references/review-protocol.md  isolation, method selection, evidence records, rounds, gate decisions
references/verification-catalog.md  how to choose a verification method
references/platforms.md      what this host actually offers, and its hazards
assets/task-ledger.md        durable state template (contract, decisions, evidence, findings, gates)
assets/coverage-ledger.csv   machine-readable coverage rows
assets/reviewer-brief.md     neutral brief for the independent reviewer
assets/critic-brief.md       neutral brief for the critic
LICENSE                      upstream MIT text, verbatim
NOTICE.md                    attribution and the list of port changes
```

## Install

Copy the **complete directory** into the global skills directory — copying only `SKILL.md` is
incomplete:

```powershell
Copy-Item -Recurse <source>\sureforge "$env:USERPROFILE\.minimax\skills\sureforge"
```

The skill then appears in the catalog under the bare name `sureforge`. MiniMax Code rescans local
plugins automatically, but a **new session** is needed for a newly installed skill to appear in
that session's catalog.

## Use

Load the skill with the `skill` tool (`sureforge`), or read `SKILL.md` directly. The skill
supplies the procedure; the team's operating parameters (concurrency cap, handover format, the
security-swarm gate) live in the bot instructions and in team memory, and they override the
skill's defaults.

## Verify

Discovery proves the package was found, not that it changed any output. Confirm the skill loads in
a fresh session, and confirm a real task follows the tier and gate rules.
