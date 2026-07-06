# PROJECT.md — Design-team configuration

**The single source of truth for everything file-specific.** Every other doc, command,
and the design-system skill reference *this* file instead of hard-coding a file key, node
id, or brand value. When you start a new project, this is the file you fill in — run
[`/startup`](.claude/commands/startup.md) and it will populate the whole thing for you.

> Until the placeholders below are replaced, the team is **not configured**. `/startup` fills
> them as it stands the file up; after that, this file only changes when the library's
> structure changes (a new Section, a renamed page).

---

## Figma file

| Field                 | Value                                                                                     |
| --------------------- | ----------------------------------------------------------------------------------------- |
| **File name**         | `<FILL IN — e.g. "Acme — Product Designs">`                                               |
| **File key**          | `<FILL IN — the 22-char key from the URL: figma.com/design/<KEY>/...>`                    |
| **URL name segment**  | `<FILL IN — the slug after the key, url-encoded, e.g. Acme-%E2%80%94-Product-Designs>`    |
| **Library page**      | `<FILL IN node id — e.g. 1:2 — the "Components" page that is the single source of truth>` |
| **Library page name** | `Components` *(rename if you prefer)*                                                     |

**Node-URL format** (used in every `Sources:` list and candidate row): `https://www.figma.com/design/<FILE KEY>/<URL NAME SEGMENT>?node-id=<A>-<B>` (convert a node id `A:B` → `A-B`).

---

## Library page Sections — Atomic Design taxonomy

The Components page is organized using **atomic design methodology**: Sections map to
composition tiers, not functional categories. Build (and reuse) strictly bottom-up — an
Organism is only as good as the Molecules and Atoms it's built from.

| Section        | Node id | Tier definition                                                                                          | Holds (starter examples)                                                                     |
| -------------- | ------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **Atoms**      | `<id>`  | Smallest indivisible UI pieces. Cannot be broken down further without ceasing to be useful on their own.   | Button, Input, Textarea, Checkbox, Chip, Badge, Icon                                            |
| **Molecules**  | `<id>`  | A small, functional group of Atoms working together as one reusable unit.                                  | Select (Input + listbox), Search Bar (Input + Button), Table Cell, Nav Item, Toast, Inline Alert, Empty State, Section Header |
| **Organisms**  | `<id>`  | Distinct, complex sections composed of Molecules and/or Atoms — a recognizable piece of an interface.       | Sidebar, Page Header, Table (full, with cells + header row), Dialog, Card, Bottom Tab Bar        |
| **Templates**  | `<id>`  | Page-level layout skeletons — placement of Organisms/Molecules with placeholder content, no real data.       | List Page Template, Detail Page Template, Form Dialog Template                                  |
| **Sandbox / WIP** | `<id>` | Unfinished work — **never reused in real screens**.                                                        | —                                                                                               |

*(**Pages** — templates filled with real content — are not part of the library; they're what
Page agents build on canvas from instances of the above.)*

*(Sections are a starting taxonomy, not a rule. Rename/merge/add sub-groupings — e.g. an
`Atoms / Icons` sub-section — to fit the product. What must **not** change is the tier order:
nothing in Molecules should reference a component outside Atoms/Molecules; nothing in
Organisms should reference anything above its own tier.)*

---

## Brand & foundations

The design tokens the whole system binds to. `/startup` sets these from your brand (or from
the neutral default in the skill) and creates the matching Figma variables / styles. Record
the decided values here so humans and agents share one reference.

| Foundation                 | Value                                                      |
| --------------------------- | ---------------------------------------------------------- |
| **Brand / primary color**  | `<hex>`                                                    |
| **Primary — dark / hover** | `<hex>`                                                    |
| **Primary — button bg**    | `<hex>`                                                    |
| **Page background**        | `<hex>`                                                    |
| **Surface / card**         | `<hex>`                                                    |
| **Foreground text**        | `<hex>`                                                    |
| **Muted / secondary text** | `<hex>`                                                    |
| **Default border**         | `<hex>`                                                    |
| **Input border**           | `<hex>`                                                    |
| **Heading font family**    | `<e.g. Geist, Inter>`                                      |
| **UI / body font family**  | `<e.g. Inter>`                                             |
| **Spacing scale**          | `<e.g. 2, 4, 6, 8, 12, 16, 24, 32>`                        |
| **Radii**                  | `<Small / Medium / Large / pill — e.g. 6 / 10 / 14 / 999>` |
| **Card shadow ("Soft")**   | `<e.g. 0 10 20 rgba(...) @ 18%>`                           |

**Variable collections in the file** *(names created by `/startup`)*: `<e.g. Brand, TailwindCSS, Mode>` — record the exact names so binding is unambiguous.

*(Foundations sit below Atoms in the hierarchy — they're the tokens Atoms bind to, not a tier
of their own.)*

---

## Status

- [ ] Figma file connected (key + access verified)
- [ ] Foundations created (color / spacing / radii variables, text styles, effect styles)
- [ ] Components page + Atomic Sections created (Atoms, Molecules, Organisms, Templates)
- [ ] Core Atom set built and published
- [ ] Core Molecule set built and published
- [ ] Core Organism set built and published
- [ ] `SKILL.md` inventory seeded
- [ ] First screen (Page) built from instances

When every box is checked, the project is bootstrapped and you run the normal
[daily loop](docs/workflow.md).
