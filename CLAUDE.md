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
- Driver sidebars: if API returns 0 but static `stats` has a value > 0, the static value is shown. Team sidebars: display `Math.max(api, TEAM_DATA.careerStats)` per stat. All-zero API results are never cached (likely rate limit).
- **Championships come from static data only**: `drivers/{id}/driverStandings` and `constructors/{id}/constructorStandings` without a season return HTTP 400 (Jolpica now requires `season_year`). Do not re-add those fetches.
- Qualifying data in the API only exists from ~1994, so `constructors/{id}/qualifying/1` undercounts poles for old teams — that's what the static `careerStats.poles` baseline compensates for.
- 2026 race results use simplified statuses: `Finished` / `Lapped` / `Retired` (not `+1 Lap` / cause-specific).
- Schedule country values include `UK` and `UAE` — both must exist as keys in `FLAGS`.
- Race done/next status uses `raceStart(r)` (`r.time` from API, fallback 14:00 UTC) + 2 hours. The **next race** is defined in exactly one place (`init()`: first race whose start+2h hasn't passed) and passed as `nextRound` to the timeline and calendar — don't reintroduce per-section date windows.

## HTML structure (f1_2026_v4.html)

The file has three logical sections:

1. **CSS** (`<style>`, lines ~1–560) — all styles, CSS variables, responsive layout
2. **HTML** (`<body>`, lines ~561–665) — page skeleton: nav, section panels, three sidebars
3. **JavaScript** (`<script>`, lines ~666–2100):
   - `init()` — fetches live standings + race results from API, renders the page
   - `TEAM_DATA` / `DRIVER_DATA` — static data objects (championships, career stats, team history)
   - `openSidebar()` / `openTeamSidebar()` / `openRaceSidebar()` — sidebar open logic
   - `fetchDriverCareerStats()` / `fetchTeamStats()` — wins/poles/podiums from Jolpica API
   - `fetchRaceDetails()` — qualifying + race results from Jolpica API (cached in `raceCache`; empty responses are not cached)
   - `renderStandingsTables()` — full driver + constructor tables in the Standings section
   - `liveNumbers` — driverId → car number from the API (filled in `renderDrivers`, used by team sidebar chips)
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
- `jolpicaId` — constructor ID used in API calls (2026 IDs: `mercedes, ferrari, mclaren, red_bull, alpine, rb, haas, audi, williams, aston_martin, cadillac`)
- `careerStats` — `{wins, poles, podiums}` static baseline (snapshotted from API 2026-07-10; for Racing Bulls it covers the whole Minardi→Toro Rosso→AlphaTauri→RB lineage). Displayed as `Math.max(api, baseline)`.
- `color` — team hex color; must match `TEAM_COLOR` (single source of truth for card + sidebar colors)
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
