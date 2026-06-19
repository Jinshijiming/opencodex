# Plan — GUI redesign to match the brand theme

## Problem
`gui/` is 6 React screens of **bare inline styles**: light-only, blue (`#3b82f6`) accent, emoji icons
(⚠️✅❌🔐↑↓▸), no design tokens, no dark mode, no brand. Page title is literally `gui`, favicon is the
old placeholder. It doesn't match the opencodex brand (the `>_` terminal logo, purple `#7c5cff`, the
clean modern docs aesthetic).

## Design Read (mini DESIGN.md)
Reading this as: a **developer control panel** (proxy admin tool) for developers, in a modern
**dark "terminal devtool"** language. Reference: Linear's dark UI + Vercel dashboard, with the
opencodex `>_` brand. A dense, fast, trustworthy tool — not a marketing page.

- **Do's:** off-black canvas, ONE purple accent, monospace for model-ids/urls/code, calm 1px borders,
  subtle elevation, real SVG icons, a left sidebar with the logo.
- **Don'ts:** no emoji icons, no purple-on-white, no oversized hero text, no centered marketing cards,
  no status colors beyond semantic green/red/amber.
- `DESIGN_VARIANCE: 3 · MOTION_INTENSITY: 2 · density: D5 (devtools)`

```yaml
colors:
  bg: "#0b0b0f"      surface: "#15151b"   raised: "#1c1c24"   border: "#2a2a34"
  text: "#e8e8ec"    muted: "#9b9ba6"     accent: "#7c5cff"   accent-soft: "rgba(124,92,255,.14)"
  green: "#34d399"   red: "#f87171"       amber: "#fbbf24"
typography:
  ui: system sans;   mono: ui-monospace (model ids, urls, code)
```
Dark-first (the terminal brand reads dark; light mode = a later round). All colors as CSS custom
properties so it's theme-extensible.

## File change map
### NEW (design system)
- `gui/src/styles.css` — token vars + reset + base + component classes (`.btn`, `.card`, `.panel`,
  `.badge`, `.table`, `.switch`, `.input`, `.nav-item`, `.stat`, `.empty`…). The heart of the redesign.
- `gui/src/icons.tsx` — small inline-SVG icon set (no new dep): check, x, alert, plus, trash, chevron,
  arrowUp/Down, search, github, externalLink, key, lock, + nav icons (grid, server, boxes, bot, list).
- `gui/src/ui.tsx` — shared primitives: `Button`, `Badge`, `Switch`, `StatCard`, `EmptyState`, `PageHeader`.
- `gui/public/logo.svg` — the `>_` mark as SVG (theme-tinted via `currentColor`) for the sidebar + favicon.

### MODIFY (restyle; preserve ALL API wiring + state)
- `gui/src/main.tsx` — import `./styles.css`.
- `gui/index.html` — `<title>opencodex · proxy dashboard</title>`, favicon → logo.svg, font preconnect.
- `gui/src/App.tsx` — branded shell: left **sidebar** (logo + product name + version, nav items w/ icons,
  GitHub link), dark canvas, content area with a max width. Keep the `page` state + apiBase wiring.
- `gui/src/pages/Dashboard.tsx` — stat cards (Status/Version/Uptime/Providers), providers table, models
  grid → restyled; emoji → SVG. Keep `/healthz`, `/api/providers`, `/api/models` + 5s poll + error state.
- `gui/src/pages/Providers.tsx` — OAuth login panel, provider cards, JSON editor, AddProvider button.
  Keep `/api/config` GET/PUT, `/api/oauth/{providers,status,login,logout}`, `/api/providers` DELETE.
- `gui/src/pages/Models.tsx` — collapsible provider groups + switches. Keep `/api/models`, `/api/disabled-models`.
- `gui/src/pages/Subagents.tsx` — featured ≤5 picker + search. Keep `/api/subagent-models` GET/PUT.
- `gui/src/pages/Logs.tsx` — request table + auto-refresh. Keep `/api/logs` + 2s poll.
- `gui/src/components/AddProviderModal.tsx` — restyle modal + preset picker; emoji 🔐🔑 → SVG. Keep
  `/api/providers` POST, `/api/key-providers`, `/api/oauth/*` flow.

## Hard rules (per dev-frontend / dev-uiux-design)
- No emoji as UI icons (STRICT) → inline SVG. Off-black not pure black. Single accent. Semantic colors only.
- A11y: semantic elements, `focus-visible` rings, `aria-label` on icon-only buttons, `prefers-reduced-motion`.
- Preserve every fetch endpoint, payload shape, and state machine exactly — this is a restyle, not a rewrite of behavior.
- Keep deps as-is (react only); no UI framework added.

## Verify (C)
`bun run build` (vite) + `tsc` clean → run dev server → screenshot every screen (dark) for visual
verification → confirm no emoji remain, contrast OK, all screens render.

## Rounds
One PABCD pass for the full dark redesign + design system. Light-mode toggle and motion polish are
candidate follow-up rounds.
