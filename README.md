# AnyRoad — Branding Settings

Prototype, guided walkthrough, and system documentation for AnyRoad's **Branding** feature in the operator dashboard. The feature covers two sub-features — **Brand Kits** (the visual layer) and **Content Profiles** (the structural layer of the booking flow).

## Viewing

Open `index.html` in a browser to land on a page with links to all three views.

| File | Purpose |
| ---- | ------- |
| `app-prototype.html` | The working prototype — free exploration of every flow |
| `app-tutorial.html` | The same prototype with a step-by-step guided walkthrough |
| `system-guide.html` | Full spec, developer reference, and Design Decisions chapter |

Every HTML file is self-contained with no external dependencies, so the repo can be hosted as static files anywhere — GitHub Pages, Vercel, Netlify, or any static web server.

## The two-layer model

Every booking page is shaped by two independent layers.

**Brand Kit** is the visual layer: logo, brand name (the wordmark fallback shown when no logo is uploaded), header color, button color, and link color. Brand Kits live at the account level and can be shared across experiences.

**Content Profile** is the structural layer: CTA label, field visibility, layout, and checkout behavior. A profile can be shared across multiple experiences (account-level) or scoped to a single experience (custom). A new account starts with one blank default profile that the operator names and configures.

The two layers are independent. Changing a Brand Kit does not affect any Content Profile, and vice versa. See `system-guide.html` for the complete spec — including the customization flow, the state machine, and the Design Decisions chapter explaining the reasoning behind every major UI choice.

## Notes for contributors

`CLAUDE.md` contains project instructions for AI agents working on the prototype, including architecture rules and naming conventions. If you're editing the files directly, it's worth reading for context.

This is an internal design preview for review and iteration, not production code.
