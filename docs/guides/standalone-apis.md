---
title: Standalone APIs
---

# Standalone APIs

Use CSVToCSVW, MapToMethod, and RDFConverter as independent microservices — no CKAN required.

This is useful for CI/CD pipelines, batch processing scripts, Jupyter notebooks, or any workflow where you want the transformation logic without deploying a full data portal.

All three services are publicly accessible. For private or intranet data, use the upload endpoints documented below.

---

## CSVToCSVW — Annotate a CSV

Convert any CSV to CSVW JSON-LD with a single HTTP call.

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d '{"data_url": "https://example.org/sample.csv", "encoding": "auto"}' \
  --output sample.csvw.json
```

For private data that is not publicly accessible, upload the file directly:

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate_upload?return_type=turtle" \
  -F "file=@/path/to/local/sample.csv" \
  --output sample.ttl
```

See [API Endpoints — CSVToCSVW](../reference/api-endpoints.md#csvtocsvw-v135) for all parameters and return type options.

---

## MapToMethod — Generate a YARRRML Mapping

Given a CSVW JSON-LD and a template pattern, generate YARRRML mapping rules.

```bash
curl -X POST "https://maptomethod.matolab.org/api/mapping" \
  -H "Content-Type: application/json" \
  -d '{
    "data_url":     "https://example.org/sample.csvw.json",
    "template_url": "https://example.org/my-pattern.ttl",
    "predicate":    "http://www.w3.org/ns/oa#hasBody",
    "map": {
      "tensile_strength_MPa": "https://example.org/pattern#TensileStrengthValue",
      "yield_strength_MPa":   "https://example.org/pattern#YieldStrengthValue"
    }
  }' \
  --output my-mapping.yaml
```

See [Author a Mapping](author-a-mapping.md) for the full workflow to build the `map` dict.

---

## RDFConverter — Convert Data to RDF

Apply a YARRRML mapping to your CSVW data and get a knowledge graph back.

```bash
curl -X POST "https://rdfconverter.matolab.org/api/createrdf?return_type=turtle" \
  -G \
  --data-urlencode "mapping_url=https://example.org/my-mapping.yaml" \
  --data-urlencode "data_url=https://example.org/sample.csvw.json" \
  --output sample-joined.ttl
```

For private data, upload the file instead:

```bash
curl -X POST "https://rdfconverter.matolab.org/api/createrdfupload?return_type=turtle" \
  -F "mapping_url=https://example.org/my-mapping.yaml" \
  -F "data_url=https://example.org/sample.csvw.json" \
  -F "file=@/path/to/local/sample.csvw.json" \
  --output sample-joined.ttl
```

---

## Chaining the Three Services

Full CSV → RDF pipeline as a shell script:

```bash
#!/usr/bin/env bash
CSV_URL="https://example.org/sample.csv"
MAPPING_URL="https://example.org/my-mapping.yaml"

# Step 1: Annotate CSV → CSVW JSON-LD
curl -s -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d "{\"data_url\": \"${CSV_URL}\", \"encoding\": \"auto\"}" \
  --output sample.csvw.json

echo "✓ CSVW created: sample.csvw.json"

# Step 2: Convert CSVW to Turtle (optional intermediate step)
curl -s -X POST "https://csvtocsvw.matolab.org/api/rdf?return_type=turtle" \
  -H "Content-Type: application/json" \
  -d "{\"metadata_url\": \"${CSV_URL%.csv}.csvw.json\"}" \
  --output sample.ttl

echo "✓ Turtle created: sample.ttl"

# Step 3: Apply mapping → joined RDF knowledge graph
curl -s -X POST "https://rdfconverter.matolab.org/api/createrdf?return_type=turtle" \
  -G \
  --data-urlencode "mapping_url=${MAPPING_URL}" \
  --data-urlencode "data_url=${CSV_URL%.csv}.csvw.json" \
  --output sample-joined.ttl

echo "✓ Knowledge graph created: sample-joined.ttl"
```

---

## Validating a Mapping (dry-run)

Before running a full conversion, check whether your mapping rules match your data:

```bash
curl -X POST "https://rdfconverter.matolab.org/api/checkmapping" \
  -G \
  --data-urlencode "mapping_url=https://example.org/my-mapping.yaml" \
  --data-urlencode "data_url=https://example.org/sample.csvw.json"
# → {"rules_applicable": 3, "rules_skipped": 0}
```

`rules_skipped == 0` means all rules matched. See [Author a Mapping — Troubleshooting](author-a-mapping.md#troubleshooting-rules_skipped--0) if `rules_skipped > 0`.

---

## Next steps

- **Full API reference:** [API Endpoints](../reference/api-endpoints.md)
- **Extend the pipeline for new data types:** [Add a Use Case](add-a-use-case.md)
- **Deploy your own instance:** [Quickstart](quickstart.md)
