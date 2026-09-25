---
title: Mat-O-Lab DataStack
---

# Mat-O-Lab DataStack

**DataStack turns your research data files into FAIR-compliant, publishable datasets — without writing any code.**

Upload a CSV, a microscopy export, or a structured data file. The pipeline enriches your data with structured metadata, links it to shared scientific vocabularies, and publishes it to a data portal where it is searchable, downloadable, and citable.

> Create complete, consistent metadata for a non-semantic resource — then use semantic technologies to transform that further.

The result: data that meets FAIR principles (Findable, Accessible, Interoperable, Reusable) by design, not as an afterthought.

---

## Where do you want to start?

=== "I'm a researcher or data manager"
    **→ [What the pipeline does](pipeline/index.md)**

    Learn what happens to your data, what outputs you receive, and how to interpret the results. No technical background needed.

    For a deeper look at how semantic enrichment works, see [Semantic Foundation](pipeline/semantic-foundation.md).

=== "I need to deploy and run this"
    **→ [Quickstart](guides/quickstart.md)**

    Get the stack running with Docker Compose. Configure data sources and connect your storage backend.

=== "I'm connecting data sources or writing mappings"
    **→ [Author a Mapping](guides/author-a-mapping.md)**

    Learn how to describe your data structure so the pipeline can enrich it automatically.

=== "I want to use the tools standalone"
    **→ [Standalone APIs](guides/standalone-apis.md)**

    Run individual pipeline components without deploying the full stack.

---

## Three domains — one pipeline

The same pipeline handles different scientific and industrial data formats:

| Domain | Source | What you get | Live example |
|---|---|---|---|
| Lab / materials science | CSV measurement file (tensile test, spectroscopy) | A structured, standards-aligned metadata record — searchable and citable | [IOFMaterialsTutorial](https://github.com/Mat-O-Lab/IOFMaterialsTutorial) |
| Microscopy imaging | OMERO image archive | Image metadata linked to instrument, acquisition parameters, and sample context | [BAMresearch DF-TEM-PAW](https://github.com/BAMresearch/DF-TEM-PAW) |
| Automotive supply chain | SAMM / Catena-X product data | Machine-readable records linked to shared automotive industry vocabularies | [futurecarproduction.materialsdata.space](https://futurecarproduction.materialsdata.space) |

---

## In production today

DataStack runs at public data portals:

- **[futurecarproduction.materialsdata.space](https://futurecarproduction.materialsdata.space/)** — 26 Catena-X datasets published with automatically generated semantic metadata · search: [SAMM](https://futurecarproduction.materialsdata.space/dataset?tags=SAMM) · [AAS](https://futurecarproduction.materialsdata.space/dataset?tags=Asset+Administration+Shell) · [microscopy](https://futurecarproduction.materialsdata.space/dataset?tags=Mikroskopie)
- **[dataportal.material-digital.de](https://dataportal.material-digital.de/)** — Cross-project materials data, including PA6GF30 / Catena-X use cases · search: [tensile tests](https://dataportal.material-digital.de/dataset?q=tensile+tests) · [Vickers](https://dataportal.material-digital.de/dataset?q=Vickers) · [Creep](https://dataportal.material-digital.de/dataset?q=Creep) · [knowledge graph](https://dataportal.material-digital.de/dataset?q=knowledge-graph)

The pipeline approach is documented in peer-reviewed publications:

> Nasrabadi, Hanke et al. (2023). *Toward a digital materials mechanical testing lab.*
> Computers in Industry, 153, 104016. [doi:10.1016/j.compind.2023.104016](https://doi.org/10.1016/j.compind.2023.104016)

> Hanke et al. (2023). *FAIR microscopy data via the Mat-O-Lab pipeline.*
> Scientific Data (Nature). [doi:10.1038/s41597-023-02244-6](https://doi.org/10.1038/s41597-023-02244-6)

---

## Ready to explore?

- **For researchers and data managers:** [Read how the pipeline works →](pipeline/index.md)
- **For data engineers and operators:** [Follow the quickstart →](guides/quickstart.md)
