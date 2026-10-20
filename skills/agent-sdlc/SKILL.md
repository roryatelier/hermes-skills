---
name: agent-sdlc
description: Use when dispatching work or tracking spend. Route roles.
---

# Agent SDLC

One inbox. Three roles. The user does not switch apps to remember the job.

## Roles

| Role | Model | Allowed | Forbidden |
|---|---|---|---|
| Clerk | `deepseek/deepseek-v4.1-flash` via `delegation.model` | Read, measure, tests, apply a locked spec, update the board | Redesign, new architecture, a second plan |
| Architect | alias `architect` = `x-ai/grok-4.7` | One spec: done-when, out-of-scope, acceptance test | Implementation |
| Reviewer | alias `reviewer` = `openai/gpt-5.6-sol` | Spec + diff + test output. Verdict only | Implementation, a long chat paste |
| Rescue | reviewer model, explicit | One task the reviewer said the clerk cannot land | A new project |

`Sol 6.1` means `openai/gpt-5.6-sol`. Do not guess a different Codex model.

## Routing

- Implementation goes to `delegate_task`. Children are pinned to the clerk. Do not implement a multi-step build in the parent if the parent is architect or reviewer.
- Architect and reviewer are **not** children. Spawn them only with `hermes chat -q --oneshot -m <model> --provider openrouter`, and only with a short brief: job card + diff + test output. Never the whole conversation.
- Two identical failures: stop. Do not spend a third attempt on the same error.
- A favourable self-check is not a release gate. The reviewer is a different model.

## Spend cap

- 4 expensive passes per calendar day. Architect and reviewer each count as one. Clerk does not.
- Ledger: `$HERMES_HOME/sdlc/ledger.json`. Read it before an expensive pass. If `passes_used` for today is already 4, refuse the pass and say so. Do not silently spend.
- After a pass, increment the ledger in the same turn. Date key is UTC `YYYY-MM-DD`.
- Wednesday rule: if the week's passes exceed 16 by Wednesday, freeze new plans. Finish open cards only. Weekly cap is 20.

## Distribution

Canonical copy lives at `roryatelier/hermes-skills` (public). Install on any agent:

```
hermes skills tap add roryatelier/hermes-skills
hermes skills install roryatelier/hermes-skills/agent-sdlc
hermes skills install roryatelier/hermes-skills/sdlc-next
hermes skills install roryatelier/hermes-skills/sdlc-architect
hermes skills install roryatelier/hermes-skills/sdlc-review-gate
```

No credential is needed — the repo is public. Edit the local copy, then re-publish;
the repo is the distribution channel, the host is the working copy.

## Board

- **State root:** `$HERMES_HOME/sdlc/` — resolve `HERMES_HOME` per host. Holds `board.json` and `ledger.json`. Each agent keeps its own board; only the skills are shared.
- Human board: `$HERMES_HOME/reports/sdlc/board.html`. Machine state never lives under `reports/`.
- Machine card: `$HERMES_HOME/sdlc/board.json`
- Update both at the end of every non-trivial task. One card in `now`. Everything else is `next`, `blocked`, or `done`.
- A task is not done until the acceptance test in the card has been run and the result is on the card.

## Job card

Required fields: `project`, `now`, `done_when`, `out_of_scope`, `acceptance_test`, `role`, `blocked`.

If the user says "next", take the first unblocked card. Do not ask which project unless two cards are equally urgent.
