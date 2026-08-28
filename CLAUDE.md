# AnyRoad — Branding Feature

This folder contains the prototype and documentation for AnyRoad's **Branding** feature in the operator dashboard. It covers two sub-features: **Brand Kits** and **Content Profiles**.

## Files

| File | Purpose |
|------|---------|
| `app-prototype.html` | Main working prototype — all feature development happens here |
| `app-tutorial.html` | Tutorial overlay version — sync from prototype after significant changes |
| `system-guide.html` | Full spec and design decisions reference — the source of truth for how and why things work |

## The Two-Layer Mental Model

Every booking page is shaped by two independent layers:

- **Brand Kit** — the visual layer: logo, brand name (wordmark fallback), header color, button color, link color, and three font roles (heading, body, button). Account-level, shared across experiences. Lives in Branding Settings > Brand Kits.
- **Content Profile** — the structural layer: CTA label, field visibility, layout, checkout behavior. Can be shared (account-level) or custom (experience-scoped). Lives in Branding Settings > Content Profiles.

These layers are completely independent. Changing a Brand Kit does not affect Content Profiles and vice versa.

Both editors expose their layer **block by block**: clicking a section of the canvas opens a panel of settings for that block. The two block sets differ by layer, and neither editor shows the other's blocks. The Content Profile editor's blocks edit copy and field visibility; the Brand Kit editor's blocks edit visual treatment, and add Header and Cover, which are not profile settings at all.

Inside the Brand Kit editor there is a second split, between the panel's two states. Ask: **would this differ between two sections of the same page?** A typeface would not, so font family is account-level and lives in the home state. A heading size would, so sizes are block-level. Getting this wrong in either direction is the most common way to break the feature: a per-block typeface lets one kit disagree with itself, and an account-level heading size makes the whole page move when you wanted one section to.

## Key Terminology

- **Brand Kit** — not "brand theme", not "kit" alone in UI copy
- **Content Profile** — not "kit", not "content kit"
- **Custom** — the name for an experience-scoped profile. Never named after the experience (e.g. never "Custom (Painted Ladies)")
- **Shared profile** — a Content Profile that can be assigned to multiple experiences. Account-level. A new account starts with one blank default profile; the operator can create additional shared profiles as needed. Prototype examples like "Classic," "Premium," and "Minimal" are illustrative, not shipped defaults.
- **Branding Settings** — the account-level settings page. Contains Brand Kits tab and Content Profiles tab
- **Design Bar** — the control bar at the top of the Experience Editor Design tab
- **Heading / Body / Button font** — the three type roles a Brand Kit sets. Say "role", not "level" or "slot". Heading covers titles, section labels, prices, and the brand name; button covers every button a guest can press; body covers everything else, and is the default. The CSS role map in `#bpCanvas` is what decides which is which
- **Soleil / Proxima Nova** — two typefaces, two audiences, never interchangeable. **Soleil** (Adobe Typekit) is the platform's *internal* face and dresses this operator dashboard only. **Proxima Nova** is the default on the *public* pages a guest sees, so it is what every canvas previews and what a brand kit starts from. Its stack lives once, in `--bk-platform-sans`. Soleil is never offered as a brand kit font
- **Font source** — where a font comes from: System, Google, Adobe, or Upload. A source is not a setting of its own; it is how the operator finds the typeface they want. **System** leads with Proxima Nova, AnyRoad's application standard, and everything else in it is a face the device already has
- **Block** — one customizable section of a canvas, wrapped in `.cs-section`. Say "block" for the canvas region and "block panel" for the controls it opens. In the Brand Kit editor the panel's other state is the **home state** (the account-level Brand Kit settings)
- No em-dashes in any UI copy or tutorial text

## Architecture Rules — Do Not Break These

