# ContentRecomend

Connect your YouTube, Netflix, and Spotify watch history. Save what is great. Share with friends.

## What It Is

ContentRecomend is a content recommendation app that turns your passive watch history into an active social experience. Bookmark videos, films, and playlists you love, then let friends discover what you're watching.

## Features

- **Multi-platform sync** — Connect YouTube, Netflix, Spotify, and Twitch
- **Bookmark & save** — Build personal collections of content you love
- **Social feed** — See what friends are watching, like and comment on their picks
- **Split-view discovery** — Browse your feed while viewing content details side-by-side
- **Dark & light mode** — Editorial design system with full theme support

## Design

Built with The Verge design system: heavy Anton display typography, Space Mono uppercase labels, hazard-color accents (Jelly Mint `#3cffd0`, Ultraviolet `#5200ff`), flat depth with 1px borders.

### Quality

- **19/20** impeccable audit score
- **29/29** contrast ratios pass WCAG AA (both dark and light mode)
- **30/30** structural checks pass
- Zero anti-patterns, zero dependencies, single HTML file

## Project Structure

```
content-recomend/
├── index.html    # Frontend prototype (complete, 1383 lines)
├── README.md     # This file
└── HANDOFF.md    # Backend development handoff document
```

## Frontend

The frontend is a complete single-page prototype with:
- 4 pages: Landing, Account, Scroll (feed), Friends (activity)
- Client-side routing via `showPage()`
- Dark/light mode toggle with `localStorage` persistence
- Full keyboard navigation and ARIA support
- Responsive design with 3 breakpoints
- No build step, no framework, no dependencies beyond Google Fonts

Open `index.html` in any browser to preview.

## Backend

Backend development is pending. See `HANDOFF.md` for:
- Data models and schema
- API endpoint specifications
- Platform OAuth integration requirements
- Sync worker architecture
- Technical considerations and open questions

## Development

No build step required. The frontend is a single HTML file.

```bash
# Preview
open index.html

# Or serve locally
python3 -m http.server 8080
```

## License

Private project.
