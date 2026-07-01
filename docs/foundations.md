# Foundations Setup — variables, text styles & effects

This is the **how-to** for standing up the design tokens: the color variables, spacing variables,
radii, shadows, and text styles that every component and screen binds to. It's the detail behind
Step 3 of [`/startup`](../.claude/commands/startup.md).

**Who does what.** *You (the human)* decide the values — the brand color, the fonts, the spacing
rhythm (or accept the neutral default in the [skill](../.claude/skills/figma-design-system/SKILL.md)).
*The Librarian (Claude)* creates them in Figma through the `use_figma` tool, which runs the Figma
Plugin API in the file. The code below is exactly what it runs — you don't have to type it, but it
tells you precisely what's being built, and you can read it back to verify.

> **Order matters.** Build tokens in the order below, and build **all of them before any
> component**. A component created before its variables exist bakes in raw hex/px that you then
> have to retrofit. Foundations first, always.

---

## The token model (what you're building)

Three **variable collections**, each a named group of variables with one or more **modes**:

| Collection | Holds | Modes |
|---|---|---|
| **Brand** | Your brand colors + primitives specific to the product | one (or Light/Dark) |
| **TailwindCSS** | A broad, reusable palette (grays + semantic scales like `green/500`) | one |
| **Mode** | Semantic `base/*` tokens (`base/foreground`, `base/card`, …) that *alias* the palette | Light / Dark |

Plus **spacing** and **radii** as float variables (put them in Brand, or a small `Scale` collection),
**effect styles** (the card shadow), and **text styles** (the type scale). Styles are file-level, not
part of a collection.

**Primitive vs semantic.** Palette entries like `green/500 = #22c55e` are *primitives* (raw values).
Semantic tokens like `base/foreground` don't hold a hex — they **alias** a primitive, and the alias
can differ per mode (Light → `zinc/900`, Dark → `zinc/50`). Bind components to **semantic** tokens
where one exists, so a theme change is a token edit, not a rebuild.

---

## 1. Collections & modes

```js
// One collection per group. The first mode already exists; rename it, add others as needed.
const brand = figma.variables.createVariableCollection("Brand");
const tw    = figma.variables.createVariableCollection("TailwindCSS");
const mode  = figma.variables.createVariableCollection("Mode");

// Modes: rename the default, add a second for theming.
const lightId = mode.modes[0].modeId;
mode.renameMode(lightId, "Light");
const darkId  = mode.addMode("Dark");   // omit if you don't need dark mode yet
```

Record the collection names you chose in [`PROJECT.md`](../PROJECT.md) so binding is unambiguous.

---

## 2. Color variables

Colors are set as `{ r, g, b }` (and optional `a`) in the **0–1** range, not hex. Convert first:

```js
// hex "#1e293b" -> {r,g,b} in 0..1   (Figma also offers figma.util.rgb("#1e293b"))
function rgb(hex) {
  hex = hex.replace('#','');
  return {
    r: parseInt(hex.slice(0,2),16)/255,
    g: parseInt(hex.slice(2,4),16)/255,
    b: parseInt(hex.slice(4,6),16)/255,
  };
}

// Create a primitive color variable and set it for a mode.
function color(name, collection, modeId, hex) {
  const v = figma.variables.createVariable(name, collection, "COLOR");
  v.setValueForMode(modeId, rgb(hex));
  return v;
}

// --- Palette primitives (TailwindCSS collection) ---
const twMode = tw.modes[0].modeId;
const zinc900 = color("zinc/900", tw, twMode, "#18181b");
const zinc500 = color("zinc/500", tw, twMode, "#71717a");
const zinc200 = color("zinc/200", tw, twMode, "#e4e4e7");
const white   = color("base/white", tw, twMode, "#ffffff");
const green700= color("green/700", tw, twMode, "#15803d");
const red600  = color("red/600",   tw, twMode, "#dc2626");
// ...add the rest of the scales you need (see the skill's default palette).

// --- Brand primitives (Brand collection) ---
const brandMode = brand.modes[0].modeId;
const primary   = color("Primary",      brand, brandMode, "#2563eb"); // <- your real brand color
const btnBg     = color("Primary-Dark", brand, brandMode, "#1e293b");
```

