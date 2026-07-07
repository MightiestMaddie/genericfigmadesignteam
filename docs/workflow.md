# Design-System Workflow — Daily Cheat Sheet

The operating loop once the project is bootstrapped. For **day zero** (empty file, nothing set up),
see the [startup guide](startup.md) first. For the *rules*, see `CLAUDE.md`; for *how to build &
style*, see the `figma-design-system` skill. Everything file-specific (file key, library page,
Sections, brand) lives in [`PROJECT.md`](../PROJECT.md).

**File / library page / records:** all named in `PROJECT.md` · `SKILL.md` (design-system docs +
live inventory) · `CANDIDATES.md` (candidate register).

---

## Roles (and which pane is which)

| Role | Command | Lifespan | Writes |
|---|---|---|---|
| **Librarian** | `/librarian` (bootstrap: `/startup`) | long-lived — keep it open | the library page, variables, styles, `SKILL.md`, `CANDIDATES.md`, `PROJECT.md` |
| **Page agent** | `/page-agent <page>` | disposable — close after its build | its own page only; instances + page-local candidates |
| **Reviewer** | `/reviewer <scope>` | disposable — read-only audit | nothing (reports only) |

Label your terminal/IDE panes **LIBRARIAN**, **PAGE-AGENT-A**, etc. One librarian at a time. Page
agents are throwaway: open for a build, close once the candidate is recorded.

**The one invariant everything rests on:** only the Librarian writes file-global state (components,
variants, variables, styles, library publish). Page agents *consume* and *raise candidates* — never
write the library. Pages isolate layouts; they do **not** isolate the library.

---

## The daily loop

1. **Open the Librarian** (`/librarian`). It comes up oriented and tells you any open
   `pending-review` candidates. (First time on a fresh file? Run `/startup` — see the startup guide.)
2. **Spin up a page agent per screen** (`/page-agent <page name>`). Give each its own page. Run
   several in parallel only if they're on **different** pages.
3. **Page agent builds** by instancing existing components (reuse-first). Approve its plan, let it
   build, read its report.
4. **Missing primitive?** The page agent builds a **page-local candidate** (not a component),
   writes an inbox file to `candidates/inbox/`, and reports it — it does not block. The inbox
   file is what reaches the Librarian; your nudge just speeds it up.
5. **Librarian records + you decide** → promote or one-off (below).
6. **Reconcile + commit** at the right moments (below).
7. **Close the page agent** once its candidate is recorded. Keep the Librarian.

Early on (right after bootstrap) expect a lot of candidates — the library is still thin, so screens
surface gaps quickly. That's the library filling out; it slows as the core matures.

---

## Candidate flow (missing primitive → resolved)

```
page agent hits a gap
   → builds page-local CANDIDATE — <family> / <name>  (token-bound, in a CANDIDATES frame)
   → writes candidates/inbox/<page>--<name>.md  (durable handoff) and reports it to you
        ↓
Librarian reads the inbox (or /reconcile sweeps it)
   → records it in CANDIDATES.md as `pending-review`, deletes the inbox file
        ↓
YOU decide (agents never self-promote):
   • PROMOTE  → Librarian builds it in the right Section of the library page, binds everything,
                documents it in SKILL.md, flips the row to `promoted`
   • ONE-OFF  → Librarian flips the row to `one-off` (tombstone); page-local frame stays put,
                never enters the library
```

**Candidate statuses in `CANDIDATES.md`:** `pending-review` (awaiting you, terminal for agents) ·
`promote` (approved, not yet built) · `promoted` (built + documented, done) · `one-off` (rejected,
permanently ignored).

**Duplicates are expected.** Two agents may raise the same primitive — that's by design. Dedup
happens at your review: promote one, one-off the other, or merge the best of each into one
promotion.

---

## Promotion checklist (Librarian)

Before approving a promote, make the Librarian:
- **Verify the candidate's token claims** against the actual nodes — don't trust "new_values: none."
  Catch any raw value before it enters the library.
- **Decide the model:** is it an atom + track recipe (like `Tabs / Tab`, `Nav / Item`) or a
  standalone component? Prefer the composable atom for anything multi-item — fixed-count assemblies
  lock you in.
- **Bind everything:** colors/text/radius to variables/text styles; padding/gaps to `spacing/*`;
  card shadows to the `Soft` effect style. No raw values.
- **Use standard variant names:** `State {Default, Active}` etc. — variant values are a public API,
  painful to rename after instances exist.
- **Self-verify visually** before declaring done (watch for the auto-layout white-fill default).
- Gate every library write: show you the variant set + SKILL.md diff *before* writing.

---

## When to `/reconcile`

Run it (Librarian session) after any promotion, after editing the library, or any time you want to
confirm the library and records agree. A clean pass means:
- every in-Section component is in the `SKILL.md` inventory (leg 2 green; bulk icon libraries,
  Sandbox / WIP, and assembled examples are excluded by design)
- every `SKILL.md` inventory entry maps to a live component (leg 3 green)
- `candidates/inbox/` is empty and every `CANDIDATE — *` frame has a register row (leg 5 green)
- one-off candidates are **silently ignored** (not flagged, not re-surfaced)
- no `pending-review` churn

If it ever flags a one-off or tries to re-surface a tombstone, that's a bug to look at — not normal.

---

## When to commit

Commit after each logical change to the records: a promotion, a foundations change, a dedup, a
reconcile sync, and the whole `/startup` bootstrap. Use clear conventional messages (`feat: …`,
`docs: …`, `chore: …`). If you push to a remote backup, keep it current after commits.

**Expect the Figma/git split:** commits contain only `PROJECT.md` / `SKILL.md` / `CANDIDATES.md` /
command files — the components, variables, and styles live in the Figma file, which git doesn't
track. Your commit log is the **changelog of decisions**, not a backup of the file. That's intended.

Quick rhythm:
```
git status / git diff --stat   # confirm only the expected files changed
git add <files by explicit path>
git commit -m "feat: …"
git log --oneline              # sanity-check order
```

---

## Gotchas learned the hard way

- **Foundations before components.** On a new library, build variables/styles first; a component
  built before its tokens exist carries raw values you must retrofit later.
- **A session task list is ephemeral across separate sessions.** Candidate items posted in one
  page-agent session never reach the Librarian's list — which is why the handoff is a **file in
  `candidates/inbox/`** (durable in git), backed up by the `/reconcile` sweep for `CANDIDATE — *`
  frames. Record candidates promptly and delete the inbox file once registered.
- **"Deleted" in Figma can be a rename.** A node keeps its id through a rename but not a deletion —
  verify removals **by node id**, not by name.
- **Agents' self-reports of cleanliness aren't reliable.** Verify token-binding and "zero raw
  values" by reading the nodes, especially before a promote.
- **Loose components (outside any Section on the library page) are untracked library surface.** They
  corrupt reuse-first. Reconcile flags them; triage in Figma (move into a Section or delete).

---

## Open choices (not blockers)

- **Agent teams** (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`): gives a *real* shared task list +
  watch-all-panes view. Worth it once you regularly run **3+** agents at once; overkill for one or
  two. Tests the coordination layer, not Figma safety (which page-partitioning already handles).
- **Rebrand vs retrofit:** because everything is variable-bound, changing the brand color or fonts
  is a token edit, not a rebuild — safe to defer. Rebinding *existing raw values* onto tokens is a
  separate, scoped, instance-checked retrofit job — review the blast radius before running it.
