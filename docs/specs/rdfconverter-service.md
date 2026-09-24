# RDFConverter Microservice — Component Specification

**Repo:** https://github.com/Mat-O-Lab/RDFConverter  
**Public deployment:** https://rdfconverter.matolab.org  
**API docs:** https://rdfconverter.matolab.org/api/docs  
**OpenAPI JSON:** https://rdfconverter.matolab.org/api/openapi.json  
**Version:** v1.3.3  
**License:** Apache 2.0  
**DOI:** 10.5281/zenodo.19084883

---

## Purpose

Transforms structured data (CSV, JSON, XML, RDF) into semantic RDF knowledge graphs using YARRRML/RML declarative mapping rules. Packages the official YARRRML Parser (Node.js) and RML Mapper (Java) behind a FastAPI (Python) orchestration layer, enabling FAIR data publication without coding.

---

## Tech Stack

| Component | Role |
|---|---|
| FastAPI (Python) | Main REST API orchestrator |
| YARRRML Parser (Node.js) | Converts human-readable YARRRML → RML |
| RML Mapper (Java) | Executes RML mapping rules on data |
| RDFLib (Python) | RDF processing and serialization |
| PySHACL | SHACL constraint validation |
| Docker Compose | 3-service orchestration (`yarrrml-parser`, `rmlmapper`, `rdfconverter`) |

Standards: YARRRML, RML (W3C), CSVW, SAMM, PROV-O, SHACL, JSON-LD, Turtle, RDF/XML.

---

## API Endpoints

### `POST /api/createrdf` — Full RDF conversion (URL-based)

**Query params:** `mapping_url`, `data_url`, `return_type`  
**Response:** `RDFResponse { graph_data, filename, mapping_statistics }`

Main endpoint. Fetches mapping and data from public URLs, produces RDF output.

---

### `POST /api/createrdfupload` — Full RDF conversion (file upload)

**Query params:** `mapping_url`, `data_url` (used as base URI), `return_type`  
**Body:** uploaded file bytes (multipart)  
**Response:** `RDFResponse`

Use when data is not publicly accessible — upload the file directly.

---

### `POST /api/checkmapping` — Dry-run mapping check

**Query params:** `mapping_url`, `data_url`  
**Response:** `CheckResponse { rules_applicable, rules_skipped }`

Checks how many mapping rules apply to the data without producing output. Used by ckanext-csvwmapandtransform for mapping discovery and selection.

---

### `POST /api/yarrrmltorml` — YARRRML → RML conversion only

**Input:** `mapping_url` (query) or body  
**Output:** RML mapping in Turtle format

Convert YARRRML to RML without executing against data.

---

### `POST /api/rdfvalidator` — SHACL validation

**Query params:** `shapes_url`, `rdf_url`  
**Response:** `ValidateResponse { conformance_report, shapes_graph }`

Validate RDF output against SHACL constraint shapes.

---

### `POST /api/test` — Detailed diagnostic run

**Query params:** `mapping_url`, `data_url`  
**Response:** `TestMappingResult { per_rule_statistics, logs, triple_count, output_preview }`

Detailed diagnostic — use when debugging mapping rules.

---

### `GET /info`

```json
{ "version": "v1.3.3" }
```

---

## Input Data Formats

CSV, JSON (JSONPath iterator syntax), XML (XPath/XQuery), or any RDF serialization (Turtle, N-Triples, N3, JSON-LD — auto-converted to JSON-LD internally).

## Output Formats (`return_type`)

`json-ld` · `n3` · `nt` · `hext` · `trig` · `turtle` · `longturtle` · `xml`

---

## Internal Architecture

```
rdfconverter (FastAPI, Python)
  ├─► yarrrml-parser (Node.js, port 3001) — YARRRML → RML
  └─► rmlmapper (Java webapi, port 4000) — RML execution
```

All three communicate over internal Docker bridge network `rdfconverter_net`.

---

## Configuration (env)

| Variable | Notes |
|---|---|
| `APP_PORT` | External port (default: 6003) |
| `PARSER_PORT` | YARRRML parser internal port (3001) |
| `MAPPER_PORT` | RML mapper internal port (4000) |
| `APP_MODE` | `development` / `production` |
| `SSL_VERIFY` | Toggle SSL verification for outbound URL fetches |
| `YARRRML_URL` | Internal yarrrml-parser URL |
| `MAPPER_URL` | Internal rmlmapper URL |

---

## Capabilities

- Any YARRRML-described data → FAIR RDF knowledge graph
- Supports CSV, JSON, XML, and RDF as input data
- Dry-run check (`/api/checkmapping`) without producing output — enables automated mapping selection
- SHACL constraint validation of output
- PROV-O provenance included in all outputs automatically
- Upload endpoint for private/local data
- 8 RDF serialization formats

## Limitations

- URL-based endpoints require data to be publicly server-accessible
- RML Mapper (Java) is the execution bottleneck — complex mappings or large files may time out
- YARRRML/RML learning curve for mapping authors (MapToMethod mitigates this)
- `rmlmapper` port 4000 is fixed in the DataStack compose — cannot be changed without rebuilding

## Resources

- YARRRML tutorial: https://rml.io/yarrrml/tutorial/
- Online YARRRML editor (Matey): https://rml.io/yarrrml/matey/
