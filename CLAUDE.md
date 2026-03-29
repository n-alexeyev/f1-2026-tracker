# F1 2026 Season Tracker

Single-file web app for tracking the 2026 Formula 1 season.

## Active file

**`f1_2026_v4.html`** — the current version. Files `f1_2026.html`, `v2`, `v3` are old drafts, do not edit them.

## Running locally

```bash
python3 serve.py        # starts server on port 8788
```

Open: http://localhost:8788/f1_2026_v4.html

The `start.sh` script is outdated (points to v2), ignore it.

## Deploying

After every code change — commit and push to GitHub:

```bash
git add f1_2026_v4.html
git commit -m "..."
git push
```

GitHub Pages serves the live site automatically from the `main` branch.
Repository: https://github.com/n-alexeyev/f1-2026-tracker

## File structure

```
f1_2026_v4.html     — entire app (CSS + HTML + JS, ~2100 lines)
photos/
  drivers/          — driver portraits (.webp)
  cars/             — car renders (.webp)
  logos/            — team logos (.webp)
serve.py            — simple HTTP server
```

## API

Live data comes from **Jolpica F1 API**: `https://api.jolpi.ca/ergast/f1`

- Rate limit: ~4 requests/second. Always add a **static fallback** for every API-fetched value — if the API returns 0 (rate-limited empty response), show the static value instead.
- The static fallback pattern is used for career wins, poles, and podiums in driver sidebars. If API returns 0 but static data has a value > 0, the static value is shown.
- Race done/next status uses `r.time` (race start UTC time from API) + 2 hours, not just `r.date`. This correctly handles timezone differences (e.g. Japan races happen Saturday evening EU time but are dated Sunday JST).

## HTML structure (f1_2026_v4.html)

The file has three logical sections:

1. **CSS** (`<style>`, lines ~1–560) — all styles, CSS variables, responsive layout
2. **HTML** (`<body>`, lines ~561–665) — page skeleton: nav, section panels, three sidebars
3. **JavaScript** (`<script>`, lines ~666–2100):
   - `init()` — fetches live standings + race results from API, renders the page
   - `TEAM_DATA` / `DRIVER_DATA` — static data objects (championships, career stats, team history)
   - `openSidebar()` / `openTeamSidebar()` / `openRaceSidebar()` — sidebar open logic
   - `fetchDriverCareerStats()` — wins/poles/podiums from Jolpica API
   - `fetchDriverChampionships()` — WDC years from Jolpica API
   - `fetchRaceDetails()` — qualifying + race results from Jolpica API (cached in `raceCache`)
   - `CIRCUIT_DATA` — static object mapping `circuitId` → `{km, laps, rec, drv, yr}`

## Key data objects

### DRIVER_DATA
Each driver entry has:
- `name`, `team`, `number`, `nationality`, `hometown`
- `careerPoints` — static career points total (string)
- `stats` — `[{val, lbl}]` for Race Wins, Pole Positions, Podiums (static fallback)
- `championships` — array of WDC years (static fallback)
- `career` — array of `{years, team, detail, color, active}` timeline entries

### TEAM_DATA
Each team entry has:
- `name`, `base`, `principal`, `founded`
- `jolpicaId` — constructor ID used in API calls
- `color` — team hex color
- `driverIds` — array of driver keys matching DRIVER_DATA
- `constructorChamps` — array of `{year, driver, pts}` (top-scoring driver + their points that season)
- `driverChamps` — array of `{year, driver}` (WDC winners driving for this team)

### CIRCUIT_DATA
Static object keyed by Ergast `circuitId` (e.g. `albert_park`, `monaco`):
- `km` — circuit length in km (string)
- `laps` — number of race laps
- `rec` — lap record time string, or `null` for new circuits
- `drv` — driver who set the lap record
- `yr` — year the lap record was set

## Sections / tabs

- **Drivers** — championship standings grid with driver cards, clickable sidebars
- **Teams** — constructor standings grid, clickable team sidebars
- **Calendar** — race schedule with track stats; done + next cards are clickable → race detail sidebar
- **Standings** — full driver + constructor championship tables

## Sidebars

There are three independent sidebar panels, each with its own overlay:

| Sidebar | ID | Trigger |
|---|---|---|
| Driver | `#driver-sb` / `#sb-overlay` | Click driver card |
| Team | `#team-sb` / `#tsb-overlay` | Click team card |
| Race | `#race-sb` / `#rsb-overlay` | Click done/next calendar card |

**Race sidebar** shows:
- **Qualifying** (P1–P20): best session time (Q3/Q2/Q1), team colour dot; Q1 eliminees are dimmed
- **Race result** (done races only): finish position, position delta (▲/▼), status for DNF/lapped, points, fastest lap (purple highlight)
- Data fetched lazily on first click, cached in `raceCache` per round
- For the current "NEXT" race (qualifying done, race not yet run): shows qualifying grid only

## Photos

All images are `.webp`. Naming conventions:
- `photos/drivers/{FirstName}_{LastName}.webp`
- `photos/cars/{TeamName}.webp` (e.g. `Red_Bull.webp`)
- `photos/logos/{TeamName}.webp`
