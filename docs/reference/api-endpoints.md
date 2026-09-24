---
title: API Endpoints
---

# API Endpoints

Public microservice endpoints for standalone use. All services expose interactive documentation at `/api/docs`.

| Service | Base URL | Interactive docs |
|---|---|---|
| CSVToCSVW | https://csvtocsvw.matolab.org | [/api/docs](https://csvtocsvw.matolab.org/api/docs) |
| MapToMethod | https://maptomethod.matolab.org | [/api/docs](https://maptomethod.matolab.org/api/docs) |
| RDFConverter | https://rdfconverter.matolab.org | [/api/docs](https://rdfconverter.matolab.org/api/docs) |

---

### CSVToCSVW (v1.3.5)

| Method | Path | Purpose | Key params | Response |
|---|---|---|---|---|
| `POST` | `/api/annotate` | Annotate CSV from URL → CSVW | `data_url`, `encoding`, `return_type` | CSVW/RDF in requested format |
| `POST` | `/api/annotate_upload` | Annotate CSV from file upload → CSVW | `encoding`, `return_type`, multipart `file` | CSVW/RDF in requested format |
| `POST` | `/api/rdf` | Convert existing CSVW to RDF serialization | `metadata_url`, `csv_url`, `return_type` | RDF in requested format |
| `GET` | `/info` | Service version and config | — | `{"app_name", "version", "server"}` |

`return_type` options: `json-ld` · `n3` · `nt` · `hext` · `trig` · `turtle` · `longturtle` · `xml`

**Curl example — annotate a CSV from URL:**

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d '{"data_url": "https://example.org/sample.csv", "encoding": "auto"}'
```

**Curl example — upload a CSV directly (no public URL required):**

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate_upload?return_type=turtle" \
  -F "file=@sample.csv"
```

---

### MapToMethod (v1.1.5)

| Method | Path | Purpose | Key params | Response |
|---|---|---|---|---|
| `GET` | `/api/types` | Get all `rdf:type` IRIs from a document | `url` | JSON array of type URIs |
| `GET` | `/api/entities` | Get named entities of specified types | `url`, `types` | Dict: name → `{uri, property, text, type}` |
| `POST` | `/api/mapping` | Generate YARRRML mapping | `data_url`, `template_url`, `predicate`, `map`, `data_types`, `template_types` | YARRRML file (YAML download) |
| `GET` | `/info` | Service version | — | `{name, version, contact, mode}` |

**Curl example — list entity types in a CSVW:**

```bash
curl "https://maptomethod.matolab.org/api/types?url=https://example.org/sample.csvw.json"
```

---

### RDFConverter (v1.3.3)

| Method | Path | Purpose | Key params | Response |
|---|---|---|---|---|
| `POST` | `/api/createrdf` | Full RDF conversion from URL-accessible data | `mapping_url`, `data_url`, `return_type` | `RDFResponse {graph_data, filename, mapping_statistics}` |
| `POST` | `/api/createrdfupload` | Full RDF conversion from file upload | `mapping_url`, `data_url`, `return_type`, multipart file | `RDFResponse` |
| `POST` | `/api/checkmapping` | Dry-run: count applicable vs skipped rules | `mapping_url`, `data_url` | `CheckResponse {rules_applicable, rules_skipped}` |
| `POST` | `/api/yarrrmltorml` | Convert YARRRML → RML only (no execution) | `mapping_url` | RML in Turtle format |
| `POST` | `/api/rdfvalidator` | Validate RDF against SHACL shapes | `shapes_url`, `rdf_url` | `ValidateResponse {conformance_report, shapes_graph}` |
| `POST` | `/api/test` | Diagnostic run with per-rule statistics | `mapping_url`, `data_url` | `TestMappingResult {per_rule_statistics, logs, triple_count}` |
| `GET` | `/info` | Service version | — | `{version}` |

**Curl example — check how many mapping rules apply:**

```bash
curl -X POST "https://rdfconverter.matolab.org/api/checkmapping" \
  -G \
  --data-urlencode "mapping_url=https://example.org/my-mapping.yaml" \
  --data-urlencode "data_url=https://example.org/sample.csvw.json"
```

**Curl example — convert URL-accessible data to Turtle:**

```bash
curl -X POST "https://rdfconverter.matolab.org/api/createrdf?return_type=turtle" \
  -G \
  --data-urlencode "mapping_url=https://example.org/my-mapping.yaml" \
  --data-urlencode "data_url=https://example.org/sample.csvw.json"
```

`return_type` options (same as CSVToCSVW): `json-ld` · `n3` · `nt` · `hext` · `trig` · `turtle` · `longturtle` · `xml`