1. **One panel element** — `#eeCustomizePanel` serves both Profile Settings (home state) and section editing. Never add a second panel alongside it.
2. **Preview mode controls canvas access** — `#eeCanvas` has the `.preview-mode` class when a shared profile is active. `eeOpenPanel()` is a no-op in this state. Remove `.preview-mode` only when a custom profile is active.
3. **profileConfigs is the source of truth** — `eeInit()` reads `profileConfigs` to restore custom profile state on re-entry, not `eeProfile`. Do not reset `eeProfile.customized` without checking `profileConfigs` first.
4. **Content profile cards are not clickable** — actions are via the `···` menu only. Do not add `onclick` navigation to `.profile-card`.
5. **Design bar pills are neutral gray** — `#f4f5f7` background, not blue. Blue implies selection state.
6. **"Customize for this experience" lives in the profile switcher** — in `#eeSwitcherCustomizeRow`, not in the design bar.
7. **Custom profile naming** — always `'Custom'`, never derived from the experience name.
8. **Topbar layout** — viewport toggle left, hint text right, no "Open full preview" link. Matches account-level profile editor exactly.
9. **Brand kit wordmark** — plain `font-weight: 600` only when no logo. No italic, uppercase, letter-spacing, or decorative treatment, and no font picker of its own. It carries the `bk-wordmark` class so it renders in the kit's **heading font** — the operator's own typeface on their own name is the point; what the rule forbids is decoration for its own sake, not the brand's face. Size is adjustable from the Header block; family, weight, and case are not.
10. **One panel element, in the brand kit editor too** — `#bkPanel` serves both the home state (`#bkPanelHome`, the static account-level settings) and block editing (`#bkPanelBody`). Never add a second panel. `#bkPanelHead` is hidden in the home state and only appears in block state, where the back link is the way out.
11. **Block panels never repeat account-level settings** — a Brand Kit block panel holds only settings that have no account-level equivalent. Colors, logo, brand name, kit name, and font family stay in the home state. If a new setting would apply across blocks, it belongs in the home state, not in a block.
12. **Brand kit block settings are kit-wide** — a block control targets every instance of that block across the canvas pages (all three header bars, every booking button), not just the Experience Details one that was clicked. The block is where you find the setting, not the scope of it. Declare the extra targets in `bkBlockControls`.
13. **Font family is account-level, font size is block-level** — never add a font-family picker to a block panel, and never add a text size to the home state. Family is applied once per role as `--bk-font-heading` / `--bk-font-body` / `--bk-font-button` on `#bpCanvas`; blocks set `fontSize` only, so the two can never collide. A new text element joins a role by being added to the CSS role map, not by any JS change.
14. **A font picker must not lie about what loaded** — `document.fonts.check()` returns `true` for a family the page has never heard of, so it reports success for a font that failed to fetch. Detection goes through `bkFontRendered()`, which compares probe-string metrics against two generics. If a face did not load, say so in the status line rather than showing a fallback silently.

## Making Changes

- Edit `app-prototype.html` first
- After significant changes, sync `app-tutorial.html` by replacing its body with the prototype body and re-attaching the tutorial system block (everything from the `<!-- TUTORIAL SYSTEM -->` comment to end of file)
- Update tutorial step copy if any UI flows, labels, or targets changed
- If a decision is significant, add it to the Design Decisions chapter in `system-guide.html`

## How to Add a Block to a Content Profile Canvas

The content profile editor's canvases use a block-level customization pattern. Each customizable section in a canvas is wrapped in `.cs-section` and shows a hover outline + "Customize" chip; clicking it opens a right-side panel of controls that mutate the canvas live.

The Brand Kit editor uses the same `.cs-section` shell but a different control mechanism — see **How to Add a Block to the Brand Kit Canvas** below. Do not mix the two: `p*` helpers do not work in a brand kit panel, and `bk*` helpers do not work in a profile panel.

### Three steps

**1. Wrap the section in `cs-section`** with a tag and chip:

```html
<div class="cs-section" id="pp-sec-NAME" onclick="profileOpenPanel('NAME')">
  <div class="cs-section-tag">Section Display Name</div>
  <div class="cs-chip"><svg>...</svg> Customize</div>
  <!-- existing content here, with IDs on anything the panel will mutate -->
</div>
```

**2. Add IDs** to the elements the panel will mutate (text nodes whose `textContent` gets rewritten, elements whose visibility toggles, etc.). Convention: `pp-co-*` for checkout-page elements, `pp-*` for details-page elements.

**3. Add a panel config** to `profilePanelConfigs` using the helpers:

