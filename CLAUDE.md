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
f1_2026_v4.html     — entire app (CSS + HTML + JS, ~1900 lines)
photos/
  drivers/          — driver portraits (.webp)
  cars/             — car renders (.webp)
  logos/            — team logos (.webp)
serve.py            — simple HTTP server
```

## API

Live data comes from **Jolpica F1 API**: `https://api.jolpi.ca/ergast/f1`

- Rate limit: ~4 requests/second. Always add a **static fallback** for every API-fetched value — if the API returns 0 (rate-limited empty response), show the static value instead.
- The static fallback pattern is already used for career wins, poles, and podiums in driver sidebars.

## HTML structure (f1_2026_v4.html)

The file has three logical sections:

1. **CSS** (`<style>`, lines ~1–490) — all styles, CSS variables, responsive layout
2. **HTML** (`<body>`, lines ~491–600) — page skeleton: nav, section panels, sidebars
3. **JavaScript** (`<script>`, lines ~601–1896):
   - `init()` — fetches live standings + race results from API, renders the page
   - `TEAM_DATA` / `DRIVER_DATA` — static data objects (championships, career stats, team history)
   - `openSidebar()` / `openTeamSidebar()` — sidebar open logic with async API calls
   - `fetchDriverCareerStats()` — wins/poles/podiums from Jolpica API
   - `fetchDriverChampionships()` — WDC years from Jolpica API

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
- `constructorChamps` — array of `{year, driver, pts}` (leading driver + their points that season)
- `driverChamps` — array of `{year, driver}` (WDC winners driving for this team)

## Sections / tabs

- **Drivers** — championship standings grid with driver cards, clickable sidebars
- **Teams** — constructor standings grid, clickable team sidebars
- **Calendar** — race schedule with track stats, results for completed rounds
- **Standings** — full driver + constructor championship tables

## Photos

All images are `.webp`. Naming conventions:
- `photos/drivers/{FirstName}_{LastName}.webp`
- `photos/cars/{TeamName}.webp` (e.g. `Red_Bull.webp`)
- `photos/logos/{TeamName}.webp`
