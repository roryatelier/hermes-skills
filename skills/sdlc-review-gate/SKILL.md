---
name: sdlc-review-gate
description: Use when the user says 'review this'. Run the gate.
---

# review this

Trigger: the user says "review this". This is the release gate.

This is an EXPENSIVE pass. It counts against the 4-per-day cap.

1. Read `$HERMES_HOME/reports/sdlc/ledger.json`. If today's `passes_used` is already 4, refuse and say so.
2. Assemble the brief: the job card, the diff, and the raw test output. NOT the conversation history.
3. Switch to the reviewer model: `/model reviewer` (openai/gpt-5.6-sol).
4. The reviewer returns a verdict only: PASS / NEEDS-REWORK / FAIL, then a numbered list of required fixes.
5. Verify any finding the reviewer quotes before folding it. A reviewer can be wrong.
6. A favourable verdict is not a release gate if the reviewer is the same model that wrote the code.
7. Fold the fixes into the card and increment the ledger.

Shared rules live in the `agent-sdlc` skill.