### Semantic tokens that alias primitives (the Mode collection)

```js
// A semantic var holds an ALIAS, and can point at different primitives per mode.
function semantic(name, aliasLight, aliasDark) {
  const v = figma.variables.createVariable(name, mode, "COLOR");
  v.setValueForMode(lightId, figma.variables.createVariableAlias(aliasLight));
  if (typeof darkId !== "undefined")
    v.setValueForMode(darkId, figma.variables.createVariableAlias(aliasDark || aliasLight));
  return v;
}

const fg   = semantic("base/foreground", zinc900, /*dark*/ white);
const muted = semantic("base/muted-foreground", zinc500, zinc500);
const card = semantic("base/card", white, zinc900);
const border = semantic("Border-Mid", zinc200, zinc500);
```

Naming with **slashes** (`base/foreground`, `green/700`) makes Figma group them into folders in the
Local variables panel — keep the family prefixes consistent.

---

## 3. Spacing variables (float, `GAP` scope)

Spacing/gap values are **FLOAT** variables. Set their **scope** to `GAP` (optionally
`WIDTH_HEIGHT`) so Figma only offers them where padding/gaps apply — this keeps the binding UI clean.

```js
function spacing(n) {
  const v = figma.variables.createVariable("spacing/" + n, brand, "FLOAT");
  v.setValueForMode(brandMode, n);       // value = name, e.g. spacing/8 = 8
  v.scopes = ["GAP", "WIDTH_HEIGHT"];     // where it can be applied
  return v;
}
[2,4,6,8,12,16,24,32].forEach(spacing);
```

Bind padding and item-spacing to these on every auto-layout frame — never type a raw px padding.

---

## 4. Radii variables (float, `CORNER_RADIUS` scope)

```js
function radius(name, value) {
  const v = figma.variables.createVariable("Radius/" + name, brand, "FLOAT");
  v.setValueForMode(brandMode, value);
  v.scopes = ["CORNER_RADIUS"];
  return v;
}
radius("Small", 6);    // buttons, inputs, square badges
radius("Medium", 10);  // cards
radius("Large", 14);   // mobile cards
radius("Pill", 999);   // tags/chips
```

---

## 5. Effect styles (the card shadow)

Shadows live as **effect styles**, not variables. Create a named `Soft` style so cards bind a style,
not a hand-rolled shadow.

```js
const soft = figma.createEffectStyle();
soft.name = "Soft";
soft.effects = [{
  type: "DROP_SHADOW",
  color: { r: 0.82, g: 0.855, b: 0.882, a: 0.18 }, // ~#D1DAE1 @ 18%
  offset: { x: 0, y: 10 },
  radius: 20,
  spread: 0,
  visible: true,
  blendMode: "NORMAL",
}];

// A deeper dialog shadow, optional:
const dialogShadow = figma.createEffectStyle();
dialogShadow.name = "Dialog";
dialogShadow.effects = [{ type:"DROP_SHADOW", color:{r:0.06,g:0.09,b:0.16,a:0.16}, offset:{x:0,y:16}, radius:24, spread:0, visible:true, blendMode:"NORMAL" }];
```

Apply with `await node.setEffectStyleIdAsync(soft.id)`.

---

## 6. Text styles (the type scale)

Type is applied as **text styles**, never raw `fontName`/`fontSize`. **Load the font first** — this
is the #1 failure point (see gotchas).

