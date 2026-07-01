# Figma Design-System Agent Team — Generic Template

A Claude Code project that coordinates a multi-agent team building a **design system in any Figma
file — from scratch**. It's the reusable, un-branded version of a proven setup: point it at an
**empty Figma file**, run **`/startup`**, and one **Librarian** stands up the foundations
(variables, text styles, effect styles) and a core component library; then one or more **Page
agents** build screens from instances of that library and raise *candidates* whenever a primitive is
missing. A human decides whether each candidate is promoted into the library or kept as a one-off,
and `/reconcile` keeps the records in sync with the live file.

This template ships with **nothing project-specific baked in** — no file key, no components, no
candidates. All of that is filled in during startup. The team rules are in [CLAUDE.md](CLAUDE.md)
(source of truth); everything file-specific lives in [PROJECT.md](PROJECT.md); how to build and
style is in the [`figma-design-system` skill](.claude/skills/figma-design-system/SKILL.md).

## Who this is for

Starting a **new** Figma project with Claude and you have no design system yet — no components, no
tokens, no candidates set up. You bring a Figma file (and, optionally, a brand); the team sets
everything up. If you already have a mature library, you'd adapt the branded original instead — this
template is specifically the greenfield starting point.

## Repository layout

```
.
├── CLAUDE.md                 # Team coordination rules — source of truth
├── PROJECT.md                # File-specific config (file key, library page, Sections, brand) — filled by /startup
├── CANDIDATES.md             # Candidate register — starts empty
├── README.md                 # This file
├── docs/
│   ├── startup.md            # Day-zero guide: bootstrapping an empty file
│   └── workflow.md           # Daily loop: roles, candidate lifecycle, reconcile, commit
└── .claude/
    ├── commands/
    │   ├── startup.md        # /startup — one-time greenfield bootstrap (Librarian)
    │   ├── librarian.md      # /librarian — sole writer of file-global state + records
    │   ├── page-agent.md     # /page-agent — page-local; consume the library, raise candidates
    │   ├── reviewer.md       # /reviewer — read-only audit of finished work
    │   ├── reconcile.md      # /reconcile — Librarian-only records↔library sync
    │   └── excalidraw-diagram.md
    └── skills/
        └── figma-design-system/
            └── SKILL.md      # How to build & style; neutral default foundations; living inventory
```

## Setup

### Requirements

- **Claude Code** with the **Figma MCP** connected (for `use_figma`, `get_metadata`,
  `get_screenshot`, etc.), signed in to an account that can edit your file.
- **A recent Opus model recommended** for the Librarian / orchestration role.
- **Git** — `PROJECT.md`, `CANDIDATES.md`, and the `SKILL.md` inventory are the durable records the
  reconciliation pass reads, so keep them version-controlled. This repo is already initialized.
- *(Optional)* **Agent teams**, once you regularly run 3+ agents at once:
  ```bash
  export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
  ```
  With the flag on, every session has one implicit team — you spawn teammates in natural language and
  they coordinate through a shared task list + `SendMessage`. It's experimental (no session
  resumption with in-process teammates, task-status lag, one team per session). Overkill for one or
  two agents.

### 1. Bootstrap the file (once)

Open Claude Code in this directory, tell the session it is the **Librarian**, and run:

```
/startup <your Figma file URL>
```

It connects the file, interviews you for the brand (or accepts a neutral default), builds the
foundations → the `Components` page + Sections → the core component set, seeds the records, and
publishes the library — **gating at every write**. Full walkthrough: [docs/startup.md](docs/startup.md).

### 2. Build screens (the daily loop)

Once bootstrapped, spin up a page agent per screen:

```
/page-agent <first screen name>
```

It builds from instances of the library and raises a **candidate** for anything missing. You decide
promote / one-off; the Librarian builds promotions. Full loop: [docs/workflow.md](docs/workflow.md).

### 3. Keep records in sync (Librarian only)

```
/reconcile
```

Compares the live library page against `SKILL.md` and `CANDIDATES.md`, builds any `promote`-status
candidates, documents undocumented components, fixes drift — **never** re-flags a `one-off`, **never**
waits on `pending-review`. Idempotent; safe to run repeatedly.

## How it fits together

- **Build & style rules + starter tokens:** the
  [`figma-design-system` skill](.claude/skills/figma-design-system/SKILL.md) — foundations-first
  order, naming conventions, recipes, a neutral default design system, and a living component
  inventory.
- **Team coordination rules:** [CLAUDE.md](CLAUDE.md) — who may write what, the single-writer
  invariant, the candidate protocol, and reconciliation.
- **File-specific config:** [PROJECT.md](PROJECT.md) — the one place file key, library page,
  Sections, and brand are recorded. Everything else references it.
- **Human guides:** [docs/startup.md](docs/startup.md) (day zero) and
  [docs/workflow.md](docs/workflow.md) (daily).
