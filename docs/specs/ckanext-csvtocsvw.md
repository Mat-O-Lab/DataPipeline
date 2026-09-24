# ckanext-csvtocsvw — Component Specification

**Repo:** https://github.com/Mat-O-Lab/ckanext-csvtocsvw  
**Type:** CKAN extension  
**Depends on:** CSVToCSVW microservice

---

## Purpose

Automatically converts uploaded CSV/TXT/ASC files into CSVW JSON-LD metadata by calling the CSVToCSVW microservice, then pushes the structured tabular data into the CKAN DataStore — making it queryable via CKAN's Data API.

---

## CKAN Integration Points

Implements `IResourceController` and `IResourceUrlChange`:

| Hook | Trigger |
|---|---|
| `after_resource_create` | New resource uploaded |
| `after_update` | Resource updated |
| `notify` | Resource URL changed |

**Trigger condition:** resource `format` (lowercased) must be in `ckanext.csvtocsvw.formats`.

---

## Processing Paths

### Path 1: CSV → CSVW annotation

Trigger: format in configured formats (default: `csv txt asc tsv`)

1. Calls CSVToCSVW `POST /api/annotate?return_type=json-ld` with the CSV URL
2. Uploads resulting JSON-LD as a new resource in the same CKAN dataset
   - Adds extras: `hadPrimarySource` (original CSV URL), `wasGeneratedBy` (microservice URL)
3. Deletes any datapusher-created DataStore on the original CSV resource (prevents double-import)
4. Parses CSVW metadata, extracts column headers + rows
5. Inserts into CKAN DataStore in 250-row chunks via `datastore_create`

### Path 2: JSON-LD → Turtle (triggered by the JSON-LD created in Path 1)

Trigger: format = `json-ld`

1. Calls CSVToCSVW `POST /api/rdf?return_type=turtle`
2. Uploads resulting Turtle file as a new resource in the same dataset

---

## Job Queue

All processing runs as CKAN RQ background jobs. Deduplication: jobs with the same resource ID title pattern are deduplicated automatically.

Job lifecycle states: `submitting → pending → running → complete / errored` (updated via `csvtocsvw_hook` CKAN action).

---

## Registered CKAN Actions

| Action | Purpose |
|---|---|
| `csvtocsvw_annotate` | Trigger annotation job |
| `csvtocsvw_transform` | Trigger JSON-LD → Turtle job |
| `csvtocsvw_hook` | Callback to update job status |

---

## Configuration

| Key | Default | Description |
|---|---|---|
| `ckanext.csvtocsvw.csvtocsvw_url` | `https://csvtocsvw.matolab.org` | Microservice URL |
| `ckanext.csvtocsvw.formats` | `csv txt asc tsv` | Formats that trigger annotation |
| `ckanext.csvtocsvw.ckan_token` | `""` | CKAN API token for background jobs |
| `ckanext.csvtocsvw.ssl_verify` | `True` | SSL verification for outbound requests |

In DataStack, configure via:
```
CKANINI__CKANEXT__CSVTOCSVW__CSVTOCSVW_URL=http://csvtocsvw:5000
CKANINI__CKANEXT__CSVTOCSVW__FORMATS=csv txt asc tsv
```

---

## Capabilities

- Zero-config for end users — upload a CSV, metadata is created automatically
- CKAN DataStore integration — CSV data becomes queryable via Data API
- Semantic enrichment via QUDT unit annotations (from CSVToCSVW)
- PROV-O provenance on all generated resources
- Works on resource update and URL change, not just initial upload
- 250-row chunked DataStore import handles large files without timeout

## Limitations

- CSV URL must be publicly accessible from the CKAN server (for microservice to fetch it)
- QUDT unit matching depends on column header naming conventions
- Only formats explicitly listed in `ckanext.csvtocsvw.formats` trigger processing
- `BACKGROUNDJOBS_API_TOKEN` must be set in DataStack or jobs will not execute
