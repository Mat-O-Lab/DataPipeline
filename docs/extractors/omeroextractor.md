---
title: OmeroExtractor
---

!!! warning "Early stage (v0.0.4) — API may change"
    OmeroExtractor is in active development. Endpoint signatures and output structure may change between releases. Pin to a specific image tag in production.

# OmeroExtractor

OmeroExtractor connects to your OMERO server via the OMERO API and produces a **JSON-LD annotation file** — the same format as a CSVW metadata file but for microscopy datasets. Where CSVToCSVW reads a CSV and outputs a structured description of that CSV's columns and values, OmeroExtractor reads an OMERO image record and outputs a structured description of that image's acquisition metadata, channels, ROIs, and provenance.

**Repo:** https://github.com/Mat-O-Lab/OmeroExtractor  
**Image:** `ghcr.io/mat-o-lab/omeroextractor:latest`  
**Version:** v0.0.4 (early stage)

!!! note "No public OMERO instance available for live testing"
    Unlike CSVToCSVW, OmeroExtractor cannot be exercised against a shared public endpoint — it must connect to an OMERO.Web instance you control. The output examples below are taken from real test fixtures in the OmeroExtractor repository (`tests/image83.*`).

---

## What it does

OMERO is a server platform for storing microscopy images together with their acquisition metadata (objectives, exposure settings, channels, timestamps, and drawn ROIs). OmeroExtractor is a FastAPI microservice that sits between your OMERO server and the rest of the pipeline:

1. Calls the OMERO.Web JSON API to retrieve image, dataset, and ROI metadata
2. Fetches raw acquisition parameters from the microscope software (key/value pairs stored as INI-style annotations)
3. Merges both into a JSON-LD file — JSON with extra `@type` and `@id` fields that make the data self-describing

!!! tip "What is JSON-LD?"
    JSON-LD is ordinary JSON with two extra conventions: every object gets an `@type` (what kind of thing it is) and an `@id` (a unique identifier for it). This is how the pipeline attaches machine-readable meaning to metadata without inventing a new file format. See [Semantic Foundation](../pipeline/semantic-foundation.md) for background.

The output is the OMERO analogue of what CSVToCSVW produces for CSV files: a complete, structured description of a microscopy resource that downstream services (MapToMethod → RDFConverter) can read and map to any target vocabulary.

---

## API

Three endpoints cover the main OMERO object types. All share the same query parameters:

| Parameter | Default | Description |
|---|---|---|
| `anonymize` | `true` | Strip PII (owner name, email) from output |
| `format` | `json-ld` | Output serialization: `json-ld` · `turtle` · `longturtle` · `n3` · `nt` · `hext` · `trig` · `xml` |

### `GET /api/image/{id}` — Extract metadata for a single image

Returns full OMERO image metadata including acquisition parameters and provenance.

### `GET /api/dataset/{id}` — Extract metadata for a dataset (collection of images)

Returns metadata for an OMERO dataset (a named group of images).

### `GET /api/rois/{id}` — Extract ROI annotations for an image

Returns Region of Interest annotations associated with an image.

### `GET /api/info` — Service health and version

---

## Output — JSON-LD

The raw JSON-LD output for image 83 begins as follows. This is the real fixture from the OmeroExtractor test suite. The `@type` values are URIs from the OME microscopy vocabulary — these act as globally unique labels for "what kind of thing this is" so any system can interpret them:

```json
// from OmeroExtractor tests/image83.json (v0.0.4)
{
    "@type": "http://www.openmicroscopy.org/Schemas/OME/2016-06#Image",
    "@id": 83,
    "omero:details": {
        "@type": "TBD#Details",
        "owner": {
            "@type": "http://www.openmicroscopy.org/Schemas/OME/2016-06#Experimenter",
            "@id": 52,
            "omero:details": {
                "@type": "TBD#Details",
                "permissions": {
                    "@type": "TBD#Permissions",
                    "perm": "rw----",
                    "canAnnotate": true,
                    "canDelete": false,
                    "canEdit": false,
                    "canLink": true,
                    "isWorldWrite": false,
                    "isWorldRead": false,
                    "isGroupWrite": false,
                    "isGroupRead": false
```

The overall structure is JSON you already understand — nested objects, key/value pairs — with `@type` and `@id` added so downstream tools know what each object represents without guessing from field names.

---

## Output — Turtle (alternative format)

When requesting `?format=turtle`, the same image 83 metadata is emitted as Turtle — a compact text format for the same structured data. You will not normally need to read Turtle directly; it is useful for debugging or loading into a graph database. The first 15 lines of the real fixture:

