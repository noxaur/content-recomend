# Handoff: Migrate ContentRecomend to Next.js

**Date**: 2026-05-20
**Source**: Single-file HTML prototype at `index.html` (1398 lines)
**Target**: Next.js application with App Router
**Status**: Frontend prototype complete, production-grade design, needs framework migration

---

## What This Is

ContentRecomend is a content recommendation app — connect YouTube/Netflix/Spotify/Twitch, bookmark content, share with friends. The current frontend is a single HTML file with zero dependencies that needs to become a proper Next.js app.

---

## Source File

`index.html` — 1398 lines, self-contained:
- CSS: ~900 lines (design tokens, components, responsive, theming)
- HTML: ~300 lines (4 pages, semantic markup, ARIA)
- JS: ~100 lines (client-side routing, theme toggle, keyboard handlers)

**Quality**: 19/20 impeccable audit score, 29/29 contrast ratios pass WCAG AA, zero anti-patterns.

---

## Pages to Extract

The prototype has 4 pages currently switched via `showPage()` client-side routing. Each becomes a Next.js route:

### 1. Landing (`/`)
- Split hero: text left, play frame right
- 3 feature tiles (mint tile, card, uv tile)
- CTA button → `/account`
- File: `app/page.tsx`

### 2. Account (`/account`)
- Profile header with avatar, username, stats
- Platform connection cards (YouTube, Netflix, Spotify) with toggles
- Recently saved stream (StoryStream rail)
- File: `app/account/page.tsx`

### 3. Scroll (`/scroll`)
- Split view: feed list (left) + detail panel (right)
- Selectable items with border highlight
- Detail panel: thumbnail, title, description, badge, meta, Save/Share/+ buttons
- File: `app/scroll/page.tsx`

### 4. Friends (`/friends`)
- Activity feed with posts
- Each post: avatar, username, timestamp, platform badge, thumbnail, text, action buttons (like/comment/share/save), comments
- File: `app/friends/page.tsx`

---

## Component Extraction

Extract these reusable components from the single-file CSS/HTML:

### Layout
- `TopNav` — Sticky top nav with wordmark, account button, theme toggle, avatar
- `PageNav` — Sticky bottom nav with page switcher buttons (prototype-only, may become sidebar/tab nav in production)
- `Container` — Max-width wrapper with responsive padding

### Typography
- `Wordmark` — Anton display, hover → deep-link-blue
- `Display` / `DisplaySm` — Anton headlines with clamp() sizing
- `Whisper` — Light uppercase body text, mint accent
- `Kicker` — Mono uppercase section labels, mint accent
- `MonoLabel` / `MonoLabelSm` — Mono uppercase metadata
- `Headline` — Space Grotesk bold, hover → deep-link-blue
- `Deck` — Secondary text, 16px

### UI Components
- `BtnMint` — Primary CTA, pill shape, mint bg
- `BtnSlate` — Secondary button, slate bg
- `BtnOutlined` — Ghost button, mint border (dark) / dark mint border (light)
- `BtnOutlinedUv` — Ghost button, ultraviolet border
- `Card` — Elevated card with 1px border, 20px radius
- `CardSlate` — Slate background card
- `TileMint` / `TileUv` / `TileWhite` — Color-block feature tiles
- `Thumb` — 16:9 thumbnail placeholder
- `Avatar` — Circular avatar (sizes: xs, sm, default, lg)
- `Badge` — Platform badge (YT/NF/SP/TW with brand colors)
- `Tag` — Filter tag with active state
- `Input` — Text input with focus state
- `Toggle` — Switch toggle (role="switch", aria-checked)
- `Divider` — 1px horizontal rule
- `Stream` / `StreamItem` — StoryStream rail with purple timeline

---

## Design Tokens → CSS/Theme

### Option A: CSS Variables (recommended — matches current approach)

Keep the CSS custom property system. Move tokens to:
- `app/globals.css` — `:root` and `[data-theme="light"]` blocks
- All component styles reference `var(--token)` — no changes needed

### Option B: Tailwind config

Map tokens to Tailwind theme config:
```ts
// tailwind.config.ts
theme: {
  extend: {
    colors: {
      canvas: 'var(--canvas)',
      'surface-slate': 'var(--surface-slate)',
      'surface-elevated': 'var(--surface-elevated)',
      'jelly-mint': 'var(--jelly-mint)',
      // ... etc
    },
    fontFamily: {
      display: ['Anton', 'Impact', 'sans-serif'],
      body: ['Space Grotesk', 'Helvetica', 'Arial', 'sans-serif'],
      mono: ['Space Mono', 'Courier New', 'monospace'],
    },
    borderRadius: {
      input: '2px',
      nested: '4px',
      card: '20px',
      feature: '24px',
      pill: '24px',
      cta: '30px',
      outlined: '40px',
    },
    spacing: {
      s1: '4px', s2: '8px', s3: '12px', s4: '16px',
      s5: '20px', s6: '24px', s8: '32px', s10: '40px',
      s12: '48px', s14: '56px', s16: '64px', s20: '80px',
    },
  },
}
```

### Option C: CSS Modules

Keep CSS variables, use `.module.css` per component.

**Recommendation**: Option A (CSS variables in globals.css) + CSS Modules per component. The token system is already solid — don't rebuild it.

---

## Theme System

Current implementation:
- `data-theme="dark"` (default) and `data-theme="light"` on `<html>`
- CSS custom properties swap in `[data-theme="light"]` block
- Toggle button in `TopNav`
- `localStorage` persistence under `cr-theme`
- System preference detection via `matchMedia('(prefers-color-scheme: light)')`

