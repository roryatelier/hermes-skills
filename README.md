# hermes-skills

Shared Hermes skills for Rory's agents. One inbox, three roles, a visible spend cap.

## Install (per agent)

No credentials needed — this repo is public.

```bash
hermes skills tap add roryatelier/hermes-skills
hermes skills search sdlc --source github
hermes skills install roryatelier/hermes-skills/sdlc-next
hermes skills install roryatelier/hermes-skills/sdlc-architect
hermes skills install roryatelier/hermes-skills/sdlc-review-gate
hermes skills install roryatelier/hermes-skills/agent-sdlc
```

The tap points at the `skills/` directory of this repo. Each skill is a self-contained
directory with a `SKILL.md`.

## Skills

| Skill | Trigger | Purpose |
|---|---|---|
| `agent-sdlc` | any dispatch or spend question | Roles, routing rules, spend cap, board format |
| `sdlc-next` | "next" | Take the top unblocked board card and dispatch it |
| `sdlc-architect` | "architect" | One spec, then stop. Counts as an expensive pass |
| `sdlc-review-gate` | "review this" | Reviewer reads card + diff + tests. Verdict only |

`agent-sdlc` is the source of truth. The three command skills are thin wrappers that
defer to it — do not restate the rules in them.

## Per-agent setup

The skills read machine state at `$HERMES_HOME/sdlc/` and write the human-readable
board to `$HERMES_HOME/reports/sdlc/`.

Each agent keeps **its own** board and ledger. Only the skills are shared. Create the
directories and the two state files before first use:

```bash
mkdir -p "$HERMES_HOME/sdlc" "$HERMES_HOME/reports/sdlc"
printf '{"updated":"","now":null,"next":[],"blocked":[],"done":[]}' > "$HERMES_HOME/sdlc/board.json"
printf '{"expensive_passes_per_day":4,"expensive_passes_per_week":20,"models":{},"days":{}}' > "$HERMES_HOME/sdlc/ledger.json"
```

Never put `board.json` or `ledger.json` under `reports/` — that directory is for
human-facing artifacts only.

## Configure the models

The skills name roles, not model IDs, wherever possible. Pin them per agent:

```bash
hermes config set model.aliases.clerk     openrouter/deepseek/deepseek-v4.1-flash
hermes config set model.aliases.architect  openrouter/x-ai/grok-4.7
hermes config set model.aliases.reviewer   openrouter/openai/gpt-5.6-sol
hermes config set delegation.provider      openrouter
hermes config set delegation.model         deepseek/deepseek-v4.1-flash
```

`delegation.model` pins **all** child agents to the cheap worker regardless of which
model the parent session is using. That is the main cost control.

Verify: `hermes config get delegation.model`

## Rules that matter

- Expensive passes (architect, reviewer) are capped at **4 per day, 20 per week**.
  Clerk work is free. The cap is enforced in the skill, not by a human remembering.
- The reviewer must be a **different model** from the one that wrote the code.
- The reviewer sees the **job card, the diff, and the test output** — never the
  conversation history.
- A task is not done until its `acceptance_test` has actually been run.

## Personal data

These skills contain no project data, credentials, or board contents. Boards and
ledgers stay on each host and are never committed here.
