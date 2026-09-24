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

- **Annotates every column** with an `rdf:type` and, where a [QUDT](https://qudt.org/) unit term can be inferred from the column header name (e.g. `Kraft [kN]` → `unit:KiloN`), a `qudt:unit` IRI
- **Wraps metadata rows** above the data table as `oa:Annotation` entries — each key-value pair in the preamble block becomes a typed RDF annotation with provenance
- **Records provenance** via [PROV-O](https://www.w3.org/TR/prov-o/) — `prov:wasGeneratedBy` linking to the service URL and `prov:generatedAtTime`
- **Detects encoding** from 30 options including `auto`

The CSVW output is the entry point for MapToMethod (mapping authoring) and for ckanext-csvtocsvw (CKAN automation).

---

## Supported CSV formats

CSVToCSVW handles the **multi-block format** common in lab instrument exports:

```
aktuelle Probe;17              ← metadata block (key-value rows)
Einspannlänge;832.756
Probenbreite b0;11.4
                               ← blank separator line (auto-detected)
Zeit [s];Maschine [mm];Kraft [kN]    ← data table header row
0.000;0.000;0.164
0.200;0.001;0.193
…
```

Both blocks are captured: the metadata rows become `oa:Annotation` entries; the data table columns become `csvw:Column` entries with QUDT unit annotations derived from the header bracket notation.

---

## API

### `POST /api/annotate` — Annotate CSV from a public URL

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d '{"data_url": "https://github.com/Mat-O-Lab/CSVToCSVW/raw/main/examples/example2.csv",
       "encoding": "auto"}' \
  --output example2-metadata.json
```

This is a real working call — the example CSV is publicly accessible on GitHub and returns the full CSVW JSON-LD immediately.

### `POST /api/annotate_upload` — Annotate a local CSV (no public URL needed)

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate_upload?return_type=json-ld" \
  -F "file=@/path/to/your/data.csv" \
  --output data-metadata.json
```

Use this for private or intranet data that is not publicly accessible from the service host.

### `POST /api/rdf` — Convert an existing CSVW to another RDF serialization

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/rdf?return_type=turtle" \
  -H "Content-Type: application/json" \
  -d '{"metadata_url": "https://raw.githubusercontent.com/Mat-O-Lab/CSVToCSVW/main/examples/example-metadata.json"}' \
  --output example.ttl
```

`return_type` options for all endpoints: `json-ld` · `n3` · `nt` · `hext` · `trig` · `turtle` · `longturtle` · `xml`

---

## Output — CSVW JSON-LD in detail

Here is the real output CSVToCSVW produces for `example.csv` and `example2.csv`. The two sections show the two kinds of semantic content the format carries.

### Metadata block as `oa:Annotation` entries

The preamble rows above the data table (probe dimensions, test parameters) become Open Annotation entries. Each has a `qudt:unit` when the value is a physical quantity:

```json
{
  "@id": "…/example.csv/Einspannlaenge1",
  "label": "Einspannlänge",
  "@type": "oa:Annotation",
  "rownum": {"@value": 1, "@type": "xsd:integer"},
  "oa:hasBody": [{
    "@type": "qudt:QuantityValue",
    "qudt:value": {"@value": 832.756, "@type": "xsd:double"},
    "qudt:unit": {
      "@id": "http://qudt.org/vocab/unit/MilliM",
      "@type": "qudt:DerivedUnit"
    }
  }]
},
{
  "@id": "…/example.csv/ProbenbreiteB03",
  "label": "Probenbreite b0",
  "@type": "oa:Annotation",
  "oa:hasBody": [{
    "@type": "qudt:QuantityValue",
    "qudt:value": {"@value": 11.4, "@type": "xsd:double"},
    "qudt:unit": {"@id": "http://qudt.org/vocab/unit/MilliM"}
  }]
}
```

### Data columns with QUDT unit annotations

Each column in the data table gets a `csvw:Column` entry. Columns whose header contains a recognized unit suffix in square brackets (e.g. `[mm]`, `[kN]`, `[s]`, `[°C]`) receive an automatic `qudt:unit` IRI:

```json
{
  "name": "MaschineMm",
  "titles": ["Maschine [mm]", "MaschineMm"],
  "@type": "Column",
  "qudt:unit": {"@id": "http://qudt.org/vocab/unit/MilliM"}
},
{
  "name": "KraftKn",
  "titles": ["Kraft [kN]", "KraftKn"],
  "@type": "Column",
  "qudt:unit": {"@id": "http://qudt.org/vocab/unit/KiloN"}
},
{
  "name": "Mx840A0Ch5C",
  "titles": ["MX840A_0_CH 5 [°C]", "Mx840A0Ch5C"],
  "@type": "Column",
  "qudt:unit": {"@id": "http://qudt.org/vocab/unit/DEG_C"}
},
{
  "name": "ZeitS",
  "titles": ["Zeit [s]", "ZeitS"],
  "@type": "Column",
  "qudt:unit": {"@id": "http://qudt.org/vocab/unit/SEC"}
}
```

The `name` field is a CamelCase-normalized version of the original header — this is what MapToMethod uses as the entity key.

---

## Try it with the example files

The `examples/` folder in the CSVToCSVW repo contains eight real lab CSV files. Use them to explore the service output without needing your own data:

| File | Format | What it tests |
|---|---|---|
| [`example.csv`](https://github.com/Mat-O-Lab/CSVToCSVW/blob/main/examples/example.csv) | Multi-block (German lab, semicolon) | Metadata block + tensile test time series |
| [`example2.csv`](https://github.com/Mat-O-Lab/CSVToCSVW/blob/main/examples/example2.csv) | Tab-separated, multi-channel INSTRON | Multiple measurement channels with units |
| [`example3.csv`](https://github.com/Mat-O-Lab/CSVToCSVW/blob/main/examples/example3.csv) | Comma-separated | Simple tabular |
| [`example5.csv`](https://github.com/Mat-O-Lab/CSVToCSVW/blob/main/examples/example5.csv) | Multi-block variant | Different separator and encoding |

Their pre-computed CSVW outputs are in the same folder as `*-metadata.json` files — inspect them to see the full CSVW structure before calling the service.

**Reproduce example2 output:**

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d '{"data_url": "https://github.com/Mat-O-Lab/CSVToCSVW/raw/main/examples/example2.csv",
       "encoding": "auto"}' \
  | python3 -m json.tool | head -60
```

Compare your output to [`example2-metadata.json`](https://raw.githubusercontent.com/Mat-O-Lab/CSVToCSVW/main/examples/example2-metadata.json) in the repo.

---

## Real-world pipeline example

The [BAMresearch/DF-TEM-PAW](https://github.com/BAMresearch/DF-TEM-PAW) repository shows a complete pipeline run using CSVToCSVW on TEM microscopy detection-run data:

| File | URL |
|---|---|
| Input CSV | [`detection_runs.csv`](https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs.csv) |
| CSVW output | [`detection_runs-metadata.json`](https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs-metadata.json) |
| YARRRML mapping | [`detection_runs-map.yaml`](https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs-map.yaml) |
| Joined knowledge graph | [`detection_runs-joined.ttl`](https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs-joined.ttl) |

Reproduce the annotation step:

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d '{"data_url": "https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs.csv",
       "encoding": "auto"}' \
  --output detection_runs-metadata.json
```

This pipeline is documented in: [Hanke et al. (2023)](https://link.springer.com/article/10.1007/s40192-023-00331-5)

---

## CKAN automation

When deployed in the DataStack, ckanext-csvtocsvw calls `/api/annotate` automatically on every CSV upload — no user action needed after initial setup. See [Default Use Case](../pipeline/default-use-case.md) for the full automation chain.

**Trigger formats** (configurable in `.env`): `csv` · `txt` · `asc` · `tsv`

---

## QUDT unit matching logic

CSVToCSVW identifies units from column header names using two patterns:

1. **Bracket notation:** `Column Name [unit]` → the bracket content is looked up in QUDT (e.g. `[mm]` → `unit:MilliM`, `[kN]` → `unit:KiloN`, `[°C]` → `unit:DEG_C`)
2. **Suffix notation:** common abbreviations in the name itself (e.g. `_MPa` → `unit:MegaPA`)

Columns without a recognizable unit pattern are typed but not unit-annotated. Unusual or institution-specific abbreviations may not match — rename columns to standard notation before annotating for best results.

---

## Limitations

- Remote `/api/annotate` requires the CSV to be publicly accessible from the service host; use `/api/annotate_upload` for private data
- QUDT unit matching is heuristic — based on header naming conventions; verify the output `qudt:unit` IRIs match your intent
- Multi-block CSV parsing targets lab instrument export formats; other multi-table layouts may need preprocessing
