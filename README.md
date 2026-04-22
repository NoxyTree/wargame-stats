# Handoff: Wargame Stats Guild Dashboard

## Overview
A comprehensive guild battle analytics dashboard for tracking player performance, match history, and rivalries. The dashboard includes:
- Overview with stats and recent games
- Calendar view of matches
- Player roster with sortable stats and period filters
- Mercenaries (allied non-guild players) tracking
- Feuds (enemy players) tracking
- **Personal Profile page** with rolling win-rate charts, stats, and match history
- **Side-by-side scoreboard modal** showing team-vs-team breakdown of individual matches
- **Player profile popup** with mini stats and win-rate trend

## About the Design Files
The files in this bundle are **high-fidelity HTML design prototypes** — they show the intended look, feel, and behavior of the final UI. These are **reference designs**, not production code to copy directly.

Your task is to **recreate these designs in your target codebase** using its existing framework (React, Vue, Angular, SwiftUI, etc.), libraries, and patterns. If no framework exists yet, choose the most appropriate one for the project (likely React for web) and implement the designs there.

The HTML files use vanilla JavaScript, inline styles, and browser APIs for rapid prototyping. You should adapt this to your stack's conventions: components, state management, TypeScript, CSS-in-JS or CSS modules, API integration, etc.

## Fidelity
**High-fidelity (hi-fi)**. The prototypes are pixel-perfect mockups with final colors, typography, spacing, interactions, and animations. Recreate the UI as closely as possible using your codebase's existing patterns and libraries.

## Architecture

### Data Flow
- **API Base**: `http://localhost:3001` (configurable)
- **Guild ID**: From URL query param `?guild=...`
- **Endpoints**:
  - `GET /api/guilds/{id}/summary` → guild name, all-time/30d/7d stats
  - `GET /api/guilds/{id}/matches` → list of matches
  - `GET /api/guilds/{id}/players` → roster aggregates
  - `GET /api/guilds/{id}/players?period=7|30` → period-filtered stats
  - `GET /api/guilds/{id}/players/{name}?scope=all` → player profile with match list
  - `GET /api/guilds/{id}/matches/{id}` → detailed scoreboard with all players
  - `GET /api/guilds/{id}/mercs` → mercenary stats
  - `GET /api/guilds/{id}/feuds` → enemy player stats
  - `GET /api/guilds/{id}/top_performers` → 14-day leaderboard (min 3 games)

- **Fallback**: If API fails, uses `seed_data.json` from the same directory as index.html

### State Management
- `STATE` object holds:
  - `summary` (guild name, stats)
  - `totals` (player roster)
  - `games` (match list)
  - `gameCache` (match detail by ID)
  - `mercs`, `feuds`, `topPerformers`
  - `periodPlayers` (7d/30d filtered data)
  - `sortCol`, `sortDir` (per-table sort state)
  - `page` (pagination for mercs/feuds)
  - `calCursor`, `selectedDate` (calendar state)
  - `playerProfileCache` (player detail cache)

- **Profile Selection**: `localStorage` key `wgs:profile:{guildId}` stores selected player name; persists across reloads
- **Discord Links**: `localStorage` key `wgs:discord:{playerName}` stores optional Discord URL per player

### Navigation
- Tabs: Overview | Calendar | Players | Mercenaries | Feuds
- Profile pill (far right of tabs) opens personal Profile view
- Clicking any match → opens side-by-side scoreboard modal
- Clicking any player name → opens player profile popup
- Profile page: search + select your player → see personal stats, charts, history

---

## Screens / Views

### 1. Overview Tab
**Purpose**: At-a-glance guild stats, top performers, recent games.

