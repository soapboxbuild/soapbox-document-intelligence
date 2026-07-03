---
name: document-intelligence
version: 0.1.0
description: >
  Turn unstructured real estate documents into typed, verifiable data. Classifies uploads
  (utility bill, energy audit, lease, PCA, drawing, regulatory filing), extracts structured
  fields per document type, digitizes charts/graphs/tables from PDFs into JSON, writes
  results to the asset record, and RAG-indexes every document so it stays searchable.
  Triggers on: "extract from this bill", "parse this audit", "what does the lease say
  about", "pull the data out of this PCA", "digitize this chart/table", "process these
  documents", "batch ingest", any document upload that should become structured data.
---

# Document Intelligence

One pipeline for every document that enters a portfolio: **classify → extract → write →
index**. The goal is typed data with provenance, not prose summaries.

## Pipeline

### 1. Classify

Read the document (you have native PDF/image vision). Emit:
`{ type, confidence, language, pages, scanned: bool }`

Types: `utility_bill | energy_audit | lease | pca | architectural_drawing |
regulatory_filing | om | rent_roll | other`. If confidence < 0.8, say what it looks like
and ask rather than guessing. Multi-document PDFs (e.g. a year of bills in one file):
split logically and process each.

### 2. Extract typed fields

Use the schema for the detected type in `references/extraction-schemas.md`. Rules:

- Every extracted value carries `{ value, unit, page, verbatim_source_text }` — the page
  reference is what makes downstream verification possible.
- Never infer a value that isn't in the document. Missing = `null` + listed in `gaps`.
- Scanned/low-quality pages: extract what is legible, mark fields `low_confidence`, never
  reconstruct illegible numbers.
- **Charts and graphs**: use the `pdf-chart-parser` MCP tools (`parse_pdf_charts`,
  `parse_pdf_tables`, `extract_specific_chart`) to digitize visual data — chart-heavy
  exhibits (OM comps, audit baseline graphs, rent-roll tables) go through it rather than
  eyeballing pixel values. If the MCP is unavailable, extract only clearly labeled data
  points and flag the rest.

### 3. Write to the platform

Attach extracted data to the right asset: update asset fields where the platform schema
has a home for them (e.g. utility consumption, audit EUI baseline), and save the full
extraction JSON alongside the source document in the asset's files. Name the extraction
file `<source-doc>.extracted.json`. If the asset is ambiguous, ask — never attach data to
a guessed asset.

### 4. Index for search

Ensure the source document is RAG-indexed (platform uploads index automatically; for
out-of-band files, upload via the Files API so indexing triggers). The extraction JSON
rides along so semantic search can find both the document and its structured data.

## Batch mode

For a folder/drop of many documents: classify all first, present a one-line-per-file
manifest (`file → type → target asset`) for confirmation, then extract in order,
finishing with a summary table: `processed / failed / fields written / gaps`. Never
silently skip a failed file — list each with the reason.

## Quality gates

- Utility bills: usage + cost + period must reconcile (rate sanity check: cost/usage
  within plausible $/kWh or $/therm range for the region — flag outliers, they're usually
  unit errors).
- Energy audits: extracted baseline EUI must be consistent with the building's utility
  data on file when both exist; if they diverge >15%, raise a finding instead of writing.
- Leases: quote clause text verbatim with page cites; legal interpretation is flagged as
  interpretation.
- Run the verifier on batch results before reporting completion when it's connected.