```js
'NAME': {
  title: 'Section Display Name', icon: '🎯',
  render: function() { return pGroup('Group label', [
    pText({ label: 'Field name', target: 'pp-co-element-id', fallback: 'Default value' }),
    pToggle({ label: 'Show something', target: 'pp-co-other-id' }),
  ]); }
}
```

### Helper reference

- **`pPanel(groups)`** — joins multiple group HTML strings with `<hr class="pdivider">` between them.
- **`pGroup(label, controls)`** — labeled `.pgroup` wrapper around an array of control HTML strings.
- **`pText({ label?, target, fallback, sublabel?, transform? })`** — text input that rewrites `target.textContent` live. The input is **empty by default**: `fallback` is shown as the placeholder and rendered in the preview, and only typed text becomes an override (clearing the field reverts to the default). `transform: 'upper'` applies `.toUpperCase()` (use for uppercase section headers). Override state round-trips via `data-ov` on the target (the default + transform are stashed on `data-fb` / `data-upper`); the generated handler calls `_setOverride`.
- **`pHeading({ label?, target, fallback, sublabel? })`** — multiline textarea variant of `pText` for a heading block. Same empty-by-default behavior; `fallback` may contain a `{Guest name}` variable (resolved to the sample guest name in the preview). First line renders as a lighter greeting, remaining lines bold. Handler calls `_setHeading`; the default lives on the target's `data-fallback`.
- **`pToggle({ label, target, sublabel?, invert? })`** — checkbox that shows/hides one or more elements via `display:none`. `target` can be a string id or an array. `invert: true` flips the semantics (checked = hidden — use for "Required" or "Disabled" labels).
- **`pToggleAll({ label, group, sublabel? })`** — checkbox that shows/hides **every** element carrying a marker class (`group`, no leading dot) via `_setVisClass`. Use when one setting must control many instances at once — e.g. the booking summary card that repeats on Checkout, Payment, Confirmation, and the previews. Tag each instance's element with the shared class (`sumImg`, `sumHost`, `sumDate`, `sumTime`, `sumGuests` for the summary). `summaryVisGroup()` is the shared control set both summary panels render so they stay identical.
- **`pClassToggle({ label, target, className, sublabel?, invert? })`** — checkbox that adds/removes a CSS class on the target.
- **`pNote(text)`** — a line of explanatory text inside a group, for signposting a setting that lives elsewhere (the block panels use it to point at account-level fonts). Not a control; takes no target.

Render functions can drop to raw template-literal HTML when they need behavior the helpers don't cover. The placeholder "Show price header" toggle in the Booking Widget panel is an example.

### Don't drift

When adding a new panel:
- Use the helpers if your control fits one of the standard patterns. Don't write a new inline `onchange` string from scratch.
- If you need a new control type that doesn't fit, add a new `p*` helper rather than inlining JS in one render function. Future blocks will want the same control.
- The `_setVis`, `_setVisClass`, `_setOverride`, `_setHeading`, `_toggleClass` runtime utilities exist so generated `onchange`/`oninput` strings stay narrow. Don't embed long expressions inline.

## How Brand Kit Typography Works

Font **family** is account-level with exactly three roles; font **size** is block-level. That split is architecture rule 13 and the reason the two never fight each other.

### Applying a family

Both roles are applied as custom properties on the canvas root, once per role:

```js
// bkApplyFonts loops BK_FONT_ROLES, so adding a role is one entry, not new code here
canvas.style.setProperty(role.prop, stack);   // --bk-font-heading | -body | -button
```

Which elements follow which role is decided entirely in CSS, in the role map near the top of the stylesheet:

```css
#bpCanvas { font-family: var(--bk-font-body); }        /* body is the default */
#bpCanvas .bk-title,
#bpCanvas .co-review-title,
#bpCanvas .bk-wordmark { font-family: var(--bk-font-heading); }
#bpCanvas button,
#bpCanvas .bk-cta-btn { font-family: var(--bk-font-button); }
```

**To put a new text element on a role**, add its selector to that map. Do not add a target list to any JS. Body is the default, so only heading and button need listing.

