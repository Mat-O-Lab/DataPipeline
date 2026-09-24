---
title: Capability Map
---

# Capability Map

What the DataStack pipeline can and cannot do — an honest overview of supported resource types, output formats, and current limitations.

> You bring the data; the pipeline brings the transformation. The pipeline is resource-type agnostic — any extractor that produces JSON-LD or RDF can plug in.

---

## Stage 1 — Metadata Creation (Source-Type Extractors)

The first stage bridges your native data format into the semantic world. Each resource type has a dedicated extractor.

| Resource type | Extractor | Output ontologies | Status | Notes |
|---|---|---|---|---|
| CSV / TSV / ASC lab files | [CSVToCSVW](../extractors/csvtocsvw.md) v1.3.5 | CSVW, QUDT, OA, PROV-O | **Production** | Full CKAN automation via ckanext-csvtocsvw |
| OMERO microscopy image server | [OmeroExtractor](../extractors/omeroextractor.md) v0.0.4 | OME, OA, QUDT, PROV-O | Early stage | Public deployment; example: BAMresearch/DF-TEM-PAW |
| OpenBIS ELN/LIMS | [OpenBISmantic](../extractors/openbismantic.md) | Schema.org, DCAT, PROV-O | Demonstrator | Requires own OpenBIS instance; also produces RO-Crate |
| SAMM / Catena-X aspect model JSON | YARRRML mapping (Stage 1) | CX/SAMM namespace (intermediate) | **Production** | 26 mappings in live instance; then SPARQL CONSTRUCT bridges to PMDco |
| IDTA / AAS submodel JSON | YARRRML mapping (direct) | PMDco | **Production** | Single-stage; AAS structure maps directly |
| SQL databases | [Ontop](../extractors/ontop.md) (external) | Any ontology (OBDA) | External tool | R2RML/OBDA; SPARQL-over-SQL; no Mat-O-Lab integration built yet |
| Any existing RDF / JSON-LD | — (direct ingest) | Already compatible | Built-in | ckanext-csvwmapandtransform checks any RDF format |
| JSON / XML (arbitrary) | RDFConverter direct | Any (via YARRRML) | Built-in | Manual YARRRML mapping; no semantic enrichment step |

After the extractor step, all resource types flow through the same MapToMethod → RDFConverter → Fuseki path.

---

## Stage 2 — Semantic Transformation Paths

Two paths are available depending on how directly your source ontology maps to the target.

| Path | Mechanism | When to use |
|---|---|---|
| YARRRML/RML (direct) | RDFConverter executes YARRRML mapping against source data | Source data maps directly to target ontology (default for CSV) |
| YARRRML/RML + SPARQL INSERT | RDFConverter → intermediate RDF in Fuseki → SPARQL CONSTRUCT query | Source has its own ontology (SAMM/CX) that differs structurally from target |

See [Pipeline Overview — Two Transformation Paths](overview.md#two-transformation-paths) for architecture detail.

---

## Pattern Authoring

A **pattern** (the current [PMDCO](https://github.com/materialdigital/core-ontology) term) is a template knowledge graph that defines the target structure for your YARRRML mapping. The term "graph prototype" is obsolete — do not use it.

| Tool | Status | Notes |
|---|---|---|
| [OntosphereIO](https://github.com/ThHanke/ontosphere) | **Recommended** | Browser-only, AI-assisted, full OWL2DL reasoning, PMDCO autoshapes (SHACL) |
| [PMDCO pattern library](https://github.com/materialdigital/core-ontology/tree/main/patterns/) | Reference | Reusable patterns for common materials science measurement types |
| IOFMaterialsTutorial | Outdated | Uses draw.io and old terminology — concepts only, not tooling |

---

## What the Pipeline CAN Do

- **Ontology agnostic** — template graph and YARRRML mapping define the target ontology; change them, change the ontology, touch nothing else
- **Fully automated end-to-end for CSV** — upload triggers all steps up to (but not including) Fuseki
- **Auto-mapping discovery** — one mapping covers all future uploads with matching column structure
- **SPARQL querying** across all RDF resources in a dataset via Fuseki union default graph
- **OWL/RDFS inference** — Fuseki reasoning modes available
- **Standalone microservice use** — all three REST APIs work without CKAN
- **DCAT-compliant metadata** — harvestable by external catalogues
- **8 RDF serialization formats** at every stage (Turtle, JSON-LD, N-Triples, N3, HEX-T, TriG, Long Turtle, RDF/XML)

---

## What the Pipeline CANNOT Do (Current Limitations)

- **Fuseki auto-sync is disabled** — `fuseki_update` requires a manual trigger via the CKAN UI or API; the automatic hooks are commented out in ckanext-fuseki
- **Private or intranet data URLs** — URL-based microservice endpoints require publicly accessible URLs; use upload endpoints (`/api/annotate_upload`, `/api/createrdfupload`) for private data
- **No built-in data validation** before mapping — bad input column names may produce empty graphs silently under the `exact` strategy
- **No knowledge graph versioning** — the Fuseki dataset is deleted and recreated on each sync; no diff or history
- **MapToMethod requires public URLs** for both data document and pattern template
- **No streaming or incremental updates** — full re-sync on every resource change
- **Non-CSV extractors need separate deployment** — OmeroExtractor and OpenBISmantic require their own container and CKAN extension wiring, not included in the default DataStack compose
