# soapbox-document-intelligence

Document Intelligence for real estate ESG — one pipeline for every document that enters a
portfolio: **classify → extract typed fields → write to the asset → RAG-index**.

Covers utility bills, energy audits, leases, PCAs, rent rolls, OMs, drawings, and
regulatory filings, with per-type extraction schemas and provenance on every value
(page + verbatim source text). Chart, graph, and table digitization is built in via the
bundled `pdf-chart-parser` MCP server (this plugin absorbed the former standalone
pdf-chart-parser plugin).

## Design

Skill-first: classification and typed extraction run on the agent's native document
vision, against strict schemas in `skills/document-intelligence/references/`. The one
piece of infrastructure is the chart/table digitization MCP
(`https://pdf-chart-parser.mcp.soapbox.build/mcp`), bundled in the plugin manifest.

Future work (per the 2026-06-26 ESG plugin architecture spec): a dedicated batch
extraction MCP for high-volume server-side ingestion, green lease scoring, and
multi-language documents.

## Install

- Soapbox: `soapbox install document-intelligence`
- Claude Code / opencode: add this repo as a plugin (skills + bundled MCP in the manifest)

## Contents

- `skills/document-intelligence/SKILL.md` — the pipeline
- `skills/document-intelligence/references/extraction-schemas.md` — per-type schemas
- `commands/extract-charts.md` — chart/table extraction command
