# DataStack Documentation — References

All URLs, repos, deployments, datasets, and publications found during research.

---

## Core Repos

| Repo | Role |
|---|---|
| https://github.com/Mat-O-Lab/DataStack | Docker Compose stack orchestration |
| https://github.com/Mat-O-Lab/CSVToCSVW | Microservice: CSV → CSVW JSON-LD |
| https://github.com/Mat-O-Lab/MapToMethod | Microservice: JSON-LD + template → YARRRML |
| https://github.com/Mat-O-Lab/RDFConverter | Microservice: YARRRML + data → RDF knowledge graph |
| https://github.com/Mat-O-Lab/ckanext-csvtocsvw | CKAN extension: auto-annotate CSV uploads |
| https://github.com/Mat-O-Lab/ckanext-csvwmapandtransform | CKAN extension: auto-discover and apply mappings |
| https://github.com/Mat-O-Lab/ckanext-fuseki | CKAN extension: Fuseki triplestore integration |
| https://github.com/Mat-O-Lab/OpenBISmantic | Extractor: OpenBIS ELN → JSON-LD / RO-Crate |
| https://github.com/Mat-O-Lab/OmeroExtractor | Extractor: OMERO microscopy server → JSON-LD |

---

## Pattern Authoring & Ontologies

| Repo / URL | Role |
|---|---|
| https://github.com/ThHanke/ontosphere | OntosphereIO — browser app for pattern authoring (AI + OWL2DL + SHACL); **recommended tool** |
| https://github.com/materialdigital/core-ontology | PMDCO (Platform MaterialDigital Core Ontology) |
| https://github.com/materialdigital/core-ontology/tree/main/patterns/ | PMDCO reference patterns library |
| https://github.com/materialdigital/core-ontology/tree/main/patterns/chemical%20composition | Example: chemical composition pattern |

---

## External Tools Referenced

| Repo / URL | Role |
|---|---|
| https://github.com/ontop/ontop | SQL → RDF bridge (OBDA/R2RML); potential Stage 1 extractor for SQL sources |
| https://rml.io/yarrrml/tutorial/ | YARRRML official tutorial |
| https://rml.io/yarrrml/matey/ | Matey — online YARRRML editor |
| https://rhizomik.net/redefer/xsd2owl | xsd2owl — used by OmeroExtractor to generate ome.ttl from ome.xsd |
| https://github.com/eclipse-tractusx/sldt-semantic-models | Catena-X / SAMM aspect model definitions (Turtle ontologies) |

---

## Industry Standards

| Standard | Notes |
|---|---|
| SAMM (Semantic Aspect Meta-Model) | Catena-X standard for automotive supply-chain data; payloads are flat JSON conforming to SAMM aspect model schemas |
| IDTA / AAS (Asset Administration Shell) | IEC 63278 / IEC 61360; hierarchical JSON submodel format for Industry 4.0 digital twins |
| W3C CSVW | CSV on the Web — metadata standard produced by CSVToCSVW |
| RML / YARRRML | W3C RDF Mapping Language and its human-readable dialect |
| QUDT | Quantities, Units, Dimensions, and Types ontology |
| PROV-O | W3C Provenance Ontology |
| BFO / OBO | Basic Formal Ontology / Open Biological Ontologies foundational relations |
| OBI | Ontology for Biomedical Investigations (scalar value specifications) |
| OME (Open Microscopy Environment) | Microscopy domain ontology; `ome.ttl` generated from official OME XSD |

---

## Public Microservice Deployments

| Service | Base URL | API Docs | OpenAPI JSON |
|---|---|---|---|
| CSVToCSVW | https://csvtocsvw.matolab.org | https://csvtocsvw.matolab.org/api/docs | https://csvtocsvw.matolab.org/api/openapi.json |
| MapToMethod | https://maptomethod.matolab.org | https://maptomethod.matolab.org/api/docs | https://maptomethod.matolab.org/api/openapi.json |
| RDFConverter | https://rdfconverter.matolab.org | https://rdfconverter.matolab.org/api/docs | https://rdfconverter.matolab.org/api/openapi.json |
| OmeroExtractor | https://metadata.omero.matolab.org | https://metadata.omero.matolab.org/api/docs | https://metadata.omero.matolab.org/api/openapi.json |
| OpenBISmantic | openbis.matolab.org | — | — (unreachable at time of research) |

