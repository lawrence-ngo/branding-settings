# AnyRoad — Branding Settings

Prototype, guided walkthrough, and system documentation for AnyRoad's **Branding** feature in the operator dashboard. The feature covers two sub-features — **Brand Kits** (the visual layer) and **Content Profiles** (the structural layer of the booking flow).

## Viewing

Open `index.html` in a browser to land on a page with links to all three views.

| File | Purpose |
| ---- | ------- |
| `app-prototype.html` | The working prototype — free exploration of every flow |
| `app-tutorial.html` | The same prototype with a step-by-step guided walkthrough |
| `system-guide.html` | Full spec, developer reference, and Design Decisions chapter |

Every HTML file is a single self-contained document with no build step, so the repo can be hosted as static files anywhere — GitHub Pages, Vercel, Netlify, or any static web server.

The only network requests are for fonts: Adobe Typekit for the dashboard's own typeface (Soleil) and for Proxima Nova, the default on the public-facing pages the canvases preview; plus Google Fonts, loaded on demand when a brand kit's font picker selects one. Everything works offline apart from those faces, and the font picker says so rather than quietly showing a fallback.

## The two-layer model

Every booking page is shaped by two independent layers.

**Brand Kit** is the visual layer: logo, brand name (the wordmark fallback shown when no logo is uploaded), header color, button color, link color, and three font roles (heading, body, button). Brand Kits live at the account level and can be shared across experiences. Within a kit, colors and fonts are account-wide, while text sizes and per-section treatment are set block by block on the editor canvas.

**Content Profile** is the structural layer: CTA label, field visibility, layout, and checkout behavior. A profile can be shared across multiple experiences (account-level) or scoped to a single experience (custom). A new account starts with one blank default profile that the operator names and configures.

The two layers are independent. Changing a Brand Kit does not affect any Content Profile, and vice versa. See `system-guide.html` for the complete spec — including the customization flow, the state machine, and the Design Decisions chapter explaining the reasoning behind every major UI choice.

## Notes for contributors

`CLAUDE.md` contains project instructions for AI agents working on the prototype, including architecture rules and naming conventions. If you're editing the files directly, it's worth reading for context.

This is an internal design preview for review and iteration, not production code.