**Next.js migration**:
- Create `ThemeProvider` context component
- Read/write `localStorage` in `useEffect`
- Apply `data-theme` to `<html>` via `document.documentElement`
- Use `next-themes` library (recommended) or custom provider

---

## Font Loading

Current: Google Fonts `<link>` in `<head>` with `preconnect`.

**Next.js**: Use `next/font/google`:
```ts
import { Anton, Space_Grotesk, Space_Mono } from 'next/font/google'

const anton = Anton({ weight: '400', subsets: ['latin'], variable: '--font-display' })
const spaceGrotesk = Space_Grotesk({ subsets: ['latin'], variable: '--font-body' })
const spaceMono = Space_Mono({ weight: ['400', '700'], subsets: ['latin'], variable: '--font-mono' })
```

---

## Routing

Current: Client-side `showPage()` with `window.history.replaceState` and URL `?page=` param.

**Next.js**: App Router with file-based routing:
- `/` → Landing
- `/account` → Account
- `/scroll` → Feed/Scroll
- `/friends` → Friends activity

Navigation: Replace `onclick="showPage('page')"` with `<Link href="/page">`.

---

## State Management

Current: None (static mock data).

**Next.js needs**:
- Auth state (user session)
- Theme state (dark/light)
- Feed selection state (scroll page)
- Platform connection state (account page)
- Like/comment state (friends page)

**Recommendation**: Start with React `useState`/`useContext`. Add Zustand or Jotai if complexity grows. Don't reach for Redux.

---

## Recommended Skills for This Session

1. **`writing-plans`** — Plan the component tree, file structure, and migration steps before coding
2. **`dispatching-parallel-agents`** — Once planned, extract components in parallel (each component is independent)
3. **`frontend-design`** — If design adjustments are needed during migration
4. **`impeccable`** — Run `impeccable audit` after migration to verify no regressions
5. **`verification-before-completion`** — Run typecheck, lint, and build before claiming done

---

## Suggested File Structure

```
content-recomend/
├── app/
│   ├── layout.tsx          # Root layout with ThemeProvider, fonts, metadata
│   ├── globals.css         # Design tokens, reset, base styles
│   ├── page.tsx            # Landing page
│   ├── account/
│   │   └── page.tsx        # Account page
│   ├── scroll/
│   │   └── page.tsx        # Feed/scroll page
│   └── friends/
│       └── page.tsx        # Friends activity page
├── components/
│   ├── layout/
│   │   ├── top-nav.tsx
│   │   ├── page-nav.tsx
│   │   └── container.tsx
│   ├── typography/
│   │   ├── wordmark.tsx
│   │   ├── display.tsx
│   │   ├── whisper.tsx
│   │   ├── kicker.tsx
│   │   ├── mono-label.tsx
│   │   ├── headline.tsx
│   │   └── deck.tsx
│   ├── ui/
│   │   ├── btn-mint.tsx
│   │   ├── btn-slate.tsx
│   │   ├── btn-outlined.tsx
│   │   ├── btn-outlined-uv.tsx
│   │   ├── card.tsx
│   │   ├── tile-mint.tsx
│   │   ├── tile-uv.tsx
│   │   ├── tile-white.tsx
│   │   ├── thumb.tsx
│   │   ├── avatar.tsx
│   │   ├── badge.tsx
│   │   ├── tag.tsx
│   │   ├── input.tsx
│   │   ├── toggle.tsx
│   │   ├── divider.tsx
│   │   ├── stream.tsx
│   │   └── stream-item.tsx
│   └── theme/
│       └── theme-provider.tsx
├── lib/
│   └── theme.ts            # Theme hook/context
├── public/
│   └── fonts/              # (if self-hosting, otherwise next/font handles)
├── index.html              # Keep as reference during migration
├── HANDOFF.md              # Backend handoff (previous)
├── HANDOFF-NEXTJS.md       # This file
├── README.md
├── AGENTS.md
├── DESIGN.md
├── package.json
├── tsconfig.json
├── next.config.ts
├── tailwind.config.ts      # (if using Tailwind)
└── postcss.config.mjs      # (if using Tailwind)
```

---

## Migration Strategy

### Phase 1: Scaffold
```bash
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir=false
```
(Or without Tailwind if using CSS variables + CSS Modules)

### Phase 2: Tokens & Base
- Move CSS variables to `app/globals.css`
- Set up font loading with `next/font`
- Create `ThemeProvider`
- Verify dark/light mode works

### Phase 3: Components
- Extract each component from `index.html`
- One component per file
- Keep exact same styles (copy-paste CSS, convert to modules or keep globals)
- Verify each component renders identically to prototype

### Phase 4: Pages
- Create page routes
- Wire up navigation with `<Link>`
- Remove `showPage()` client-side routing
- Verify all 4 pages work

### Phase 5: Polish
- Run `impeccable audit` — verify 19/20 score maintained
- Fix any regressions
- Add loading states, error boundaries
- Set up proper metadata per page

---

## Constraints

- **Do not redesign** — The design is approved (19/20 audit score). Match the prototype exactly.
- **Do not add dependencies** unless necessary. The prototype has zero dependencies.
- **Keep CSS variables** — The token system works. Don't replace it with inline styles or utility-only approaches.
- **Maintain accessibility** — All ARIA labels, roles, keyboard handlers, focus-visible styles must be preserved.
- **Maintain responsive behavior** — 3 breakpoints, fluid typography, reduced motion support.

---

## Open Questions

1. **Tailwind or CSS Modules?** — Current code uses CSS variables. Tailwind can coexist but isn't required.
2. **Server vs Client Components?** — Most UI will be client components (interactive). Layout and static sections can be server components.
3. **Deployment target?** — Vercel (default for Next.js), or elsewhere?
4. **Backend integration timing?** — Should this migration include API calls, or stay with mock data for now?