Heading covers titles, section labels, prices, totals, and the wordmark. Button covers every button a guest can press, including the two styled as links rather than `<button>` elements. Body covers everything else: descriptions, form labels, inputs, links.

**To add a role**, add one entry to `BK_FONT_ROLES` (`{ id, label, prop, hint }`), one `:root` default, and its selectors to the role map. `bkApplyFonts`, `bkFontCheckStatus`, `bkFontRenderRoles` and `bkResetFonts` all loop the list, so none of them need touching.

### The four sources

`bkFontCatalog` holds one array per source, and every entry needs `id`, `name`, and `stack`:

| Source | Entry shape | Loading |
|--------|-------------|---------|
| `system` | `{ id, name, stack }`, plus `verify` on a platform-served face | none |
| `google` | `+ g: 'Family+Name:wght@400;700'` | `bkLoadGoogleFont()` injects a `fonts.googleapis.com` link |
| `adobe` | `+ kit: '<kit id>'` | `bkConnectAdobeKit()` injects a `use.typekit.net/<id>.css` link. Starts **empty** — no kit connected |
| `upload` | `+ uploaded: true` | `bkUploadFont()` registers the file via the `FontFace` API |

**To add a source**, add its array to `bkFontCatalog`, an entry to `BK_FONT_SOURCES`, and — only if it needs UI beyond the family list (a kit field, a dropzone) — a branch in `bkFontPickerBody`. Anything that needs fetching before the list can preview itself belongs in `bkFontPreloadSource`.

`system` is the floor: it must render with no Adobe kit, no Google request, and no upload. Its sans entry **is** the platform stack — `stack: 'var(--bk-platform-sans)'`, pointing at the one definition rather than restating it, so the canvases, the role defaults, and the picker can never disagree about what Proxima Nova resolves to. It carries `verify: 'proxima-nova'` because platform-served is not the same as universal: if it is missing, the operator is told. The serif and mono entries have universal fallbacks and stay silent. Do not add a face to `system` that needs fetching.

Do not add Soleil to any source. It is the dashboard's own face; offering it would invite an operator to dress their public pages in AnyRoad's typeface.

### Reporting what actually loaded

`document.fonts.check()` is not usable here: with no matching family it still returns `true`, so it reports success for a font that failed to load. `bkFontRendered(name)` measures a probe string against two generics instead, and `bkFontCheckStatus()` writes the result into the role's status line — green when the face is really there, amber when the page is showing a fallback. Keep that honest; a picker that hides a failed load is worse than one with fewer options.

### Runtime reference

- **`bkFonts`** — `{ heading: { source, id }, body: { source, id } }`, the active selection per role.
- **`bkSetFont(role, source, id)`** — select one role, load if needed, re-render, apply. The roles are set independently; there is no combined control.
- **`bkApplyFonts()`** — write both role variables to the canvas.
- **`bkFontRenderRoles()`** — redraw the two role rows. Each row previews its font **in that font**, which is the whole affordance; a row rendering in the fallback means the face has not loaded.
- **`bkResetFonts()`** — both roles back to the platform default (Proxima Nova). Called from `bkPopulateSettings`, so fonts are per kit like block styles. Uploaded faces stay registered in the browser; only the selection resets.

## How to Add a Block to the Brand Kit Canvas

The Brand Kit editor's canvas (`#bpCanvas`) uses the same `.cs-section` shell, but its controls work differently. A profile control mutates *content* (text, visibility) on one target id. A brand kit control applies a chosen *style value* to every instance of a block, and has to round-trip that choice when the panel re-renders. So instead of describing the control inline, you declare it once in a registry and both the renderer and the applier read it.

### Four steps

**1. Wrap the section in `cs-section`**, same shell as the profile canvas but calling `bkOpenPanel`:

```html
<div class="cs-section" id="bk-sec-NAME" onclick="bkOpenPanel('NAME')">
  <div class="cs-section-tag">Section Display Name</div>
  <div class="cs-chip"><svg>...</svg> Customize</div>
  <!-- existing content here -->
</div>
```

**2. Add IDs** to the elements the controls will style. Convention: `bp*` with a trailing page number, matching the ids already in that canvas (`bpH1`, `bpTitle1`, `bpPrice1`). If the block repeats on Checkout / Payment / Confirmation, give every instance an id and list them all in step 3.

