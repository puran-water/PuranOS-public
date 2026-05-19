# GIS MCP Server

`gis-mcp` is the spatial-reasoning surface for PuranOS. It exposes typed tools for project-area-of-interest analysis, world-scoped geocoding and gazetteer lookup, river-network topology reasoning, raster permanence sampling, and KMZ export for Google Earth handoff. All backends are anonymous-read OSS endpoints — no commercial license is required to operate the server end-to-end.

## Why This Exists

Industrial water-supply and infrastructure tenders require defensible geospatial reasoning that historically depended on either pay-per-call commercial APIs (Google Maps Platform geocoding/places) or commercial petabyte-scale raster platforms (Google Earth Engine, AWS Sentinel commercial tier). PuranOS treats geospatial reasoning the same way it treats engineering computation: a deterministic, typed, session-aware surface that an agent can call as part of a structured skill.

`gis-mcp` closes the planetary-raster reasoning gap that Earth Engine fills — for the specific subset of questions an industrial-water proposal stage agent needs to answer — using only OSS data sources that allow anonymous read.

## What The Server Does

The tool surface is organized in three groups, each addressing a different reasoning scope.

### Group A — Project-AOI Vector and Raster Queries

Operate against a pre-clipped project area of interest staged on the project filesystem. Used for the "what is inside this project's geographic envelope" class of question.

- AOI lifecycle: load or create an AOI bounding box from project metadata or an operator-provided GeoJSON
- Vector queries within a buffered alignment: OSM roads, OSM/Overture buildings, OSM landuse polygons
- Raster sampling: DEM profile along a linestring (Copernicus GLO-30 by default; FABDEM optional, license-gated for commercial deliverables)
- Population analysis: WorldPop 100m gridded population zonal sum within a polygon
- Ecological pre-screen: KBA + GBIF intersection along a candidate alignment, plus optional WDPA when a commercial license is configured
- Stakeholder map rendering: matplotlib + contextily PNG/PDF for proposal documents, or Folium interactive HTML for SharePoint review

### Group B — World-Scoped Search and Reasoning

Operate without a pre-loaded AOI. Used for the "where on Earth is X" and "what is around Y" class of question.

- Forward geocoding with transliteration-fuzzy fallback: OSM Nominatim is the primary backend; on miss within a region-bias bounding box, falls back to Overpass `name~regex,i` with diacritic-stripped stems and Jaro-Winkler ranking. This recovers from cases where the corpus spelling differs from the OSM tag (a common failure mode in francophone West Africa, for example).
- Reverse geocoding via Nominatim
- Tagged-feature search via OSM Overpass with a curated preset library (`mine`, `settlement`, `water_intake`, `drain`, `industrial`, `power_plant`, `urban_area`, river name-regex search) plus raw Overpass-QL when the agent needs full flexibility
- Structured-knowledge gazetteer via Wikidata SPARQL with optional parent-organization fallback, tagged explicitly so callers cannot mistake company metadata for asset metadata
- River-network topology: HydroRIVERS Africa shapefile with NEXT_DOWN adjacency traversal to walk upstream from a snap point, returning a selected reach plus densified candidate-node points plus branch alternatives where tributary ambiguity is high
- Spatial primitives via shapely: nearest-feature lookup, buffer/intersect/within/exclude filtering, with metric buffer round-trips through local UTM

### Group C — Raster Permanence and Land Cover

The Earth-Engine-equivalent layer for the questions an industrial-water proposal stage actually needs to answer at planet scale.

- JRC Global Surface Water v1.4 (Pekel et al. 2021): occurrence (% of time water present 1984–2021), seasonality (months per year), recurrence (inter-annual %), transitions (categorical change classification). Sampled in a buffered window — typically 90 to 150 metres radius — to handle 30 metre mixed-pixel reaches on narrow rivers
- ESA WorldCover v200 (10 m, 11 classes) for intake-bank land-cover classification: permanent water, built-up, cropland, tree cover, bare ground, and so on
- Preflight tile-cache tool that bulk-downloads required JRC GSW tiles before the first sampling call, avoiding network-I/O blocking inside the agent's reasoning loop
- KMZ export with attribute-based styling, matplotlib LUT colormaps, PNG legend ScreenOverlays, and automatic foldering for large networks — produces Google-Earth-Web-openable artifacts for human field-team handoff

The deliberate omission is Sentinel-2 monthly time series (NDWI / NDTI / NDVI for turbidity and surface-water-extent tracking) — this is captured under "Out of Scope, Forward Paths" below.

