# Hydraulic MCP Server

`hydraulic-mcp` is the municipal-water-distribution and pump-station sizing surface for PuranOS. It wraps WNTR (the Sandia Water Network Tool for Resilience) around EPANET 2.2 and exposes the resulting solver, sizing toolkit, and GIS-export pipeline as typed MCP tools. The default contract is stateless: pass an `.inp` path, get a result-artifact path back.

## Why This Exists

Municipal water-supply tenders require defensible hydraulic modelling — pressure at every junction, flow in every pipe, fire-flow residual, storage sizing, pump-station OpEx, transmission-corridor selection — all with provenance that survives the handoff to a downstream design engineer who will open the model in standalone EPANET 2.2 GUI. PuranOS treats hydraulic computation the same way it treats every other simulation: API-first, session-persistent only when an iteration loop demands it, and produces results with explicit credibility metadata.

## What The Server Does

### Core Stateless Tools

- `hydraulic_load_model(inp_path)` — parse a WNTR `.inp` and return a topology summary (junction / pipe / pump / tank / reservoir / valve counts, total demand, options, topology hash)
- `hydraulic_simulate(inp_path, sim_type, options)` — stateless solve. Supports steady-state and extended-period simulation, demand-driven (DDA) or pressure-driven (PDD) modes. Persists results as HDF5 or pickled DataFrames; returns the result artifact path plus a summary
- `hydraulic_scenario_run(inp_path, scenario_patch, out_inp_path, out_result_path)` — apply a scenario patch (demand multipliers, pipe diameter swaps, pump speed changes, Hazen-Williams roughness overrides), persist the patched `.inp`, solve
- `hydraulic_scenario_compare(result_path_a, result_path_b)` — file-based delta between two result artifacts: per-node pressure delta, per-pipe flow delta, global extremes
- `hydraulic_query_pressure_at_node` / `hydraulic_query_flow_in_pipe` / `hydraulic_query_node_pressure_grid` — focused reads off a persisted result artifact
- `hydraulic_export_inp(inp_path, out_path, target_version="2.2")` — re-emit `.inp` with a target-version header for the downstream EPANET 2.2 GUI smoke test that PuranOS requires before declaring a handoff complete

### GIS Export

- `hydraulic_to_geojson(inp_path, out_geojson_path?, include_results?)` — round-trip via `wntr.network.to_gis` to emit GeoJSON FeatureCollections for junctions, pipes, pumps, tanks, reservoirs, valves. When `include_results` points at a trusted result artifact, joins simulation-result scalars onto each feature's properties: `pressure_peak_m`, `pressure_min_m`, `pressure_t0_m`, `flow_peak_lps` (signed), `flow_abs_peak_lps`, `velocity_max_mps` (derived from flow and pipe diameter), plus scalar timestamps `pressure_peak_time` / `velocity_max_time`. Every feature carries `element_type` and `element_id` for downstream styling, plus a full `_provenance` block including the result-artifact path, simulation timestamp, and units provenance. Path-restricted: the result loader refuses pickles outside the project workspace or the hydraulic-mcp cache root. ID-join validation against `.inp` element IDs raises on mismatch rather than silently dropping.

### Sizing Tools

- `hydraulic_size_storage` — AWWA M32 storage sizing combining equalization, fire-flow reserve, and emergency components
- `hydraulic_size_pump_station` — pump-station hydraulic + electrical + annual OpEx rollup with configurable duty and standby counts
- `hydraulic_fire_flow_screen` — single-hydrant fire-flow screen with residual-target threshold and deficit-node enumeration
- `hydraulic_least_cost_corridor` — least-cost-path transmission corridor over a DEM with road-preference and ecological-avoidance weighting plus a max-slope constraint
- `hydraulic_pipe_km_rollup` — km-of-pipe by DN by right-of-way type for the CAPEX driver in a proposal

### Optional Session Cache

- `hydraulic_session_open` / `hydraulic_session_close` — opt-in TTL-bounded in-memory cache for what-if iteration loops against the same base `.inp`. Use sparingly; the default contract is stateless

## Upstream OSS Stack

- **WNTR** (BSD-3-Clause) — Sandia National Laboratories Water Network Tool for Resilience
- **EPANET 2.2** (public-domain reference engine) — US EPA
- **NetworkX** (BSD-3-Clause) — graph utilities for topology reasoning
- **pandas / numpy / shapely / pyproj / rasterio** — the standard PyData geospatial stack

No commercial license is required. The downstream design engineer's smoke test runs in EPA's official EPANET 2.2 GUI installer, which is publicly distributed.

## Per-Feature Provenance

Same contract as `gis-mcp`. Every GeoJSON feature emitted by `hydraulic_to_geojson` carries `_provenance` with `source`, `source_url_or_path`, `query`, `retrieved_at`, `as_of`, `license`, `method`, `spatial_accuracy_m`, `confidence`, `confidence_reason`, and the split field-verification flags. Element-level fields (`element_type` ∈ `{junction, pipe, tank, reservoir, pump, valve}` and `element_id`) are stamped uniformly so downstream styling and analysis tools can dispatch on them without hardcoding hydraulic semantics.

## How Hydraulic MCP Composes With GIS MCP

The two servers cooperate cleanly on the boundary: hydraulic-mcp owns WNTR-result interpretation, gis-mcp owns KML rendering and spatial reasoning. A municipal-water proposal stage composes them in two reasoning arcs:

1. **Network rendering**: `hydraulic_simulate` produces a result artifact, then `hydraulic_to_geojson(include_results)` emits enriched per-element GeoJSON, then `gis_render_kmz` styles junctions by peak pressure and pipes by max velocity, with PNG legend ScreenOverlays embedded in the KMZ. The result is a Google-Earth-Web-openable network overlay that a project review meeting can rotate around in real time.
2. **Intake-decision rendering**: a separate orchestrator combines the tender-default intake, the HydroRIVERS-derived Niger-upstream candidate intakes, the SAG / mine polygons from `gis_overpass_query(preset="mine")`, the JRC GSW permanence overlay, the least-cost transmission corridor from `hydraulic_least_cost_corridor`, and the urban-outfall layer into a single styled KMZ for the field team.

The hand-off between the two servers is a JSON-serializable feature collection. Neither server depends on the other's process; both are stateless by default.

## Out of Scope

- EPANET-MSX integration for disinfectant residual + chlorine decay — there is a placeholder hook (`hydraulic_water_age`) for v2 work.
- Sub-network parallelisation for very-large transmission grids — current scale (single tank + a few thousand pipes) solves in seconds; larger problems would need a different decomposition strategy.

## Cross-Reference

- `gis-mcp` — the spatial-reasoning peer; see `docs/servers/gis-mcp.md`.
- `engineering-engines.md` — the broader PuranOS perspective on first-class simulation infrastructure.
