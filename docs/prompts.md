# Prompts for Brand & Foundations Setup

`/startup` is **interactive** — at Step 2 it asks you for the brand, and at Step 3 it builds the
foundations from what you gave it. The quality of everything downstream (every component, every
screen) rests on those two moments. This doc gives you **copy-paste prompts** for the common
situations, worked **examples**, and — just as important — **what each piece of information is and
why the system needs it**, so you can give good answers instead of guessing.

Read alongside [foundations.md](foundations.md) (the mechanics of what gets built) and the
[startup guide](startup.md) (the overall flow).

> **Where these go.** You type these in your **Librarian** session, either as the argument to
> `/startup` or as replies when it asks. You don't have to front-load everything — a one-line
> `/startup <url>` is fine and it'll interview you. These prompts just let you be precise and skip
> the back-and-forth.

---

## The mental model: why it asks before it builds

Foundations are **tokens** — named variables and styles that every component binds to. Because
components bind them, the tokens are decided **once, up front**, and changing a token later
re-flows everywhere automatically. That's the payoff — but it means the setup needs a few
decisions before it can build anything:

- **Colors** → so every fill/stroke/text binds to a variable, never a raw hex.
- **Fonts** → so text uses **text styles**, not raw font settings (and the fonts must exist in the
  file before a style can be made).
- **Spacing & radii** → so padding/gaps/corners bind to a scale, giving consistent rhythm.
- **Shadow** → so cards share one named effect instead of hand-rolled shadows.

Give it these and it can build a clean, fully-bound system. Give it nothing and it uses a sensible
**neutral default** — which is fine, and you can rebrand later cheaply (a token edit, not a rebuild).

---

## Pick your situation

### A. You have a full brand spec
Best case — hand it everything and let it confirm before writing.

```
/startup https://www.figma.com/design/ABC123/My-Product

Here's our brand. Set the foundations from this, show me the plan before you touch Figma:

- Primary / brand color: #2F6BFF
- Primary dark (hover/emphasis): #1E4FD0
- Primary button background: #0B1B3F   (near-navy; buttons are dark, not bright blue)
- Page background: #F7F8FA
- Card / surface: #FFFFFF
- Text (foreground): #101828
- Muted / secondary text: #667085
- Default border: #E4E7EC   ·   Input border: #CBD2DC
- Semantic: success #067647, danger #D92D20, warning #B54708, info #175CD3
- Heading font: Geist   ·   UI/body font: Inter
- Spacing scale: 2, 4, 6, 8, 12, 16, 24, 32
- Radii: small 6, medium 10, large 14, pill 999
- Card shadow "Soft": 0 10 20 rgba(16,24,40,0.08)
- Single platform (web app only — no mobile prefix)
- Include a Dark mode
```

### B. You have a logo or reference screenshot (extract the brand)
Let it read the colors and fonts off an image, then confirm with you.

```
/startup https://www.figma.com/design/ABC123/My-Product

I've attached our logo and a screenshot of our marketing site. Pull the brand palette and
fonts from these — primary color, a dark shade for buttons, neutrals, and the type families.
Propose the full foundations table (fill any gaps with sensible neutrals), show me the values,
and wait for my go before building.
```
*(Attach the image(s) in the same message.)*

### C. You have just a color and fonts (fill the rest from the default)
The most common real case.

```
/startup https://www.figma.com/design/ABC123/My-Product

Brand basics: primary is #7C3AED, headings in Geist, body in Inter. Everything else — neutrals,
spacing, radii, shadow, semantics — use the neutral default from the skill. Buttons should use a
dark neutral background, not the purple. Show me the resolved table before writing.
```

### D. You have nothing yet (accept the neutral default)
Get a working system now; rebrand later.

```
/startup https://www.figma.com/design/ABC123/My-Product

No brand yet — use the neutral default foundations as-is. I'll swap the brand color and fonts
later. Just confirm what you're about to create, then build it.
```

---

## What each input is, and why it's needed

