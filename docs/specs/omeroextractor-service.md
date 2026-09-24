# OmeroExtractor — Component Specification

**Repo:** https://github.com/Mat-O-Lab/OmeroExtractor  
**Type:** Metadata extractor microservice — stage 1 of pipeline for OMERO microscopy image data  
**Public deployment:** https://metadata.omero.matolab.org  
**API docs:** https://metadata.omero.matolab.org/api/docs  
**OpenAPI JSON:** https://metadata.omero.matolab.org/api/openapi.json  
**Version:** v0.0.4  
**Container:** `ghcr.io/mat-o-lab/omeroextractor`

---

## Purpose

Extracts metadata from an OMERO.Server (Open Microscopy Environment — standard open-source image data management platform for microscopy) and exposes it as semantically enriched JSON-LD / RDF.

Steps internally:
1. Connects to OMERO.Web JSON API — pulls structured image, dataset, and ROI metadata
2. Fetches "original metadata" from OMERO webclient — raw key/value pairs from microscope acquisition software (INI-style config)
3. Converts merged JSON to RDF triples using a custom OME ontology (`ome.ttl`) generated from the official OME XSD schema via `xsd2owl`

### What is OMERO?

OMERO (Open Microscopy Environment Remote Objects) is an open-source platform for managing and analyzing fluorescence/light microscopy image data. It stores images and acquisition metadata (objectives, exposure, channels, timestamps, ROIs, etc.) on a server. OMERO.Web is its HTTP API layer.

---

## Inputs

| Input | Details |
|---|---|
| OMERO image ID | Integer — path param on `/api/image/{id}` |
| OMERO dataset ID | Integer — path param on `/api/dataset/{id}` |
| OMERO image ID (ROIs) | Integer — path param on `/api/rois/{id}` |
| `anonymize` | Boolean query param (default `true`) — strips PII |
| `format` | Output serialization (default `json-ld`) |
| OMERO credentials | Set via env vars `OMERO_WEB_USER`, `OMERO_WEB_PASS`, `OMERO_WEB_HOST` — end users need no OMERO credentials |

---

## Outputs

RDF graphs in: `json-ld` · `turtle` · `longturtle` · `n3` · `nt` · `hext` · `trig` · `xml`

### Ontologies Used

| Vocabulary | Namespace | Role |
|---|---|---|
| OME (custom) | `https://github.com/Mat-O-Lab/OmeroExtractor/raw/main/ome.ttl#` | Core microscopy types (Image, Dataset, Channel, etc.) — generated from `ome.xsd` |
| Open Annotation | `http://www.w3.org/ns/oa#` | Wraps key/value acquisition metadata as `oa:Annotation` |
| QUDT | `http://qudt.org/schema/qudt/` | Numeric values typed as `qudt:QuantityValue` |
| PROV-O | Standard | `prov:wasGeneratedBy`, `prov:generatedAtTime` |

---

## Key API Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/api/image/{id}` | Extract metadata for a single OMERO image → RDF |
| GET | `/api/dataset/{id}` | Extract metadata for a dataset (collection of images) → RDF |
| GET | `/api/rois/{id}` | Extract ROI (Region of Interest) annotations for an image → RDF |
| GET | `/api/info` | Service health / version |
| GET | `/` | HTML form for ad-hoc image ID lookup |

All three data endpoints share query params: `anonymize` (bool, default `true`), `format` (string, default `json-ld`).

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI + Uvicorn + Starlette |
| RDF | rdflib |
| HTTP client | requests (to OMERO.Web) |
| Templating | Jinja2 + starlette-wtf + WTForms |
| Container | Docker Compose — API + OMERO.Web sidecar (`omeroweb:4080`) |

---

## Pipeline Position

Stage 1 extractor for OMERO microscopy data:

```
OMERO.Server (images + acquisition metadata)
  → OmeroExtractor (/api/image, /api/dataset, /api/rois)
  → JSON-LD (OME ontology + OA + QUDT + PROV-O)
  → [MapToMethod → RDFConverter for domain-specific ontology mapping]
```

Its output is the OMERO analogue of what CSVToCSVW produces for tabular data — complete, consistent metadata for a non-semantic resource (microscopy images), expressed as JSON-LD. From there the standard MapToMethod → RDFConverter path applies to map to any target ontology.

---

## Capabilities

- Extracts full OMERO image + dataset + ROI metadata in one call
- Merges structured API data with raw acquisition metadata from microscope software
- OME ontology alignment via `ome.ttl` (generated from official schema)
- QUDT unit annotation on numeric acquisition parameters
- Anonymization option
- 8 RDF serialization formats
- Web form UI for ad-hoc exploration

## Real-World Example

- **Dataset:** https://github.com/BAMresearch/DF-TEM-PAW — dark-field TEM data from BAM; demonstrates OmeroExtractor in a real materials characterization workflow
- **Publication:** https://link.springer.com/article/10.1007/s40192-023-00331-5 — peer-reviewed paper describing the use of the Mat-O-Lab pipeline (including OmeroExtractor) for FAIR microscopy data

Use these as the primary documentation example for the OMERO use case.

---

## Limitations

- Requires accessible OMERO.Web instance (configured via env vars)
- v0.0.4 — early stage; API surface may change
- OME ontology mapping to domain-specific target ontologies (e.g. materials science) requires downstream YARRRML mapping
- No bulk/batch extraction endpoint — one image/dataset per call
