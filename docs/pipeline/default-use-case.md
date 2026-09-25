---
title: Default Use Case — CSV Lab Data
---

# Default Use Case — CSV Lab Data

> **Create complete, consistent metadata for a non-semantic resource — then use semantic technologies to transform that further.**

---

## Where this fits

This page describes the **core transformation path** in Block 2 — Semantic Data Foundation. DataStack takes a plain CSV file and turns it into a richly described, machine-readable dataset that meets FAIR data principles — without you writing any code.

If you are new to what "semantic" or "linked data" means and why it matters, start with [Semantic Foundation](semantic-foundation.md) first. This page focuses on what happens step by step, and what you end up with.

---

## Pipeline Overview

The DataStack pipeline has three parallel lanes working together. The diagram below shows the full picture; the rest of this page walks through the **CSV path** in detail.

- **Data Sources lane** (left): the file types the system can ingest — CSV lab files, microscopy images, database entries
- **Processing lane** (centre): the automated services that read, annotate, and transform your data
- **CKAN lane** (right): CKAN orchestrates the services automatically and stores every output as a resource in your dataset

```mermaid
graph LR
  subgraph Sources ["Data Sources (Stage 1)"]
    direction TB
    csv[CSV Lab File<br/>tensile test, spectroscopy…]
    img[Microscopy Image<br/>OMERO server]
    eln[ELN / LIMS entry<br/>openBIS]
    db[(SQL Database)]
  end

  subgraph Extractors ["Stage 1 — Extractors"]
    direction TB
    CSVToCSVW["CSVToCSVW<br/>→ CSVW JSON-LD"]
    OmeroExtractor["OmeroExtractor<br/>→ OME JSON-LD"]
    OpenBISmantic["OpenBISmantic<br/>→ Schema.org JSON-LD"]
    Ontop["Ontop<br/>→ Virtual SPARQL"]
  end

  subgraph Semantics ["Stage 2 — Mapping & Conversion"]
    MapToMethod["MapToMethod<br/>(mapping authoring)"]
    RDFConverter["RDFConverter<br/>YARRRML/RML execution"]
    KG["RDF Knowledge Graph<br/>target ontology aligned"]
  end

  subgraph CKAN ["CKAN Integration (parallel automation lane)"]
    direction TB
    ext1["ckanext-csvtocsvw<br/>auto-calls CSVToCSVW on CSV upload"]
    ext2["ckanext-csvwmapandtransform<br/>auto-selects mapping + calls RDFConverter"]
    ext3["ckanext-fuseki<br/>syncs RDF to Fuseki triple store"]
    Fuseki[(Fuseki<br/>SPARQL endpoint)]
  end

  csv --> CSVToCSVW
  img --> OmeroExtractor
  eln --> OpenBISmantic
  db --> Ontop

  CSVToCSVW --> MapToMethod
  OmeroExtractor --> MapToMethod
  OpenBISmantic --> MapToMethod

  MapToMethod --> RDFConverter
  RDFConverter --> KG

  ext1 -. automates .-> CSVToCSVW
  ext2 -. automates .-> RDFConverter
  KG --> ext3
  ext3 --> Fuseki
```

The rest of this page details the **CSV lab data path** — the default and most automated route.

---

## What is the "default use case"?

You have a CSV file from a lab instrument — tensile test results, spectroscopy measurements, material property tables. The file sits on a network drive. DataStack turns it into a queryable, FAIR dataset that other researchers and systems can understand.

The pipeline runs **automatically** after upload. You do not write any code. The only step that requires a manual action is loading the final dataset into the query database (Step 7 below), and even that is a single button click.

---

## The Automation Chain

Upload your CSV to CKAN and the following seven steps happen:

---

**Step 1 — CKAN detects the format**

The file extension (`.csv`, `.asc`, `.tsv`, `.txt`) tells CKAN to start the pipeline. Any other format is ignored. No configuration is needed on your part.

---

**Step 2 — CKAN creates a column-description file**