**3. Declare each control in `bkBlockControls`**, keyed `'block.control'`:

```js
'NAME.size': { targets: ['bpThing1'], prop: 'fontSize', def: '14px', options: [
                 { label: 'S', value: '13px' },
                 { label: 'M', value: '14px' },
                 { label: 'L', value: '17px' } ] },
```

- `targets` + `prop` — the simple case: set one style property on each id.
- `apply: function(v)` — replaces `targets`/`prop` when one property can't express it (the header alignment control also clears padding; the two cover-overlay controls compose one `rgba()`).
- `def` — **must match what the markup or CSS already renders**, or the segmented control will show a state the canvas isn't in.
- `type: 'color'` / `type: 'range'` (with `min`, `max`, `step`, `unit`) for the non-segmented controls.

**4. Add a panel config** to `bkPanelConfigs`, reusing `pPanel` / `pGroup` for layout:

```js
'NAME': {
  title: 'Section Display Name',
  render: function() { return pPanel([
    pGroup('Group label', [
      bkSeg({ label: 'Size', control: 'NAME.size' }),
      bkSeg({ label: 'Font', control: 'NAME.font', sublabel: 'Optional hint.' }),
    ])
  ]); }
}
```

### Helper reference (brand kit blocks)

- **`bkSeg({ label, control, sublabel? })`** — segmented picker over the control's declared `options`. Reads the active option back from `bkBlockStyles`, so reopening a panel shows what is actually applied. Options pass their **index** to `bkSetStyle`, not their value, so font stacks full of quotes never have to survive an inline attribute string.
- **`bkColor({ label, control, sublabel? })`** — hex readout + swatch, using the account-level `.bk-color-field` markup so a color control looks the same in either panel state.
- **`bkRange({ label, control, sublabel? })`** — slider with a live readout, for a degree rather than a choice. Reads `min` / `max` / `step` / `unit` from the registry.
- **`bkField(label, controlHtml, sublabel?)`** — the `.pfield` wrapper the three above share. Use it if you add a fourth.

`pPanel` and `pGroup` are shared with the profile panels; everything below them is not.

### Runtime reference

- **`bkBlockStyles`** — the active overrides, keyed `'block.control'`. An absent key means the control's declared default.
- **`bkSetStyle(control, value, el)`** — store and apply one control. Segmented callers pass an option index; color and range callers pass the raw value.
- **`bkApplyStyle(control)`** / **`bkApplyBlockStyles()`** — re-apply one or all. Call `bkApplyStyle` from anything that changes an input a control reads (`bkUpdateLinkColor` does this for the At a Glance icon tint).
- **`bkResetBlockStyles()`** — clear every override and write the defaults back. Called from `bkPopulateSettings`, so entering a kit starts from that kit's baseline instead of the last kit's blocks.
- **`bkOpenPanel(key)`** / **`bkReturnToGlobal()`** — the two panel states. `bkReturnToGlobal` is what `navigate('brand-editor-*')` calls on entry.

### Don't drift

- A new control means a new `bkBlockControls` entry, not new inline JS in a render function. If it needs a control type the three helpers don't cover, add a fourth `bk*` helper.
- Anything that would apply across blocks goes in the home state (architecture rule 11).
- If a click target inside a block does its own thing (a `<select>`, a "More" link), give it `event.stopPropagation()` at the call site so it acts on the preview instead of opening the block panel.

## Inline Decision Comments — Required Behavior

Any time a non-obvious design or architecture decision is made, Claude MUST add a `<!-- DECISION: ... -->` comment in HTML/CSS or a `// DECISION: ...` comment in JS, placed immediately before the relevant code. This is not optional. The comment should explain WHY, not just what — future AI and developers need to understand the reasoning, not just the outcome.

Any time existing code is changed or removed, Claude MUST check whether any nearby `DECISION` comment has become stale or misleading, and update or remove it accordingly. A comment that describes old behavior is worse than no comment at all.

## Full Reference

See `system-guide.html` for the complete spec, including the Design Decisions chapter which explains the reasoning behind all major UI choices.
