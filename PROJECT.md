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
| **File name**         | `Claude Test — Website`                                                                   |
| **File key**          | `KwsqCVnJzicChzUpwz55cS`                                                                  |
| **URL name segment**  | `Claude-Test---Website`                                                                   |
| **Library page**      | `5:2`                                                                                     |
| **Library page name** | `Components`                                                                              |

**Node-URL format** (used in every `Sources:` list and candidate row): `https://www.figma.com/design/<FILE KEY>/<URL NAME SEGMENT>?node-id=<A>-<B>` (convert a node id `A:B` → `A-B`).

---

## Library page Sections — Atomic Design taxonomy

The Components page is organized using **atomic design methodology**: Sections map to
composition tiers, not functional categories. Build (and reuse) strictly bottom-up — an
Organism is only as good as the Molecules and Atoms it's built from.

| Section        | Node id | Tier definition                                                                                          | Holds (starter examples)                                                                     |
| -------------- | ------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **Atoms**      | `5:3`   | Smallest indivisible UI pieces. Cannot be broken down further without ceasing to be useful on their own.   | Button, Input, Textarea, Checkbox, Chip, Badge, Icon                                            |
| **Molecules**  | `5:4`   | A small, functional group of Atoms working together as one reusable unit.                                  | Select (Input + listbox), Search Bar (Input + Button), Table Cell, Nav Item, Toast, Inline Alert, Empty State, Section Header |
| **Organisms**  | `5:5`   | Distinct, complex sections composed of Molecules and/or Atoms — a recognizable piece of an interface.       | Sidebar, Page Header, Table (full, with cells + header row), Dialog, Card, Bottom Tab Bar        |
| **Templates**  | `5:6`   | Page-level layout skeletons — placement of Organisms/Molecules with placeholder content, no real data.       | List Page Template, Detail Page Template, Form Dialog Template                                  |
| **Sandbox / WIP** | `5:7` | Unfinished work — **never reused in real screens**.                                                        | —                                                                                               |

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
| **Brand / primary color**  | `#2563eb`                                                  |
| **Primary — dark / hover** | `#1d4ed8`                                                  |
| **Primary — button bg**    | `#1e293b`                                                  |
| **Page background**        | `#f8fafc`                                                  |
| **Surface / card**         | `#ffffff`                                                  |
| **Foreground text**        | `#18181b`                                                  |
| **Muted / secondary text** | `#71717a`                                                  |
| **Default border**         | `#e2e8f0`                                                  |
| **Input border**           | `#cbd5e1`                                                  |
| **Heading font family**    | `Inter`                                                    |
| **UI / body font family**  | `Inter`                                                    |
| **Spacing scale**          | `2, 4, 6, 8, 12, 16, 24, 32`                                |
| **Radii**                  | `Small 6 / Medium 10 / Large 14 / Pill 999`                |
| **Card shadow ("Soft")**   | `0 10 20 rgba(209,218,225,1) @ 18%`                        |

**Variable collections in the file** *(names created by `/startup`)*: `Brand, TailwindCSS, Mode` — record the exact names so binding is unambiguous.

**Platform:** multi-platform product — mobile-specific components use the `Mobile ...` prefix right after the tier prefix (e.g. `Atom / Mobile Button`).

**Foundation IDs** *(for binding reference — see `figma-use` variable-patterns for lookup by name)*:

| Item | ID |
| --- | --- |
| Brand collection | `VariableCollectionId:4:2` |
| TailwindCSS collection | `VariableCollectionId:4:3` |
| Mode collection | `VariableCollectionId:4:4` (Light `4:2`, Dark `4:3`) |
| `base/foreground` | `VariableID:4:25` |
| `base/muted-foreground` | `VariableID:4:26` |
| `base/card` | `VariableID:4:27` |
| `base/background` | `VariableID:4:28` |
| `Border-Mid` | `VariableID:4:29` |
| `Border-Dark` | `VariableID:4:30` |
| `Primary` / `Primary-Dark` / `Primary-Light` / `Primary-Button-Bg` / `Page-Background` | `4:20` / `4:21` / `4:22` / `4:23` / `4:24` |
| `spacing/2…32` | `4:31`–`4:38` |
| `Radius/Small,Medium,Large,Pill` | `4:39`–`4:42` |
| Effect style `Soft` | `S:6ba2fc10321c62bc801cdbb5ec8940f43cefbcba,` |
| Effect style `Dialog` | `S:2c87670608c2e88ce3e2be2cc2ca26f67f737d77,` |
| Text styles (heading-md, 2xl-semibold, lg-semibold, sm-semibold, sm-normal, xs-normal) | see `figma.getLocalTextStylesAsync()` by name |

*(Foundations sit below Atoms in the hierarchy — they're the tokens Atoms bind to, not a tier
of their own.)*

---

## Status

- [x] Figma file connected (key + access verified)
- [x] Foundations created (color / spacing / radii variables, text styles, effect styles)
- [x] Components page + Atomic Sections created (Atoms, Molecules, Organisms, Templates)
- [x] Core Atom set built *(publish pending — human action in Figma UI)*
- [x] Core Molecule set built *(publish pending)*
- [x] Core Organism set built *(publish pending)*
- [x] `SKILL.md` inventory seeded
- [ ] First screen (Page) built from instances

When every box is checked, the project is bootstrapped and you run the normal
[daily loop](docs/workflow.md).
