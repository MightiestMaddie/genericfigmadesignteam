---
name: figma-design-system
description: Design-system knowledge base for building and editing screens in this project's Figma file so new work matches the existing library. Use this skill whenever the user asks to create, edit, extend, or mock up any screen, dialog, component, or mobile view — even if they don't say "design system". It defines HOW to build: foundations (variables, text styles, effect styles), the Atomic Design tier taxonomy (Atoms → Molecules → Organisms → Templates → Pages), naming and platform conventions, the reuse-first rule, component recipes, and which variables to bind. The file key, library page, Sections, and brand values live in PROJECT.md; the live component inventory below is filled in as the library is built (via /startup) and kept in sync (via /reconcile). Also use it when asked to make a design "match the house style".
---

# Figma Design System

Knowledge base for designing in **this project's Figma file** so new and edited designs match
the existing screens. This skill is the *how to build & style* reference; team coordination
rules are in [`CLAUDE.md`](../../../CLAUDE.md), and everything file-specific (file key, library
page, Sections, brand) is in [`PROJECT.md`](../../../PROJECT.md).

- **File key, library page, Sections, brand:** see [`PROJECT.md`](../../../PROJECT.md). Do not
hard-code these anywhere — read them from `PROJECT.md`.
- **Library page:** the "Components" page named in `PROJECT.md` is the single source of truth —
every component is defined there and screens consume **instances** of it.
- **Greenfield?** If `PROJECT.md` still has placeholders (no file key, no Sections, empty
inventory below), the library doesn't exist yet — run [`/startup`](../../commands/startup.md) to
establish foundations and the core component set before trying to consume the library.

## Atomic Design taxonomy (how the library is organized)

This library follows **Atomic Design** (Brad Frost): every component belongs to exactly one
composition tier, and higher tiers are built only from lower ones. Never build "sideways or up" —
an Organism may use Molecules and Atoms; a Molecule may use Atoms; an Atom uses nothing but
tokens.

| Tier | Definition | Examples in this library |
| --- | --- | --- |
| **Atoms** | Smallest indivisible pieces — a single element that still makes sense on its own. | Button, Input, Textarea, Checkbox, Chip, Badge, Icon |
| **Molecules** | A small, functional group of Atoms working together as one reusable unit. | Select, Search Bar, Table Cell, Nav Item, Toast, Inline Alert, Empty State, Section Header |
| **Organisms** | A distinct, complex section of an interface, composed of Molecules and/or Atoms. | Sidebar, Page Header, Table, Dialog, Card, Bottom Tab Bar |
| **Templates** | A page-level layout skeleton — placement of Organisms/Molecules with placeholder content, no real data. | List Page Template, Detail Page Template, Form Dialog Template |
| **Pages** | A Template filled with real content. | Not part of the library — this is what Page agents build on canvas. |

**Classifying a new component:** ask "what is the smallest thing this stops being useful at?"
If it can't be broken into smaller reusable UI pieces → Atom. If it's 2–4 Atoms that only ever
appear together → Molecule. If it's a recognizable, self-contained chunk of a real screen →
Organism. If it's a full-page layout skeleton reused across multiple screens → Template.

## Reuse-first rule (READ THIS BEFORE DRAWING ANYTHING)

The most common mistake is rebuilding a component that already exists. Do **not** draw a
button, input, badge, dialog, table cell, sidebar, etc. from scratch.

1. **Find it first.** Before creating any UI element, locate the matching component on the
library page using the inventory below (or `get_metadata` on the library page) — check the
tier you'd expect it in (Atoms first for primitives, then Molecules, then Organisms).
2. **Instantiate, don't recreate.** Create an instance of the existing component
(`component.createInstance()` / `componentSet.defaultVariant.createInstance()`), then set
its variant properties (`instance.setProperties({...})`) to the state you need.
3. **Only build new** if nothing in the inventory fits. Under the **team model** (see
`CLAUDE.md`) this differs by role:
  - **Page agent** → do **not** build a main component. Build a page-local **candidate** and
raise it (candidate protocol) — note which tier you believe it belongs to. Keep moving.
  - **Librarian** → build it as a proper component **inside the matching tier Section**,
following the naming rules below, bind everything, document it in the inventory, and tell the
user.

## Foundations first (the greenfield order of operations)

