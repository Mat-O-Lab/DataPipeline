---
title: Pipeline Setup Workflow
---

# Pipeline Setup Workflow

This is the complete technical workflow for wiring a new data type into the DataStack pipeline — from raw file to CKAN-published, FAIR-linked knowledge graph. Each step links to the relevant tool or guide.

**Audience:** data engineers and operators setting up a new use case.

**For domain scientists** who want to understand the process conceptually, see [Add a Use Case](add-a-use-case.md) instead.

---

## The workflow

### Step 1 — Make your raw data file accessible

The pipeline services fetch files over HTTP. Your raw data (CSV, JSON, etc.) must be reachable by URL:

- **CKAN-deployed DataStack:** upload your file to CKAN — the URL is automatically assigned. Files in a public or intranet CKAN are accessible to the local pipeline services.
- **dataportal.material-digital.de or other public portals:** the file must be on the public web so remote services can fetch it.
- **Development / testing:** serve the file from a public GitHub repo, a public S3 bucket, or your institution's web server.

---

### Step 2 — Generate the metadata file (CSVW)

The pipeline enriches data through metadata, not the raw file directly. For CSV files the metadata is a CSVW JSON-LD document describing every column with types, units, and provenance.

**If you are using CKAN with `ckanext-csvtocsvw`:** this step is automatic. Uploading a `.csv` file triggers the metadata generation immediately — no action needed.

**If you are working standalone:** use [CSVToCSVW](https://csvtocsvw.matolab.org/) directly.

| Access method | URL |
|---|---|
| Web UI (form-based, no code) | [csvtocsvw.matolab.org](https://csvtocsvw.matolab.org/) |
| Interactive API explorer | [csvtocsvw.matolab.org/api/docs](https://csvtocsvw.matolab.org/api/docs) |

Paste your CSV URL into the `data_url` field and submit. Save the returned CSVW JSON-LD file — you will need it in Step 6.

See [CSVToCSVW](../extractors/csvtocsvw.md) for the full API reference.

---

### Step 3 — (Optional) Make the metadata file accessible

If CKAN is not hosting the CSVW file, upload it to a public URL (GitHub, S3, web server). MapToMethod fetches both the metadata file and the prototype graph by URL in Step 6.

Skip this step if CKAN is handling file hosting — the CSVW URL is shown in the CKAN dataset resource list.

---

### Step 4 — Create or locate a prototype graph

A prototype graph is a Turtle file that defines the semantic structure your data should be enriched into — the entities, their types, and relationships for your measurement or process type.

**If a prototype graph already exists** for your measurement type (e.g. in the [PMDCO pattern library](https://github.com/materialdigital/core-ontology/tree/main/patterns/)), use it directly — copy the raw URL.

**If you need to author a new prototype graph:** use [Ontosphere](https://thhanke.github.io/ontosphere) — a browser-based RDF/OWL 2 DL editor. See [Author a Prototype Graph](author-a-prototype-graph.md) for the full Ontosphere workflow.

---

### Step 5 — Make the prototype graph accessible

Upload the prototype graph `.ttl` file to a public URL. MapToMethod and RDFConverter fetch it over HTTP.

- Commit it to a GitHub repo (use the raw URL)
- Upload to your CKAN instance in a dedicated dataset (e.g. a `prototypes` group)

---

### Step 6 — Create the mapping

The mapping defines the rules between your resource metadata (CSVW) and the prototype graph's named individual slots.

**Option A — MapToMethod (recommended for standard cases):**

| Access method | URL |
|---|---|
| Web UI (form-based) | [maptomethod.matolab.org](https://maptomethod.matolab.org/) |
| Interactive API explorer | [maptomethod.matolab.org/api/docs](https://maptomethod.matolab.org/api/docs) |

Provide the CSVW URL and prototype graph URL. MapToMethod lists the mappable columns and template slots — you pair them and it generates the YARRRML file. See [Author a Mapping](author-a-mapping.md) for the step-by-step workflow.

**Option B — Author YARRRML directly (complex or non-CSV cases):**

Write the YARRRML by hand or use an AI assistant to generate it. For the YARRRML language reference and an interactive editor, see [Matey](https://rml.io/yarrrml/matey/). Real working examples are in the [Author a Mapping — Real-world examples](author-a-mapping.md#real-world-examples) section.

---

### Step 7 — Test the mapping

Before uploading to CKAN, verify your mapping produces correct output.

**Check all rules match:**

```bash
curl -X POST "https://rdfconverter.matolab.org/api/checkmapping" \
  -G \
  --data-urlencode "mapping_url=<your_mapping_url>" \
  --data-urlencode "data_url=<your_csvw_url>"
```

`rules_skipped == 0` is the quality gate. If any rules are skipped, a column name in the mapping does not match any column in the CSVW — check entity names character-by-character.

**Run a test conversion:**

```bash
curl -X POST "https://rdfconverter.matolab.org/api/test" \
  -G \
  --data-urlencode "mapping_url=<your_mapping_url>" \
  --data-urlencode "data_url=<your_csvw_url>"
```

Returns triple count and per-rule statistics. Check that `triple_count` is non-zero and that the triples reference the expected ontology IRIs.

| Access method | URL |
|---|---|
| Web UI (form-based) | [rdfconverter.matolab.org](https://rdfconverter.matolab.org/) |
| Interactive API explorer | [rdfconverter.matolab.org/api/docs](https://rdfconverter.matolab.org/api/docs) |

---

### Step 8 — Verify the output graph

Produce the full joined TTL and inspect it:

```bash
curl -X POST "https://rdfconverter.matolab.org/api/createrdf?return_type=turtle" \
  -G \
  --data-urlencode "mapping_url=<your_mapping_url>" \
  --data-urlencode "data_url=<your_csvw_url>" \
  --output output.ttl
```

**Verification options:**

- **Ontosphere** — load the TTL into [Ontosphere](https://thhanke.github.io/ontosphere), run OWL 2 DL reasoning (Konclude), and optionally run SHACL shapes if your community provides them. Amber dashed edges indicate inferred triples; red nodes indicate inconsistencies.
- **Manual inspection** — open the TTL file and verify that the expected ontology IRIs appear and that literal values are typed correctly.

---

### Step 9 — Upload the mapping to CKAN

1. In CKAN, open or create a dataset for this mapping
2. **Resources → Add Resource → Upload File** — upload your `.yaml` mapping file
3. Set format to `YAML`
4. Add the dataset to the `mappings` group: **Dataset → Groups tab → add `mappings`**

Once the dataset is in the `mappings` group, the pipeline will automatically test it against every future CSV upload. If column names match exactly, the mapping fires without any user action.

---

## Quick reference — all service UIs and API explorers

| Service | What it does | Web UI | API explorer |
|---|---|---|---|
| [CSVToCSVW](https://csvtocsvw.matolab.org/) | CSV → CSVW metadata | [UI](https://csvtocsvw.matolab.org/) | [/api/docs](https://csvtocsvw.matolab.org/api/docs) |
| [MapToMethod](https://maptomethod.matolab.org/) | Generate YARRRML mappings | [UI](https://maptomethod.matolab.org/) | [/api/docs](https://maptomethod.matolab.org/api/docs) |
| [RDFConverter](https://rdfconverter.matolab.org/) | Test and run YARRRML | [UI](https://rdfconverter.matolab.org/) | [/api/docs](https://rdfconverter.matolab.org/api/docs) |
| [Ontosphere](https://thhanke.github.io/ontosphere) | Prototype graph authoring and verification | [UI](https://thhanke.github.io/ontosphere) | — |
