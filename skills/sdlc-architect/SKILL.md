---
name: sdlc-architect
description: Use when the user says 'architect'. One spec, then stop.
---

# architect

Trigger: the user says "architect" or "architect this".

This is an EXPENSIVE pass. It counts against the 4-per-day cap.

1. Read `$HERMES_HOME/sdlc/ledger.json`. If today's `passes_used` is already 4, refuse and say so. Do not spend.
2. Switch to the architect model: `/model architect` (x-ai/grok-4.7).
3. Produce exactly one document: `done_when`, `out_of_scope`, `acceptance_test` (a runnable command and its expected output), files in play, risks.
4. Do not write implementation code. Do not produce a second plan.
5. Increment today's pass count in the ledger.
6. Fold the spec into the card in `board.json`.

Shared rules live in the `agent-sdlc` skill.