**Layout**:
- **Stats Bar**: 5 stat cards in a responsive grid (`grid-template-columns: repeat(auto-fit, minmax(180px, 1fr))`)
  - Background: `rgba(20, 25, 34, 0.65)` + `backdrop-filter: blur(6px)`
  - Border: `1px solid rgba(42, 50, 68, 0.8)`
  - Radius: `10px`
  - Padding: `20px 22px`
  - Each card: label (uppercase, `0.68rem`, `#707a8a`), value (`2.2rem`, `#e6edf3`), sub-text (`0.72rem`, `#707a8a`)

- **Top Performers Panel**:
  - Title: "Top Performers" | Subtitle: "Last 14 Days · Per-Game Avg · Min 3 Games"
  - Grid: 4 columns on desktop (`repeat(4, 1fr)`), 2 on tablet
  - Each card: player name (uppercase, `0.82rem`, `#e6edf3`), 3 stat rows (label + value)
  - Hover: lift 3px, accent border glow

- **Recent Games List**:
  - Each game row: enemy name, map, result badge (win green `#3ddc97` / loss red `#ef6464`), date
  - Clickable → opens scoreboard modal

### 2. Calendar Tab
**Purpose**: Monthly view of all matches, colored by result.

**Layout**:
- Month header with prev/next arrows
- 7-column grid (Sun–Sat), 6 rows max
- Each day cell:
  - Gray if out-of-month
  - Shows date number (top-left)
  - Match dots below (green for wins, red for losses, max 3 visible + "+" badge if more)
- Click a day → highlights all matches that day in a list below
- Click a match → opens scoreboard modal

### 3. Players Tab
**Purpose**: Guild roster with sortable stats.

**Layout**:
- Period toggle pills: "All Time" | "30 Days" | "7 Days" (active has gold `#f5a524` background)
- Search input (filters by player name)
- Table with columns:
  - Rank | Name | Games | Wins | Losses | Win % | Defeats | Assists | Damage Dealt | Damage Taken | Healed
- Click column header → sort (arrow indicates direction)
- Click row → opens player profile popup

**Typography**:
- Headers: `0.62rem`, uppercase, `#707a8a`, letter-spacing `0.14em`
- Cells: monospace `0.8rem`, `#e6edf3`
- Name cell: sans-serif `0.82rem`, bold

### 4. Mercenaries Tab
**Purpose**: Track allied players outside your guild.

**Layout**: Same table structure as Players, with added "Guild" column and "Last Seen" date. Pagination at bottom (50 per page).

### 5. Feuds Tab
**Purpose**: Track enemy players you've faced.

**Layout**: Same as Mercs, plus "Copy Top 10" button (copies names to clipboard, newline-separated). Guild filter dropdown at top.

### 6. Profile Page (Personal View)
**Purpose**: Selected player's personal stats, trends, and full match history.

**Layout**:
- **Empty State** (no player selected):
  - Centered panel with search input
  - As user types, dropdown suggests matching names from roster + mercs
  - Click suggestion → selects player, renders full profile

- **Profile Hero Card**:
  - Left: large avatar (2-letter initials, gold gradient `linear-gradient(135deg, #f5a524, #b4791a)`)
  - Center: player name (`1.9rem`, uppercase), stats line (games · W · L · win %), W/L streak tape (last 20 matches as colored chips)
  - Right: "Link Discord" button (purple `#5865F2`, Discord logo) + "Change player" button
  - Discord click: if not set, prompts for URL (saved to `localStorage:wgs:discord:{name}`); if set, opens in new tab

- **Stat Cards Row**: 5 cards (Avg Damage, Defeats, Assists, Healed, Taken) — same style as Overview

- **Charts Grid** (2 columns on desktop, 1 on mobile):
  - **Left: Rolling Win Rate**:
    - Line chart, last 30 matches
    - X-axis: match index, Y-axis: cumulative win%
    - Gradient fill under curve
    - Each match is a dot (green for win, red for loss)
    - Grid lines at 0%, 25%, 50%, 75%, 100%
  - **Right: Win/Loss Donut**:
    - 170×170px SVG donut (stroke `14px`)
    - Win arc (green), Loss background (dim red)
    - Center: large % (`1.9rem`), label "Win Rate" (`0.65rem`)
    - Legend below: green dot "Victories {N}" | red dot "Defeats {N}"

