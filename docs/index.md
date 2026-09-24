---
title: Mat-O-Lab DataStack
---

# Mat-O-Lab DataStack

> **Create complete, consistent metadata for a non-semantic resource — then use semantic technologies (RML/YARRRML) to transform that further.**

The Mat-O-Lab DataStack is a production-proven pipeline that turns raw laboratory and industrial data files into FAIR, queryable RDF knowledge graphs — without requiring researchers to write ontology code.

---

## Where do you want to start?

=== "I'm a researcher or data manager"
    **→ [Pipeline](pipeline/index.md)**

    Understand what happens to your data, what outputs you get, and how the pipeline handles different resource types.

=== "I'm a data engineer or DevOps"
    **→ [Guides](guides/index.md)**

    Deploy and configure the stack. Create mappings. Use the microservices standalone.

=== "I'm an ontology engineer"
    **→ [Reference](reference/index.md)**

    API contracts, configuration options, and data format specifications.

=== "I'm a developer or integrator"
    **→ [Components](components/index.md)**

    Deep-dive into individual services, their repos, images, and integration points.

---

## Real-world examples

Three domains — one pipeline pattern:

| Domain | Source format | Output | Real example |
|---|---|---|---|
| Lab / materials science | CSV measurement file (tensile test, spectroscopy) | PMDco-aligned RDF knowledge graph | [IOFMaterialsTutorial](https://github.com/Mat-O-Lab/IOFMaterialsTutorial) |
| Microscopy imaging | OMERO server metadata (OME-XML) | OME-ontology RDF, SPARQL-queryable | [BAMresearch DF-TEM-PAW](https://github.com/BAMresearch/DF-TEM-PAW) |
| Automotive supply chain | SAMM / Catena-X JSON payload | SAMM-aligned RDF, PMDco cross-walk | [futurecarproduction.materialsdata.space](https://futurecarproduction.materialsdata.space) |

---

## What the pipeline does

The DataStack handles three distinct resource types through one consistent pattern:

| Pipeline step | CSV lab data | OMERO microscopy | SAMM / Catena-X |
|---|---|---|---|
| **Metadata creation** | CSVToCSVW annotates CSV with CSVW JSON-LD | OmeroExtractor extracts OME metadata | Flat JSON payload from Catena-X dataspace |
| **YARRRML mapping** | MapToMethod + OntosphereIO pattern | MapToMethod + OME ontology pattern | YARRRML with JSONPath iterators |
| **RDF output** | RDFConverter executes RML → knowledge graph | RDFConverter executes RML → knowledge graph | RDFConverter → SAMM-aligned intermediate RDF |
| **SPARQL endpoint** | Fuseki (auto-loaded via ckanext-fuseki) | Fuseki (auto-loaded via ckanext-fuseki) | Fuseki + SPARQL CONSTRUCT → PMDco graph |

The pipeline is **ontology-agnostic**: the same infrastructure handles materials science (PMDco), automotive supply chains (SAMM/CX), and digital twin metadata (IDTA/AAS).

---

## Live instances

The pipeline runs in production at two public portals:

- **[futurecarproduction.materialsdata.space](https://futurecarproduction.materialsdata.space/)** — Future Car Production Materials Data Space (26 Catena-X SAMM mapping datasets)
- **[dataportal.material-digital.de](https://dataportal.material-digital.de/)** — Material Digital Data Portal (cross-project use case, PA6GF30 / Catena-X → PMDco)

---

## Publication

The microscopy pipeline path is described in a peer-reviewed paper:

> Hanke et al. (2023). *FAIR microscopy data via the Mat-O-Lab pipeline.*
> Scientific Data (Nature).
> [doi:10.1038/s41597-023-02244-6](https://doi.org/10.1038/s41597-023-02244-6)
