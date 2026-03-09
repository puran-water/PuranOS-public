# PuranOS

**Puran Water's integrated operating system for wastewater and waste-to-value engineering and delivery (EPC, own/operate).**

PuranOS consolidates ~60 MCP servers, 20+ agent personas, automated workflows, and Docker service orchestration into a single monorepo — purpose-built for wastewater and waste-to-value engineering and delivery.

## What It Does

- **Process Engineering**: Design tools for aerobic/anaerobic treatment, RO, IX, evaporation, degassing, mixing CFD, corrosion engineering, and more
- **Project Delivery**: CPM scheduling, cost estimation, procurement, inventory (InvenTree), CMMS (Atlas), and project finance modeling
- **Compliance**: EPA EGGRT reporting, CA-GREET pathway analysis, LCFS credit verification
- **Lead Generation**: Industrial facility targeting, EPA ECHO data mining, CRM integration (Twenty)
- **Knowledge Management**: Semantic search across engineering reference libraries, RAG-powered Q&A
- **Business Operations**: Bookkeeping (QuickBooks/Wave), legal document management, sales proposals

## Architecture

| Layer | Count | Examples |
|-------|-------|---------|
| MCP Servers | ~60 | Water chemistry, fluids hydraulics, WaterTAP/QSDsan simulation engines |
| Agent Personas | 20+ | Process engineer, compliance officer, project manager, procurement specialist |
| Skills | 36+ | PFD generation, P&ID digitization, control philosophy, equipment lists |
| Docker Services | 8+ | OpenProject, InvenTree, Atlas CMMS, NocoDB, Twenty CRM |
| Workflows | n8n | Automated engineering document routing and approval |

## Tech Stack

- **Runtime**: Python 3.11+ (uv workspaces), Node.js 22 (office integrations)
- **AI Framework**: Model Context Protocol (MCP) servers, Claude Code + Codex agent personas
- **Infrastructure**: Docker Compose, systemd services, Cloudflare Tunnels
- **Data**: PostgreSQL, SQLite, semantic vector search (FTS5 + embeddings)

## Contact

**Puran Water** — Wastewater and waste-to-value engineering and delivery  
[puranwater.com](https://puranwater.com)