| Input | What it is | Why the system needs it | If you skip it |
|---|---|---|---|
| **Primary / brand color** | Your accent — links, focus rings, active states, selected tints | The one color that signals "brand"; used sparingly for emphasis | Default blue `#2563eb` |
| **Primary — dark** | A darker shade of the brand | Hover/pressed/emphasis states and gradients | Derived darker shade |
| **Primary — button background** | The fill for the main button | Often **not** the bright brand color — many systems use a near-black/navy so buttons read as "primary action" without shouting. Kept separate so you decide deliberately | Dark neutral |
| **Page background** | The canvas behind cards | The base layer; everything else sits on it | `#f8fafc` |
| **Card / surface** | Fill of cards, dialogs, inputs | The "raised" layer; usually white | `#ffffff` |
| **Foreground text** | Primary text color | Bound to `base/foreground`; the default for headings/body | `#18181b` |
| **Muted / secondary text** | Dimmer text | Labels, captions, table headers, placeholders | `#71717a` |
| **Default border** | Card/row/divider lines | The quiet 1px structure lines | `#e2e8f0` |
| **Input border** | Field outlines | Usually a touch darker than default borders so inputs read as interactive | `#cbd5e1` |
| **Semantic accents** | success / danger / warning / info | Status badges, toasts, inline alerts, destructive buttons, validation | Tailwind green/red/amber/blue |
| **Heading font** | Family for big titles | Text styles are made per font; the font must be **enabled in the Figma file** first | Inter (or your body font) |
| **UI / body font** | Family for everything else | Same — decided up front so the type scale can be built | Inter |
| **Spacing scale** | The set of padding/gap steps | Padding/gaps bind to these floats → consistent rhythm, no raw px | `2,4,6,8,12,16,24,32` |
| **Radii** | Corner sizes (small/med/large/pill) | Corners bind to these → consistent roundness | `6 / 10 / 14 / 999` |
| **Card shadow** | The "Soft" elevation | Cards bind one named effect style instead of ad-hoc shadows | `0 10 20` neutral @ ~18% |
| **Platform(s)** | Web only, or also mobile? | Decides whether components get a `Mobile ...` prefix (mobile ≠ desktop pieces) | Single-surface (no prefix) |
| **Dark mode?** | One theme or Light + Dark | Adds a second **mode** so semantic tokens flip per theme | Light only (can add later) |

**The three that matter most to get right up front:**
1. **Fonts** — because a text style can't be created for a font Figma can't load, and re-typing all
   text later is painful. Decide these before Step 3. (Add them to the file first — see
   [foundations.md](foundations.md#gotchas).)
2. **Spacing scale** — everything binds to it; an odd scale is felt in every screen.
3. **Whether you need dark mode** — cheap to include now (semantic tokens alias per mode), more work
   to retrofit once components exist.

Everything else is safe to take the default and refine later.

---

## Prompts for the build step (Step 3)

Once the values are agreed, these steer *how* it builds and let you verify:

**Confirm the plan before any write** (it should do this anyway — this reinforces it):
```
Before writing anything to Figma: list the exact collections, every variable with its value,
the text styles, and the Soft shadow you'll create. I'll approve, then you build.
```

**Ask for the collection structure you want:**
```
Use three collections: Brand (brand + spacing + radii), TailwindCSS (the full palette),
and Mode (semantic base/* tokens that alias the palette, with Light and Dark modes).
```

**Make semantics alias primitives (not hold raw hex):**
```
base/* tokens must alias palette primitives per mode, not store their own hex — so a rebrand or
theme switch is one edit. Show me which primitive each semantic points to in Light vs Dark.
```

**Verify before moving on to components:**
```
Build a throwaway rectangle + a line of text, bind them to base/card, base/foreground, a
spacing/* padding, and the Soft shadow, screenshot it, then delete it — I want to see binding
works end-to-end before we build any real component.
```

**Record the results:**
```
Write the final values into PROJECT.md's brand table and tick the foundations checkbox.
```

---

## Good vs vague — a quick calibration

| Vague (forces guessing) | Precise (builds correctly) |
|---|---|
| "Make it look modern and clean." | "Neutral grays, one blue accent `#2F6BFF`, generous spacing, radius 8–10." |
| "Use our brand colors." | "Primary `#2F6BFF`, buttons `#0B1B3F`, text `#101828`, borders `#E4E7EC`." |
| "Pick some nice fonts." | "Headings Geist, body Inter — both are already enabled in the file." |
| "Set up the usual spacing." | "Spacing `2,4,6,8,12,16,24,32`; radii small 6 / med 10 / large 14 / pill 999." |
| "Add colors for statuses." | "success `#067647`, danger `#D92D20`, warning `#B54708`, info `#175CD3`." |

Precise answers don't have to be *complete* — say what you know, and explicitly hand the rest to the
neutral default: *"…everything else, use the default."* That single sentence removes all the
guessing.

---

## After it's built

- Confirm the values landed in [`PROJECT.md`](../PROJECT.md) and in Figma's **Local variables**
  panel (see [foundations.md — Verify it](foundations.md#verify-it-human-in-figmas-ui)).
- Only now move on to components (`/startup` Step 5). Foundations before components, always.
- Changed your mind on the brand color or a font later? Just tell the Librarian to update that one
  variable/text style — because everything binds to it, the change re-flows. Rebrand is cheap;
  retrofitting raw values is not.
