---
title: OpenBISmantic
---

# OpenBISmantic

If your lab uses OpenBIS to manage experiments, OpenBISMantic can extract that data automatically and make it machine-readable for sharing and reuse.

[OpenBIS](https://openbis.ch/) is a widely used electronic lab notebook (ELN) and laboratory information management system (LIMS) — software that labs use to record experiments, organise samples, and store measurement files. OpenBISmantic is a service that wraps your OpenBIS server and exposes everything it contains as structured, linkable data that other systems and scientists can access programmatically.

**Repo:** https://github.com/Mat-O-Lab/OpenBISmantic  
**Image:** `ghcr.io/mat-o-lab/openbismantic`

!!! info "Demonstrator — no active public deployment"
    OpenBISmantic has no hosted instance available for public testing. All curl examples below use a placeholder URL — replace it with the address of your own OpenBISmantic deployment.

---

## What your OpenBIS data becomes

Here is something that often surprises researchers: your experiment records in OpenBIS already describe materials, processes, and measurements in a structured way — you just may not think of them that way. Each sample has an identity, properties, and a history. Each dataset belongs to a specific experiment performed on a specific material. That structure is real and valuable; it just isn't visible to other systems.

OpenBISmantic makes that implicit structure explicit. It reads your OpenBIS content and turns every entry — samples, experiments, datasets, files — into a record with its own permanent web address. Each record is described using shared scientific vocabulary that other tools and data portals can understand. The result is that your lab's data can be:

- **Found** — each sample and dataset has a stable address that doesn't change
- **Understood by machines** — the records use standard vocabularies that search engines and data catalogues recognise
- **Archived** — experiments can be exported as self-contained bundles containing both the data files and a full description of the context
- **Connected** — the records can be linked to domain-specific ontologies (shared scientific glossaries) that describe exactly what was measured and how

You do not need to know anything about how this works technically. A data engineer on your team, or your institution's research data management service, handles the setup.

---

## How OpenBIS content is organised

OpenBIS organises all experimental data in a five-level hierarchy. OpenBISmantic preserves this hierarchy exactly:

```text
Space
└── Project
    └── Collection (Experiment)
        └── Object (Sample)
            └── Dataset
```

Each level is a named container: a **Space** groups projects by team or topic; a **Project** holds one or more **Collections** (experimental campaigns); each Collection contains **Objects** (individual samples or specimens); and each Object can have multiple **Datasets** — the actual measurement files and results. OpenBISmantic assigns each of these a permanent address and makes the relationships between them visible to other systems.

---

## What gets extracted and why

When OpenBISmantic reads your OpenBIS instance, it produces two kinds of output:

**Structured descriptions** — each entity (sample, experiment, dataset, project) is described using three widely adopted scientific vocabularies:

| Vocabulary | What it captures |
|---|---|
| Schema.org | Dataset identity, names, file information |
| DCAT | Where files can be downloaded, their size and format |
| PROV-O | When and how data was generated, and from what |

These vocabularies are not domain-specific — they describe the research data *structure*, not the materials science content. That is intentional: getting the structure right is the first step. Mapping your data to domain-specific scientific terminology (for example, linking a "tensile test" to the precise ontology term for that measurement method) happens in a downstream step using the RDFConverter service.

**Exportable bundles** — the `/export_bundle` endpoint packages all the files from a set of experiments into a ZIP archive that includes a machine-readable description of the whole collection. This format, called [RO-Crate](https://www.researchobject.org/ro-crate/) (Research Object Crate), is a standard way to archive data together with its provenance so the context is never separated from the files.

---

## Content negotiation

The same web address can return your data in different formats depending on what the requesting system asks for. This is called *content negotiation*. A web browser asking for an object gets a human-readable page; a data pipeline asking for the same address gets structured data.

For data engineers integrating this into a pipeline:

```bash
# replace with your OpenBISmantic URL

# Request structured data in JSON-LD format (readable by most data tools)
curl -H "Accept: application/ld+json" \
  "https://your-openbismantic.example.org/object/20230601123456789-42"

# Request the same data in Turtle format (compact RDF notation)
curl -H "Accept: application/x-turtle" \
  "https://your-openbismantic.example.org/object/20230601123456789-42"
```

---

## API endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/object/{perm_id}` | Sample/object as structured data |
| `GET` | `/dataset/{perm_id}` | Dataset + file listing |
| `GET` | `/distribution/{perm_id}/{path}` | Specific file from a dataset |
| `GET` | `/collection/{perm_id}` | Experiment/collection |
| `GET` | `/project/{perm_id}` | Project with experiments |
| `GET` | `/space/{perm_id}` | Space with all projects |
| `GET` | `/instance/` | All spaces (top-level index) |
| `GET` | `/user/{user_id}` | Person/user record |
| `GET` | `/class/{object_type_code}` | Object or collection type definition |
| `GET` | `/object_property/{property_type_code}` | Property type definition |
| `POST` | `/export_bundle` | Select entities → produce RO-Crate ZIP with all files |
| `GET` | `/search/` | Full-text search |

---

## RO-Crate export

`POST /export_bundle` accepts a selection of entities and produces a self-contained ZIP archive. The archive contains all the referenced dataset files plus a `ro-crate-metadata.json` file that describes the entire collection — who created it, when, what it contains, and how the pieces relate.

This format ([RO-Crate](https://www.researchobject.org/ro-crate/)) is designed for long-term archival: the description travels with the data, so the context is never lost. The ZIP can be ingested directly into RDFConverter for the next mapping step.

```bash
# replace with your OpenBISmantic URL
curl -X POST "https://your-openbismantic.example.org/export_bundle" \
  -H "Content-Type: application/ld+json" \
  -d @selected-objects.json \
  --output export.zip
```

---

## Example output

An openBIS sample object as structured data (JSON-LD format). The permanent ID becomes the stable address; Schema.org and DCAT describe the data structure:

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

*Illustrative example — shape matches actual output.*

---

## Deployment

OpenBISmantic requires an OpenBIS instance running version 20.10 or later (needed for the authentication method it uses). Two deployment modes are available.

**Full stack** — includes OpenBIS itself, plus a database and web proxy. Use this if your lab does not already have an OpenBIS instance:

```bash
git clone https://github.com/Mat-O-Lab/OpenBISmantic
cd OpenBISmantic
cp .env.example .env   # fill in ADMIN_PASS, HOST_NAME, POSTGRES_PASSWORD
docker compose up -d
```

**API only** — points at an existing OpenBIS instance. Use this if your institution already runs OpenBIS:

```bash
export OPENBIS_URL=https://openbis.yourinstitution.org
export BASE_URL=https://your-openbismantic.example.org
uvicorn app:app
```

Key environment variables:

| Variable | Description |
|---|---|
| `OPENBIS_URL` | URL of the OpenBIS server to connect to |
| `BASE_URL` | Public base URL for the addresses OpenBISmantic assigns |
| `ADMIN_PASS` | OpenBIS admin password |
| `OPENBISMANTIC_URL` | Self-referencing URL for internal API links |
| `HOST_NAME` | Hostname for the full-stack deployment |

---

## Where this fits in the pipeline

OpenBISmantic is the extraction step for ELN/LIMS data. Its output feeds into the mapping and conversion steps that add domain-specific scientific terminology:

```text
OpenBIS ELN/LIMS (samples + datasets + files)
  → OpenBISmantic   (assigns permanent addresses, describes structure)
  → Structured data (Schema.org + DCAT + PROV-O)
  → MapToMethod     (maps structure to domain-specific scientific vocabulary)
  → RDFConverter    (applies the mapping)
  → Fuseki          (queryable knowledge graph)
```

---

## Limitations

- Requires an accessible OpenBIS instance running version 20.10 or later — no standalone operation
- No active public deployment for testing
- The initial output uses Schema.org, DCAT, and PROV-O — these describe data structure, not domain-specific scientific content. Mapping to domain terminology (for example, linking measurement results to specific material science concepts) requires a downstream mapping step using RDFConverter
