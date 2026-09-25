---
title: Capability Map
---

# Capability Map

What the DataStack pipeline can and cannot do — an honest overview of supported data types, outputs, and current limitations.

> **New to linked data?** See [Semantic Foundation](semantic-foundation.md) for a plain-language explanation of what the pipeline produces and why it matters for FAIR data.

---

## Does DataStack Support My Data?

DataStack currently produces FAIR-compliant linked data from the following sources:

- **Spreadsheets and tabular lab files** (CSV, TSV, ASC) — fully automated
- **Microscopy images stored in OMERO** — working, early-stage deployment
- **ELN/LIMS records in OpenBIS** — demonstrator-level, requires your own OpenBIS instance
- **Catena-X / SAMM aspect model JSON** — production, 26 mappings live
- **IDTA / AAS submodel JSON** — production
- **SQL databases** — via external [Ontop](../extractors/ontop.md) tool, no built-in integration yet
- **Existing RDF or JSON-LD** — accepted directly, no transformation needed
- **Arbitrary JSON or XML** — supported with a manually authored mapping

If your data type is not listed, contact the project team — the pipeline is designed to accept any extractor that produces linked data output.

---

## Stage 1 — Supported Data Sources

The table below shows which extractor handles each source type, what vocabularies the linked data output uses, and the current readiness level.

| Data source | Extractor | Output vocabularies | Status | Notes |
|---|---|---|---|---|
| CSV / TSV / ASC lab files | [CSVToCSVW](../extractors/csvtocsvw.md) | CSVW, QUDT, OA, PROV-O | **Production** | Upload a CSV to CKAN and the full transformation runs automatically |
| OMERO microscopy image server | [OmeroExtractor](../extractors/omeroextractor.md) | OME, OA, QUDT, PROV-O | Early stage | Publicly deployed; example dataset: BAMresearch/DF-TEM-PAW |
| OpenBIS ELN/LIMS | [OpenBISmantic](../extractors/openbismantic.md) | Schema.org, DCAT, PROV-O | Demonstrator | Requires your own OpenBIS instance; output also includes RO-Crate |
| Catena-X / SAMM aspect model JSON | Mapping-based (two-stage) | PMDco | **Production** | 26 live mappings; first maps to Catena-X vocabulary, then bridges to PMDco |
| IDTA / AAS submodel JSON | Mapping-based (direct) | PMDco | **Production** | Single-stage — AAS structure maps directly to PMDco |
| SQL databases | [Ontop](../extractors/ontop.md) (external tool) | Any (configurable) | External tool | Enables SPARQL queries over relational databases; no built-in DataStack integration yet |
| Existing RDF or JSON-LD | Direct ingest | Already compatible | Built-in | Pipeline accepts all standard RDF formats without transformation |
| Arbitrary JSON or XML | Mapping-based (direct) | Any (configurable) | Built-in | Requires a manually authored mapping; no automatic semantic enrichment |

After the extraction step, all source types flow through the same transformation and storage path.

---

## Stage 2 — How Transformation Works

Once your source data is extracted, it is transformed into linked data using a **pattern** — a template that defines the target structure. There are two paths depending on whether your source already has its own data model.

| Path | When it applies | What happens |
|---|---|---|
| Direct | Source data maps directly to the target vocabulary — default for CSV and most lab files | Data is converted to linked data in a single step |
| Two-stage via intermediate format | Source has its own vocabulary (e.g. Catena-X SAMM, AAS submodels) that differs structurally from the target | Data is first converted to an intermediate format, then re-mapped to the target vocabulary |

See [Pipeline Overview — Two Transformation Paths](overview.md#two-transformation-paths) for more detail.

---

## Pattern Authoring

A **pattern** (the [PMDCO](https://github.com/materialdigital/core-ontology) term) is a template knowledge graph that defines the target structure your data transforms into. Patterns determine which vocabulary terms appear in the output and how concepts relate to each other.

| Tool | Status | Notes |
|---|---|---|
| [OntosphereIO](https://github.com/ThHanke/ontosphere) | **Recommended** | Browser-based, AI-assisted, supports OWL2DL reasoning and PMDCO autoshapes (SHACL) |
| [PMDCO pattern library](https://github.com/materialdigital/core-ontology/tree/main/patterns/) | Reference | Reusable patterns for common materials science measurement types |
| IOFMaterialsTutorial | Outdated | Uses draw.io and old terminology — concepts only, not current tooling |

The term "graph prototype" is obsolete — the correct PMDCO term is **pattern**.

---

## What the Pipeline CAN Do

- **Supports all common research data formats** — CSV, TSV, microscopy images (OMERO), ELN records (OpenBIS), Catena-X/SAMM JSON, AAS JSON, and existing RDF
- **Fully automated end-to-end for CSV** — uploading a spreadsheet triggers the complete transformation without any manual steps (up to the knowledge graph sync)
- **Auto-mapping discovery** — one mapping covers all future uploads with matching column structure; no per-file configuration needed
- **Query across all datasets** — once transformed, all linked data in a CKAN dataset can be queried together through a single endpoint
- **Reasoning over your data** — the knowledge graph supports OWL/RDFS inference so implicit relationships can be surfaced automatically
- **Use components independently** — the transformation services work without the CKAN data portal for integration into other workflows
- **Harvestable metadata** — DCAT-compliant metadata makes your datasets discoverable by external catalogues
- **Export in any standard format** — linked data output is available in multiple standard formats for use in other tools

---

## What the Pipeline CANNOT Do (Current Limitations)

- **Knowledge graph sync is not fully automatic** — after uploading data, you need to manually trigger the sync to the knowledge graph from the CKAN portal interface; the automatic hook is present but disabled
- **Files on private networks are not directly accessible** — some pipeline steps require your files to be reachable via a public URL; uploading via the portal bypasses this restriction
- **No input data validation before transformation** — if your file has unexpected column names, the transformation may produce an empty result without warning
- **No versioning of the knowledge graph** — the knowledge graph is rebuilt from scratch on each sync; there is no history or change tracking
- **No incremental updates** — every change triggers a full rebuild, not just an update of changed records
- **Microscopy and ELN support requires additional setup** — OMERO and OpenBIS extractors are not included in the standard installation; they require their own deployment configuration
