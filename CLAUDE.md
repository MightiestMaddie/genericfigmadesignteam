# CLAUDE.md — Design-System Agent Team

Standing rules for every Claude session working on **this project's Figma file**. The file
key, library page, Sections, and brand live in [`PROJECT.md`](PROJECT.md) — read it first;
it is the single source of truth for everything file-specific. If `PROJECT.md` still has
`<FILL IN>` placeholders, the project is **not set up yet** → run [`/startup`](.claude/commands/startup.md).

**Precedence.** This file governs *team coordination* — who may write what, and how missing
pieces are handled. The **`figma-design-system`** skill governs *how to build and style*
(foundations, naming, recipes, variable/text-style binding, sources citation). When this file
restricts an action the skill describes, this file wins. Everything the skill says about
reuse-first, variant properties, variable binding, and the closing **Sources:** section still
applies to everyone.

---

## Roles

Exactly one **Librarian**, one or more **Page agents**, and optionally a **Reviewer** per
team. Know which you are before acting; if unsure, ask the lead.

- **Librarian** — the single writer of all *file-global* state. Owns the component library
  page, variables, text styles, effect styles, library publishing, and the design-system
  records (`SKILL.md`, `CANDIDATES.md`, `PROJECT.md`). Also runs the one-time **`/startup`**
  bootstrap that creates the foundations and the initial component set.
- **Page agent** — works inside one assigned page (or page set). Consumes the library by
  instancing; never writes file-global state.
- **Reviewer** (`/reviewer`) — read-only auditor. May *inspect* any page (reads and
  screenshots only); never edits Figma and never writes the records or any file. Produces a
  findings report and routes each fix to the Librarian (file-global) or the owning page
  agent (page-local).

---

## The single-writer invariant (read this first)

These are **file-global** in Figma — a change to any of them is seen by every page and every
agent. **Only the Librarian** may create, edit, rename, delete, or publish them:

- Main components and component sets (anything on the **library page**, in any Section
  including **Sandbox / WIP**)
- Variant properties on existing components
- Variables / variable collections and modes
- Text styles and effect styles
- Publishing / updating the shared library
- `SKILL.md`, `CANDIDATES.md`, and `PROJECT.md`

Pages isolate *layouts*. They do **not** isolate the library. Worktrees only protect the
repo files in git — never the shared Figma document. So the library is coordinated, not
partitioned: it has exactly one writer.

---

## Two phases: bootstrap, then the loop

1. **Bootstrap (once, Librarian only).** On a brand-new / empty file there is nothing to
   consume yet — no variables, no styles, no components. The Librarian runs **`/startup`**:
   verify file access → establish foundations → create the library page + Sections → build the
   core component set → seed the records → publish. This is the *only* time file-global
   creation happens in bulk, and it is gated the same as any other library write (show the
   human the plan, wait for go). See [`docs/startup.md`](docs/startup.md).
2. **The loop (ongoing).** Once foundations + a core set exist, page agents build screens from
   instances and raise **candidates** when a primitive is missing; the human decides
   promote / one-off; the Librarian builds promotions and reconciles the records. See
   [`docs/workflow.md`](docs/workflow.md).

Until bootstrap is done, a page agent has almost nothing to instance — expect it to raise many
candidates, which is exactly how the library grows in the early days.

---

## Page agent rules

1. **Stay on your page.** Edit only your assigned page. Never touch another agent's page or the
   library page.
2. **Consume, don't create.** Build screens from **instances** of existing components
   (`createInstance()` → `setProperties({...})`), per the skill's reuse-first rule. Respect the
   platform prefix (e.g. `Mobile ...`) and the slash/middot family naming.
3. **Never write file-global state.** Do not create main components, edit variants, add
   variables / styles, or publish the library. If the skill's reuse-first step 3 ("build a new
   component in the Section") would apply, **raise a candidate instead** (next section).
4. **Don't block on missing pieces.** If a needed primitive doesn't exist, build a *candidate*
   and keep moving. Do not wait on the Librarian.
5. **Bind as you go.** Every fill, stroke, text color, and text style on nodes you create must
   be bound to file variables / text styles (skill: "never raw values"), including padding/gaps
   to the spacing variables. This applies to candidates too.

---

## Candidate protocol (when a primitive is missing)

A page agent that hits a missing primitive **builds it anyway**, as a reviewable candidate, then
flags it. The human decides later whether it's promoted into the library or kept as a one-off.

**How to build a candidate**

- Build it **page-local**: a styled frame/group on your own page. It is **NOT** a main
  component and goes **nowhere near the library page**.
- Place it in a frame named **`CANDIDATES`** on your page, and name the element
  **`CANDIDATE — <family> / <name>`** so it's findable and the family is obvious.
- **Token-first.** Compose from existing instances where possible, and bind every value to
  existing variables / text styles. Only introduce a new raw value if genuinely unavoidable —
  and list it explicitly in the callout so review is fast. A candidate that already speaks the
  token language is a *promotion*; one that invents values is a *rebuild*.
- Follow the platform + naming conventions as if it were going into the library, so promotion
  is a lift, not a redesign.

**How to flag it (do both)**

