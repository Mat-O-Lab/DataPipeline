---
title: OpenBISmantic
---

# OpenBISmantic

FastAPI microservice that exposes an [openBIS](https://openbis.ch/) ELN/LIMS instance as Linked Data. Every openBIS entity — samples, datasets, experiments, projects — becomes a resolvable RDF resource with a stable IRI. It is the Stage 1 extractor for ELN/LIMS data in the Mat-O-Lab pipeline.

**Repo:** https://github.com/Mat-O-Lab/OpenBISmantic  
**Image:** `ghcr.io/mat-o-lab/openbismantic`

!!! info "Demonstrator — no active public deployment"
    OpenBISmantic has no hosted instance available for public testing. All curl examples below use a placeholder URL — replace it with the address of your own OpenBISmantic deployment.

---

## What it does

### The openBIS hierarchy

openBIS organises all experimental data in a five-level hierarchy:

```text
Space
└── Project
    └── Collection (Experiment)
        └── Object (Sample)
            └── Dataset
```

Each level is a named container: a **Space** groups projects by team or topic; a **Project** holds one or more **Collections** (experimental campaigns); each Collection contains **Objects** (individual samples or specimens); and each Object can have multiple **Datasets** — the actual measurement files. OpenBISmantic maps every node in this hierarchy to a resolvable IRI.

### What OpenBISmantic exposes

OpenBISmantic wraps the openBIS server with a FastAPI layer that:

- Assigns **persistent IRIs** to every entity based on openBIS permanent IDs (permIds)
- Serves each entity as RDF via content negotiation — the same URL returns JSON-LD, Turtle, or HTML depending on the `Accept` header
- Exports **RO-Crate** bundles (ZIP with all dataset files + `ro-crate-metadata.json`) for portable archival
- Uses Schema.org, DCAT, and PROV-O to describe the research data structure

The output feeds MapToMethod + RDFConverter for domain-specific ontology mapping (e.g. to MSEO or PMDco).

---

## API endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/object/{perm_id}` | Sample/object as RDF |
| `GET` | `/dataset/{perm_id}` | Dataset + file listing as RDF |
| `GET` | `/distribution/{perm_id}/{path}` | Specific file from a dataset |
| `GET` | `/collection/{perm_id}` | Experiment/collection as RDF |
| `GET` | `/project/{perm_id}` | Project with experiments as RDF |
| `GET` | `/space/{perm_id}` | Space with projects as RDF |
| `GET` | `/instance/` | All spaces (top-level index) |
| `GET` | `/user/{user_id}` | Person/user as RDF |
| `GET` | `/class/{object_type_code}` | Object or collection type definition |
| `GET` | `/object_property/{property_type_code}` | Property type definition |
| `POST` | `/export_bundle` | Accept a JSON-LD graph → produce RO-Crate ZIP with all files |
| `GET` | `/search/` | Full-text search |

Content negotiation — same URL, different `Accept` header:

```bash
# replace with your OpenBISmantic URL
# JSON-LD
curl -H "Accept: application/ld+json" \
  "https://your-openbismantic.example.org/object/20230601123456789-42"

# replace with your OpenBISmantic URL
# Turtle
curl -H "Accept: application/x-turtle" \
  "https://your-openbismantic.example.org/object/20230601123456789-42"
```

---

## Output — JSON-LD (illustrative example)

An openBIS sample object expressed as JSON-LD. The `permId` becomes the stable IRI; Schema.org and DCAT describe the data structure:

```json
{
  "@context": {
    "schema": "https://schema.org/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "prov": "http://www.w3.org/ns/prov#"
  },
  "@id": "https://your-openbismantic.example.org/object/20230601123456789-42",
  "@type": "schema:Dataset",
  "schema:name": "Sample_PA6GF30_Run1",
  "schema:identifier": "20230601123456789-42",
  "dcat:distribution": [
    {
      "@type": "dcat:Distribution",
      "dcat:downloadURL": "https://your-openbismantic.example.org/distribution/20230601123456789-42/data.xlsx",
      "dcat:byteSize": 48320
    }
  ],
  "prov:wasGeneratedBy": {
    "@id": "https://your-openbismantic.example.org"
  }
}
```

*# illustrative example — shape matches actual output*

---

## Ontologies used

| Vocabulary | Role |
|---|---|
| Schema.org | Dataset, File, Person metadata |
| DCAT | Distributions, download URLs, byte sizes |
| PROV-O | Activities, derivation chains, timestamps |
| Custom Mat-O-Lab/openBIS namespace | Dataset codes, datastore relationships |

---

## RO-Crate export

`POST /export_bundle` accepts a JSON-LD graph of selected entities and produces a ZIP containing all referenced dataset files plus a `ro-crate-metadata.json` conforming to the [RO-Crate spec](https://www.researchobject.org/ro-crate/).

```bash
# replace with your OpenBISmantic URL
curl -X POST "https://your-openbismantic.example.org/export_bundle" \
  -H "Content-Type: application/ld+json" \
  -d @selected-objects.json \
  --output export.zip
```

The ZIP can be ingested directly into RDFConverter since RDFConverter accepts JSON-LD input.

---

## Deployment

OpenBISmantic requires an openBIS instance. Two modes:

**Full stack** (includes openBIS + PostgreSQL + nginx):

```bash
git clone https://github.com/Mat-O-Lab/OpenBISmantic
cd OpenBISmantic
cp .env.example .env   # fill in ADMIN_PASS, HOST_NAME, POSTGRES_PASSWORD
docker compose up -d
```

**API only** (point at an existing openBIS instance):

```bash
export OPENBIS_URL=https://openbis.yourinstitution.org
export BASE_URL=https://your-openbismantic.example.org
uvicorn app:app
```

Key environment variables:

| Variable | Description |
|---|---|
| `OPENBIS_URL` | URL of the openBIS server |
| `BASE_URL` | Public base URL for OpenBISmantic IRIs |
| `ADMIN_PASS` | openBIS admin password |
| `OPENBISMANTIC_URL` | Self-referencing URL for API links |
| `HOST_NAME` | Hostname for the full-stack deployment |

---

## Pipeline position

```text
openBIS ELN/LIMS (samples + datasets + files)
  → OpenBISmantic /object, /dataset, /collection, …
  → JSON-LD / RDF (Schema.org + DCAT + PROV-O)
  → MapToMethod  (YARRRML mapping to target ontology)
  → RDFConverter (RML execution → domain-specific RDF)
  → Fuseki       (SPARQL endpoint)
```

---

## Limitations

- Requires an accessible openBIS instance — no standalone operation
- No active public deployment for testing
- Output uses Schema.org/DCAT/PROV-O — domain-specific mapping (e.g. to MSEO) requires a downstream YARRRML mapping
- No SPARQL knowledge required by end users