---

## Container Images

| Image | Service |
|---|---|
| `ghcr.io/mat-o-lab/csvtocsvw:latest` | CSVToCSVW |
| `ghcr.io/mat-o-lab/maptomethod:latest` | MapToMethod |
| `ghcr.io/mat-o-lab/rdfconverter:latest` | RDFConverter |
| `ghcr.io/mat-o-lab/yarrrml-parser` | YARRRML Parser (Node.js, used by RDFConverter) |
| `ghcr.io/mat-o-lab/rmlmapper-webapi` | RML Mapper (Java, used by RDFConverter) |
| `ghcr.io/mat-o-lab/omeroextractor:latest` | OmeroExtractor |
| `ghcr.io/mat-o-lab/openbismantic` | OpenBISmantic |
| `secoresearch/fuseki:4.9.0` | Apache Jena Fuseki (used in DataStack) |

---

## Live CKAN Instances

| URL | Name |
|---|---|
| https://futurecarproduction.materialsdata.space/ | Future Car Production Materials Data Space |
| https://dataportal.material-digital.de/ | Material Digital Data Portal |

### CKAN API entry points

```
GET https://futurecarproduction.materialsdata.space/api/3/action/package_list
GET https://futurecarproduction.materialsdata.space/api/3/action/package_search?q=samm&rows=20
GET https://futurecarproduction.materialsdata.space/api/3/action/package_show?id=<slug>
GET https://dataportal.material-digital.de/api/3/action/package_list
```

---

## Notable Datasets

### futurecarproduction.materialsdata.space

| Dataset | URL | Notes |
|---|---|---|
| SAMM mapping — material data legacy | https://futurecarproduction.materialsdata.space/dataset/samm-mapping-material-data-legacy | Reference SAMM mapping dataset; contains YARRRML + 2 SPARQL CONSTRUCT files |
| EDCar JSON payloads | https://futurecarproduction.materialsdata.space/dataset/edcar_json_payloads | 29 Catena-X JSON payloads + 30 pre-computed joined TTLs + pmdco-full.owl + live Fuseki SPARQL endpoint |
| SAMM mapping datasets (26 total) | pattern: `samm-mapping-*` | Cover full automotive materials lifecycle: material, composition, batch, sustainability, recycling, vehicle info, component, simulation, homologation, dismantling, circular economy, feedback-to-design |

### dataportal.material-digital.de

| Dataset | URL | Notes |
|---|---|---|
| Cross-project use case | https://dataportal.material-digital.de/dataset/cross-project-use-case | Full PA6GF30 / Catena-X → PMDco pipeline with Jupyter notebook |
| SPARQL INSERT helper | https://dataportal.material-digital.de/dataset/cross-project-use-case/resource/32e3bc55-a022-4edd-b5bc-7402c8ff5991 | `pmdco-mapping-insert.html` — browser page that POSTs SPARQL INSERT to Fuseki update endpoint |

---

## Real-World Example Projects

| Repo | Notes |
|---|---|
| https://github.com/BAMresearch/DF-TEM-PAW | Dark-field TEM microscopy data from BAM; demonstrates OmeroExtractor → Mat-O-Lab pipeline |

---

## Publications

| Citation | URL |
|---|---|
| Hanke et al. (2023) — FAIR microscopy data via Mat-O-Lab pipeline, Integrating Materials and Manufacturing Innovation | https://link.springer.com/article/10.1007/s40192-023-00331-5 |

---

## Outdated / Superseded

| URL | Status | Why outdated |
|---|---|---|
| https://github.com/Mat-O-Lab/IOFMaterialsTutorial | Outdated | Uses draw.io for "graph prototypes" (now: OntosphereIO + "patterns"); pipeline without CKAN; old IOF ontology versions. Useful for conceptual understanding only — do not recommend tooling. |
