---
description: Reconcile the component library against SKILL.md and CANDIDATES.md — Librarian only.
---

# /reconcile — design-system reconciliation pass

**Run this only in the Librarian session.** Page agents must never run it (see
[CLAUDE.md](../../CLAUDE.md), "Reconciliation"). This pass does **documentation and sync only** —
it never promotes a candidate (that's a human decision) and never waits on a human-gated state.

The **library page and file key are in [`PROJECT.md`](../../PROJECT.md)** — read them from there,
don't hard-code. This command is **idempotent**: running it repeatedly with no new work is a no-op
that reports "goal met" and stops. It must not loop or churn on human-gated rows.

## Goal — stop when ALL FIVE are true

1. **No candidate is in status `promote`** — every approved candidate has been built and moved to
   `promoted`.
2. **Every component on the library page has a matching `SKILL.md` inventory entry.**
   **Documented-by-design exclusions — never violations, never itemized:**
   - a bulk icon library counts as documented by its single family line in `SKILL.md`;
   - **Sandbox / WIP** is intentionally not itemized;
   - assembled example patterns are recipes, not components to itemize.
3. **Every `SKILL.md` inventory entry maps to a real component on the library page** (no orphan
   docs).
4. **`pending-review` and `one-off` rows are ignored** — they are *not* violations and do *not*
   keep the loop running.
5. **No unregistered candidates:** every file in `candidates/inbox/` and every frame named
   `CANDIDATE — *` anywhere in the Figma file has a matching `CANDIDATES.md` row. Missing ones
   are **recorded as `pending-review`** — recording is not deciding; the status decision stays
   with the human.

If all five already hold, report "✅ goal met — nothing to do" and exit without edits.

## Steps

1. **Read the records.** Load `CANDIDATES.md`, `PROJECT.md`, and
   `.claude/skills/figma-design-system/SKILL.md`.
2. **Read the live library.** Get the current component inventory of the library page (from
   `PROJECT.md`) in the project file (read-only — `get_metadata` on the library page). Do not edit
   components you are only inspecting.
3. **Act on `promote` rows only** (condition 1). For each `CANDIDATES.md` row with status exactly
   `promote`:
   - Build the real component in the correct Section of the library page, following the skill's
     naming / variant / variable-binding rules.
   - Add its entry to the `SKILL.md` inventory.
   - Set the register row to `promoted` and fill the `Decided` column.
   - Coordinate with the owning page agent to swap their local candidate for instances (message
     them first; do not silently edit their page).
   - **Skip** any row whose status is `pending-review` or `one-off`. Never change a
     `pending-review` row's status (only a human may). **Never re-flag a `one-off`.**
4. **Fix documentation drift** (conditions 2 & 3, honoring the exclusions in condition 2):
   - For every component on the library page with no `SKILL.md` entry → add the missing entry.
   - For every `SKILL.md` entry with no live component → flag it as an orphan and remove/correct the
     stale entry. (Also refresh any Section node ids in `PROJECT.md` that have drifted.)
5. **Sweep for unregistered candidates** (condition 5):
   - Read `candidates/inbox/` — for each file with no matching register row, add a
     `pending-review` row from its schema fields, then delete the inbox file (the register row
     supersedes it).
   - Scan the file's pages for frames named `CANDIDATE — *` with no register row. Match against
     existing rows by node id **and** name — including `one-off` tombstones, which are never
     re-recorded. For each true miss, add a `pending-review` row citing the node and note it was
     sweep-recorded (no inbox file). Do **not** edit the frames themselves.
6. **Re-check the goal.** If any `promote` row, drift, or unregistered candidate remains, repeat
   steps 3–5. Otherwise stop. Do **not** wait on `pending-review`; do **not** touch `one-off`.

## Guardrails

- **Single-writer:** only the Librarian may write components, variables, text styles, `SKILL.md`,
  `CANDIDATES.md`, or `PROJECT.md`. This command assumes the Librarian session.
- **Never auto-decide a candidate.** `pending-review` → `promote` is a human action only.
- **No publishing** unless explicitly instructed — reconciliation syncs records, it does not
  publish the library.
- Close with a short report: components added to docs, orphans fixed, `promote` rows built, and
  confirmation that `pending-review` / `one-off` rows were left untouched.
