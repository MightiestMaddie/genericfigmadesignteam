# Startup Guide — bootstrapping a brand-new Figma project

This is the guide for **day zero**: you have a Figma file with nothing in it (no variables, no
styles, no components) and you want a Claude Code design team to stand the whole system up and
then keep building on it. If you already have a mature library, skip this — go straight to the
[daily workflow](workflow.md).

The mental model: **one Librarian builds the foundations and the core component library once;
after that, page agents consume it to build screens and raise candidates for anything missing.**
Until the foundations exist, there's nothing to consume — so bootstrap comes first.

---

## Prerequisites

- **Claude Code** with the Figma MCP connected (so `use_figma`, `get_metadata`, `get_screenshot`,
  etc. are available), signed in to the account that can edit your file.
- **Git** — `PROJECT.md`, `CANDIDATES.md`, and `SKILL.md`'s inventory are the durable records the
  reconciliation loop reads, so they should be version-controlled. This template is a repo already.
- **A Figma file** you can edit, and its **URL** (you'll pull the file key from it).
- Optionally: your brand — a primary color, fonts, and any reference screenshot or logo. If you
  don't have one yet, the team ships a sensible **neutral default** you can accept and rebrand
  later.
- **The fonts you'll use must be available in the Figma file** (e.g. Inter, and a heading font like
  Geist). Text styles can't be created for a font Figma can't load — add/enable them before
  bootstrapping. See [foundations.md](foundations.md) for the details.

Enable **agent teams** if you'll run 3+ agents at once (optional early on):
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

---

## The bootstrap, in one move

Open Claude Code in this directory, identify the session as the **Librarian**, and run:

```
/startup <paste your Figma file URL>   (add brand notes if you have them)
```

`/startup` runs the greenfield sequence, **gating at every write** (it shows you the plan and waits
for your go before touching Figma or the records):

1. **Connect the file** — pulls the file key from your URL, verifies access, and reports what's in
   the file so you confirm it's the right, empty one. *No writes yet.*
2. **Gather the brand** — a short interview (or reads your notes / a reference). You can accept the
   neutral default wholesale. Decided values are written to [`PROJECT.md`](../PROJECT.md).
   **Not sure what to tell it?** [prompts.md](prompts.md) has copy-paste prompts for every
   situation (full spec, logo/screenshot, just-a-color, or nothing yet) and explains what each
   input is for.
3. **Foundations** — creates variable collections, color variables, the spacing scale, radii,
   effect styles (a `Soft` card shadow), and text styles. **This comes before any component** —
   building components first would bake in raw values. The exact mechanics (what variables get
   created, how, and how to verify them in Figma's Local variables panel) are in
   **[foundations.md](foundations.md)** — read it if you want to understand or check what's being
   built.
4. **Components page + Sections** — creates the `Components` page and its Sections (Actions, Inputs,
   Navigation, Data Display, …), recording each node id in `PROJECT.md`.
5. **Core component set** — builds the minimum primitives that make real screens buildable
   (Button, Input, Select, Checkbox, Badge, Card, Dialog, Tabs, Page Header, Toast/Inline Alert,
   an icon set…), every value bound to the foundations.
6. **Seed the records** — writes each component into the `SKILL.md` inventory; leaves
   `CANDIDATES.md` empty; ticks the `PROJECT.md` status checklist.
7. **Publish + hand off** — publishes the library so page agents can instance it, runs `/reconcile`
   once to confirm records match the file, and tells you the next step.

You don't have to build everything up front. A **lean core plus the candidate flow** is the point —
the library grows as screens demand it.

---

## After bootstrap: what you have

- A `Components` page that is the single source of truth, organized into Sections.
- Foundations every component and screen binds to (colors, spacing, radii, shadow, text styles).
- A core set of components, documented in `SKILL.md` and recorded in `PROJECT.md`.
- Empty `CANDIDATES.md`, ready for the first gaps.

From here you're in the normal loop. Start your first screen:

```
/page-agent <first screen name>
```

The page agent builds by instancing the core set and **raises a candidate** for anything missing
(expected to be frequent early on — that's how the library fills out). You decide promote / one-off;
the Librarian builds promotions and reconciles. See the [daily workflow](workflow.md) for that loop.

---

## Tips & gotchas

- **Foundations before components — always.** The single most common early mistake is building a
  button before the spacing/color variables exist, then having to retrofit raw values. `/startup`
  enforces the order; don't shortcut it.
- **One Librarian, one writer.** Don't spawn page agents during bootstrap. All the file-global
  creation in `/startup` must come from a single session.
- **Verify token claims by reading nodes**, not by trusting "everything's bound." This matters most
  before you publish.
- **Commit after each stage.** `PROJECT.md` / `SKILL.md` / `CANDIDATES.md` are your changelog of
  decisions — the components themselves live in Figma, which git doesn't track.
- **Rebrand later is cheap; retrofit is not.** If you're unsure on brand, accept the neutral default
  now and swap the primary color + fonts later — because everything is variable-bound, that's a
  token edit, not a rebuild.
