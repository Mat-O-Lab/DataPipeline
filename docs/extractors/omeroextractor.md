---
title: OmeroExtractor
---

!!! warning "Early stage (v0.0.4) — API may change"
    OmeroExtractor is in active development. Endpoint signatures and ontology alignment may change between releases. Pin to a specific image tag in production.

# OmeroExtractor

FastAPI microservice that extracts metadata from an [OMERO](https://www.openmicroscopy.org/) microscopy image server and outputs it as semantically enriched JSON-LD / RDF. It is the Stage 1 extractor for microscopy data in the Mat-O-Lab pipeline.

**Repo:** https://github.com/Mat-O-Lab/OmeroExtractor  
**Image:** `ghcr.io/mat-o-lab/omeroextractor:latest`  
**Version:** v0.0.4

!!! note "No public OMERO instance available for live testing"
    Unlike CSVToCSVW, OmeroExtractor cannot be exercised against a shared public endpoint — it must connect to an OMERO.Web instance you control. The output examples below are taken from real test fixtures in the OmeroExtractor repository (`tests/image83.*`).

---

## What it does

OMERO stores microscopy images and their acquisition metadata (objectives, exposure, channels, timestamps, ROIs) on a server. OmeroExtractor bridges OMERO and the semantic web:

1. Connects to the OMERO.Web JSON API and pulls structured image, dataset, and ROI metadata
2. Fetches raw acquisition parameters from the microscope software (INI-style key/value pairs)
3. Merges both into RDF triples using the OME ontology (`ome.ttl` — generated from the official OME XSD schema via xsd2owl)

The output is the OMERO analogue of what CSVToCSVW produces for CSV files: complete, consistent metadata for a non-semantic resource, expressed as JSON-LD. From there, the standard MapToMethod → RDFConverter path applies to map to any target ontology.

---

## API

Three endpoints cover the main OMERO object types. All share the same query parameters:

| Parameter | Default | Description |
|---|---|---|
| `anonymize` | `true` | Strip PII (owner name, email) from output |
| `format` | `json-ld` | Output serialization: `json-ld` · `turtle` · `longturtle` · `n3` · `nt` · `hext` · `trig` · `xml` |

### `GET /api/image/{id}` — Extract metadata for a single image

Returns full OMERO image metadata including acquisition parameters and provenance as RDF.

### `GET /api/dataset/{id}` — Extract metadata for a dataset (collection of images)

Returns metadata for an OMERO dataset (a named group of images).

### `GET /api/rois/{id}` — Extract ROI annotations for an image

Returns Region of Interest annotations associated with an image as `oa:Annotation` entries.

### `GET /api/info` — Service health and version

---

## Output — JSON-LD

The raw JSON-LD output for image 83 begins as follows. This is the real fixture from the OmeroExtractor test suite — the `@type` and `@id` fields use the full OME XSD namespace directly:

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

The `@type` values are OME XSD IRIs. OmeroExtractor maps these to the `ome.ttl` vocabulary when serializing to Turtle or other RDF formats.

---

## Output — Turtle RDF

When requesting `?format=turtle`, the same image 83 metadata is emitted as Turtle with full prefix declarations and PROV-O provenance. The first 15 lines of the real fixture:

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

The `ome:OriginalMeta` block further in the file captures raw microscope acquisition parameters (magnification, voltage, aperture, etc.) as `oa:Annotation` entries — the same pattern CSVToCSVW uses for CSV metadata rows.

---

## Ontologies used

| Vocabulary | Namespace | Role |
|---|---|---|
| OME (custom) | `https://github.com/Mat-O-Lab/OmeroExtractor/raw/main/ome.ttl#` | Core microscopy types (Image, Dataset, Channel, ROI) — generated from `ome.xsd` |
| Open Annotation | `http://www.w3.org/ns/oa#` | Wraps raw acquisition key/value pairs as `oa:Annotation` |
| QUDT | `http://qudt.org/schema/qudt/` | Numeric acquisition values typed as `qudt:QuantityValue` |
| PROV-O | `http://www.w3.org/ns/prov#` | `prov:wasGeneratedBy`, `prov:generatedAtTime` |

---

## Real-world example — BAMresearch/DF-TEM-PAW

The [BAMresearch/DF-TEM-PAW](https://github.com/BAMresearch/DF-TEM-PAW) repository demonstrates OmeroExtractor in a materials characterization workflow: dark-field TEM precipitate analysis on steel samples at BAM (Federal Institute for Materials Research and Testing). It is a complete pipeline run from OMERO image extraction through semantic mapping to a published FAIR dataset.

| Stage | File |
|---|---|
| OmeroExtractor output (JSON-LD) | `image*.json` in the repository |
| YARRRML mapping | `*-map.yaml` files |
| Joined knowledge graph | `*-joined.ttl` files |

This pipeline is documented in:

> Hanke et al. (2023). *A FAIR and open data infrastructure for materials science.*
> [doi:10.1038/s41597-023-02244-6](https://doi.org/10.1038/s41597-023-02244-6)

---

## Downstream: MapToMethod → RDFConverter

The JSON-LD output of OmeroExtractor feeds directly into the same Stage 2 services used for CSV data:

```
OMERO.Server (images + acquisition metadata)
  → OmeroExtractor /api/image/{id}
  → JSON-LD  (OME ontology + oa:Annotation + QUDT + PROV-O)
  → MapToMethod   (YARRRML mapping authoring — map ome: to PMDco or other target)
  → RDFConverter  (RML execution → target ontology RDF)
  → Fuseki        (SPARQL endpoint)
```

The `ome:OriginalMeta` acquisition annotations are the primary mapping target. Each acquisition parameter (e.g. accelerating voltage, magnification, aperture) becomes an `oa:Annotation` with a `qudt:QuantityValue` body — the same structure MapToMethod expects from CSVToCSVW output. A YARRRML mapping file authored in MapToMethod translates these OME-annotated values to whatever target ontology (e.g. PMDco) the domain requires.

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

- Requires an accessible OMERO.Web instance configured via env vars — no shared public OMERO server is available
- v0.0.4 — early stage; API surface may change
- No bulk/batch extraction — one image or dataset per API call
- Domain-specific mapping from OME ontology to target ontologies (e.g. PMDco) requires a YARRRML mapping via MapToMethod