On a new file, nothing can be reused until the **foundations** exist, and nothing above Atoms
can be reused until the Atoms exist. Build strictly bottom-up — a component built before its
tokens (or before its lower-tier dependencies) exist will carry raw values or ad-hoc children
you have to retrofit later. The concrete Figma Plugin API recipe for every step here
(hex→rgb, collections/modes, semantic aliasing, spacing/radii scopes, effect + text styles) is
in [`docs/foundations.md`](../../../docs/foundations.md).

1. **Variable collections & modes.** Create the collections named in `PROJECT.md` (recommended: a **Brand** collection, a broad **TailwindCSS**-style palette, and a **Mode** collection for light/dark `base/*` semantics). One writer only (the Librarian).
2. **Color variables.** Brand + neutrals + semantic accents (success/danger/warning/info).
Bind by exact hex; see the default palette below.
3. **Spacing scale.** Float variables (`spacing/2, /4, /6, /8, /12, /16, /24, /32`) with a
`GAP` scope — bind every padding and gap to these, never hard-code.
4. **Radii.** `Radius/Small`, `Medium`, `Large`, and a pill value (999).
5. **Effect styles.** At least a **"Soft"** card shadow so cards bind a named style, not a raw
shadow.
6. **Text styles.** The type scale as local text styles (see Typography). Apply styles with
`setTextStyleIdAsync`, never raw fontName/fontSize.
7. **Then Atoms.** Build the core Atom set (Button, Input, Checkbox, Badge, Icon, …), binding
every value to the tokens above.
8. **Then Molecules.** Build Molecules strictly from the Atoms above (e.g. Select from Input +
listbox, Search Bar from Input + Button).
9. **Then Organisms.** Build Organisms from the Molecules/Atoms above (e.g. Page Header, Table,
Dialog, Sidebar).
10. **Then Templates (as needed).** Once 2+ real screens share a layout skeleton, promote that
layout into a Template in its Section.

`/startup` walks this exact sequence and records the created ids in `PROJECT.md`.

## Naming & platform convention

- **Tier prefix is mandatory.** Every component name starts with its tier: `Atom / Button`,
`Molecule / Search Bar`, `Organism / Page Header`, `Template / List Page`. This makes the tier
visible in the layers panel and in every instance name, and stops accidental cross-tier misuse.
- **Platform prefix** (e.g. **`Mobile ...`**) comes right after the tier, e.g.
`Atom / Mobile Button`. Everything **without** the platform prefix is for the primary
(desktop/web) application. Never use a mobile component on a desktop screen or vice-versa.
(Decide the prefix in `PROJECT.md` if the product is multi-platform; skip it for a
single-surface product.)
- Within a tier, components use **slash grouping** where they belong to a family
(`Molecule / Nav / Item`, `Molecule / Filter / Row`, `Molecule / Table Cell · Badge`). Match the
existing group when extending a family.
- Use **variant properties** (Type, State, Size, Color, Align…) instead of duplicate frames.
- **Variant values are a public API** — painful to rename after instances exist. Use standard
names from the start: `State {Default, Active}`, `Type {Primary, Secondary, …}`, etc.

## Component inventory

> **This table is the living record of the library.** On a new project it starts effectively
> empty; `/startup` seeds it with the core set it builds, and `/reconcile` keeps it in sync with
> the live file thereafter. Each Section maps to a Section node id recorded in `PROJECT.md`.
> Desktop unless marked **[Mobile]**. IDs are the component / component-set nodes.

### Atoms

Smallest indivisible pieces — bind directly to tokens, no other components as children.

| Component  | Node   | Variant properties                                                              |
| ---------- | ------ | -------------------------------------------------------------------------------- |
| **Atom / Button** | `7:27` | Type {Primary, Secondary, Destructive, Link} × State {Default, Hover, Disabled} |
| **Atom / Input**    | `7:36` | State {Default, Focused, Error, Disabled} |
| **Atom / Textarea** | `7:45` | State {Default, Focused, Error, Disabled} |
| **Atom / Checkbox** | `8:11` | State {Unchecked, Checked, Indeterminate, Disabled} |
| **Atom / Chip**     | `8:16` | Type {Default, Selected} |
| **Atom / Badge**    | `8:27` | Color {Neutral, Success, Danger, Warning, Info} |

### Molecules

Small groups of Atoms functioning as a single reusable unit.