- **Match History Table**:
  - Columns: Date | vs | Map | Result | Defeats | Assists | Damage | Taken | Healed
  - Rows clickable → opens scoreboard modal
  - `cursor: pointer` on rows

### 7. Scoreboard Modal (Match Detail)
**Purpose**: Side-by-side team breakdown for a single match.

**Layout**:
- **Header**:
  - Breadcrumb: "Matches › {date}"
  - Title row: "VICTORY" or "DEFEAT" (large, color-coded) + "vs {Enemy Guild}"
  - Meta pills: Map | Mode | Players (e.g. "5v5")
  - Close button (top-right X)

- **Body** (full-width, no side column):
  - Two panels side-by-side:
    - **Your Team** (green accent)
    - **Opponent Team** (red accent)
  - Each panel:
    - Header bar: "YOUR TEAM" | "5 Players"
    - Table: Player | Defeats | Assists | Dmg Dealt | Dmg Taken | Healing
    - NO avatars — just name + guild-label below
    - Sortable columns (click header)
    - Rows clickable → opens player profile popup

- **Responsive**: On mobile (<780px), stacks vertically; hides columns 5-6

**Colors**:
- Ally header label: `#3ddc97`
- Enemy header label: `#ef6464`
- Background: `rgba(13, 17, 23, 0.7)`
- Modal max-width: `1600px`

### 8. Player Profile Popup (Small Modal)
**Purpose**: Quick player stats + trend when clicking a name in tables/scoreboards.

**Layout**:
- **Header**: Player name (large, uppercase), meta line (games · W · L · win %)
- **Stat Cards**: Same 5-card row as Profile page
- **Rolling Win Rate Mini-Chart**: 
  - 600×180px SVG, last 30 matches
  - Same style as Profile page chart but smaller
  - Container: dark background, border, 8px padding
- **Match History Table**: Same columns as Profile page, rows clickable → opens scoreboard

---

## Design Tokens

### Colors
```
--bg-void: #0d1117
--bg-deep: #161b22
--bg-surface: #21262d
--bg-hover: rgba(42, 50, 68, 0.4)
--border: rgba(42, 50, 68, 0.8)
--border-hi: rgba(42, 50, 68, 1)

--text-primary: #e6edf3
--text-secondary: #96a0ad
--text-muted: #707a8a

--accent: #f5a524
--accent-hi: #ffb84d
--accent-glow: rgba(245, 165, 36, 0.3)

--win: #3ddc97
--win-dim: rgba(61, 220, 151, 0.1)
--loss: #ef6464
--loss-dim: rgba(239, 68, 100, 0.1)
```

### Typography
```
--font-display: "Space Grotesk", sans-serif
--font-body: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif
--font-mono: "JetBrains Mono", "SF Mono", Consolas, monospace
```

**Font Weights**: 400 (regular), 600 (semi-bold), 700 (bold), 900 (black)

**Letter-spacing**:
- Uppercase labels: `0.14em` to `0.2em`
- Body: normal
- Mono: `0.05em` to `0.1em`

### Spacing
```
--radius-sm: 6px
--radius-md: 10px
--radius-lg: 14px
```

Padding/margin scale: 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30px (even numbers)

### Shadows
```
Stat card hover: 0 8px 24px rgba(245, 165, 36, 0.08)
Top card hover: 0 8px 24px rgba(245, 165, 36, 0.08)
Avatar glow: 0 0 30px rgba(245, 165, 36, 0.35)
```

---

## Interactions & Behavior

### Tabs
- Click tab → switch active view (`.active` class)
- Active tab: gold background `#f5a524`, black text, shadow
- Inactive: muted gray, hover lightens