```js
async function textStyle(name, family, weight, size, lineHeight) {
  await figma.loadFontAsync({ family, style: weight });   // MUST load before use
  const s = figma.createTextStyle();
  s.name = name;                                           // slash-grouped
  s.fontName = { family, style: weight };
  s.fontSize = size;
  s.lineHeight = { unit: "PIXELS", value: lineHeight };
  return s;
}

await textStyle("custom/heading-md",                 "Inter", "Bold",      30, 36);
await textStyle("text-2xl/leading-8/font-semibold",  "Inter", "Semi Bold", 24, 32);
await textStyle("text-lg/leading-none/font-semibold","Inter", "Semi Bold", 18, 18);
await textStyle("text-sm/leading-5/font-semibold",   "Inter", "Semi Bold", 14, 20);
await textStyle("text-sm/leading-5/font-normal",     "Inter", "Regular",   14, 20);
await textStyle("text-xs/leading-4/font-normal",     "Inter", "Regular",   12, 16);
```

Apply with `await node.setTextStyleIdAsync(s.id)`.

---

## Binding cheat-sheet (once the tokens exist)

| What you're styling | How to bind |
|---|---|
| **Fill / stroke color** | build a paint, then `paint = figma.variables.setBoundVariableForPaint(paint, "color", colorVar); node.fills = [paint];` |
| **Padding** | `node.setBoundVariable("paddingLeft", spacingVar)` (also `paddingRight/Top/Bottom`) |
| **Gap between children** | `node.setBoundVariable("itemSpacing", spacingVar)` (and `counterAxisSpacing` for wrap) |
| **Corner radius** | `node.setBoundVariable("topLeftRadius", radiusVar)` (all four corners) |
| **Text style** | `await node.setTextStyleIdAsync(styleId)` |
| **Shadow** | `await node.setEffectStyleIdAsync(effectStyleId)` |

Prefer the **semantic** token when one exists (`base/foreground` over `zinc/900`). Match colors by
exact hex: Brand first, then the Tailwind palette, then nearest.

---

## Verify it (human, in Figma's UI)

After the Librarian reports the foundations are built, sanity-check in Figma:

- Open the file → deselect everything → in the right-hand panel, the **Local variables** collections
  (Brand, TailwindCSS, Mode) should list your colors, `spacing/*`, and `Radius/*`, grouped by their
  slash prefixes. Toggle the **Mode** collection between Light/Dark to confirm semantics flip.
- **Local styles** (text + effects) should list the type scale and the `Soft` shadow.
- Ask the Librarian to bind a throwaway rectangle + text to a couple of tokens and screenshot it,
  then delete it — a fast end-to-end check that binding works before any real component is built.

The values it created are also recorded in [`PROJECT.md`](../PROJECT.md) → confirm that table matches
what you see.

---

## Gotchas

- **Fonts must be available in the file.** `loadFontAsync` fails if the font isn't installed/enabled.
  Add **Inter** (and your heading font, e.g. Geist) to the file before building text styles. For
  **Inter**, the weight string is **`"Semi Bold"`** (with a space), not `"SemiBold"`.
- **Colors are 0–1, sRGB.** `#ffffff` → `{r:1,g:1,b:1}`. Use the `rgb()` helper (or `figma.util.rgb`);
  don't pass 0–255 values.
- **Scopes keep binding sane.** Without `scopes = ["GAP"]` on spacing, every float var shows up in
  every numeric field. Scope spacing to `GAP`/`WIDTH_HEIGHT` and radii to `CORNER_RADIUS`.
- **Semantic ≠ primitive.** Don't give `base/foreground` a hex — alias it to a palette primitive so
  Light/Dark and future rebrands are one edit.
- **One writer.** Only the Librarian creates or edits any of this (single-writer invariant in
  `CLAUDE.md`). Don't run this from a page-agent session.
- **Retrofit is expensive; rebrand is cheap.** Because components bind these tokens, changing the
  brand color later is a value edit. Rebinding *raw values that were never tokenized* is a separate,
  scoped job — which is exactly why you build foundations before components.
