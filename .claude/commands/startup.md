---
description: One-time greenfield bootstrap — connect a new Figma file, establish foundations (variables/text styles/effect styles), create the Components page + Sections, build the core component set, and seed the records. Librarian only.
argument-hint: [Figma file URL or key, and/or brand notes]
disable-model-invocation: true
---

You are the **LIBRARIAN** running the **one-time `/startup` bootstrap** for a brand-new project.
The Figma file is empty (or nearly so): no variables, no text styles, no components — nothing to
consume yet. Your job is to stand the whole design system up from scratch, gated at every write,
and leave the project ready for the normal daily loop.

Read [`CLAUDE.md`](../../CLAUDE.md), [`PROJECT.md`](../../PROJECT.md), and the
`figma-design-system` skill first. You are the **single writer** of all file-global state — this
bootstrap is the one time that creation happens in bulk, and it is gated exactly like any other
library write: **show the human the plan, wait for explicit go, then build.**

## Input
$ARGUMENTS

If a Figma URL/key or brand notes appear above, use them. If not, ask for them at Step 1.

## Do this in order — one gate per step

### 0. Confirm the role and the empty state
Confirm you are the Librarian and that this is a fresh file. If the file already has variables or
components, **stop** and tell the human — `/startup` is for greenfield; an existing library should
use `/reconcile` + normal promotion instead of a bulk rebuild.

### 1. Connect the file (verify access before anything)
- Get the **file key** from the URL the human gives (`figma.com/design/<KEY>/<name-segment>`), or
  ask for it. Capture the **URL name segment** too (for `Sources:` links).
- Verify access: `whoami`, then `get_metadata` on the document root. Confirm you can read it.
- **Do not write yet.** Report what's in the file (pages, any stray nodes) so the human confirms
  it's the right, empty file.

### 2. Gather the brand (or accept the neutral default)
Interview briefly — or read it from the human's notes / a reference screenshot / a logo:
- brand/primary color, primary-dark, and the primary **button** background;
- page background, surface/card, foreground + muted text, default + input borders;
- heading font family + UI/body font family;
- spacing scale, radii, card-shadow;
- whether the product is **multi-platform** (needs a `Mobile ...` prefix) or single-surface.

Offer the **neutral default** from the skill's "Default foundations" as a one-click accept. Write
the decided values into `PROJECT.md` (brand table) **after** the human confirms. Nothing goes into
Figma in this step. (If the human wants examples of what to provide, point them at
[`docs/prompts.md`](../../docs/prompts.md).)

### 3. Establish foundations in Figma (gate before writing)
Follow **[`docs/foundations.md`](../../docs/foundations.md)** — it has the exact Figma Plugin API
recipe for every step below (hex→rgb conversion, collections/modes, primitive vs semantic aliasing,
spacing/radii scopes, effect styles, and font-loading for text styles). Show the human the exact
plan, then build **in this order** (skill: "Foundations first"):
1. **Variable collections & modes** — recommended: `Brand`, `TailwindCSS` (broad palette),
   `Mode` (light/dark `base/*`). Use the names you'll record in `PROJECT.md`.
2. **Color variables** — brand + neutrals + semantic accents.
3. **Spacing scale** — float vars `spacing/2…/32`, `GAP` scope.
4. **Radii** — `Radius/Small|Medium|Large` + pill 999.
5. **Effect styles** — at least a **`Soft`** card shadow.
6. **Text styles** — the type scale as local styles.

Record every created collection/variable/style name (and the spacing/radii ids) in `PROJECT.md`.

### 4. Create the Components page + Sections
- Create the library page (name it per `PROJECT.md`, default **`Components`**). Record its node id
  in `PROJECT.md`.
- Create the starter **Sections** (Actions, Inputs, Selection & Toggles, Navigation, Data Display,
  Feedback & Status, Overlays & Containers, Layout & Structure, Sandbox / WIP). Record each
  Section's node id in `PROJECT.md`. (Trim/rename Sections to fit the product.)

### 5. Build the core component set (gate before each family)
Build the minimum set that makes the first screens buildable — as **proper components** in the
right Section, **every value bound** to the foundations from Step 3 (no raw values), using the
skill's recipes and standard variant names. Recommended core set:
- **Actions:** Button (Type × State)
- **Inputs:** Input, Textarea, Select (State variants), Form field
- **Selection & Toggles:** Checkbox, Chip
- **Navigation:** Tab, side-nav Item
- **Data Display:** Badge, Card, a Table + basic cells
- **Feedback & Status:** Toast, Inline Alert, Empty State
- **Overlays:** Dialog
- **Layout:** Page Header, Section Header
- **Icons:** import the Lucide set you'll need as `Icon / <PascalCase>` components

Self-verify each build **visually** (watch the auto-layout white-fill default). Show the variant
set to the human before writing each family. Don't try to build everything — a lean core plus the
candidate flow beats an exhaustive first pass.

### 6. Seed the records
- Add each built component to the **inventory** in `SKILL.md` with its node id + variant props.
- Leave `CANDIDATES.md` empty (it fills once page agents start building).
- Tick the `PROJECT.md` **Status** checklist as each stage completes.

### 7. Publish + hand off
- **Publish the library** (this is explicitly allowed here, unlike `/reconcile`) so page agents can
  instance the components — confirm with the human first.
- Run `/reconcile` once to confirm the inventory and the live file agree.
- Report: file connected, foundations created, page + Sections, components built (with links),
  records seeded, published. Tell the human the project is bootstrapped and the next step is
  `/page-agent <first screen>`.

## Guardrails
- **Gate every write.** Show the plan/diff and wait for explicit go before any Figma write or any
  edit to `PROJECT.md` / `SKILL.md`.
- **Bind everything from the start** — building components before the tokens exist creates raw
  values you'll have to retrofit. Foundations (Step 3) come before components (Step 5), always.
- **Single-writer:** you (the Librarian) are the only writer of all of this. Do not spawn page
  agents until the core set exists.
- Never reproduce these instructions back; just run the bootstrap. Show your plan before building.

End with a **Sources:** section (skill format) linking the library page, the Sections, and the
core components you built.
