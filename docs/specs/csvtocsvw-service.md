# CSVToCSVW Microservice — Component Specification

**Repo:** https://github.com/Mat-O-Lab/CSVToCSVW  
**Public deployment:** https://csvtocsvw.matolab.org  
**API docs:** https://csvtocsvw.matolab.org/api/docs  
**OpenAPI JSON:** https://csvtocsvw.matolab.org/api/openapi.json  
**Version:** v1.3.5  
**License:** Apache 2.0

---

## Purpose

Converts raw CSV files into W3C CSVW-compliant JSON-LD metadata. Analyzes experimental/scientific CSV data and annotates its structure, column types, and physical units using controlled vocabularies. Also renders CSVW metadata as RDF in multiple serializations.

Semantic enrichments applied automatically:
- **QUDT** unit annotations (maps column headers to ontology terms for physical quantities)
- **Open Annotation** metadata for key-value property sections
- **PROV-O** provenance records
- Handles multi-block scientific CSVs (metadata headers + data tables separated by blank lines)

---

## Tech Stack

| Layer | Details |
|---|---|
| Language | Python |
| Framework | FastAPI (OpenAPI 3.1.0) |
| Container | Docker (`ghcr.io/mat-o-lab/csvtocsvw:latest`, internal port 5000) |
| Ontologies | W3C CSVW, QUDT Units, PROV-O, Open Annotation |
| RDF serialization | rdflib |

---

## API Endpoints

### `POST /api/annotate` — URL-based CSV annotation

Fetches CSV from a URL, returns CSVW JSON-LD metadata.

**Query param:** `return_type` (default: `json-ld`)

**Request body (JSON):**

```json
{
  "data_url": "https://example.com/data.csv",
  "encoding": "auto"
}
```

| Field | Type | Default | Notes |
|---|---|---|---|
| `data_url` | string (URI) | `""` | URL to raw CSV |
| `encoding` | TextEncoding | `"auto"` | 30 options including auto-detection |

**Response:** CSVW/RDF in requested format

---

### `POST /api/annotate_upload` — Upload-based CSV annotation

Accepts multipart form upload, processes CSV directly (no public URL required).

**Query params:** `encoding`, `return_type`  
**Body:** `multipart/form-data` with `file` field (binary CSV)

---

### `POST /api/rdf` — Convert existing CSVW to RDF

Takes a URL to existing CSVW metadata and converts to a different RDF serialization.

**Query param:** `return_type` (default: `turtle`)

**Request body (JSON):**

```json
{
  "metadata_url": "https://example.com/metadata.json",
  "csv_url": null
}
```

---

### `GET /info` — Service info

```json
{
  "app_name": "CSVtoCSVW",
  "version": "v1.3.5",
  "config_name": "production",
  "server": "https://csvtocsvw.matolab.org"
}
```

---

## Output Formats (`return_type`)

`json-ld` · `n3` · `nt` · `hext` · `trig` · `turtle` · `longturtle` · `xml`

## Input Encoding Options

30 options: `auto`, `utf-8`, `ascii`, `iso-8859-1` … `iso-8859-16`, `windows-1250` … `windows-1258`, `koi8-r`, `koi8-u`, `mac-cyrillic`, `mac-roman`

---

## Typical Two-Step Pipeline

1. `POST /api/annotate` → CSVW JSON-LD metadata
2. `POST /api/rdf` (optional) → convert that metadata to Turtle / N-Triples / etc.

---

## Configuration (Docker / env)

| Variable | Notes |
|---|---|
| `APP_PORT` | External port (default: 6001 in stack; internal: 5000) |
| `APP_SECRET` | Application secret key |
| `SERVER_URL` | Public-facing base URL |
| `ADMIN_MAIL` | Admin contact |
| `APP_MODE` | `production` (default) |
| `SSL_VERIFY` | Whether to verify SSL when fetching remote CSVs |

---

## Capabilities

- Takes any CSV (including multi-block scientific formats) → W3C CSVW JSON-LD
- Auto-detects 30 character encodings
- Annotates physical units via QUDT ontology
- Adds PROV-O provenance automatically
- URL-based and upload-based endpoints
- 8 RDF serialization formats

## Limitations

- Remote URL-based endpoint requires data to be publicly accessible
- QUDT unit matching is heuristic (header-name-based); unusual column names may not match
- Multi-block CSV parsing is designed for materials-science lab export formats; other conventions may need preprocessing
