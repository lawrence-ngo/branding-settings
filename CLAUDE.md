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

## Inline Decision Comments — Required Behavior

Any time a non-obvious design or architecture decision is made, Claude MUST add a `<!-- DECISION: ... -->` comment in HTML/CSS or a `// DECISION: ...` comment in JS, placed immediately before the relevant code. This is not optional. The comment should explain WHY, not just what — future AI and developers need to understand the reasoning, not just the outcome.

Any time existing code is changed or removed, Claude MUST check whether any nearby `DECISION` comment has become stale or misleading, and update or remove it accordingly. A comment that describes old behavior is worse than no comment at all.

## Full Reference

See `system-guide.html` for the complete spec, including the Design Decisions chapter which explains the reasoning behind all major UI choices.
