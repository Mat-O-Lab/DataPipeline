---
title: OmeroExtractor
---

# OmeroExtractor

FastAPI microservice that extracts metadata from an [OMERO](https://www.openmicroscopy.org/) microscopy image server and outputs it as semantically enriched JSON-LD / RDF. It is the Stage 1 extractor for microscopy data in the Mat-O-Lab pipeline.

**Repo:** https://github.com/Mat-O-Lab/OmeroExtractor  
**Public API:** https://metadata.omero.matolab.org · [API docs](https://metadata.omero.matolab.org/api/docs)  
**Image:** `ghcr.io/mat-o-lab/omeroextractor:latest`  
**Version:** v0.0.4 (early stage)

---

## What it does

OMERO stores microscopy images and their acquisition metadata (objectives, exposure, channels, timestamps, ROIs) on a server. OmeroExtractor bridges OMERO and the semantic web:

1. Connects to the OMERO.Web JSON API and pulls structured image, dataset, and ROI metadata
2. Fetches raw acquisition parameters from the microscope software (INI-style key/value pairs)
3. Merges both into RDF triples using the OME ontology (`ome.ttl` — generated from the official OME XSD schema)

The output is the OMERO analogue of what CSVToCSVW produces for CSV files: complete, consistent metadata for a non-semantic resource, expressed as JSON-LD. From there, the standard MapToMethod → RDFConverter path applies to map to any target ontology.

---

## API

### `GET /api/image/{id}` — Extract metadata for a single image

```bash
curl "https://metadata.omero.matolab.org/api/image/83?anonymize=true&format=turtle" \
  --output image83.ttl
```

### `GET /api/dataset/{id}` — Extract metadata for a dataset (collection of images)

```bash
curl "https://metadata.omero.matolab.org/api/dataset/12?anonymize=true&format=json-ld" \
  --output dataset12.json
```

### `GET /api/rois/{id}` — Extract ROI annotations for an image

```bash
curl "https://metadata.omero.matolab.org/api/rois/83?anonymize=true&format=json-ld" \
  --output rois83.json
```

All three endpoints share the same query parameters:

| Parameter | Default | Description |
|---|---|---|
| `anonymize` | `true` | Strip PII (owner name, email) from output |
| `format` | `json-ld` | Output serialization: `json-ld` · `turtle` · `longturtle` · `n3` · `nt` · `hext` · `trig` · `xml` |

---

## Output — Turtle RDF

Real output for image 83 from the public OMERO instance (truncated — anonymized):

```turtle
@prefix oa:   <http://www.w3.org/ns/oa#> .
@prefix ome:  <https://github.com/Mat-O-Lab/OmeroExtractor/raw/main/ome.ttl#> .
@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix qudt: <http://qudt.org/schema/qudt/> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .

<> a ome:Image ;
    rdfs:label "190C-1000h_Sample1_Stelle 10 DF 30s.dm3"^^xsd:string ;
    prov:generatedAtTime "2023-06-28T13:51:40"^^xsd:dateTime ;
    prov:wasGeneratedBy <https://metadata.omero.matolab.org/api/image> ;
    ome:download "https://omero.matolab.org/webgateway/archived_files/download/2"^^xsd:anyURI ;
    ome:relates_to [
        a ome:OriginalMeta ;
        ome:notes <GlobalMetadataActualMagnification>,
                  <GlobalMetadataAcceleratingVoltage>,
                  <GlobalMetadataSpotSize>,
                  <GlobalMetadataAperture>
        # ... acquisition parameters as oa:Annotation entries
    ] .
```

The `ome:OriginalMeta` block captures raw microscope acquisition parameters (magnification, voltage, aperture, etc.) as `oa:Annotation` entries — the same pattern CSVToCSVW uses for CSV metadata rows.

---

## Ontologies used

| Vocabulary | Namespace | Role |
|---|---|---|
| OME (custom) | `https://github.com/Mat-O-Lab/OmeroExtractor/raw/main/ome.ttl#` | Core microscopy types (Image, Dataset, Channel, ROI) — generated from `ome.xsd` |
| Open Annotation | `http://www.w3.org/ns/oa#` | Wraps acquisition key/value pairs as `oa:Annotation` |
| QUDT | `http://qudt.org/schema/qudt/` | Numeric acquisition values typed as `qudt:QuantityValue` |
| PROV-O | `http://www.w3.org/ns/prov#` | `prov:wasGeneratedBy`, `prov:generatedAtTime` |

---

## Real-world example

The [BAMresearch/DF-TEM-PAW](https://github.com/BAMresearch/DF-TEM-PAW) repository demonstrates OmeroExtractor in a materials characterization workflow — dark-field TEM precipitate analysis on steel samples. The pipeline is documented in:

> Hanke et al. (2023). *FAIR microscopy data via the Mat-O-Lab pipeline.*  
> [doi:10.1007/s40192-023-00331-5](https://link.springer.com/article/10.1007/s40192-023-00331-5)

---

## Deployment

OmeroExtractor connects to an OMERO.Web instance configured via environment variables. End users need no OMERO credentials — the service authenticates via a configured service account.

```bash
# Minimal .env for running against an existing OMERO.Web instance
APP_PORT=80
OMERO_WEB_HOST=https://omero.yourinstitution.org
OMERO_WEB_PUBLIC_USER=publicuser
OMERO_WEB_PUBLIC_PASSWORD=<password>
```

```bash
docker pull ghcr.io/mat-o-lab/omeroextractor:latest
docker compose up
```

The HTML form UI for ad-hoc image lookup is available at `/`.

---

## Pipeline position

```
OMERO.Server (images + acquisition metadata)
  → OmeroExtractor /api/image/{id}
  → JSON-LD (OME ontology + OA annotations + QUDT + PROV-O)
  → MapToMethod  (YARRRML mapping authoring)
  → RDFConverter (RML execution → target ontology RDF)
  → Fuseki       (SPARQL endpoint)
```

---

## Limitations

- Requires an accessible OMERO.Web instance configured via env vars
- v0.0.4 — early stage; API surface may change
- No bulk/batch extraction — one image or dataset per API call
- Domain-specific mapping from OME ontology to target ontologies (e.g. PMDco) requires a YARRRML mapping via MapToMethod
