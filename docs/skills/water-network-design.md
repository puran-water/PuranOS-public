# Water Network Design Skill

`water-network-design` is the proposal-stage standard operating procedure for municipal potable-water tender pursuits. It is the first skill in the PuranOS catalog that crosses both `gis-mcp` (spatial reasoning) and `hydraulic-mcp` (network hydraulics) end-to-end, producing a draft EPANET model and the geographic evidence that justifies its source selection.

## Why This Exists

Municipal water-supply tenders demand a defensible chain of reasoning from "we will draw water from here" all the way through "this is the network that delivers it." Historically that reasoning is fragmented across desktop GIS sessions, paper hydraulic calculations, and Excel CAPEX rollups, with no provenance carried between them. The deterministic gates fail the moment a basis-of-design assumption changes upstream.

`water-network-design` is the skill that makes the chain auditable: every candidate intake carries the raster evidence that drove its confidence band, every transmission corridor carries the DEM + roads + ecological cost weights that produced it, every pipe carries the demand allocation and EPANET solve that sized it, and the final CAPEX rollup carries the km-of-pipe by DN by right-of-way breakdown that feeds the bid sheet.

## Scope

This skill is for the **proposal stage** — pre-award, pre-detail-engineering. It produces:

- a draft EPANET `.inp` and steady-state + EPS solve, with explicit pressure-driven mode and minimum-pressure / required-pressure settings
- ranked candidate intake set with raster-backed permanence and land-cover evidence
- transmission-corridor alternatives over a least-cost-path surface
- pressure-zone overlays with PRV and booster placement recommendations
- a CAPEX km-of-pipe rollup by DN by right-of-way type
- a Google-Earth-openable KMZ for the field team plus a stakeholder Folium HTML and a proposal-document matplotlib PNG of the same feature set
- easement / right-of-way conflict register against OSM landuse and ecological pre-screen layers

It is **not** a substitute for detail engineering. The deliverable is what a senior PE can hand to a downstream design team with explicit provenance on every number — not a turn-key construction set.

## Reasoning Arcs

The skill executes along four arcs. Each arc has explicit gates and produces persisted artifacts on the project filesystem.

### Arc 1 — AOI and Data Staging

- Resolve the project AOI from operator-provided GeoJSON or from a centroid plus buffer
- Stage OSM roads, buildings, landuse, waterways via Geofabrik PBF extraction (single-pass, cached per AOI)
- Stage DEM tiles (Copernicus GLO-30 by default; FABDEM env-gated for commercial bare-earth)
- Stage WorldPop 100m gridded population (year ≤ 2030; later years require an explicit demographic-authority citation)
- Stage ecological pre-screen layers: KBA + GBIF density (WDPA optional with commercial license)
- Stage JRC GSW v1.4 tiles (occurrence + seasonality + recurrence + transitions) for any AOI not already covered

The data-quality gate is explicit: low-coverage AOIs trigger a `LOW_COVERAGE` flag that prevents downstream stages from proceeding without operator override.

### Arc 2 — Source-Water Siting

- Read the tender corpus for any prescribed source via `wiki-project` MCP — tender-anchored sites are rank-1 candidates, not optional reference data
- Snap the tender intake (or operator-supplied seed point) to HydroRIVERS via `gis_river_upstream_of`; walk upstream within a configured distance, returning the selected reach plus densified candidate-node points plus branch alternatives where tributary ambiguity is high
- For each candidate node sample JRC GSW in a buffered window (typically 120 m radius) — occurrence p95, seasonality p95, dominant transition class, valid-water-pixel count
- For each candidate node sample ESA WorldCover at 10 m resolution to classify the bank
- Apply the confidence-upgrade rule deterministically in the orchestrator (not in any tool):
  - HIGH when `(occurrence_p95 ≥ 80 OR occurrence_max ≥ 90) AND (seasonality_p95 ≥ 10 OR transition_class ∈ {permanent, new_permanent, seasonal_to_permanent}) AND n_valid_water_pixels ≥ 5`
  - LOW when `occurrence_p95 < 30 AND transition_class not in the permanent set`
  - MEDIUM otherwise