- Leave the visual marker in place (the `CANDIDATES` frame + name prefix).
- Write **one inbox file** to **`candidates/inbox/<your-page>--<candidate-name>.md`** in the
  repo — this is the durable notification channel (a session-scoped task list is invisible
  to other sessions and dies with yours; a file in git does not). Contents = this schema:

  ```
  candidate-review
  name:        CANDIDATE — <family> / <name>
  raised_by:   <agent> · <page>
  gap:         <what existing component was missing / why an instance didn't fit>
  node:        <figma node link to the candidate>
  built_from:  <instances/components reused, e.g. "Input + Badge">
  new_values:  <none | list each raw value not bound to a variable>
  ```

The inbox file is the *only* record you write — do not also edit `CANDIDATES.md` (that's the
Librarian's single-writer record; each agent writes only its **own** inbox files, so the
single-writer rule holds per file). Also mention the candidate in your report to the human.
Expect duplicates across pages (two agents may both need a "pill badge"); that's fine —
dedup happens at human review. As a safety net, `/reconcile` also sweeps for `CANDIDATE — *`
frames that never got an inbox file — but don't rely on the net: write the file.

---

## Librarian rules

- **Sole writer** of everything in the single-writer invariant list.
- **Bootstrap the file** with `/startup` on a new project (foundations → Sections → core set).
- **Record candidates.** When an inbox file appears in `candidates/inbox/` (or the
  reconciliation sweep finds an unregistered `CANDIDATE — *` frame), add a row to
  `CANDIDATES.md` with status `pending-review`, then delete the inbox file — the register
  row supersedes it. Never auto-decide.
- **On a human `promote` decision:** build the real component in the correct Section of the
  library page following the skill's naming/variant rules, bind variables + text styles + the
  spacing scale, add its entry to `SKILL.md`, set the register row to `promoted`, and
  (coordinating with the page agent) swap that page's local candidate for instances of the new
  component.
- **On a human `one-off` decision:** set the register row to `one-off`. Leave the page-local
  element as-is; it never enters the library.
- **Run reconciliation** (`/reconcile`). Page agents never run it.
- Don't enter page agents' pages except to replace a promoted candidate with real instances,
  and message the owning agent first.

---

## Candidate register — `CANDIDATES.md`

Librarian-owned, in the repo (git = the durable record the reconciliation loop reads). One row
per candidate:

```
| ID | Name | Raised by (agent · page) | Gap | Node | Built from | New values | Status | Decided |
```

**Status values**

- `pending-review` — awaiting the human. **Terminal for agents.** Only the human moves it on.
- `promote` — human approved; Librarian has not built it yet. **Loop-actionable.**
- `promoted` — built on the library page and documented in `SKILL.md`. **Done.**
- `one-off` — human rejected for the library. **Tombstone — permanently ignored.**

Only a human moves a row out of `pending-review`. Agents never self-promote.

---

## Reconciliation — `/reconcile` (Librarian session only)

The loop's job is **documentation and sync**, not promotion. Human-gated states are explicitly
**not** loop work, so the loop must not churn waiting on them.

**Goal (stop condition — all true):**

1. No candidate is in status `promote` (every approved candidate has been built → `promoted`).
2. Every component on the library page has a matching `SKILL.md` entry. *Documented-by-design
   exclusions (not violations):* a bulk icon library counts as documented by its single
   family line; **Sandbox / WIP** is intentionally not itemized; assembled example patterns
   are recipes, not entries.
3. Every `SKILL.md` entry maps to a real component on the library page.
4. `pending-review` and `one-off` rows are **ignored** — they are *not* violations and do *not*
   keep the loop running.
5. Every file in `candidates/inbox/` and every frame named `CANDIDATE — *` in the Figma file
   has a matching `CANDIDATES.md` row — the loop **records** missing ones as `pending-review`
   (recording is not deciding; the status decision stays with the human).

This makes "pending review" a valid resting state. The loop scans the library against the
records, builds/docs anything in `promote`, fixes drift, records unregistered candidates, and
stops — it never re-flags a `one-off`, and it never waits on a human decision. It is
idempotent — safe to run repeatedly.

---

## Quick reference

| Action | Page agent | Reviewer | Librarian |
|---|---|---|---|
| Run one-time `/startup` bootstrap | ❌ | ❌ | ✅ |
| Instance existing components on own page | ✅ | ❌ (read-only) | ✅ |
| Edit another agent's page | ❌ | ❌ (may **read** any page) | only to swap a promoted candidate (coordinate first) |
| Create main component / variant on the library page | ❌ | ❌ | ✅ |
| Add/edit variables, text styles, or effect styles | ❌ | ❌ | ✅ |
| Publish the library | ❌ | ❌ | ✅ |
| Build a page-local candidate + write inbox file | ✅ | ❌ (flags untracked primitives in its report) | n/a |
| Write `CANDIDATES.md` / `SKILL.md` / `PROJECT.md` | ❌ | ❌ | ✅ |
| Promote a candidate | ❌ (humans only) | ❌ (humans only) | executes the human's decision |
| Run reconciliation loop | ❌ | ❌ | ✅ |
