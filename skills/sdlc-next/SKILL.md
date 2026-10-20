---
name: sdlc-next
description: Use when the user says 'next'. Work the top board card.
---

# next

Trigger: the user says "next".

1. Read `$HERMES_HOME/sdlc/board.json`.
2. Take the first card in `next` with no blocker. If every card is blocked, name the blocker on each and stop. Do not invent work.
3. Move that card to `now` and write the board back before starting.
4. Dispatch the work to the clerk (`delegate_task`). Do not implement it in the parent session.
5. Done means the card's `acceptance_test` was run and the result is written on the card. Then move it to `done` and refresh `board.html`.

Shared rules, roles, and the spend cap live in the `agent-sdlc` skill. Load it if unsure.