- Stamp the split field-verification flags onto each candidate's provenance: `requires_field_verification_for_water_availability` is set false only when the rule fires HIGH; `requires_field_verification_for_bank_stability_bathymetry_access` is always true (raster evidence cannot answer it)

The output is a ranked candidate intake set with raster numbers attached to each pin's description.

### Arc 3 — Network Synthesis

- Derive a candidate pipe network from OSM road centerlines and the demand surface (population-calibrated against WorldPop)
- Allocate demands to junctions per population density × per-capita-demand × peak-hour factor
- Site treatment-plant candidate(s) via multi-criteria scoring over the AOI
- Compute least-cost transmission corridor from source to plant and plant to network entry via `hydraulic_least_cost_corridor` — DEM + road preference + ecological avoidance + max-slope constraint
- Build a WNTR `WaterNetworkModel`, write the EPANET 2.2 `.inp`, and solve steady-state plus a 24-hour EPS in pressure-driven mode
- Identify pressure-zone boundaries from elevation bands; place PRVs and booster pumps where the hydraulics demand
- Run a fire-flow screen at representative junctions
- Size storage per AWWA M32 (equalization + fire reserve + emergency)
- Size the pump station(s) per `hydraulic_size_pump_station` with annual OpEx rollup

The EPANET 2.2 GUI smoke test is mandatory before declaring handoff complete: the `.inp` is re-emitted with a `Target-EPANET-Version: 2.2` header comment, and the downstream design engineer opens it in standalone EPANET 2.2 to compare five sampled junction pressures against the WNTR result within 1 % tolerance.

### Arc 4 — Stakeholder Output

- `gis_render_kmz` produces two KMZs: an intake-decision KMZ (tender default + Niger-upstream candidates + mine polygons + urban outfalls + transmission corridor + AOI envelope) and a network-overlay KMZ (junctions colored by peak pressure, pipes weighted by diameter and colored by max velocity, tanks / reservoirs / pumps / valves with distinct icons, PNG legend ScreenOverlays embedded)
- `gis_render_proposal_map` produces matplotlib PNG / PDF for the proposal document and Folium HTML for SharePoint stakeholder review
- `hydraulic_pipe_km_rollup` produces the CAPEX driver JSON: km of pipe by DN by right-of-way type (in-road, off-road, watercourse-crossing, etc.)

Every deliverable carries the per-feature provenance contract — source, query, license, spatial accuracy, confidence band, and the split field-verification flags — visible in the KMZ placemark descriptions and the HTML popups.

## OSS Stack

The skill operates entirely on free / anonymous-read OSS data:

- **OpenStreetMap** via Geofabrik PBF + Overpass (ODbL)
- **Copernicus DEM GLO-30** (free worldwide; DLR/Airbus attribution)
- **WorldPop** (CC-BY 4.0)
- **HydroSHEDS / HydroRIVERS** (HydroSHEDS Data License Agreement; free for non-commercial)
- **JRC Global Surface Water v1.4** (Copernicus Programme; free with acknowledgement)
- **ESA WorldCover v200** (CC-BY 4.0)
- **KBA / GBIF** for ecological pre-screen (open APIs; WDPA optional with commercial license)
- **WNTR + EPANET 2.2** (BSD-3-Clause + US EPA public-domain) for hydraulics

## Hard Failures

The skill is deliberately strict about data quality. Two failure modes are non-negotiable:

- **LOW_COVERAGE** without operator override: if the OSM building density, road density, DEM completeness, or WorldPop coverage falls below the configured threshold for the AOI, the skill halts. Forcing through requires an explicit operator override flag, which is logged in the project artifacts.
- **Demographic authority for years past 2030**: WorldPop's Global_2000_2020 dataset is the projection authority through 2030. For longer-horizon population growth, the skill requires an explicit demographic-authority citation (e.g. national census bureau projection, World Bank dataset reference) persisted before any downstream demand calculation runs.

## Cross-Reference

- `gis-mcp` — the spatial-reasoning surface; see `docs/servers/gis-mcp.md`
- `hydraulic-mcp` — the network-hydraulics surface; see `docs/servers/hydraulic-mcp.md`
- `engineering-engines.md` — the broader PuranOS perspective on first-class simulation infrastructure