A service called CSVToCSVW reads your CSV and produces a companion metadata document. This file — called a **CSVW file** (short for "CSV on the Web", a [W3C standard](https://www.w3.org/TR/tabular-data-primer/)) — annotates every column with:

- **Standard unit labels.** Column names like `tensile_strength_MPa` are matched to internationally recognised measurement unit identifiers from [QUDT](https://qudt.org/) — a public vocabulary of physical quantities and units. This makes the column machine-readable to any system that understands units.
- **Provenance.** A record of which service produced the file, following the [PROV-O](https://www.w3.org/TR/prov-o/) standard. Think of it as a digital audit trail.
- **Annotation rows.** Any header rows above your data table are preserved as structured annotations.

The CSVW file appears as a new resource in your CKAN dataset. The excerpt below is from the real [`example2-metadata.json`](https://raw.githubusercontent.com/Mat-O-Lab/CSVToCSVW/main/examples/example2-metadata.json) produced by CSVToCSVW for a typical lab instrument export — German-language headers, multi-channel measurement data. You do not need to understand this format — it is produced entirely automatically:

```json
{
  "@context": [
    "http://www.w3.org/ns/csvw",
    {
      "oa":   "http://www.w3.org/ns/oa#",
      "qudt": "http://qudt.org/schema/qudt/",
      "prov": "http://www.w3.org/ns/prov#",
      "csv":  "https://github.com/Mat-O-Lab/CSVToCSVW/raw/main/examples/example2.csv/"
    }
  ],
  "@type": "http://www.w3.org/ns/csvw#TableGroup",
  "notes": [{
    "@id": "csv:Probe0",
    "@type": "oa:Annotation",
    "label": "Probe",
    "oa:hasBody": [{ "oa:value": "AWA_3_03_Rz" }]
  }],
  "tables": [{
    "tableSchema": {
      "columns": [
        {
          "name": "MaschineMm",
          "titles": ["Maschine [mm]", "MaschineMm"],
          "qudt:unit": { "@id": "http://qudt.org/vocab/unit/MilliM" }
        },
        {
          "name": "KraftKn",
          "titles": ["Kraft [kN]", "KraftKn"],
          "qudt:unit": { "@id": "http://qudt.org/vocab/unit/KiloN" }
        }
      ]
    }
  }]
}
```

*The original header `"Kraft [kN]"` is preserved alongside the normalised name `"KraftKn"`, and the column now carries `unit:KiloN` — an internationally agreed identifier for kilonewtons. The metadata row `"Probe"` with its specimen identifier is captured as an `oa:Annotation`. Everything is produced automatically from the CSV.*

---

**Step 3 — The data table becomes browsable**

CKAN imports the CSV rows into its built-in data store. Your measurements are now accessible through a table preview in the browser — you can sort, filter, and explore the numbers without downloading anything.

---

**Step 4 — CKAN converts the column-description file to linked-data format**

The column-description file is converted into a format called **Turtle** — a compact notation for linked data (structured information where every concept and relationship has a stable, web-addressable identifier). A second resource file appears in the dataset. This file is the input for the mapping step.

You do not need to work with this file directly.

---

**Step 5 — CKAN identifies what the data is about**

CKAN searches its `mappings` group — a library of rule files, each encoding the semantic structure of one data type (tensile test, TEM detection run, Catena-X material record, …). It tests each rule file against your column structure. When one matches, two things happen simultaneously:

1. The pipeline knows *which rule file to apply* — no manual selection needed.
2. The pipeline knows *what kind of measurement or process this data represents* — the matched rule file carries that semantic identity.

!!! note "Matching = identification"
    The rule file is not just a transformation recipe. It is the pipeline's way of recognising what your data is about. Every column name, unit annotation, and structural pattern in your CSV narrows the match until only one rule file fits. This is what makes the pipeline self-organising: upload data in a known format and the pipeline classifies and enriches it automatically.

These rule files are what data engineers author once per data type. See [Author a Mapping](../guides/author-a-mapping.md) if your data needs a new rule file.

---

**Step 6 — CKAN produces the linked dataset**

The matching rule file and your data are sent to a transformation service. It applies the rules and produces a **joined linked-data file** — your measurements expressed using a shared materials science vocabulary called [PMDco](https://github.com/materialdigital/core-ontology) (Platform MaterialDigital Core Ontology). This means concepts like "tensile test result" or "yield strength" now carry the same identifier they would in any other dataset that uses PMDco — making your data directly comparable and combinable with other labs' data.

The joined file appears in your CKAN dataset. The excerpt below is from the real [`detection_runs-joined.ttl`](https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs-joined.ttl) — a pipeline output from BAMresearch TEM microscopy detection data. The structure is the same regardless of data type: PMDco process and quality nodes, QUDT quantity values, provenance links.

```turtle
@prefix co:    <https://w3id.org/pmd/co/> .
@prefix qudt:  <http://qudt.org/schema/qudt/> .
@prefix qunit: <http://qudt.org/vocab/unit/> .
@prefix prov:  <http://www.w3.org/ns/prov#> .

row0:ArtificalAging a co:AgingProcess ;
    co:input  row0:specimenAgingTemperature,
              row0:specimenAgingTime ;
    co:output row0:transmissionElectronMicroscopeSpecimen ;
    co:nextProcess row0:darkfieldTransmissionElectronMicroscopyImaging .

ns1:table-1-AgingTempC
    [ a qudt:QuantityValue ;
      qudt:unit  qunit:DEG_C ;
      qudt:value 1.9e+02 ] .
```

*Each measurement is now a typed PMDco process node (`co:AgingProcess`) with typed inputs and outputs. Numeric values carry QUDT unit identifiers (`qunit:DEG_C`). The same structure applies to tensile tests, composition measurements, or any other data type — only the process class and property IRIs change.*

---

**Step 7 — Load into the query database (manual)**

!!! warning "Manual step"
    Loading your linked dataset into the query database is **not automatic**. Automatic sync is implemented in the code but currently disabled. You trigger it with a single button in the CKAN dataset view — no commands or technical steps needed.

    This is a known limitation — see [Capability Map](capability-map.md) for context.

Once triggered, your linked dataset is loaded into **Fuseki** — a dedicated query database for linked data. CKAN then adds a query-interface link to your dataset so you and your collaborators can search and explore the data directly in a browser, using tools like Sparklis or YASGUI, without downloading or installing anything.

---

## What you end up with

After the full pipeline runs, your CKAN dataset contains four resources:

| Resource | Format | What it is |
|---|---|---|
| `yourfile.csv` | CSV | Your original measurement file, unchanged |
| `yourfile.csvw.json` | CSVW (JSON-LD) | Column-description file: every column annotated with standard unit identifiers and provenance |
| `yourfile.ttl` | Turtle | The column descriptions in linked-data notation — input for the mapping step |
| `yourfile-joined.ttl` | Turtle | Your measurements expressed in the shared PMDco vocabulary — the FAIR output |

Plus, after loading into the query database:

- A live query interface scoped to your dataset
- A browser link in the CKAN resource list — click it to start exploring your data with Sparklis or YASGUI

---

## Where to go next

- **Deploy the stack yourself:** [Quickstart](../guides/quickstart.md)
- **Create a mapping rule file for your data:** [Author a Mapping](../guides/author-a-mapping.md)
- **See what data types and sources the pipeline supports:** [Capability Map](capability-map.md)

---

---

**Publications describing this pipeline:**

- Nasrabadi, Hanke et al. (2023). *Toward a digital materials mechanical testing lab.* Computers in Industry, 153, 104016. [doi:10.1016/j.compind.2023.104016](https://doi.org/10.1016/j.compind.2023.104016) — describes the CSV→CSVW→YARRRML→RDF workflow with real tensile test data
- Hanke et al. (2023). *FAIR microscopy data via the Mat-O-Lab pipeline.* Scientific Data (Nature). [doi:10.1038/s41597-023-02244-6](https://doi.org/10.1038/s41597-023-02244-6)