| Component  | Node   | Composed of                        | Variant properties |
| ---------- | ------ | ------------------------------------ | ------------------- |
| **Molecule / Select**         | `9:21` | Atom / Input + Atom / Icon / ChevronDown + listbox | State {Closed, Open} |
| **Molecule / Tab**            | `9:34` | Text + indicator | State {Default, Active, Hover} |
| **Molecule / Table Cell**     | `11:57` | Text or Atom / Badge | Type {Text, Badge, Header} |
| **Molecule / Toast**          | `10:37` | Atom / Icon (status-tinted) + text | Status {Success, Warning, Error, Info} |
| **Molecule / Inline Alert**   | `10:61` | Atom / Icon + text on tinted panel | Status {Info, Success, Warning, Error} |
| **Molecule / Empty State**    | `11:34` | Atom / Icon / Search + text + Atom / Button (Secondary) | — |
| **Molecule / Section Header** | `11:49` | Text + optional Atom / Button (Link) | Action {None, Link} |
| **Molecule / Nav / Item**     | `9:50` | Atom / Icon + text | State {Default, Active, Hover} |

### Organisms

Distinct, self-contained sections composed of Molecules and/or Atoms.

| Component  | Node   | Composed of                        | Variant properties |
| ---------- | ------ | ------------------------------------ | ------------------- |
| **Organism / Card**       | `12:65` | Header + content + footer w/ Atom / Button ×2 | Header {With, Without} × Footer {With, Without} |
| **Organism / Dialog**     | `13:44` | Header text + Atom / Icon / X + content + Atom / Button ×2 | — |
| **Organism / Page Header**| `13:70` | Title + subtitle + Atom / Button ×2 | Layout {With Buttons, Without Buttons} |
| **Organism / Table**      | `13:71` | Header row + 3 rows of Molecule / Table Cell | — |

*(No Sidebar yet — raise as a candidate when a screen needs one.)*

### Templates

Page-level layout skeletons — populated as repeated patterns emerge across screens. Empty on a
new project; add rows once 2+ real screens share a structure.

| Component  | Node   | Composed of         | Notes |
| ---------- | ------ | --------------------- | ----- |
| *(none yet)* | — | — | Promote once a layout repeats across screens |

### Icons (Atoms)

Prefer a full **Lucide** icon library imported as components named `Atom / Icon / <PascalCase>`
(24×24, stroke bound to a color var). Instance and override the stroke color; **scale the stroke
with the size** (Lucide is drawn 24px/2px, so stroke ≈ `size ÷ 12`).

Current set (stroke bound to `base/foreground`; retint per use): Check `6:8`, X `6:12`,
ChevronDown `6:15`, ChevronRight `6:18`, Search `6:22`, Plus `6:26`, AlertCircle `6:31`,
AlertTriangle `6:36`, Info `6:41`, CheckCircle `6:45`, Settings `6:49`, User `6:53`.
Need another Lucide glyph? Raise a candidate (page agents) or add it here (Librarian).

### Sandbox / WIP

Holds unfinished work, any tier. **Do not reuse** anything from this Section in real screens.

## Working rules

1. **Look before you draw.** Screenshot the relevant Section or a reference screen with
`get_screenshot` before creating, so the new design matches the latest file state, then
follow the reuse-first rule above.
2. **Bind variables, don't hard-code.** Prefer Brand variables, fall back to the Tailwind
palette, then nearest. After building a frame, run a bind pass over every fill / stroke /
text-size / padding / gap before finishing.
3. Use `use_figma` for edits (load the `figma-use` skill first). Verify with a final screenshot
and compare against neighbors.
4. Typical frame sizes: desktop 1440×1024–1080; mobile ~390–402 wide. (Confirm the product's
canonical sizes in `PROJECT.md` if they differ.)
5. **Respect the tier boundary.** When building an Organism, only reach into Molecules/Atoms —
never invent a one-off "shortcut" primitive that skips a tier. If a needed Atom doesn't exist,
raise it as an Atom-tier candidate first (Page agent) or build it as an Atom first (Librarian).

## Default foundations (neutral starter — rebrand in `/startup`)

A sensible, production-grade neutral baseline (shadcn/Tailwind-flavored). Use it as-is for a
quick start, or have `/startup` swap the brand color + fonts for the real brand. Record the
decided values in `PROJECT.md`.

### Color palette

Brand (Brand collection) — **replace the primary with the real brand color**:

