---
title: CSVToCSVW
---

# CSVToCSVW

FastAPI microservice that reads a CSV file and produces a [W3C CSVW](https://www.w3.org/TR/tabular-data-primer/) JSON-LD metadata document with semantic column annotations. It is Stage 1 of the default pipeline for CSV lab data.

**Repo:** https://github.com/Mat-O-Lab/CSVToCSVW  
**Public API:** https://csvtocsvw.matolab.org · [API docs](https://csvtocsvw.matolab.org/api/docs)  
**Image:** `ghcr.io/mat-o-lab/csvtocsvw:latest`  
**Version:** v1.3.5

---

## What it does

CSVToCSVW analyzes the structure of a CSV file and produces a CSVW JSON-LD document that:

- **Annotates every column** with an `rdf:type` and, where a QUDT unit term can be inferred from the column header name, a `qudt:unit` IRI from the [QUDT vocabulary](https://qudt.org/)
- **Wraps metadata rows** (key-value pairs above the data table in multi-block scientific CSVs) as `oa:Annotation` entries
- **Records provenance** via [PROV-O](https://www.w3.org/TR/prov-o/) — `prov:wasGeneratedBy` pointing to the service URL

The CSVW output is the input to MapToMethod and RDFConverter. Without this step, the pipeline has no column-level semantic metadata to map from.

---

## Supported CSV formats

CSVToCSVW handles the **multi-block format** common in lab instrument exports:

```
material,PA6GF30            ← metadata block (key-value rows)
test_standard,DIN EN ISO 527
                            ← blank separator line
temperature_C,strength_MPa  ← data table header row
23,180
60,140
```

Both blocks are captured: the metadata rows become `oa:Annotation` entries; the data table columns become `csvw:Column` entries with QUDT unit annotation.

---

## API

### `POST /api/annotate` — Annotate CSV from URL

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d '{"data_url": "https://example.org/sample.csv", "encoding": "auto"}' \
  --output sample.csvw.json
```

### `POST /api/annotate_upload` — Annotate local CSV (no public URL needed)

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate_upload?return_type=turtle" \
  -F "file=@/path/to/sample.csv" \
  --output sample.ttl
```

### `POST /api/rdf` — Convert existing CSVW to a different RDF serialization

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/rdf?return_type=turtle" \
  -H "Content-Type: application/json" \
  -d '{"metadata_url": "https://example.org/sample.csvw.json"}'
```

`return_type` options: `json-ld` · `n3` · `nt` · `hext` · `trig` · `turtle` · `longturtle` · `xml`

---

## Output — CSVW JSON-LD

Real output from the [BAMresearch/DF-TEM-PAW](https://github.com/BAMresearch/DF-TEM-PAW) dark-field TEM dataset (truncated):

```json
{
  "@context": ["http://www.w3.org/ns/csvw", {
    "oa":   "http://www.w3.org/ns/oa#",
    "qudt": "http://qudt.org/schema/qudt/",
    "prov": "http://www.w3.org/ns/prov#",
    "csv":  "https://github.com/BAMresearch/DF-TEM-PAW/raw/main/detection_runs.csv/"
  }],
  "@id":   "https://github.com/BAMresearch/DF-TEM-PAW/raw/main/detection_runs.csv",
  "@type": "http://www.w3.org/ns/csvw#TableGroup",
  "tables": [{
    "tableSchema": {
      "columns": [
        {
          "@id":   "...table-1-Specimenname",
          "name":  "Specimenname",
          "@type": "Column",
          "format": { "@id": "http://www.w3.org/2001/XMLSchema#string" }
        },
        {
          "@id":   "...table-1-DiskradiusvaluePx",
          "name":  "DiskradiusvaluePx",
          "@type": "Column",
          "format": { "@id": "http://www.w3.org/2001/XMLSchema#integer" }
        }
        // ... (additional columns truncated)
      ]
    }
  }]
}
```

When column headers contain recognizable physical quantity names (e.g. `tensile_strength_MPa`, `temperature_C`), CSVToCSVW adds a `qudt:unit` IRI to each column entry automatically.

---

## Real-world example

The [BAMresearch/DF-TEM-PAW](https://github.com/BAMresearch/DF-TEM-PAW) repository shows a complete pipeline run using CSVToCSVW:

| File | Role |
|---|---|
| [`detection_runs.csv`](https://github.com/BAMresearch/DF-TEM-PAW/blob/main/detection_runs.csv) | Input CSV (TEM detection runs) |
| [`detection_runs-metadata.json`](https://github.com/BAMresearch/DF-TEM-PAW/blob/main/detection_runs-metadata.json) | CSVW JSON-LD output from CSVToCSVW |
| [`detection_runs-map.yaml`](https://github.com/BAMresearch/DF-TEM-PAW/blob/main/detection_runs-map.yaml) | YARRRML mapping authored with MapToMethod |
| [`detection_runs-joined.ttl`](https://github.com/BAMresearch/DF-TEM-PAW/blob/main/detection_runs-joined.ttl) | Final knowledge graph produced by RDFConverter |

This is documented in: [Hanke et al. (2023)](https://link.springer.com/article/10.1007/s40192-023-00331-5)

---

## CKAN automation

When deployed in the DataStack, ckanext-csvtocsvw triggers CSVToCSVW automatically on every CSV upload — no user action needed. See [Default Use Case](../pipeline/default-use-case.md) for the full automation chain.

**Formats that trigger processing** (configurable): `csv` · `txt` · `asc` · `tsv`

---

## Limitations

- Remote URL-based endpoint requires the CSV to be publicly accessible from the service host
- QUDT unit matching is heuristic — based on column header naming conventions; unusual names may not match
- Multi-block CSV parsing targets lab instrument export formats; other conventions may need preprocessing