## OSS Backends and Licenses

Every dataset and library in the stack is anonymous-read, with these license requirements:

- **OpenStreetMap** (Nominatim, Overpass, Geofabrik) — Open Database License (ODbL). Usage policies require a truthful User-Agent and reasonable request rates; the server applies module-level throttling per backend.
- **Wikidata** — CC0; SPARQL endpoint is open.
- **HydroSHEDS / HydroRIVERS** — HydroSHEDS Data License Agreement; free for non-commercial use and many commercial uses with attribution.
- **JRC Global Surface Water** — Copernicus Programme data; free with acknowledgement.
- **ESA WorldCover** — CC-BY 4.0.
- **Copernicus DEM (GLO-30)** — free worldwide, DLR/Airbus attribution required.
- **WorldPop** — CC-BY 4.0.

No commercial API key is required for any production code path. A FABDEM (bare-earth DEM) license can be wired in via an environment-gated commercial-license flag for projects that require sub-canopy elevation accuracy.

## Per-Feature Provenance Contract

Every output feature carries a `_provenance` block in its `properties`. The contract is uniform across all three tool groups:

- `source` — backend identifier (e.g. `osm_overpass`, `wikidata`, `jrc_gsw_v1_4_pekel_2021`, `esa_worldcover_v200_2021`)
- `source_url_or_path` — the actual URL or local file consulted
- `query` — the literal Overpass-QL / SPARQL / point coordinates / bbox used
- `retrieved_at` and `as_of` — ISO timestamps for the query and the underlying dataset epoch
- `license` — the dataset license, cited verbatim
- `method` — the analysis method (`direct`, `diacritic_stripped_stem_regex`, `rasterio_buffered_window_p95`, `hydrorivers_next_down_traversal`, etc.)
- `spatial_accuracy_m` — numeric uncertainty when known
- `confidence` and `confidence_reason` — qualitative band plus the numeric basis
- Split field-verification flags: `requires_field_verification_for_water_availability` (raster-answerable) and `requires_field_verification_for_bank_stability_bathymetry_access` (always true — raster evidence cannot answer it)

The split field-verification design exists because raster evidence can confirm "the river runs here in the dry season" but cannot confirm "the bank is stable, deep enough, accessible, and within a permit boundary." Conflating those two questions under a single flag was an early design mistake; the split keeps the contract honest.

## How A Proposal Stage Uses GIS MCP

A municipal-water tender pursuit calls `gis-mcp` along three reasoning arcs:

1. **Site discovery**: forward-geocode the city, load the AOI, query OSM roads and landuse, run the ecological pre-screen. Output is the project envelope and its sensitive-area exclusions.
2. **Source-water siting**: walk upstream from the tender-default intake using HydroRIVERS topology, sample JRC GSW permanence in a buffered window at each candidate node, classify the surrounding land cover with ESA WorldCover, exclude candidates that intersect urban-outfall buffers or mining-permit buffers via the spatial-filter primitive. Output is a ranked list of defensible Niger / Tinkisso / equivalent candidate intakes, each carrying the raster evidence that drove its confidence band.
3. **Handoff**: render the candidate set plus the transmission corridor and tender-default option as a styled KMZ. The field team opens it in Google Earth Web; the engineering team opens the same feature collection rendered as a Folium HTML or a matplotlib PNG inside the proposal document.

## Out of Scope, Forward Paths

These are explicitly not in the current server and are deferred to future plans:

- **Sentinel-2 NDWI / NDTI / NDVI monthly time series** — would answer the upstream-mining-turbidity-in-rainy-season risk question that the current Tier-A permanence layer cannot. Element84 STAC + AWS open-data buckets are anonymous-read, so the OSS path stays viable when the work lands.
- **Sentinel-1 SAR flood-extent mapping** — cloud-penetrating, useful for tropical-rainy-season inundation.
- **Earth Engine commercial license** — the OSS substitutes cover the highest-value subset of questions at zero license cost. Earth Engine would unlock more capability but is explicitly off the table today.

## Cross-Reference

`gis-mcp` composes with `hydraulic-mcp` — the hydraulic server's `hydraulic_to_geojson` tool emits per-element-typed GeoJSON with optional simulation-result attributes, and `gis-mcp`'s `gis_render_kmz` consumes that GeoJSON to produce Google-Earth-ready EPANET network overlays. The boundary is clean: hydraulic-mcp owns WNTR-result interpretation, gis-mcp owns KML rendering and spatial reasoning. Neither server hardcodes the other's domain semantics.
