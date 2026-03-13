# Japan 2026 Trip Planner

## Project Overview

A single-file, mobile-first HTML web app for a group trip to Japan (March 13–22, 2026). Hosted on GitHub Pages at `https://jwr456.github.io/Japan-2026/`. Password-gated (password: `ppfdhurryup`, hashed client-side).

The entire app lives in **`index.html`** — one self-contained file with inline CSS + vanilla JS. No build step, no frameworks, no dependencies beyond two CDN imports (Leaflet.js for maps, Google Fonts for typography).

## Architecture

### Single File Structure

```
index.html
├── <style>        — All CSS (~120 lines), mobile-first, max-width 480px
├── <body>
│   ├── .lock      — Password gate overlay
│   └── .app       — Main app container (hidden until authenticated)
│       ├── .tabs   — Sticky tab bar: 📅 Calendar | 🗼 Tokyo | ⛩ Kyoto | ✈️ Travel
│       ├── #pane-calendar — Calendar grid + drill-down detail cards
│       ├── #pane-tokyo    — Leaflet map + filtered POI legend
│       ├── #pane-kyoto    — Leaflet map + filtered POI legend
│       └── #pane-travel   — Flight + Shinkansen cards
└── <script>       — All JS (~400 lines), vanilla, no transpilation
```

### Key Data Structures

All data is inline in the `<script>` block. No external JSON files.

**`DAYS[]`** — Array of 8 day objects (Mar 14–21). Each day has:
- `date` (ISO), `dow`, `label`, `badge` (push/hold/flex), `city` (tokyo1/tokyo2/kyoto)
- `wx` — Weather: `{ic, hi, lo, rain, d, live?}` (temperatures in °F, rain in mm)
- `sakura` — Optional cherry blossom note (string, only on days 19–21)
- `meals[]` — Array of meal objects: `{type, name, cuisine, price, attire, addr, lat, lng, hrs, note, bk}`
- `pm[]` — Optional afternoon activity POIs: `{n, d, lat, lng}`
- `after[]` — After-dark suggestions: `{n, d, lat, lng}`
- `sleep` — Sleep strategy note (string)

**`HOTELS[]`** — Accommodation cards: `{c (city key), n, sub (who), d (date range), x (nights)}`

**`TPOI[]` / `KPOI[]`** — Map POI arrays for Tokyo/Kyoto. Seeded with hotels and sakura spots, then dynamically extended from DAYS data on load. Each POI: `{n, lat, lng, t (type), s (city segment), days[], who?, tip?}`

**POI types (`t`):** `hotel`, `rest`, `pm`, `night`, `sakura`

**City segments:** `tokyo1` (Akasaka, first visit), `tokyo2` (Ginza, second visit), `kyoto`

