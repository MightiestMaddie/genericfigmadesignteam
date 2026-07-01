# CANDIDATES.md — Candidate Register

**Librarian-owned.** This file is file-global state: only the Librarian writes it
(see [CLAUDE.md](CLAUDE.md), single-writer invariant). Page agents never edit this
file — they raise a `candidate-review` item on the shared task list (and report it to
the human) instead, and the Librarian records it here as a new row with status
`pending-review`.

One row per candidate. **No candidates yet** — this table fills in once page agents start
building screens and hit primitives the library doesn't have. On a freshly bootstrapped
project it is expected to stay empty until the first screens are built.

| ID | Name | Raised by (agent · page) | Gap | Node | Built from | New values | Status | Decided |
|----|------|--------------------------|-----|------|------------|------------|--------|---------|
| *(none yet)* | | | | | | | | |

## Status legend

| Status | Meaning | Who moves it | Loop behavior |
|--------|---------|--------------|---------------|
| `pending-review` | Awaiting the human's decision. **Terminal for agents.** | Only a human moves it on. | **Ignored** — not a violation, never keeps the loop running. |
| `promote` | Human approved; Librarian has not built it yet. | Set by a human; executed by the Librarian. | **Loop-actionable** — build it on the library page, document in `SKILL.md`, then set `promoted`. |
| `promoted` | Built on the library page and documented in `SKILL.md`. **Done.** | Librarian, after building. | Stable resting state. |
| `one-off` | Human rejected for the library. **Tombstone — permanently ignored.** | Set by a human. | **Never re-flagged.** Page-local element stays as-is. |

Only a human moves a row out of `pending-review`. Agents never self-promote.
