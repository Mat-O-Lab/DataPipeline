# MapToMethod Microservice — Component Specification

**Repo:** https://github.com/Mat-O-Lab/MapToMethod  
**Public deployment:** https://maptomethod.matolab.org  
**API docs:** https://maptomethod.matolab.org/api/docs  
**OpenAPI JSON:** https://maptomethod.matolab.org/api/openapi.json  
**Version:** v1.1.5  
**License:** Apache 2.0

---

## Purpose

Generates YARRRML mapping rules that connect JSON-LD data documents to knowledge graph templates — without requiring users to write RML directly. Sits at the critical "authoring" stage of the pipeline: once a mapping is authored and stored in CKAN, it can be applied automatically to any compatible future upload.

Pipeline position: CSV → [CSVToCSVW] → JSON-LD metadata → **[MapToMethod]** → YARRRML rules → [RDFConverter] → RDF knowledge graph

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI + Uvicorn |
| Templating | Jinja2, WTForms, starlette-wtf |
| RDF / Semantic web | rdflib ≥ 6.2.0, SPARQL |
| Config | Pydantic ≥ 2.0, pydantic_settings |
| Output | YARRRML (YAML via PyYAML) |
| Container | Docker / Docker Compose, `ghcr.io/mat-o-lab/maptomethod` |
| Middleware | CORS (all origins), SessionMiddleware, ProxyHeadersMiddleware |

---

## API Endpoints

### `GET /api/types`

Get all unique `rdf:type` IRIs from a semantic document.

**Query param:** `url` (optional — defaults to CSVToCSVW example metadata)  
**Response:** `["https://...", "https://..."]`

Use this to explore what entity types exist in a JSON-LD / CSVW document before building a mapping.

---

### `GET /api/entities`

Query entities of specified types from a semantic document.

**Query params:**
- `url` — URL to JSON-LD / RDF document
- `types` — comma-separated type URIs (default: `oa:Annotation` and `csvw:Column`)

**Response:** dict mapping entity names to metadata (IRI, properties)

Use this to get the list of named entities (columns, annotations) available for mapping.

---

### `POST /api/mapping`

Generate a YARRRML mapping file.

**Request body (JSON):**

```json
{
  "data_url": "https://...",
  "template_url": "https://...",
  "predicate": "https://...",
  "map": {
    "ColumnName": "https://...entity"
  },
  "data_types": ["https://..."],
  "template_types": ["https://..."],
  "use_template_rowwise": false
}
```

| Field | Required | Description |
|---|---|---|
| `data_url` | yes | URL to data metadata (JSON-LD/CSVW) |
| `template_url` | yes | URL to template knowledge graph (Turtle/RDF) |
| `predicate` | yes | RDF property linking data entity → template entity |
| `map` | yes | entity-name → entity-IRI dict (from `/api/entities` output) |
| `data_types` | no | RDF types to query from data |
| `template_types` | no | RDF types to query from template |
| `use_template_rowwise` | no | if true, duplicate template block per data row |

**Response:** YAML file download (`Content-Disposition: attachment`) — YARRRML mapping rules ready for RDFConverter.

---

### `GET /info`

```json
{
  "name": "MapToMethod",
  "version": "v1.1.5",
  "contact": "maptomethod@matolab.org",
  "mode": "development"
}
```

---

## Ontology Namespaces Used

- BFO: `http://purl.obolibrary.org/obo/`
- IOF Core: `https://spec.industrialontologies.org/ontology/core/Core/`
- Open Annotation: `http://www.w3.org/ns/oa#`
- CSVW: `https://www.w3.org/ns/csvw.ttl`

---

## Typical Workflow (Mapping Authoring)

1. Upload a CSVW JSON-LD (from CSVToCSVW) and a template graph (Turtle)
2. `GET /api/types` on both to understand available types
3. `GET /api/entities` to enumerate columns/annotations and template entities
4. Build the `map` dict pairing data columns to template entities
5. `POST /api/mapping` → download YARRRML file
6. Upload YARRRML to CKAN `mappings` group as a YAML resource
7. Future CSV uploads with matching structure will auto-apply this mapping via ckanext-csvwmapandtransform

---

## Configuration (env)

| Variable | Default | Description |
|---|---|---|
| `APP_MODE` | `development` | App mode |
| `PORT` | `5005` | Listen port |
| `SERVER_URL` | `https://maptomethod.matolab.org` | Base URL for OpenAPI |
| `SSL_VERIFY` | — | Toggle SSL cert verification for outbound requests |

---

## Capabilities

- Interactive mapping authoring without writing RML
- Supports any JSON-LD / RDF as data source (not just CSVW)
- Supports any Turtle template graph (any ontology)
- Row-wise template duplication for tabular data patterns
- Output is standard YARRRML — works with any YARRRML-compatible toolchain

## Limitations

- Both data and template URLs must be publicly accessible
- `map` dict must be pre-built (caller's responsibility, but `/api/entities` helps)
- No built-in mapping validation — validate via RDFConverter `/api/checkmapping`
- Mapping quality depends on template ontology design