### Tables
- Click column header → sort (toggle asc/desc)
- Sorted column shows arrow (▲ or ▼)
- Default sort: numeric columns desc, string columns asc
- Hover row: background `rgba(42, 50, 68, 0.4)`, name turns accent gold

### Modals
- Fade in 0.3s ease
- Backdrop: `rgba(13, 17, 23, 0.85)` + `backdrop-filter: blur(4px)`
- Click backdrop or X → close
- Escape key → close (closes player popup if open, else scoreboard)

### Profile Pill
- Lives on far right of tabs row (not in header)
- Shows "?" avatar if no selection, else player's initials
- Click → opens Profile tab, focuses search if no selection
- Updates to selected player's name on selection

### Animations
- Fade-in views: 0.45s ease (translateY 12px → 0)
- Stat cards: 0.3s hover lift (-3px)
- Status dot pulse: 2s infinite
- Chart line draw-in: 0.7s cubic-bezier(.2,.8,.2,1)
- Reveal on scroll: 0.6s ease, staggered by 0.05s

### Pagination
- Mercs/Feuds: 50 per page
- Prev/Next buttons disabled at bounds
- Display: "Page N of M · X total"

### Period Toggle
- Players tab: All Time | 30 Days | 7 Days
- Fetches filtered data from API endpoint `?period=7` or `?period=30`
- Updates table heading subtitle (e.g. "Last 30 Days · Per-Game Avg")

### Search
- Players/Mercs: filter by name (case-insensitive substring match)
- Profile suggest: shows matches as user types, max 12 results
- Enter key in Profile search → selects first suggestion

### Calendar
- Click day → highlight, show matches for that day below grid
- Click match in list → open scoreboard
- Prev/Next month buttons update grid + URL param (optional)

### Copy Button
- Feuds "Copy Top 10" → writes newline-separated names to clipboard
- Button text changes to "Copied!" for 1.5s

---

## State Management

### Key Variables
```js
STATE = {
  summary: { guildName, allTime, last30d, last7d },
  totals: [{ name, games, wins, losses, defeats, assists, dealt, taken, healed }],
  games: [{ id, label, enemy, map, result, date }],
  gameCache: { [id]: { ...match, players: [...] } },
  mercs: [...],
  feuds: [...],
  topPerformers: [...],
  periodPlayers: { "7": [...], "30": [...] },
  playerPeriod: "" | "7" | "30",
  sortCol: { players, game, mercs, feuds },
  sortDir: { players, game, mercs, feuds },
  page: { mercs, feuds },
  calCursor: Date,
  selectedDate: "YYYY-MM-DD" | null,
  playerProfileCache: { "{name}:all": { matches: [...] } }
}
```

### LocalStorage
- `wgs:profile:{guildId}` → selected player name (string)
- `wgs:discord:{playerName}` → Discord URL (string, optional)

### Caching
- Match details cached after first fetch (`gameCache`)
- Player profiles cached (`playerProfileCache`)
- Persist across tab switches (no refetch)

---

## Assets

### Icons
- Sigil (guild logo): inline SVG shield with checkmark path
- Aurora background: 3 animated gradient divs (pure CSS)
- Discord logo: inline SVG (24×24 viewBox)
- Sort arrows: CSS `::after` with content `▲` or `▼`

### Images
- `favicon.png` (64×64 PNG) — used in `<link rel="icon">`
- No other external images; all UI is code + gradients

### External Dependencies
- Google Fonts: Space Grotesk (400, 600, 700, 900), JetBrains Mono (400, 600)
- **No JS libraries** — vanilla DOM manipulation
- Fallback to system fonts if Google Fonts fails

---

## Files

### Reference HTML Files
- **`guild.html`** — Full dashboard implementation (2800+ lines)
  - All tabs, modals, profile view, chart rendering, API integration
  - Self-contained: HTML + CSS + JS in one file
  - Uses `seed_data.json` as fallback if API unavailable
  
