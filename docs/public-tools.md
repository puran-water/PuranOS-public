# Public engineering tools

## Current standalone releases

| Repository | Calculation scope |
| --- | --- |
| [fluids-mcp](https://github.com/puran-water/fluids-mcp) | Pipe flow, pipe and valve sizing, fluid properties and bounded sweeps. |
| [heat-transfer-mcp](https://github.com/puran-water/heat-transfer-mcp) | Heat exchangers, air cooling, heat loss, insulation and thermal properties. |
| [water-chemistry-mcp](https://github.com/puran-water/water-chemistry-mcp) | PHREEQC speciation, mixing, saturation and chemical-dosing screens. |
| [corrosion-engineering-mcp](https://github.com/puran-water/corrosion-engineering-mcp) | Corrosion chemistry and model-specific screening with applicability limits. |
| [qsdsan-engine-mcp](https://github.com/puran-water/qsdsan-engine-mcp) | Dynamic treatment-process simulation, job state and numerical time series. |
| [watertap-engine-mcp](https://github.com/puran-water/watertap-engine-mcp) | Steady-state flowsheets using WaterTAP, IDAES and Pyomo with solver diagnostics. |
The six servers provide local MCP stdio entrypoints. Installation uses public Git sources, including the narrow `puran-mcp-support` package hosted inside the fluids repository. Each README contains an installation command, client configuration, known issues and licence inventory. PyPI publication is not part of the current installation path.

`v0.1.0` preserves the initial release. The default branch receives scoped improvements from PuranOS through a one-way mirror. Its source commit is recorded in `.puran-mirror.json`. Privacy, secrets, rights and packaging checks can refuse an export. Installed tools update only when the user reinstalls or upgrades them; pin both the server and support commits for repeatability.

Tools return calculations and structured evidence. The calling LLM interprets them and writes the explanation. The published packages do not include customer records, supplier pricing, internal project approvals or the complete proposal/BEP orchestration.

## Other public repositories

- [autocad-mcp](https://github.com/puran-water/autocad-mcp): Windows AutoCAD LT automation and a separate headless DXF backend.
- [freecad-mcp](https://github.com/puran-water/freecad-mcp): a FreeCAD integration fork; its add-on and runtime are required.
- [pandid](https://github.com/puran-water/pandid): a process-drawing fork; stream labels are supplied data, not a mass-balance calculation.
- [quetzal](https://github.com/puran-water/quetzal): a FreeCAD workbench fork with its own installation and upstream requirements.
- [docuseal-customizations](https://github.com/puran-water/docuseal-customizations): published source for the Docuseal customization, with its AGPL obligations.
- [adm1-mcp](https://github.com/puran-water/adm1-mcp) and [mathcad-mcp](https://github.com/puran-water/mathcad-mcp): earlier integrations. They are not members of the six-server automatic mirror.

Archived repositories retain historical implementations. Candidate archives may gain documentation notices while staying archived; a candidate is not a new release. Follow the active server READMEs for current installation.

[Built by Puran Water](https://puranwater.com/) · [Describe your engineering need](https://puranwater.com/contact/)
