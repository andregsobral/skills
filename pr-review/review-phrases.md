# Review Phrases & Voice Guide

Comments should sound like the reviewer — direct, collegial, never condescending.
Use the phrases and examples below as stylistic anchors.

---

## Tone Principles

### Tone & Communication

- **Be respectful and constructive**: Assume good intent
- **Be specific**: Point to exact lines and explain the issue clearly
- **Suggest, don't demand**: prefer "I'd suggest" / "worth considering" for style; use "this will break" / "this is a bug" when it actually is
- **Provide context**: Explain WHY something is a problem
- **Offer solutions**: Suggest concrete fixes or alternatives
- **Ask questions**: Use "Have you considered...?" when uncertain
- **Praise good work**: Call out clever solutions or good practices 'Nice one', 'God job'
- **Reference documentation**: Link to style guides, best practices, or examples
- **Explain the why**: don't just flag — give the reason and (when useful) the fix

---

## Opener Phrases

Use these to start a comment naturally:

- "This will cause an N+1 query — worth adding `select_related(\"...\")` here."
- "Heads up: `datetime.utcnow()` is deprecated in 3.12, use `datetime.now(UTC)` instead."
- "Minor: `Optional[X]` can be written as `X | None` in Python 3.10+."
- "This could silently swallow exceptions — might be worth re-raising or at least logging with a traceback."
- "Just to flag: using a mutable default here (`[]`) is a classic gotcha — `None` as sentinel is safer."
- "This looks like an N+1 — every iteration will hit the database. Adding `prefetch_related` should sort it."
- "Worth wrapping this in `transaction.atomic()` — if the second write fails, the first won't be rolled back."
- "This bypasses the ORM's atomic update — if two requests race, one update will be lost. `F()` expressions avoid that."

---

## Closing Phrases

For longer comments, optionally close with:

- "Happy to chat through the tradeoffs if useful."
- "Let me know if you'd like a hand refactoring this."
- "Not a blocker, just something to keep in mind."
- "This one I'd fix before merging — it's a real bug in concurrent scenarios."

---

## Severity Markers (use internally, not in comments)

Tag each finding before presenting to the reviewer:

- `[BUG]` — incorrect behaviour, data loss risk, security issue
- `[PERFORMANCE]` — performance issue (N+1, missing index, unnecessary computation)
- `[STYLE]` — modernisation, readability (not a bug)
- `[SAFETY]` — could break under edge cases (race condition, unhandled exception)
- `[MINOR]` — small improvement, take or leave it

---

## USER PHRASES

<!-- 
  Add your own recurring phrases here so comments sound like you.
  Examples:
  
  - "Nit: ..."
  - "Could we consider: ..."
  - "Nice one!"
-->