- **`index.html`** — Landing page (guild selector)
  - Minimal: lists available guilds, links to `guild.html?guild={id}`

- **`seed_data.json`** — Demo data for offline use
  - Structure matches API responses
  - Contains ~50 matches, ~20 players, mercs, feuds, etc.

### Key Functions (JS Reference)
**In `guild.html`**:

**Data Loading**:
- `loadData()` — fetches all endpoints, populates STATE
- `normalizeMatch(m)`, `normalizePlayer(p)`, `normalizeMercFeud(m)` — data transform

**Rendering**:
- `renderOverview()` — stats bar, top performers, recent games
- `renderCalendar()` — month grid, match dots
- `renderPlayers()` — roster table with period/search filter
- `renderMercs()`, `renderFeuds()` — paginated tables
- `renderProfileView(name)` — personal stats, charts, history

**Charts**:
- SVG line chart built inline (no library)
- Win-rate computed as rolling average
- Path coords: `xs(i)` and `ys(value)` helpers
- Gradient fill under curve

**Modals**:
- `openGame(id)` — scoreboard with ally/enemy teams
- `openPlayerProfile(name, scope)` — small popup with stats + chart
- `closeGame()`, `closePlayerProfile()`

**Profile**:
- `getSelectedProfile()`, `setSelectedProfile(name)` — localStorage sync
- `updateProfilePill(name)` — update avatar + name display
- `renderProfileSuggest(query)` — search dropdown
- `selectProfile(name)` — persist + render

**Utilities**:
- `fmt.date()`, `fmt.num()`, `fmt.compact()`, `fmt.pct()` — formatters
- `escapeHtml(str)` — XSS protection
- `setupSorting()` — wire table header clicks
- `setupReveal()` — intersection observer for scroll animations

---

## Implementation Notes

### Responsive Breakpoints
- Desktop: >1000px
- Tablet: 640–1000px
- Mobile: <640px

**Mobile Adaptations**:
- Tabs: horizontal scroll if needed
- Stats bar: single column
- Top performers: 2 columns → 1
- Tables: hide columns 5+ on small screens
- Scoreboard: stack teams vertically
- Profile grid: single column

### Performance
- Lazy-load profile data (only fetch when opened)
- Cache match details (avoid refetch)
- Virtualize long tables if >500 rows (not in prototype, but recommended)

### Accessibility
- All buttons have `aria-label` or visible text
- Modals trap focus (implement keyboard nav)
- Tables have proper `<thead>` + `<tbody>`
- Status indicator has `aria-live` region (add in production)

### Testing Checklist
- [ ] Sort each table column (asc/desc)
- [ ] Filter players by name
- [ ] Toggle period (7d/30d/all)
- [ ] Click calendar day → see matches
- [ ] Open scoreboard from calendar, recent games, profile history
- [ ] Click player name in scoreboard → popup
- [ ] Select profile → see charts, history
- [ ] Link Discord → prompt + save + open
- [ ] Change player → return to empty state
- [ ] Paginate mercs/feuds
- [ ] Copy Top 10 from Feuds
- [ ] Responsive: test mobile layout
- [ ] Escape key closes modals
- [ ] Backdrop click closes modals

---

## Next Steps

1. **Set up your environment**: Install dependencies, configure API endpoints
2. **Break into components**: Tabs, Table, Modal, StatCard, Chart, ProfileHero, etc.
3. **Implement state management**: Use React Context, Redux, Zustand, or your preferred solution
4. **API integration**: Replace `fetch()` calls with your API client
5. **TypeScript types**: Define `Guild`, `Match`, `Player`, `Merc`, `Feud` interfaces
6. **Accessibility audit**: Add ARIA labels, focus management, keyboard nav
7. **Performance optimization**: Memoization, virtualization, lazy loading
8. **Testing**: Unit tests for formatters, integration tests for flows

If you have questions or need clarification on any design detail, ask before implementing. Good luck!