| Token             | Default hex                   | Use                                    |
| ----------------- | ------------------------------ | --------------------------------------- |
| Primary           | `#2563eb`                     | Brand accent: links, active nav, focus |
| Primary-Dark      | `#1d4ed8`                     | Hover/emphasis                         |
| Primary button bg | `#1e293b` (or the brand dark) | Primary button background              |
| Primary-Light     | `#dbeafe`                     | Selected/active tint                   |
| Background        | `#f8fafc`                     | Page/canvas behind cards               |

Neutrals (Mode / `base/*` collection):

| Token                      | Default hex | Use                                    |
| --------------------------- | ------------ | ---------------------------------------- |
| base/foreground            | `#18181b`   | Primary text                           |
| base/muted-foreground      | `#71717a`   | Secondary text, table headers          |
| Placeholder text           | `#a1a1aa`   | Input placeholders                     |
| Border-Mid                 | `#e2e8f0`   | Default borders: cards, rows, dividers |
| Border-Dark                | `#cbd5e1`   | Input borders, tertiary stroke         |
| base/muted                 | `#f4f4f5`   | Active nav bg, muted fills             |
| base/background, base/card | `#ffffff`   | Card & dialog surfaces                 |

Semantic accents (Tailwind collection): success `green/700 #15803d` on `green/50 #f0fdf4`
(border `green/200`); danger `red/600 #dc2626` on `red/50 #fef2f2` (border `red/200 #fecaca`);
warning `amber/700 #b45309` on `amber/50 #fffbeb`; info `blue/600 #2563eb` on `blue/50 #eff6ff`.

### Typography

Default families: **Inter** for UI, and a heading face (Inter, or Geist for a distinct heading
voice). Apply as local text styles:

| Style                                 | Spec            | Use                               |
| --------------------------------------- | ----------------- | ------------------------------------ |
| custom/heading-md                     | Bold 30/36      | Page titles                       |
| text-2xl/leading-8/font-semibold      | Semi Bold 24/32 | Dialog titles                     |
| text-lg/leading-none/font-semibold    | Semi Bold 18    | Card/section titles               |
| text-sm/leading-5/font-semibold       | Semi Bold 14/20 | Input labels, emphasized cells    |
| text-sm/leading-5/font-normal         | Regular 14/20   | Body, input values, table cells   |
| text-xs/leading-4 (normal & semibold) | 12/16           | Table headers, captions, sub-text |

Sizes are on-scale (12/14/16/18/20/24/30/36…); canonical leadings 12→4, 14→5, 16→6, 18/20→7,
24→8, 30→9. Note: for **Inter** in `use_figma` the weight string is "Semi Bold" (with a space),
not "SemiBold".

### Spacing, radii, borders, shadows

- **Spacing scale** (`spacing/*`): float variables `2, 4, 6, 8, 12, 16, 24, 32` (value = name),
`GAP` scope. Bind padding/gaps to these. Dialog padding 24; card/segment padding 12–16;
label→input gap 8.
- **Radii**: Small **6** (buttons, inputs, square badges), Medium **10** (cards), Large **14**
(mobile cards); dialogs **16**; pills/tags **999**.
- **Borders**: 1px solid throughout. Dashed border marks computed/read-only fields and "add
another" zones.
- **Shadows**: cards use a named **"Soft"** effect style (e.g. `0 10 20` neutral @ ~18%). Apply
the style, not a raw shadow. Dialogs a deeper shadow (e.g. `0 16 24 rgba(15,23,41,0.16)`).
- **Icons**: Lucide line icons, 24×24 base. Sizes 14 (in buttons), 16 (inputs/nav/inline), 18
(icon-chips), 18–20 standalone. Scale the stroke with the size.

## Component recipes (visual specs — patterns, not fixed nodes)

### Buttons (Atom)

- **Button** (height ~36, radius 6, px 16): **Primary** = brand-dark bg, white Semi Bold 14,
optional 14px icon + 4px gap. **Secondary** = white bg, 1px Border-Dark stroke, Medium 14.
**Destructive** = red. **Link** = brand-blue Medium 13.5. Dialog footers: Secondary + Primary,
equal widths, 16px gap. Exactly one Primary per view.

### Inputs (Atom) / Select (Molecule)

