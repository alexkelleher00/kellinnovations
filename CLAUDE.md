# kellinnovations — Frontend Website

Repo: `C:\Users\PC\Desktop\workspace\Website\kellinnovations\`  
Live: **kellinnovations.com**  
No build step — plain HTML/CSS/JS, deploy by pushing to git (auto-deploys via Cloudflare Pages or similar).

## File Map

| File | Purpose |
|------|---------|
| `index.html` | Home / landing page |
| `psg.html` | Punk Squad Games hub (history tabs, countdown, registration) |
| `scoreboard.html` | **PSG Live Scoreboard** — Scoreboard, Bracket, Timed Games, Setup tabs |
| `transit.html` | **Live Transit Tracker** — real-time MBTA arrivals, polls `kellinnovations-server`'s `transit` blueprint |
| `blog.html` | Blog listing |
| `portfolio.html` | Portfolio page |
| `slide.html` | Presentation / slide viewer |
| `3man.html` | Three.js 3D bar/dice game |
| `letterfall.html` | Letterfall word game |
| `propertytrade.html` | **Property Empire** — online property-trading board game, up to 5 players |
| `2048.html` | 2048 tile-merging game with a global leaderboard |
| `uno.html` | **Uno** — online card game, 2-10 players, polls `kellinnovations-server`'s `uno_game` blueprint |
| `style.css` | Global dark theme; all CSS variables defined here |
| `portfolio.css` | Portfolio-specific styles |

Woodworking (formerly `woodwork.html`/`gallery.html`/`shop.html`) now lives in its own repo and
is rebranded as **Kelleher Workshop**, live at **kelleherworkshop.com** via GitHub Pages (custom
domain set via the repo's `CNAME` file). The "Woodwork" nav links and the homepage project card
link straight to that domain. Repo is currently still named `kelleherheartwood` on GitHub
(`git@github.com:alexkelleher00/kelleherheartwood.git`, local copy at `Website/kelleherheartwood/`)
pending a rename to `kelleherworkshop` — update this note and the local folder path once that's done.

## Design System (`style.css`)

All pages use CSS custom properties from `style.css`. Key vars:
```css
--bg, --bg2, --surface   /* background layers */
--text, --muted, --dim   /* text colors */
--accent                  /* cyan #00c8c8 */
--border                  /* border color */
--pad                     /* horizontal padding */
```
**Never hardcode hex colors** — always use `var(--*)`.

## PSG Scoreboard (`scoreboard.html`)

This is the most complex file. Key architecture:
- **API base:** `https://api.kellinnovations.com/psg`
- **4 tabs:** Scoreboard, Tournament Bracket, Timed Games, Setup (admin only)
- **Auth:** PIN-based cookie via `POST /auth/login` — sets `psg_admin` cookie
- **Auto-refresh:** every 10 seconds, paused if admin has unsaved edits (`scoresDirty`)
- **State vars:** `scoreData`, `bkData`, `isAdmin`, `scoresDirty`, `timedDirty`

### Key JS Functions
| Function | What it does |
|----------|-------------|
| `init()` | Bootstrap: check admin, fetch scores+bracket, start auto-refresh |
| `fetchScores()` | GET /scoreboard → `scoreData`, calls `renderScores()` + `renderTimedPanel()` |
| `fetchBracket()` | GET /bracket → `bkData`, calls `renderBracket()` |
| `renderScores()` | Builds grid HTML from `scoreData`; admin gets `<input>` cells |
| `renderBracket()` | Renders single-elimination bracket from `bkData.matches` + `bkData.round_names` |
| `renderTimedPanel()` | Renders timed event tables for Treasure Hunt, Watering Hole, Gauntlet |
| `initBracket()` | POST /bracket/init — generates bracket from standings |
| `advanceBracket(id, winner)` | POST /bracket/advance — marks match winner |
| `applyBracketScores()` | POST /bracket/apply_scores — writes bracket placements to scoreboard |
| `applyTimedScores(eventEnc)` | POST /timed/update — applies timed event times → points |
| `recalcPreview(event)` | Live rank/points preview from time inputs (no API call) |
| `saveAllScores()` | POST /scoreboard/update — saves admin score edits |
| `saveTeams()` | POST /scoreboard/teams — saves team names/members |

### Constants (must match server-side `psg/routes.py`)
```js
const TOURNAMENT_EVENTS = ["Volleyball", "Pickleball", "Gladiator Rumble"];
const TIMED_EVENTS_LIST = ["Treasure Hunt", "Watering Hole", "Gauntlet"];
const ROUND_ROBIN_PTS   = [10, 8, 6, 5, 4, 3, 2, 1]; // rank >= 8 → 1 pt
```

### Bracket Rendering
- `renderBracket()` is fully generic — handles any number of rounds/matches from the server
- BYE matches display with `.br-team.bye` CSS class
- Admins click team cells to advance winners (`advanceBracket`)
- `canClick = isAdmin && !m.winner && m.team1 && m.team2 && m.team2 !== "BYE"`

## Live Transit Tracker (`transit.html`)

- **API base:** `https://api.kellinnovations.com/transit`
- Four endpoints: `GET /predictions` (arrival countdown cards), `GET /vehicles` (live GPS positions
  for the map), `GET /geo` (stops + route shape polylines, fetched once — geometry barely changes),
  `GET /places` (static home/work landmarks, fetched once alongside geo)