```turtle
# from OmeroExtractor tests/image83.ttl (v0.0.4)
@prefix oa: <http://www.w3.org/ns/oa#> .
@prefix ome: <https://github.com/Mat-O-Lab/OmeroExtractor/raw/main/ome.ttl#> .
@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix qudt: <http://qudt.org/schema/qudt/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

<> a ome:Image ;
    rdfs:label "190C-1000h_Sample1_Stelle 10 DF 30s.dm3"^^xsd:string ;
    prov:generatedAtTime "2023-06-28T13:51:40.971051"^^xsd:dateTime ;
    prov:wasGeneratedBy <https://metadata.omero.matolab.org/api/image> ;
    ome:datasets "https://omero.matolab.org/api/v0/m/images/83/datasets/"^^xsd:string ;
    ome:download "https://omero.matolab.org/webgateway/archived_files/download/2"^^xsd:anyURI ;
    ome:rawMeta "https://omero.matolab.org/api/v0/m/images/2"^^xsd:anyURI ;
```

The `ome:OriginalMeta` block further in the file captures raw microscope acquisition parameters (magnification, voltage, aperture, etc.) as key/value annotation entries — the same pattern CSVToCSVW uses for CSV column metadata.

---

## Vocabularies used

OmeroExtractor borrows from four established shared vocabularies to label different parts of the output. Think of each vocabulary as a standardised naming scheme for a specific kind of information. See [Semantic Foundation](../pipeline/semantic-foundation.md) for a broader introduction.

| Vocabulary | What it labels |
|---|---|
| **OME** (custom) | Core microscopy objects — Image, Dataset, Channel, ROI. Generated from the official OME XML Schema. |
| **Open Annotation** | Wraps raw acquisition key/value pairs (magnification, voltage, aperture) so they can be reliably identified and mapped |
| **QUDT** | Numeric acquisition values with their physical units (e.g. "30 kV accelerating voltage") |
| **PROV-O** | Provenance — when this file was generated and by which service |

Namespace prefixes (`ome:`, `oa:`, `qudt:`, `prov:`) in the output are shorthand for the full vocabulary URIs shown in the Turtle header above.

---

## Real-world example — BAMresearch/DF-TEM-PAW

The [BAMresearch/DF-TEM-PAW](https://github.com/BAMresearch/DF-TEM-PAW) repository demonstrates OmeroExtractor in a materials characterization workflow: dark-field TEM precipitate analysis on steel samples at BAM (Federal Institute for Materials Research and Testing). It is a complete pipeline run from OMERO image extraction through semantic mapping to a published FAIR dataset.

| Stage | What you find in the repo |
|---|---|
| OmeroExtractor output (JSON-LD) | `image*.json` files |
| Mapping rules | `*-map.yaml` files |
| Joined knowledge graph | `*-joined.ttl` files |

This pipeline is documented in:

> Hanke et al. (2023). *A FAIR Data Infrastructure for Materials Characterization.*
> [doi:10.1007/s40192-023-00331-5](https://doi.org/10.1007/s40192-023-00331-5)

---

## Downstream: MapToMethod → RDFConverter

The JSON-LD output of OmeroExtractor feeds directly into the same Stage 2 services used for CSV data:

```
OMERO Server (images + acquisition metadata)
  → OmeroExtractor /api/image/{id}
  → JSON-LD  (OME vocabulary + acquisition annotations + QUDT + provenance)
  → MapToMethod   (author mapping rules — translate OME fields to your target vocabulary)
  → RDFConverter  (execute the mapping → output in target vocabulary)
  → Fuseki        (queryable graph database)
```

The acquisition annotations in `ome:OriginalMeta` are the primary mapping target. Each acquisition parameter (e.g. accelerating voltage, magnification, aperture) is an annotation entry with a typed numeric value — the same structure MapToMethod expects from CSVToCSVW output. A mapping file authored in MapToMethod translates these OME-annotated values to whatever target vocabulary the domain requires (e.g. PMDco for materials science).

See [Author a mapping](../guides/author-a-mapping.md) to get started with MapToMethod.

---

## Deployment

OmeroExtractor connects to an OMERO.Web instance you control, configured via environment variables. End users need no OMERO credentials — the service authenticates via a configured service account.

| Variable | Required | Description |
|---|---|---|
| `OMERO_WEB_HOST` | yes | OMERO.Web base URL |
| `OMERO_WEB_PUBLIC_USER` | no | Service account username (default: `publicuser`) |
| `OMERO_WEB_PUBLIC_PASSWORD` | yes | Service account password |
| `APP_PORT` | no | External port (default: `80`) |

```bash
docker pull ghcr.io/mat-o-lab/omeroextractor:latest
docker compose up
```

The HTML form UI for ad-hoc image lookup is available at `/`. The OpenAPI specification is at `/api/docs`.

---

## Limitations

- Requires an accessible OMERO.Web instance — no shared public OMERO server is available for testing
- v0.0.4 — early stage; API surface and output structure may change between releases
- One image or dataset per API call — no bulk/batch extraction
- Translating OME acquisition fields to a target vocabulary requires authoring a mapping in MapToMethod; this is not automatic
