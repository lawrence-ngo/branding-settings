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

- **Brand Kit** — the visual layer: logo, brand name (wordmark fallback), header color, button color, link color. Account-level, shared across experiences. Lives in Branding Settings > Brand Kits.
- **Content Profile** — the structural layer: CTA label, field visibility, layout, checkout behavior. Can be shared (account-level) or custom (experience-scoped). Lives in Branding Settings > Content Profiles.

These layers are completely independent. Changing a Brand Kit does not affect Content Profiles and vice versa.

## Key Terminology

- **Brand Kit** — not "brand theme", not "kit" alone in UI copy
- **Content Profile** — not "kit", not "content kit"
- **Custom** — the name for an experience-scoped profile. Never named after the experience (e.g. never "Custom (Painted Ladies)")
- **Shared profile** — a Content Profile that can be assigned to multiple experiences. Account-level. A new account starts with one blank default profile; the operator can create additional shared profiles as needed. Prototype examples like "Classic," "Premium," and "Minimal" are illustrative, not shipped defaults.
- **Branding Settings** — the account-level settings page. Contains Brand Kits tab and Content Profiles tab
- **Design Bar** — the control bar at the top of the Experience Editor Design tab
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
9. **Brand kit wordmark** — plain `font-weight: 600` only when no logo. No italic, uppercase, letter-spacing, or decorative fonts.

## Making Changes

- Edit `app-prototype.html` first
- After significant changes, sync `app-tutorial.html` by replacing its body with the prototype body and re-attaching the tutorial system block (everything from the `<!-- TUTORIAL SYSTEM -->` comment to end of file)
- Update tutorial step copy if any UI flows, labels, or targets changed
- If a decision is significant, add it to the Design Decisions chapter in `system-guide.html`

## How to Add a Customizable Block to a Canvas

The content profile editor's canvases use a block-level customization pattern. Each customizable section in a canvas is wrapped in `.cs-section` and shows a hover outline + "Customize" chip; clicking it opens a right-side panel of controls that mutate the canvas live.

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
- **`pClassToggle({ label, target, className, sublabel?, invert? })`** — checkbox that adds/removes a CSS class on the target.

Render functions can drop to raw template-literal HTML when they need behavior the helpers don't cover. The placeholder "Show price header" toggle in the Booking Widget panel is an example.

### Don't drift

When adding a new panel:
- Use the helpers if your control fits one of the standard patterns. Don't write a new inline `onchange` string from scratch.
- If you need a new control type that doesn't fit, add a new `p*` helper rather than inlining JS in one render function. Future blocks will want the same control.
- The `_setVis`, `_setText`, `_setOverride`, `_setHeading`, `_toggleClass` runtime utilities exist so generated `onchange`/`oninput` strings stay narrow. Don't embed long expressions inline.

## Inline Decision Comments — Required Behavior

Any time a non-obvious design or architecture decision is made, Claude MUST add a `<!-- DECISION: ... -->` comment in HTML/CSS or a `// DECISION: ...` comment in JS, placed immediately before the relevant code. This is not optional. The comment should explain WHY, not just what — future AI and developers need to understand the reasoning, not just the outcome.

Any time existing code is changed or removed, Claude MUST check whether any nearby `DECISION` comment has become stale or misleading, and update or remove it accordingly. A comment that describes old behavior is worse than no comment at all.

## Full Reference

See `system-guide.html` for the complete spec, including the Design Decisions chapter which explains the reasoning behind all major UI choices.