- Each prediction card renders **both directions** of its line as separate sections (`line.directions`),
  each with its own heading (`direction.name` — `direction.destination`) — a direction with no upcoming
  service still renders its heading with a "no scheduled arrivals" message, it never just disappears
- Map is [Leaflet](https://leafletjs.com) + CARTO's free `dark_all` tile layer (dark basemap, no API
  key) loaded via CDN — no build step, consistent with the rest of the site's no-build approach. Stop
  markers get a white stroke and each line's tracked "focus" stop renders larger than the rest of that
  route's stops, so they don't disappear against a busy dark map next to fainter regular stops.
- `refreshAll()` re-fetches predictions + vehicles every 15s; a separate 1s timer re-renders "in X min"
  countdowns from cached `when` timestamps so the page feels live between polls without extra calls
- Vehicle markers are a 40px `L.divIcon` badge (mode emoji — 🚋 Green Line, 🚇 Red Line, 🚌 buses — keyed
  off `line.id`'s prefix client-side) plus a separate chevron `div` that pivots around the badge's own
  center via inline `transform:rotate(bearingdeg)` to show heading, without rotating the emoji itself
  (Leaflet has no native marker rotation, hence two layered divs instead of one rotated icon)
- **Route dropdown** (`#routeSelect`) filters both the cards grid and the map to one line at a time, or
  "All lines". Map layers are pre-built per line id into `lineGeoLayers`/`lineVehicleLayers` and
  toggled on/off via `applyLineFilter()` rather than rebuilt, so switching is instant
- **Geolocation**: `locateUser()` (auto-fires once, silently, after `/geo` loads; also a manual "Use my
  location" button) drops a "you are here" marker and picks whichever line's `focus_stop` is physically
  nearest (haversine in `distanceMeters()`), auto-setting the dropdown to that line. This is why the
  tracked `stop`s in `transit/routes.py` need to actually match where the rider boards — picking the
  nearest of the *wrong* stops still nails the direction-finding, just onto stale data
- Which lines are tracked is decided **server-side** (`transit/routes.py`'s `LINES` list) — this page
  just renders whatever the API returns, so adding a line to the backend is enough, no frontend change.
  Two lines can share the same route (e.g. `bus-354-medford` / `bus-354-cummings` track the same
  physical 354 route at both ends of a commute) — the frontend treats them as fully independent cards/
  map layers since it only ever keys off `line.id`, never assumes one line per route
- Each card/map element's color comes from `line.color` (set via inline CSS var `--line-color` for
  cards; passed directly into inline styles for map polylines/markers)
- Map's initial view fits to each line's `focus_stop` (the specific tracked stop), not every stop on
  the route — otherwise long routes like the 354 (Woburn) or Red Line (Alewife–Braintree) would zoom
  the map out past the point of being useful. Full route paths/vehicles are still drawn, just
  pannable/zoomable into from the default local view.

## Uno (`uno.html`)

Same overall shape as Property Empire (`propertytrade.html`) — room-code multiplayer, a
`token` in `localStorage` (`uno_session`), 1s state polling — but unlike Property Empire,
Uno hands are **hidden information**, not public board state.

- **API base:** `https://api.kellinnovations.com/uno` (`http://localhost:4567/uno` on localhost)
- **Auth:** no passwords — `POST /rooms` (create) / `POST /rooms/<code>/join` returns a
  session `token`; `POST /rooms/<code>/rejoin` recovers a lost session with just the room
  code + the exact name you were already using (same no-real-auth model as Property Empire)
- **Per-player state:** `GET /rooms/<code>/state?token=...` returns *your* hand under
  `your_hand`, but only card **counts** for every other player (`players[].hand_count`) —
  the server builds this response per-requester (`Game.to_state(pid)`), it's never a single
  shared blob like Property Empire's. Polling without a token (or with an invalid one)
  returns a valid spectator view (`your_hand: null`) rather than an error, so "Watch" works.
- **Turns:** `draw_card` → `play_card` (wild/wild4 require a `color`) → `end_turn` (only
  legal right after a voluntary draw you choose not to play). One action funnels through
  `POST /rooms/<code>/action` same as Property Empire.
- **Declaring UNO:** dropping to 1 card puts your `id` in the room's vulnerable set
  (`players[].pending_uno`) until your own next turn starts. `declare_uno` clears it safely;
  any other player can `catch_uno` (`{target_id}`) before then for a 2-card penalty — CPU
  players do this opportunistically too (and occasionally "forget" to self-declare, on
  purpose, so there's something to catch).
- **Bots:** unlike Property Empire's one-CPU cap, any number of `CPU N` players can be
  added in the lobby (host-only). They resolve their entire turn instantly when polled,
  same tick-driven pattern as Property Empire's `_maybe_take_bot_turn`.
- **Scoring:** first to empty their hand wins the round (+ opponents' remaining card
  points); first to the lobby's configurable `target_score` (default 500) wins the match.
  The host advances `next_round` from the round-over screen; hands reshuffle, scores carry over.
- Chat + emoji reactions are a straight port of Property Empire's FAB pattern (`.pt-*` →
  `.un-*`), including the "flash on screen + separate persistent history in the drawer" split.

## Common Pitfalls

- `escHtml(s)` is used everywhere for user-provided content — never skip it
- `encodeURIComponent` on team names in `data-` attributes; always `decodeURIComponent` when reading back
- The `scoresDirty` flag pauses auto-refresh during admin edits — don't break this guard
- TV mode hides header/tabs/admin controls via `body.tv-mode` class — don't add elements that need explicit TV-mode hiding without updating that CSS block
