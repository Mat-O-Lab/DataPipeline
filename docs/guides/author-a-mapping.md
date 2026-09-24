---
title: Author a Mapping
---

# Author a Mapping

A YARRRML mapping connects the shape of your data (CSVW column annotations) to the shape of your target ontology (a pattern graph). MapToMethod acts as a compiler: it reads both documents, matches entity names to IRI targets, and writes the YARRRML rules for you.

Once a mapping is in the CKAN `mappings` group, it applies automatically to every future upload with a matching column structure.

---

## Prerequisites

- A CSVW JSON-LD document at a **publicly accessible URL** (produced by CSVToCSVW — see [Default Use Case](../pipeline/default-use-case.md))
- A pattern (template graph) in Turtle at a **publicly accessible URL** — e.g., from the [PMDCO pattern library](https://github.com/materialdigital/core-ontology/tree/main/patterns/)
- A running CKAN instance with the `mappings` group created

!!! note "Local or intranet URLs"
    MapToMethod fetches both URLs directly. If your CSVW or pattern is only reachable on a local network, you need a public reverse proxy or file host before proceeding.

---

## Step 1 — Explore data types

`GET /api/types` returns all unique `rdf:type` IRIs in your CSVW document. Use this to understand what entity types are available for mapping.

```bash
curl "https://maptomethod.matolab.org/api/types?url=https://example.org/sample.csvw.json"
```

Response for a typical CSVW:

```json
[
  "http://qudt.org/schema/qudt/QuantityValue",
  "http://qudt.org/schema/qudt/Unit",
  "http://www.w3.org/ns/csvw#Column",
  "http://www.w3.org/ns/csvw#TableGroup",
  "http://www.w3.org/ns/oa#Annotation",
  "http://www.w3.org/ns/prov#Activity",
  "http://www.w3.org/ns/prov#SoftwareAgent"
]
```

The types you'll map from are typically `oa:Annotation` (metadata key-value pairs) and `csvw:Column` (data table columns).

---

## Step 2 — Enumerate data entities

`GET /api/entities` returns a dict of named entities of the specified types. By default it queries for `oa:Annotation` and `csvw:Column`.

```bash
curl "https://maptomethod.matolab.org/api/entities?url=https://example.org/sample.csvw.json"
```

Response (truncated — shows 3 of potentially many columns):

```json
{
  "temperature_C": {
    "uri": "https://example.org/sample.csv/temperature_C",
    "property": "name",
    "text": "temperature_C",
    "type": "http://www.w3.org/ns/csvw#Column"
  },
  "tensile_strength_MPa": {
    "uri": "https://example.org/sample.csv/tensile_strength_MPa",
    "property": "name",
    "text": "tensile_strength_MPa",
    "type": "http://www.w3.org/ns/csvw#Column"
  },
  "material0": {
    "uri": "https://example.org/sample.csv/material0",
    "property": "label",
    "text": "PA6GF30",
    "type": "http://www.w3.org/ns/oa#Annotation"
  }
}
```

Note the entity names (`temperature_C`, `tensile_strength_MPa`) — these are the keys you'll use in the `map` dict in Step 4.

---

## Step 3 — Explore the pattern template

Run the same two calls against your pattern (template) URL to enumerate what entity types and named entities the pattern graph exposes:

```bash
curl "https://maptomethod.matolab.org/api/types?url=https://example.org/my-pattern.ttl"
curl "https://maptomethod.matolab.org/api/entities?url=https://example.org/my-pattern.ttl"
```

The template entity IRIs are what you'll map *to* — they become the object values in the `map` dict.

A minimal [PMDCO pattern](https://github.com/materialdigital/core-ontology/tree/main/patterns/) for a tensile test might expose entities like:
`TensileStrengthValue`, `YieldStrengthValue`, `ElongationValue`.

---

## Step 4 — Build the map dict

The `map` dict pairs each data column name with the IRI of the template entity it should map to:

```json
{
  "tensile_strength_MPa": "https://example.org/pattern#TensileStrengthValue",
  "yield_strength_MPa":   "https://example.org/pattern#YieldStrengthValue",
  "elongation_pct":       "https://example.org/pattern#ElongationValue"
}
```

Use the `exact` column names from `/api/entities` output — case-sensitive. Mismatches produce `rules_skipped > 0` in the validation step.

---

## Step 5 — Generate the mapping

`POST /api/mapping` with the full request body:

```bash
curl -X POST "https://maptomethod.matolab.org/api/mapping" \
  -H "Content-Type: application/json" \
  -d '{
    "data_url":     "https://example.org/sample.csvw.json",
    "template_url": "https://example.org/my-pattern.ttl",
    "predicate":    "http://www.w3.org/ns/oa#hasBody",
    "map": {
      "tensile_strength_MPa": "https://example.org/pattern#TensileStrengthValue",
      "yield_strength_MPa":   "https://example.org/pattern#YieldStrengthValue",
      "elongation_pct":       "https://example.org/pattern#ElongationValue"
    }
  }' \
  --output my-mapping.yaml
```

The response is a YARRRML file download. First ~15 lines of a typical result:

```yaml
# prefixes block
prefixes:
  ex: "https://example.org/"
  pmdco: "https://example.org/pattern#"
  qudt: "http://qudt.org/schema/qudt/"

mappings:
  TensileStrengthValue:
    sources:
      - ["https://example.org/sample.csvw.json~jsonpath", "$['tables'][0]['tableSchema']['columns'][*]"]
    subjects: "ex:result_$(tensile_strength_MPa)"
    predicates-objects:
      - predicates: pmdco:hasValue
        objects:
          value: "$(tensile_strength_MPa)"
          datatype: xsd:double
```

For the full [YARRRML specification](https://rml.io/yarrrml/spec/) and an interactive editor, use [Matey](https://rml.io/yarrrml/matey/).

---

## Step 6 — Validate: check mapping rules

`POST /api/checkmapping` at RDFConverter tests the mapping against your data without producing output. `rules_skipped == 0` is the quality gate.

```bash
curl -X POST "https://rdfconverter.matolab.org/api/checkmapping" \
  -G \
  --data-urlencode "mapping_url=https://example.org/my-mapping.yaml" \
  --data-urlencode "data_url=https://example.org/sample.csvw.json"
```

Successful response:

```json
{ "rules_applicable": 3, "rules_skipped": 0 }
```

`rules_applicable` matches your `map` dict size. `rules_skipped` is 0 — every rule found a match.

---

## Step 7 — Debug if needed

`POST /api/test` returns per-rule statistics and an output preview — use it when `rules_skipped > 0`:

```bash
curl -X POST "https://rdfconverter.matolab.org/api/test" \
  -G \
  --data-urlencode "mapping_url=https://example.org/my-mapping.yaml" \
  --data-urlencode "data_url=https://example.org/sample.csvw.json"
```

Response includes `triple_count`, per-rule stats, and the first N triples of output — enough to diagnose which rules failed and why.

---

## Troubleshooting: `rules_skipped > 0`

`rules_skipped > 0` means one or more column names in your `map` dict do not appear in the data document.

**Common causes:**
- Column name has different capitalisation (the match is case-sensitive)
- You mapped an `oa:Annotation` key name that differs from the CSVW entity name — re-check `/api/entities` output
- The CSVW was regenerated and column names changed

**Quick fix during development:** switch `ckanext.csvwmapandtransform.mapping_strategy` to `best_match` temporarily — it selects the highest-rated partial match so you can see output while iterating. Switch back to `exact` for production.

---

## Step 8 — Upload to CKAN

1. In CKAN, open the dataset that should hold the mapping (or create a new one)
2. Go to **Resources** → **Add Resource**
3. Upload `my-mapping.yaml` — format must be `YAML`
4. Add the dataset to the `mappings` group (Dataset → Groups tab)

All future CSV uploads whose CSVW columns match your mapping exactly will now auto-transform.

---

## What happens next

Every time a new CSV with the same column structure is uploaded to any dataset in your CKAN instance, ckanext-csvwmapandtransform will:

1. Scan the `mappings` group
2. Run `/api/checkmapping` against your mapping
3. If `rules_skipped == 0`, call `/api/createrdf` and add the joined Turtle to the dataset

No user action required after this point.

---

*Next: [Standalone APIs](standalone-apis.md) — use MapToMethod and RDFConverter outside of CKAN.*