- **Input**: label above (Semi Bold 14, 8px gap); field height ~28–36, white, 1px Border-Dark,
radius 6, 12px horizontal padding, value Regular 14, placeholder muted. Disabled = muted bg,
dashed border. Use State variants.
- **Textarea**: same border, radius 8, padding 12×10. **Select**: trigger (Atom / Input styling)
+ dropdown popover (Molecule-level composition).

### Dialogs (Organism)

White, radius **16**, 1px border, deep shadow. Header: 24px padding, title Semi Bold 24, 18px X
top-right. Content: 24px side padding, 18px gaps. Nested option groups in a muted panel (radius
10, subtle border, padding 12). Widths: forms 640–660, confirms ~512.

### Tables / list pages (Organism, built from Molecule cells)

Page: white top bar (h ~64, bottom border, logo) → **Organism / Page Header** → filter row
(status select, Filter button + count Badge, right-aligned search) → table card: white, radius
~10, Border-Mid, **Soft** shadow. Assemble from an **Organism / Table** + **Molecule** header/cell
components. Status **Atom / Badge** pills. Footer: "1–20 out of 100" left; rows-per-page + pager
right.

### Sidebar (Organism, built from Molecule nav items)

An **Organism / Sidebar** shell (~256 wide) with **Molecule / Nav / Item**,
**Molecule / Nav / Sub Item**, section labels (**Molecule / Nav / Group Label**), user card
(**Molecule / Nav / Footer User**). Active item: muted bg, radius 6.

### Tags / chips (Atom)

Pill radius 999, px 8, py 2, Type by color.

### Mobile app (if the product has one; frames ~390–402 wide, platform-prefixed components)

- **Header** (Organism): brand gradient, white Semi Bold ~18 title, back arrow, subtitle 12
white/80.
- **Content** (layout, not a component): page bg; white cards radius 14 with a mobile shadow,
padding 16.
- **Primary CTA** (Atom / Mobile Button): full-width, h ~48, brand-dark bg, white Semi Bold
14–16, radius 8–10.
- **Bottom tab bar** (Organism): white, 5 items, 12px labels; active = brand icon in a tint pill.

## Always use file variables & text styles — never raw values

Every fill, stroke, text color, and text style on new or edited nodes MUST be bound to this
file's variables/styles. Never leave raw hex or raw font settings.

**Colors** → bind with `figma.variables.setBoundVariableForPaint`. Match by exact hex, preferring
the **Brand** collection, then the **Tailwind** palette, then nearest opaque var. Neutral dark
text → `base/foreground`; gray text → `base/muted-foreground`; white/accent text keeps its own
color var.

**Typography** → apply a local **text style** with `node.setTextStyleIdAsync` (do NOT set raw
fontName/fontSize). Pick the nearest scale size + matching weight.

**Spacing** → bind padding/gaps to `spacing/*`. **Radii** → `Radius/*`. **Card shadow** → the
`Soft` effect style.

**Workflow:** after building any frame, run a bind pass over it (colors + text styles + spacing)
before finishing — this is part of "done," not optional. Instances inherit from their components,
so styling a component covers all its instances. For large sections, run the pass per-frame or
skip recursing into INSTANCE interiors to avoid timeouts.

## Checklist before finishing

- [ ] Reused existing components (instances), did **not** rebuild anything in the inventory
- [ ] New/edited component correctly tier-prefixed (`Atom / …`, `Molecule / …`, `Organism / …`, `Template / …`) and built only from its own tier or lower
- [ ] Right platform set: platform-prefixed components only on their platform's frames
- [ ] Page bg + surfaces + borders bound to the neutral tokens
- [ ] Exactly one Primary Button per view; everything else Secondary/Link
- [ ] Titles use the heading style; body 14; correct text styles applied (not raw fonts)
- [ ] Cards carry the "Soft" shadow style; radii 6/10/14/16; Lucide icons only
- [ ] Status colors from the semantic set (green/red/amber/blue tints)
- [ ] Variables bound (colors + text + `spacing/*`); final screenshot compared to neighbors

## Citing sources

After any Figma work, end the response with a **Sources:** section listing every artifact touched
as node-specific Figma links.

- Format: `- [Human-readable name](URL)` — one per line, grouped logically.
- URL: build from the file key + URL name segment in `PROJECT.md`:
`https://www.figma.com/design/<FILE KEY>/<URL NAME SEGMENT>?node-id=<A>-<B>` (convert node id
`A:B` → `A-B`).
- List the deliverable frames/components, not scratch nodes; put the most useful entry point
first.
