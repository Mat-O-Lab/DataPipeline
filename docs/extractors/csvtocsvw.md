---
title: CSVToCSVW
---

# CSVToCSVW

FastAPI microservice that reads a CSV file and produces a W3C CSVW metadata document that annotates every column with data types, measurement units, and semantic term links.

**Repo:** https://github.com/Mat-O-Lab/CSVToCSVW  
**Public API:** https://csvtocsvw.matolab.org · [API docs](https://csvtocsvw.matolab.org/api/docs)  
**Image:** `ghcr.io/mat-o-lab/csvtocsvw:latest`  
**Version:** v1.3.5

---

## What is CSVW and why does it matter?

A plain CSV file is just text. The column header `"Kraft [kN]"` means something to a person who knows German and materials science — but to any downstream system it is an opaque string.

**CSVW** (CSV on the Web) solves this: it is a JSON sidecar file that annotates your CSV columns with data types, units, and ontology term links — turning a plain spreadsheet into self-describing data. When a system reads the CSVW alongside your CSV, it knows that the `Kraft [kN]` column contains force measurements in kilonewtons, expressed using the internationally agreed [QUDT](https://qudt.org/) unit vocabulary. Any tool that understands QUDT can then compare, combine, or reason over that column correctly, without a human in the loop.

The CSVW file format is a [W3C standard](https://www.w3.org/TR/tabular-data-primer/). CSVToCSVW automates its production: you give it a CSV URL, it gives you back the annotated CSVW — no manual JSON authoring required.

### What is JSON-LD?

The CSVW file CSVToCSVW produces is written in **JSON-LD** — a format that embeds semantic meaning directly in a JSON file. Your CSV column `"temperature"` gets linked to a shared ontology term so any system knows exactly what it means, regardless of what the column was named locally.

Practically, JSON-LD looks like ordinary JSON with two extra conventions:

- `@context` — a dictionary that maps short names (like `qudt:unit`) to full web URLs
- `@id` — a URL that unambiguously identifies the thing being described

You can read and edit a CSVW file with any JSON tool. The semantic meaning comes from the URLs, not from a special file format.

---

## What CSVToCSVW produces

Given a CSV file, CSVToCSVW returns a CSVW JSON-LD document that:

- **Annotates every data column** with its inferred measurement unit (e.g. `Kraft [kN]` → [`unit:KiloN`](https://qudt.org/vocab/unit/KiloN))
- **Captures metadata rows** found above the data table — common in lab instrument exports — as typed key-value annotations with their own unit links
- **Records provenance** (which service produced the file, at what time), so downstream systems can trace where the annotations came from

The CSVW output is Stage 1 of the default pipeline: it feeds directly into MapToMethod (mapping authoring) and into the CKAN automation layer.

For a full explanation of the ontology terms that appear in the CSVW output (QUDT, PROV-O, Open Annotation), see [Semantic Foundation](../pipeline/semantic-foundation.md).

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

Both blocks are captured:

- **Metadata rows** (above the blank separator) become typed annotation entries — each key-value pair is preserved with its physical unit where one can be inferred from the value
- **Data table columns** (below the separator) become column descriptors with QUDT unit annotations derived from the header bracket notation

---

## Column header notation — how units are detected

CSVToCSVW reads units from column names using two patterns:

**1. Bracket notation:** `Column Name [unit]`

The content inside square brackets is looked up against the QUDT unit vocabulary:

| Column header | Detected unit IRI |
|---|---|
| `Zeit [s]` | `http://qudt.org/vocab/unit/SEC` |
| `Maschine [mm]` | `http://qudt.org/vocab/unit/MilliM` |
| `Kraft [kN]` | `http://qudt.org/vocab/unit/KiloN` |
| `MX840A_0_CH 5 [°C]` | `http://qudt.org/vocab/unit/DEG_C` |
| `Druck [MPa]` | `http://qudt.org/vocab/unit/MegaPA` |
| `Weg [µm]` | `http://qudt.org/vocab/unit/MicroM` |

**2. Suffix notation:** common abbreviations in the column name itself

`ForceMPa` → `unit:MegaPA`, `DisplacementMm` → `unit:MilliM`

Columns without a recognizable unit pattern are typed but not unit-annotated. If your column headers use non-standard or institution-specific abbreviations, rename them to the bracket notation before calling the service for best results.

---

## Output — CSVW JSON-LD in detail

Below is the real CSVW JSON-LD that CSVToCSVW produces for `example2.csv`. Two sections show the two kinds of content the format carries.

### Data columns with unit annotations

Each column in the data table becomes a `Column` entry. Columns with recognized bracket notation get a `qudt:unit` link — a URL pointing to the internationally agreed definition of that unit:

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

The `name` field is a CamelCase-normalized version of the original header. This is the key MapToMethod uses to refer to that column in mapping rules.

### Metadata block as annotation entries

The preamble rows above the blank separator become Open Annotation entries. Each has a `qudt:unit` link when the value is a physical quantity:

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

What `oa:Annotation` and `qudt:QuantityValue` mean, and how they fit into the broader ontology stack, is covered in [Semantic Foundation](../pipeline/semantic-foundation.md).

---

## API

### Endpoints at a glance

| Method | Path | Input | Use when |
|---|---|---|---|
| `POST` | `/api/annotate` | JSON body with `data_url` | CSV is publicly accessible by URL |
| `POST` | `/api/annotate_upload` | Multipart form with `file` | CSV is private or on your local machine |
| `POST` | `/api/rdf` | JSON body with `metadata_url` | Re-serialize an existing CSVW to another RDF format |
| `GET` | `/api/docs` | — | Interactive API explorer (Swagger UI) |

`return_type` options for all endpoints: `json-ld` · `n3` · `nt` · `hext` · `trig` · `turtle` · `longturtle` · `xml`

---

### `POST /api/annotate` — Annotate CSV from a public URL

#### Request

- Method: `POST`
- Path: `/api/annotate`
- Query param: `return_type=json-ld` (or any format listed above)
- `Content-Type: application/json`
- Body:

```json
{
  "data_url": "https://…/your-file.csv",
  "encoding": "auto"
}
```

#### Response

The CSVW document as JSON-LD (or the requested serialization format).

#### Example — annotate `example2.csv`

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d '{
    "data_url": "https://github.com/Mat-O-Lab/CSVToCSVW/raw/main/examples/example2.csv",
    "encoding": "auto"
  }' \
  --output example2-metadata.json
```

This is a real working call — the example CSV is publicly accessible on GitHub.

---

### `POST /api/annotate_upload` — Annotate a local CSV (no public URL needed)

#### Request

- Method: `POST`
- Path: `/api/annotate_upload`
- Query param: `return_type=json-ld`
- `Content-Type: multipart/form-data`
- Form field: `file` — your CSV file

#### Example

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate_upload?return_type=json-ld" \
  -F "file=@/path/to/your/data.csv" \
  --output data-metadata.json
```

Use this for private or intranet data that is not publicly accessible from the service host.

---

### `POST /api/rdf` — Convert an existing CSVW to another RDF serialization

#### Request

- Method: `POST`
- Path: `/api/rdf`
- Query param: `return_type=turtle` (or any format listed above)
- `Content-Type: application/json`
- Body:

```json
{
  "metadata_url": "https://…/your-file-metadata.json"
}
```

#### Example — get Turtle output from an existing CSVW

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/rdf?return_type=turtle" \
  -H "Content-Type: application/json" \
  -d '{
    "metadata_url": "https://raw.githubusercontent.com/Mat-O-Lab/CSVToCSVW/main/examples/example-metadata.json"
  }' \
  --output example.ttl
```

---

## Try it with the example files

The `examples/` folder in the CSVToCSVW repo contains eight real lab CSV files. Use them to explore the service output without needing your own data:

| File | Format | What it tests |
|---|---|---|
| [`example.csv`](https://github.com/Mat-O-Lab/CSVToCSVW/blob/main/examples/example.csv) | Multi-block (German lab, semicolon) | Metadata block + tensile test time series |
| [`example2.csv`](https://github.com/Mat-O-Lab/CSVToCSVW/blob/main/examples/example2.csv) | Tab-separated, multi-channel INSTRON | Multiple measurement channels with units |
| [`example3.csv`](https://github.com/Mat-O-Lab/CSVToCSVW/blob/main/examples/example3.csv) | Comma-separated | Simple tabular |
| [`example5.csv`](https://github.com/Mat-O-Lab/CSVToCSVW/blob/main/examples/example5.csv) | Multi-block variant | Different separator and encoding |

Pre-computed CSVW outputs are in the same folder as `*-metadata.json` files — inspect them to see the full structure before calling the service.

**Reproduce example2 output and inspect it:**

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d '{
    "data_url": "https://github.com/Mat-O-Lab/CSVToCSVW/raw/main/examples/example2.csv",
    "encoding": "auto"
  }' \
  | python3 -m json.tool | head -60
```

Compare your output to [`example2-metadata.json`](https://raw.githubusercontent.com/Mat-O-Lab/CSVToCSVW/main/examples/example2-metadata.json) in the repo.

---

## Real-world pipeline examples

### BAMresearch DF-TEM-PAW (TEM microscopy)

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
  -d '{
    "data_url": "https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs.csv",
    "encoding": "auto"
  }' \
  --output detection_runs-metadata.json
```

This pipeline is documented in: [Hanke et al. (2023)](https://link.springer.com/article/10.1007/s40192-023-00331-5)

### IOFMaterialsTutorial (tensile test data)

The [IOFMaterialsTutorial](https://github.com/Mat-O-Lab/IOFMaterialsTutorial) repository provides a complete worked example of a tensile test CSV going through the full pipeline. The CSVW output is at [`measurements-metadata.json`](https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json).

---

## CKAN automation

When deployed in the DataStack, `ckanext-csvtocsvw` calls `/api/annotate` automatically on every CSV upload — no user action needed after initial setup. See [Default Use Case](../pipeline/default-use-case.md) for the full automation chain.

**Trigger file extensions** (configurable in `.env`): `csv` · `txt` · `asc` · `tsv`

---

## Limitations

- Remote `/api/annotate` requires the CSV to be publicly accessible from the service host; use `/api/annotate_upload` for private data
- QUDT unit matching is heuristic — based on header naming conventions; verify the output `qudt:unit` IRIs match your intent
- Multi-block CSV parsing targets lab instrument export formats; other multi-table layouts may need preprocessing
