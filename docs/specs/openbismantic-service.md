# OpenBISmantic — Component Specification

**Repo:** https://github.com/Mat-O-Lab/OpenBISmantic  
**Type:** Metadata extractor microservice — stage 1 of pipeline for openBIS ELN/LIMS data  
**Public deployment:** No active public instance found (`openbis.matolab.org` unreachable)  
**Container:** `ghcr.io/mat-o-lab/openbismantic`  
**License:** (check repo)

---

## Purpose

Bridges an openBIS ELN/LIMS instance and the semantic web. Wraps an openBIS server with a FastAPI layer that exposes all openBIS entities (objects, datasets, collections, projects, spaces, users, property types) as **resolvable RDF resources** via persistent IRIs built from openBIS permanent IDs (permIds).

Also produces **RO-Crate** export bundles (ZIP with files + `ro-crate-metadata.json`) for portable archival.

### What is openBIS?

openBIS (open Biology Information System) is an open-source ELN/LIMS from ETH Zürich. It stores experimental metadata, samples, datasets, and file attachments in a hierarchy: Space → Project → Collection/Experiment → Object/Sample → Dataset.

---

## Inputs

| Input | Details |
|---|---|
| openBIS permId | Path parameter on every entity endpoint (e.g. `/object/{perm_id}`) |
| HTTP `Accept` header | Controls RDF serialization format |
| JSON-LD graph (POST) | `/export_bundle` — accepts a JSON-LD graph, produces RO-Crate ZIP |
| openBIS credentials | Set via `OPENBIS_URL` / `BASE_URL` env vars; authenticated via `PyBIS` session token |

End users do not need openBIS credentials — the service authenticates via a configured service account.

---

## Outputs

**RDF serializations** (content-negotiated via `Accept` header):

`application/ld+json` (default) · `application/x-turtle` · `application/n-triples` · `application/rdf+xml` · `text/html`

**RO-Crate export**: ZIP containing all dataset files from openBIS datastore + `ro-crate-metadata.json`.

### Ontologies Used

| Vocabulary | Role |
|---|---|
| Schema.org | Dataset, File metadata |
| DCAT | Distributions, download URLs, byte sizes |
| PROV-O | Activities, derivation chains, timestamps |
| Custom Mat-O-Lab/openBIS namespace | Dataset codes, datastore relationships |

Core JSON→RDF mapping handled by `openbis_json_parser` (Mat-O-Lab library).

---

## Key API Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/object/{perm_id}` | Sample/object as RDF |
| GET | `/dataset/{perm_id}` | Dataset + file listing as RDF |
| GET | `/distribution/{perm_id}/{path}` | Specific file from a dataset |
| GET | `/collection/{perm_id}` | Experiment/collection as RDF |
| GET | `/project/{perm_id}` | Project with experiments as RDF |
| GET | `/space/{perm_id}` | Space with projects as RDF |
| GET | `/instance/` | All spaces (top-level) |
| GET | `/user/{user_id}` | Person/user as RDF |
| GET | `/class/{object_type_code}` | Object or collection type definition |
| GET | `/object_property/{property_type_code}` | Property type definition |
| POST | `/export_bundle` | RO-Crate ZIP from a JSON-LD graph |
| GET | `/search/` | Full-text search |

---

## Tech Stack

| Component | Technology |
|---|---|
| Framework | FastAPI 0.114 + Uvicorn 0.30 |
| openBIS client | PyBIS 1.36.3 |
| RDF | rdflib + openbis_json_parser (Mat-O-Lab) |
| Templating | Jinja2 |
| Infrastructure | Docker Compose: nginx + openBIS server + PostgreSQL + API |

---

## Deployment

Requires its own openBIS server — not a standalone microservice. Two modes:
1. Full stack: `docker compose up -d` (includes openBIS, PostgreSQL, nginx)
2. API only: point at an existing openBIS instance via `OPENBIS_URL` env var

---

## Pipeline Position

Stage 1 extractor for openBIS ELN data:

```
openBIS ELN/LIMS
  → OpenBISmantic (/object, /dataset, /collection, …)
  → JSON-LD / RDF (Schema.org + DCAT + PROV-O)
  → [can feed MapToMethod + RDFConverter for ontology-specific mapping]
```

Its output is the openBIS analogue of what CSVToCSVW produces for tabular data — complete, consistent metadata for a non-semantic resource, expressed as JSON-LD. From there the standard MapToMethod → RDFConverter path applies.

**Note:** The RO-Crate output (`/export_bundle`) can also feed directly into RDFConverter since RDFConverter accepts JSON-LD input.

---

## Capabilities

- Exposes entire openBIS hierarchy as Linked Data with stable IRIs
- Content negotiation — same endpoint, multiple RDF formats
- Portable RO-Crate export with all files and metadata
- Anonymization option (strips PII from output)
- No SPARQL knowledge required by end users

## Limitations

- Requires an accessible openBIS instance (no standalone operation)
- No active public deployment for testing
- Output uses Schema.org/DCAT/PROV-O — domain-specific ontology mapping (e.g. to MSEO) requires a downstream YARRRML mapping via RDFConverter
- RO-Crate → RDFConverter integration not explicitly documented but architecturally possible