### Color System (CSS variables)
- `--ink` (#1a1a2e) — Text, hotel markers
- `--t1` (#e67e22) — Tokyo I (orange)
- `--t2` (#8e44ad) — Tokyo II (purple)
- `--ky` (#27ae60) — Kyoto (green)
- `--push` (#c0392b) — Push Late sleep badge
- `--hold` (#2980b9) — Hold Steady sleep badge
- `--flex` (#d4a017) — Flex sleep badge
- Sakura POIs: #e91e63 (pink)
- Afternoon POIs: #d35400 (burnt orange)
- Nightlife: #999 at 40% opacity

### Key Functions

| Function | Purpose |
|----------|---------|
| `unlock()` | Password check → show app, persist to sessionStorage |
| `initApp()` | Set up tabs, render calendar, init map panes |
| `renderCal()` | Build calendar grid + selected day's detail card |
| `pick(date)` | Select a day, re-render calendar |
| `switchTab(id)` | Switch between calendar/tokyo/kyoto/travel panes |
| `initMap(id, pois, center, zoom)` | Create Leaflet map instance |
| `updateMapMarkers(id, pois)` | Clear and redraw markers based on filter |
| `filterMap(id, date)` | Toggle day filter on map, re-render |
| `renderMapPanel(id, pois, isKyoto)` | Build day-filter buttons + legend + POI list below map |
| `renderTravel()` | Build flight + shinkansen cards |
| `fetchLiveWeather()` | Hit Open-Meteo API, update wx data, re-render |

### Live Weather Integration

On page load (`setTimeout(fetchLiveWeather, 500)`), the app fetches forecasts from the Open-Meteo API for both Tokyo and Kyoto. The API is free, needs no key, and supports CORS. It requests daily max/min temps (Fahrenheit), precipitation, and weather codes for the trip date range. Results silently overwrite the static `wx` data in DAYS and re-render the calendar. A `live:true` flag triggers a "⚡ Live forecast" badge in the UI. Static fallback data remains if the API is unreachable.

### Map Marker Types

| Type | Visual | Size |
|------|--------|------|
| `hotel` | Triangle (▲) | 14×12px div icon |
| `rest` | Filled circle | radius 7, color by city segment |
| `pm` | Filled circle | radius 8, burnt orange |
| `night` | Filled circle | radius 5, gray, 40% opacity |
| `sakura` | 🌸 emoji | 20×20px div icon |

### Google Maps Links

All POIs link to Google Maps search, which works natively on iOS/Android:
- Tokyo: `gm(name, lat, lng)` → `google.com/maps/search/{name}+Tokyo+Japan`
- Kyoto: `gmk(name)` → `google.com/maps/search/{name}+Kyoto+Japan`
- Address: `gmAddr(addr)` → `google.com/maps/search/?api=1&query={addr}`

## Trip Context

### Travel Party
- **Joel** — Trip organizer. Night owl (born 1980, based in New Orleans). Arrives one day late (Mar 15 via SQ11).
- **Omair, Ted, Raj** — Travel companions. Arrive Mar 14.
- **Shinji** — Local contact in Tokyo. Tentative dinner on the 20th.

### Accommodations (3 phases)
1. **Tokyo I (Mar 14–16):** Joel at Airbnb in Akasaka (2-chōme-20-15 Akasaka). Others at ANA InterContinental.
2. **Kyoto (Mar 17–18):** Everyone at Ace Hotel Kyoto (above Karasuma Oike Station).
3. **Tokyo II (Mar 19–21):** Joel at Hyatt Centric Ginza. Others at Imperial Hotel. (~800m apart, 10 min walk.)

### Transport
- **Outbound flight:** SQ11 LAX 1:25 PM Mar 14 → NRT 5:20 PM Mar 15, Business, 777-300ER
- **Shinkansen:** NOZOMI 251, Tokyo 10:00 → Kyoto 12:15, Mar 17, Green Car 9 (Seats 10-A, 11-B)
- **Return flight:** SQ12 NRT 6:15 PM Mar 22 → LAX 11:55 AM Mar 22, Business, 777-300ER

### Sleep Strategy
Joel is a night owl managing jet lag across the trip. Each day has a "badge" indicating the sleep approach:
- **Push Late** — Intentionally stay up late (2–3 AM), ride jet lag or build CDT shift for return
- **Hold Steady** — Anchor around 12:30–1 AM, particularly around big omakase dinners
- **Flex** — Flexible target, drift naturally

### Cherry Blossom Forecast (JMC, 8th forecast, Mar 12 2026)
- **Tokyo:** Flowering Mar 19, full bloom Mar 27 (5 days early)
- **Kyoto:** Flowering Mar 23, full bloom Apr 1 (3 days early)
- Sakura POIs on the Tokyo map are tagged to Mar 19–21

### Dining Highlights
- Two high-end omakase nights with enforced dress codes: **Sushi Masuda** (Mar 16, 8 PM) and **Sushi Kanesaka** (Mar 19). Both require smart casual — no T-shirts, shorts, sandals, or strong fragrances.
- Open/unbooked slots: Mar 18 lunch (Kyoto), Mar 19 lunch (transit day), Mar 21 (free day)
- Tentative: Dinner with Shinji on Mar 20 (possibly Masa Ishibashi in Ginza)

### Joel's Interests (for activity suggestions)
Retro Japanese gaming (Nintendo, Sega), Japanese consumer electronics (Sony, Panasonic), vintage toys (Transformers), animal cafes (owls, cats).

## Development Notes

### Editing Workflow
1. Edit `index.html` directly — it's one file
2. Commit and push to `main` branch
3. GitHub Pages auto-deploys within ~60 seconds

### Common Modifications

**Add a new restaurant/POI:**
1. Find the relevant day in `DAYS[]`
2. Add to `meals[]` (for restaurants) or `pm[]` (for activities) with `lat`, `lng`, `name`, etc.
3. POIs with coordinates are auto-added to the map via the dynamic build loop at line ~180

**Add a new sakura spot:**
Add to `TPOI[]` with `t:"sakura"` and appropriate `days[]` array.

**Change weather data:**
Modify the `wx` object on each day. Static data is overridden by live fetch when Open-Meteo is reachable. Temperatures are in °F.

**Add a new day:**
Add an object to `DAYS[]` following the existing schema. The calendar grid auto-renders from the array. If adding more than 8 days, the 4-column grid may need CSS adjustment.

**Add a new tab:**
1. Add `<div id="pane-{id}" class="pane"></div>` in the HTML
2. Add button to `initApp()` tab bar string
3. Add render function and call it from `switchTab()`

### Gotchas
- **No newlines in data strings** — all DAYS entries are single-line JS objects. Multi-line breaks the minified structure.
- **Map container divs** are created dynamically by `initApp()` inside `#pane-tokyo` and `#pane-kyoto`. Don't put static content in those panes.
- **Leaflet invalidateSize** — called after tab switch with a setTimeout to let the DOM render first. Without this, maps render at 0px height.
- **sessionStorage for auth** — password persists per browser session (cleared on tab close). Not localStorage.
- **POI deduplication** — the dynamic build loop can create duplicate POIs if a restaurant appears on multiple days. Currently not deduplicated (each day gets its own marker). This is intentional for day-filtering.
- **CSS is mobile-first** — max-width 480px container. Works on desktop but is designed for phone screens.
- **No build step** — just edit the HTML and push. No npm, no bundler, no transpiler.
