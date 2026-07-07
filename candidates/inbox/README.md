# candidates/inbox — candidate-review drop box

The durable channel from **page agents** to the **Librarian** (see `CLAUDE.md`, candidate
protocol). A session-scoped task list is invisible to other sessions and dies with the
session that made it; a file in git does not.

**Page agent:** after building a page-local candidate, write **one file here** —
`<your-page>--<candidate-name>.md` — with the schema below. This file is the only record
you write; never edit `CANDIDATES.md`. Each agent writes only its **own** inbox files, so
the single-writer rule holds per file.

**Librarian:** on seeing a file here, add a `pending-review` row to `CANDIDATES.md`, then
delete the file — the register row supersedes it. `/reconcile` sweeps this directory (and
the Figma file for unregistered `CANDIDATE — *` frames) on every pass.

Schema (one file per candidate):

```
candidate-review
name:        CANDIDATE — <family> / <name>
raised_by:   <agent> · <page>
gap:         <what existing component was missing / why an instance didn't fit>
node:        <figma node link to the candidate>
built_from:  <instances/components reused, e.g. "Input + Badge">
new_values:  <none | list each raw value not bound to a variable>
```

This README is the only permanent file in the directory — anything else here is an
unprocessed candidate awaiting the Librarian.
