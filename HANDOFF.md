# Handoff: ContentRecomend Backend Development

**Date**: 2026-05-20
**Project**: ContentRecomend (CR) — Watch history sharing and content recommendations
**Frontend**: Single-page prototype at `/Users/per/WebstormProjects/content-recomend/index.html`
**Status**: Frontend prototype complete (19/20 impeccable audit score). Backend needed.

---

## What This Is

ContentRecomend lets users connect their YouTube, Netflix, Spotify, and Twitch accounts, bookmark content they love, and share collections with friends. Think "goodreads for watch history."

## Frontend Architecture

**Single file**: `index.html` (1383 lines) — no build step, no framework, no dependencies beyond Google Fonts.

**4 pages** (JS client-side routing via `showPage()`):
1. **Landing** — Split hero, 3 feature tiles (mint/card/uv), CTA
2. **Account** — Profile header, platform connection cards (YT/NF/SP) with toggles, recently saved stream
3. **Scroll** — Split view: feed list (left) + detail panel (right). Selectable items with Save/Share actions
4. **Friends** — Activity feed with posts, likes, comments, platform badges

**Design system** (The Verge aesthetic):
- Display: Anton (heavy, condensed)
- Body: Space Grotesk
- Labels: Space Mono (uppercase, tracked out)
- Colors: Jelly Mint `#3cffd0`, Ultraviolet `#5200ff`, warm neutrals
- Dark mode (default) + light mode with `data-theme` toggle, `localStorage` persistence
- All contrast ratios pass WCAG AA (29/29 verified)

---

## Backend Requirements

### 1. Data Models

Based on the frontend UI, the backend needs to support:

#### Users
```
id, username, display_name, avatar_url, created_at, friend_count, save_count
```

#### Platform Connections
```
id, user_id, platform (youtube|netflix|spotify|twitch), platform_user_id, access_token, refresh_token, connected_at, sync_enabled, item_count
```

#### Content Items
```
id, platform, platform_item_id, title, description, thumbnail_url, duration, url, metadata (JSON), created_at
```

#### Saves (Bookmarks)
```
id, user_id, content_id, saved_at, collection_id (nullable), note (nullable)
```

#### Friends/Follows
```
id, follower_id, following_id, created_at
```

#### Activity Posts
```
id, user_id, content_id, post_text, created_at, like_count, comment_count
```

#### Likes
```
id, user_id, post_id, created_at
```

#### Comments
```
id, user_id, post_id, body, created_at
```

### 2. API Endpoints (Expected by Frontend)

The frontend currently uses mock data. These endpoints need to be wired up:

#### Auth
- `POST /api/auth/register` — Create account
- `POST /api/auth/login` — Login
- `POST /api/auth/logout` — Logout
- `GET /api/auth/me` — Current user profile

#### OAuth Platform Connections
- `GET /api/oauth/:platform/authorize` — Redirect to platform OAuth
- `GET /api/oauth/:platform/callback` — Handle OAuth callback
- `GET /api/connections` — List connected platforms
- `PUT /api/connections/:id/toggle` — Enable/disable sync
- `DELETE /api/connections/:id` — Disconnect platform

#### Content/Saves
- `GET /api/saves` — List user's saved items (with pagination)
- `POST /api/saves` — Save a content item
- `DELETE /api/saves/:id` — Remove save
- `GET /api/content/:platform/:platformId` — Fetch content details

#### Friends/Activity
- `GET /api/friends` — List friends
- `GET /api/activity` — Activity feed (friends' posts)
- `POST /api/posts` — Create a post (share content)
- `POST /api/posts/:id/like` — Like a post
- `DELETE /api/posts/:id/like` — Unlike
- `POST /api/posts/:id/comments` — Add comment
- `GET /api/posts/:id/comments` — List comments

#### User Profile
- `GET /api/users/:username` — Public profile
- `GET /api/users/:username/saves` — Public saves (if shared)

### 3. Platform Sync

Each platform needs a sync worker that:
- Fetches watch history via platform API
- Deduplicates against existing content items
- Creates content items if new
- Updates sync timestamps

**YouTube**: YouTube Data API v3 (watch history requires OAuth + privacy consent)
**Netflix**: No public API — may need workaround (browser extension, manual import, or third-party)
**Spotify**: Spotify Web API (recently played, playback history)
**Twitch**: Twitch API (followed channels, watch history)

### 4. Technical Considerations

- **Rate limiting**: Platform APIs have strict rate limits. Queue sync jobs.
- **Token storage**: Encrypt OAuth tokens at rest. Rotate refresh tokens.
- **Privacy**: Watch history is sensitive. Users control what's shared publicly vs. friends-only.
- **Pagination**: Feed and save lists need cursor-based pagination (frontend shows 6+ items per list).
- **Real-time**: Activity feed would benefit from WebSocket/SSE for live updates (optional for v1).

---

## File Structure

```
content-recomend/
├── index.html              # Frontend prototype (complete)
├── DESIGN.md               # Design system tokens (if exists)
├── PRODUCT.md              # Product context (if exists)
└── [backend to be created]
```

---

## Recommended Skills for Next Session

1. **`supabase`** — If using Supabase for Postgres, Auth, Edge Functions. Covers schema design, RLS policies, OAuth setup.
2. **`writing-plans`** — Before writing code, plan the API surface, data models, and auth flow.
3. **`brainstorming`** — For decisions around Netflix integration (no public API), privacy model, and sync architecture.
4. **`security-review`** — OAuth token handling, RLS policies, data privacy review before launch.
5. **`handoff`** — When backend work is done, create a handoff back to frontend for API integration.

---

## Open Questions for Backend Dev

1. **Tech stack preference?** (Node/Express, Python/FastAPI, Go, Supabase, etc.)
2. **Netflix integration** — No public API exists. Options: browser extension, manual CSV import, skip for v1?
3. **Auth strategy** — Email/password + OAuth for platforms, or magic links, or social login?
4. **Hosting** — Vercel, Railway, Fly.io, self-hosted?
5. **Database** — Postgres (recommended for relational data + RLS), or something else?

---

## Frontend Integration Notes

When the backend is ready, the frontend needs:
1. Replace `showPage()` mock navigation with real auth state
2. Replace static content in each page with `fetch()` calls to API
3. Add loading states and error handling
4. Wire up form submissions (save, like, comment, connect platform)
5. Add pagination/infinite scroll to feed lists
6. Handle OAuth redirect flows

The frontend is intentionally framework-free (vanilla JS) to keep it simple. Backend can return JSON; no SSR required.
