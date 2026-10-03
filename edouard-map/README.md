# Edouard Rainfall Map

Interactive map of Tropical Storm Edouard (Sep 1–2, 2026) storm total rainfall over Hardin, Jefferson, Orange, Tyler and Polk counties, with land cover, hydrography and the Big Thicket preserve underneath.

`southeast-texas-edouard-map.html` — one self-contained file. Open it by double-clicking; it needs an internet connection because every layer is pulled live from a public service. Nothing to install, no build step.

## What is on it

| Layer | Source | Service and layer |
|---|---|---|
| Storm total rainfall | NWS MRMS radar QPE, via the Iowa Environmental Mesonet archive | `mesonet.agron.iastate.edu/cache/tile.py/1.0.0/`, layers `mrms::p72h-202609041200` + `mrms::p24h-202609011200` |
| Gauge storm totals | NWS post-storm reports, both offices covering this area | `PSHLCH_2026AL05_Edouard_Rainfall.csv` (Lake Charles) + `PSHHGX_2026AL05_Edouard_Rainfall.csv` (Houston/Galveston), 128 stations, built into the file |
| Land cover | MRLC NLCD 2021 | WMS `www.mrlc.gov/geoserver/mrlc_display/wms`, layer `NLCD_2021_Land_Cover_L48`; shown either as all 17 classes or as developed vs undeveloped |
| Rivers and streams | USGS NHD | `basemap.nationalmap.gov/arcgis/rest/services/USGSHydroCached/MapServer`, the cached cartographic rendering |
| Big Thicket National Preserve | NPS Land Resources Division | boundary feature service, `UNIT_CODE = 'BITH'` |
| County boundaries | Census TIGERweb | State_County layer 1, FIPS 48199, 48245, 48361, 48457, 48373 |
| Gauge KOUT2 | USGS 08041500 / NOAA NWPS | live stage read from `api.water.noaa.gov/nwps/v1/gauges/KOUT2` |
| Basemaps | USGS The National Map, Esri | blank (default), Esri World Light Gray Canvas, USGSTopo, USGSImageryOnly |

Every layer name above was verified against the service's own GetCapabilities or service metadata, not assumed. If a layer fails to load, the map says which one and why instead of failing quietly.

## The rainfall window

The radar layer covers **7 AM CDT Mon Aug 31 → 7 AM CDT Fri Sep 4**, matching the window the NWS used for its storm totals, so the radar and the gauge points describe the same storm. It is built by adding IEM's 72-hour total ending Sep 4 12Z to its 24-hour total ending Sep 1 12Z. The 24-hour totals immediately before and after that window show almost no rain over these counties.

MRMS tiles are colored images, not numbers. The page converts each pixel back to millimeters using IEM's published lookup table for `mrms_p72h` / `mrms_p24h`, then redraws it on an inch scale. That conversion was checked two ways: against IEM's raw data rasters (mean difference 0.06 mm over 262,144 pixels), and in the browser, where every tile pixel matched the table exactly.

## Two offices, two gauge files

These counties straddle a forecast office boundary, so the gauge totals come from two post-storm reports:

| Counties | Office | Issued | Stations |
|---|---|---|---|
| Hardin, Jefferson, Orange, Tyler | WFO Lake Charles (PSHLCH) | 2:10 PM CDT Sep 8, 2026 | 120 |
| Polk | WFO Houston/Galveston (PSHHGX) | 11:45 AM CDT Sep 4, 2026 | 8 |

Both files flag incomplete and estimated values; none of the 128 stations here carries a flag. Houston's report went out four days earlier than Lake Charles's and describes its totals as covering Aug 31 through Sep 3, one day shorter than the radar window.

**Polk sits at the western edge of the storm, so the window was checked there separately** (Polk box, 4,420 MRMS grid cells):

| 24 h period | Max | Mean |
|---|---|---|
| Aug 30 7 AM → Aug 31 7 AM (before the window) | 0.00 in | 0.00 in |
| Sep 3 7 AM → Sep 4 7 AM (the window's last day) | 2.51 in | 0.32 in |
| Sep 4 7 AM → Sep 5 7 AM (after the window) | 1.43 in | 0.04 in |
| Sep 5 7 AM → Sep 6 7 AM (after the window) | 4.08 in | 0.61 in |

Nothing is clipped at the start: no rain at all fell over Polk in the 24 hours before the window opens. Almost nothing falls in the day after it closes. The Sep 5–6 rain is a separate system — Edouard had dissipated by Sep 3 — and both the radar layer and the gauge totals correctly exclude it.

The one-day difference between Houston's gauge window and the radar window accounts for 0.05–0.60 in at the eight Polk stations, so the two are close enough to compare directly. Polk's wettest cell in the window is 16.93 in, near the Tyler County line.

## Known limits

- **The radar layer is capped at 19.5 in.** That is the top of IEM's value table, so the darkest class is open-ended and the readout says "at least" where it saturates. The highest gauge total was 24.36 in, at Little Pine Island Bayou at SH 787 (Hardin).
- **One gauge disagrees with everything near it.** `VILLAGE CREEK NEAR KOUNTZE 4NE` (HADS) reports 1.37 in inside an area where radar and neighboring gauges show 10 in or more. It is carried as published; treat it as suspect.
- **The gauge data is preliminary.** Both offices note their post-storm data are subject to correction.
- **Radar reads slightly high against gauges here**, median ratio about 1.24 across the Lake Charles counties and 1.19 in Polk, which is normal for radar-only QPE in tropical rain.
- **Three gauges around Livingston read well below both the radar and their neighbours.** `Livingston` (CWOP, 1.97 in), `4.2 W Goodrich (RR)` (HADS, 3.20 in) and `1.4 W Livingston` (HADS, 3.46 in) sit where radar shows about 6 in, while the two CoCoRaHS stations nearby report 6.52 and 8.18 in. This is not a window problem — the window's final day only added 0.2–0.3 in at those sites — so it looks like gauge undercatch or a short record. Treat the low Livingston values with the same caution as the Kountze one.

## Reading values out of a colour-coded tile

Two layers decode published map tiles back into the numbers behind them, using the colour table each service publishes. The rainfall layer turns MRMS tiles into inches; the land cover layer reads NLCD class colours and groups codes 21–24 into one developed class against everything else. Both were checked pixel by pixel against the source tables, so the map shows the published data regrouped, never a reclassified copy.

## A note on the NHD dynamic service

An earlier version drew the streams through the dynamic export endpoint `hydro.nationalmap.gov/arcgis/rest/services/nhd/MapServer/export`, lowering the request DPI so its scale-dependent flowline layers would still draw when zoomed out. Two problems showed up in use. Its two flowline layers swap at 1:73,220, so tiles near that boundary drew different amounts of detail and coverage looked patchy across a view. And the endpoint renders on demand: asked for a screenful of tiles at once it returns 502s and times out — on 23 Sep 2026 that cost roughly two thirds of the tiles, whether they were requested all at once or two at a time. `USGSHydroCached` carries the same NHD data pre-rendered and cached to zoom 23, which is why the map now uses it.

## Presenting it

The map opens on a blank ground so the data layers carry it; the basemap list also offers a plain light canvas, USGS topographic and aerial imagery. The **Reset view** button returns to the five-county extent. Each layer has a checkbox; rainfall and land cover have opacity sliders. Pointing anywhere reads the radar estimate in inches; clicking a gauge point compares that gauge against the radar above it. If the venue's network is unreliable, open the file beforehand and screenshot the key views as a backup.
