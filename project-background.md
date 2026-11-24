# Tide Flood Map: Background

## What it is
Enter a tide height (plus optional "extra water") and see which ground floods on a satellite map. A plain-language verdict tells whether the pinned address is reached. Single file `index.html` (Leaflet, geotiff.js), runs entirely in the browser, hosted on GitHub Pages. Companion file `walls.geojson` holds known barriers and is auto-loaded from the repo.

## Why
Newport Harbor neighbors face king tides, El Niño extra water (strong El Niño through about JFM 2027; more Kelvin waves possible Nov-Jan) and storm surge. The City of Newport Beach is estimating about 22 in of extra water (Oct 2026); that is not an upper bound (past El Niños: 6-10 in sustained, storm peaks 1.5-3 ft above predicted).

## Data sources (all keyless)
- Imagery/roads: Esri World Imagery + transportation tiles (confirm tile terms for public use).
- Elevation: USGS 3DEP ImageServer exportImage (Float32 TIFF, parsed with geotiff.js) and EPQS point query; 1-2 m resolution.
- Tides: NOAA CO-OPS datagetter + mdapi. Predictions at Newport Bay Entrance 9410580 (6-min and hilo); live observed water level, air pressure, wind at Los Angeles 9410660 (about 35 miles away, a proxy).
- Alerts/forecast: api.weather.gov. Geocoder: OSM Nominatim (usage limits).

## Datums
Tide tables are MLLW; USGS elevation is NAVD88. At 9410580: NAVD88 3.51 ft, MLLW 3.33 ft, so `SCALE_GAP = 0.18`. Ground on tide scale = NAVD88 + 0.18 (read live from mdapi, fallback constant).

## Flood logic
- water = tide + extra/12 (extra in inches); worse = water + storm margin (None / +12 / +24 in).
- Colors: blue = forecast tide, orange = with extra water, red = worse case, no color = not reached.
- **Barriers hold** (default): 4-neighbor flood fill from the harbor source across connected ground (NoData passable). Low ground that is not connected is not flooded. **Barriers fail**: legacy bathtub shading (everything below the level).
- Water source: auto = lowest valid cell of the largest connected body of cells below 2 m; user can add a source on the map (Advanced section at bottom).
- Domains: close-up (~0.8 km, <=2000 px, ~1.3 m cells) or whole inner harbor (bbox s33.594 n33.625 w-117.924 e-117.872, <=3000 px, ~1.5 m cells); falls back to close-up with a visible message.
- Extra-water sources: City 22 in; live 24 h average (observed minus predicted at LA); live highest hour. Default = live highest if available, else City 22.

## Known Barriers
Sandbags, berms, walls. User draws a line (or loads GeoJSON) with `height_ft`; rasterized into a copy of the grid, raise-only (max), later lines win height assignment; only used in "Barriers hold". Drawn lines persist in localStorage `tfm_lines`. `walls.geojson` = one closed line around Balboa Island(s) at an assumed 9 ft (bridges intentionally crossed), so the island is not shown flooded when the City's sandbagging is assumed to hold. Height is editable in the app if the City raises it.

## Phone UI (<=760 px)
Map 34dvh; quick panel with tide tabs (Right now / Highest tides / Type it), extra-water stepper + chips, storm margin, mirrored verdict; bottom-sheet list of NOAA predicted highs >=6.5 ft over the next 90 days (max 60). 16 px inputs, 44 px targets, safe-area insets. Pin from `?lat=&lng=`, localStorage `tfm_home`, or default 64 Beacon Bay (33.61020749943019, -117.89731682863939). `?debug=1` exposes `window.__tfm`.

## Files
- `index.html`: the app (about 1500 lines).
- `walls.geojson`: barrier lines; must sit beside index.html in the repo.
- Repo: geoffdean1/tide_flood_map (public, no keys).

## Known limits
Elevation vintage and 1-2 m resolution; thin walls missed; storm drains, tidal valves, pumps not modeled; waves ignored; LA station is a proxy; barrier heights are user assumptions; planning aid, not an official forecast; public repo exposes the default home address; Nominatim and Esri tile terms; real-iPhone testing not done, 3DEP behavior on Balboa and the wide-area request untested against live service.

## Ideas / open items
- Laptop-first workflow; planned DJI Mini 2 / Litchi drone-elevation flight planner.
- Possible: save multiple pins, share links with level preset, better barrier editor on phone, tide-height-by-date picker, verifying wide-area load on real network.

## History notes
Removed gray "can't be reached" shading (looked like flooding). Replaced wall boxes with digitized barrier lines. Renamed "Known walls" to "Known Barriers"; barrier line color distinct from orange.
