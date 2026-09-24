---
title: Standalone APIs
---

# Standalone APIs

Use CSVToCSVW, MapToMethod, and RDFConverter as independent microservices — no CKAN required.

All three services are publicly accessible. Every curl command below is reproducible as-is.

---

## CSVToCSVW — Annotate a CSV

Convert any CSV to CSVW JSON-LD. Works on any publicly accessible CSV file.

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d '{"data_url": "https://github.com/Mat-O-Lab/CSVToCSVW/raw/main/examples/example2.csv",
       "encoding": "auto"}' \
  --output example2-metadata.json
```

This annotates a real INSTRON tensile-test CSV (Zeit [s], Maschine [mm], Kraft [kN]) — try it immediately.

Compare with the pre-computed result in the repo:

```bash
diff <(python3 -m json.tool example2-metadata.json) \
     <(curl -s "https://raw.githubusercontent.com/Mat-O-Lab/CSVToCSVW/main/examples/example2-metadata.json" | python3 -m json.tool)
```

For private data (not publicly accessible from the service host):

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate_upload?return_type=json-ld" \
  -F "file=@/path/to/local/sample.csv" \
  --output sample.csvw.json
```

---

## MapToMethod — Explore types and entities

Before building a mapping, explore what types and entities a document exposes.

**Types in the IOFMaterialsTutorial CSVW:**

```bash
curl "https://maptomethod.matolab.org/api/types?url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

```json
["http://qudt.org/schema/qudt/DerivedUnit", "http://www.w3.org/ns/csvw#Column",
 "http://www.w3.org/ns/csvw#TableGroup", "http://www.w3.org/ns/prov#Activity",
 "http://www.w3.org/ns/prov#SoftwareAgent"]
```

**Entities (columns) in the same CSVW:**

```bash
curl "https://maptomethod.matolab.org/api/entities?url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

```json
{
  "entities": {
    "table-1-LengthMm": {
      "uri": "https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/measurements.csv/table-1-LengthMm",
      "property": "name", "text": "LengthMm", "type": "http://www.w3.org/ns/csvw#Column"
    }
  },
  "base_namespace": "https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json/"
}
```

## MapToMethod — Generate a YARRRML Mapping

Once you know the entity names and template IRIs, generate the mapping:

```bash
curl -X POST "https://maptomethod.matolab.org/api/mapping" \
  -H "Content-Type: application/json" \
  -d '{
    "data_url":     "https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json",
    "template_url": "https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/LengthMeasurement.ttl",
    "predicate":    "http://purl.obolibrary.org/obo/RO_0010002",
    "map": {
      "table-1-LengthMm": "https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/LengthMeasurement.ttl/LengthData"
    }
  }' \
  --output my-mapping.yaml
```

The result is a valid YARRRML file ready for RDFConverter. See [Author a Mapping](author-a-mapping.md) for the full step-by-step guide on building the `map` dict.

---

## RDFConverter — Validate a mapping (dry-run)

Check whether all mapping rules match the data before converting:

```bash
curl -X POST "https://rdfconverter.matolab.org/api/checkmapping" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

```json
{"rules_applicable": 1, "rules_skipped": 0}
```

`rules_skipped == 0` is the quality gate. See [Author a Mapping — Troubleshooting](author-a-mapping.md#troubleshooting) if `rules_skipped > 0`.

---

## RDFConverter — Convert to RDF

Apply the mapping and produce a joined knowledge graph:

```bash
curl -X POST "https://rdfconverter.matolab.org/api/createrdf?return_type=turtle" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json" \
  --output measurements-joined.ttl
```

The response also contains `num_mappings_applied` and `num_mappings_skipped` in the JSON body if you omit `--output`:

```bash
curl -X POST "https://rdfconverter.matolab.org/api/createrdf?return_type=turtle" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

```json
{
  "filename": "measurements-metadata-joined.ttl",
  "graph": "@prefix iof: ...\n<.../table-1-LengthMm> ...",
  "num_mappings_applied": 1,
  "num_mappings_skipped": 0
}
```

---

## Full pipeline as a shell script

CSV → CSVW → validate → RDF, using real IOFMaterialsTutorial data throughout:

```bash
#!/usr/bin/env bash
set -e

CSV_URL="https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements.csv"
MAPPING_URL="https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml"

# Step 1: Annotate CSV → CSVW JSON-LD
curl -s -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d "{\"data_url\": \"${CSV_URL}\", \"encoding\": \"auto\"}" \
  --output measurements-metadata.json
echo "✓ CSVW: measurements-metadata.json"

# Step 2: Validate mapping (dry-run)
CHECK=$(curl -s -X POST "https://rdfconverter.matolab.org/api/checkmapping" \
  -G \
  --data-urlencode "mapping_url=${MAPPING_URL}" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json")
echo "✓ Check: ${CHECK}"

# Step 3: Apply mapping → joined RDF
curl -s -X POST "https://rdfconverter.matolab.org/api/createrdf?return_type=turtle" \
  -G \
  --data-urlencode "mapping_url=${MAPPING_URL}" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json" \
  --output measurements-joined.ttl
echo "✓ Knowledge graph: measurements-joined.ttl"
```

---

## Other real working examples

| Dataset | CSV | Mapping | Output |
|---|---|---|---|
| IOFMaterialsTutorial (length) | [measurements.csv](https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements.csv) | [measurements-map.yaml](https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml) | [measurements-joined.ttl](https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-joined.ttl) |
| BAMresearch DF-TEM-PAW | [detection\_runs.csv](https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs.csv) | [detection\_runs-map.yaml](https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs-map.yaml) | [detection\_runs-joined.ttl](https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs-joined.ttl) |

---

*Next: [Full API Reference](../reference/api-endpoints.md) — all endpoints with parameters and response schemas.*
