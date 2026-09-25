---
title: Author a Prototype Graph
---

# Author a Prototype Graph

Before you can map a CSV to a shared vocabulary, you need a **prototype graph** — a small Turtle file that defines the semantic structure for your measurement type: what entities exist, their types, and how they relate.

This page shows how to create one using [Ontosphere](https://thhanke.github.io/ontosphere), a browser-based RDF/OWL 2 DL editor. No installation required.

If a prototype graph for your measurement type already exists in your community's ontology library (e.g. the [PMDCO pattern library](https://github.com/materialdigital/core-ontology/tree/main/patterns/)), you can skip directly to [Author a Mapping](author-a-mapping.md).

---

## What is a prototype graph?

A prototype graph is a reusable Turtle file that encodes one semantic concept — for example, "a tensile test result has a yield strength quality inhering in a specimen, measured as a scalar value in MPa."

In the mapping workflow it serves as the **template**: MapToMethod wires your CSV columns into the named slots the prototype graph defines. Every row in your CSV produces a new set of connected entities shaped like the prototype.

A minimal prototype graph for a length measurement looks like this:

```turtle
@prefix owl:  <http://www.w3.org/2002/07/owl#> .
@prefix iof:  <https://spec.industrialontologies.org/ontology/core/Core/> .
@prefix qual: <https://spec.industrialontologies.org/ontology/qualities/> .
@prefix qudt: <http://qudt.org/schema/qudt/> .
@prefix unit: <http://qudt.org/vocab/unit/> .

<LengthMeasurement>  a owl:Ontology .

:Specimen            a owl:NamedIndividual, iof:MaterialArtifact .
:SpecimenLength      a owl:NamedIndividual, qual:Length .
:MeasurementProcess  a owl:NamedIndividual, iof:MeasurementProcess .
:LengthData          a owl:NamedIndividual, qudt:QuantityValue ;
    qudt:unit unit:MilliM .
```

The `owl:NamedIndividual` entries are the **named slots** that MapToMethod links your CSV columns to. The mapping you author later defines the rules between these slots and the resource metadata (CSVW).

---

## Ontosphere

[Ontosphere](https://thhanke.github.io/ontosphere) is a zero-install, browser-based RDF/OWL 2 DL editor. It combines a visual graph canvas (Reactodia), an in-browser triple store (N3.js), and a full OWL 2 DL reasoner compiled to WebAssembly (Konclude). No backend, no account.

### Overview video

<iframe width="560" height="315" src="https://www.youtube.com/embed/85b4WqGQMkE?si=QKMicE7TwuO4DxcK" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For a 3-minute walkthrough of all features:
[iswc2026-comprehensive.mp4](https://thhanke.github.io/ontosphere/demo-videos/iswc2026-comprehensive.mp4)

---

## Authoring workflow

The standard prototype graph authoring sequence:

```
Load base ontology → Add nodes → Add links → Run layout →
Run reasoning → Export graph (TTL)
```

### 1 — Load your base ontology

Open [Ontosphere](https://thhanke.github.io/ontosphere). In the **Load** panel, enter the URL of your target ontology (e.g. PMDCO, MTO, EMMO) or upload a local TTL file. Ontosphere loads the class hierarchy and property definitions from the ontology — these become available as types when you create nodes.

<video controls width="100%">
  <source src="https://thhanke.github.io/ontosphere/demo-videos/feat-loading.mp4" type="video/mp4">
</video>

You can also load from a SPARQL endpoint if your ontology is published there.

### 2 — Build the graph: add nodes and links

Switch to the **Authoring** tab. For each entity in your prototype graph:

1. **Add a node** — drag a class from the ontology panel onto the canvas, or use **New Node** and set its type IRI manually
2. **Name it** — give it a local name (e.g. `Specimen`, `YieldStrengthDatum`) — this becomes the named individual IRI slot MapToMethod links to
3. **Draw edges** — drag from one node's connection port to another; set the property IRI (e.g. `obo:RO_0000080` — *inheres in*)

<video controls width="100%">
  <source src="https://thhanke.github.io/ontosphere/demo-videos/feat-authoring.mp4" type="video/mp4">
</video>

Use **Run Layout** (Dagre or ELK) after adding nodes to automatically arrange the graph readably.

### 3 — Explore and verify structure

Use the **TBox / ABox toggle** to switch between the ontology class hierarchy (TBox) and your instance graph (ABox). The search bar finds any node by IRI or label. Use the minimap for large graphs.

<video controls width="100%">
  <source src="https://thhanke.github.io/ontosphere/demo-videos/feat-exploration.mp4" type="video/mp4">
</video>

### 4 — Run OWL 2 DL reasoning

Click **Run Reasoning**. Konclude checks your prototype graph for OWL 2 DL consistency and adds any entailed relationships as **amber dashed edges** — these are inferred triples not present in your source, derived from the ontology axioms. If a node is unsatisfiable (contradictory classification), the reasoner flags it here.

<video controls width="100%">
  <source src="https://thhanke.github.io/ontosphere/demo-videos/feat-reasoning.mp4" type="video/mp4">
</video>

Fix any unsatisfiabilities before exporting — an inconsistent prototype graph will produce incorrect RDF downstream.

### 5 — SHACL validation (optional)

If your community provides SHACL shapes (e.g. PMDCO ships shapes for its core prototype graphs), load them in the **Validation** panel. Ontosphere runs the shapes against your graph and highlights which nodes fail which constraints, with repair suggestions.

<video controls width="100%">
  <source src="https://thhanke.github.io/ontosphere/demo-videos/feat-shacl.mp4" type="video/mp4">
</video>

### 6 — Export the TTL

**Export Graph** → **Turtle** (`.ttl`). The export uses W3C RDFC-1.0 canonicalization — the output is deterministic and diff-friendly.

Upload the exported TTL to a publicly accessible URL (GitHub, public S3, institution web server). MapToMethod fetches the prototype graph by URL — it must be reachable over HTTP.

---

## Materials science benchmark tasks

The [OntoAuthor-Mat benchmark](https://github.com/ThHanke/ontosphere/tree/main/benchmarks/ontoauthor-mat/) provides six materials science prototype graph tasks with reference solutions, SHACL shapes, and SPARQL competency questions. Use them to learn the core OWL 2 DL structures before authoring your own:

| Task | OWL construct | Scenario |
|------|--------------|----------|
| [T1](https://github.com/ThHanke/ontosphere/tree/main/benchmarks/ontoauthor-mat/T1) | `rdfs:subClassOf` | Steel alloy classification hierarchy |
| [T2](https://github.com/ThHanke/ontosphere/tree/main/benchmarks/ontoauthor-mat/T2) | `owl:someValuesFrom` | Composite materials and their constituents |
| [T3](https://github.com/ThHanke/ontosphere/tree/main/benchmarks/ontoauthor-mat/T3) | `owl:allValuesFrom` | Certified material supplier constraints |
| [T4](https://github.com/ThHanke/ontosphere/tree/main/benchmarks/ontoauthor-mat/T4) | `owl:disjointWith` | Metallic vs. ceramic material categories |
| [T5](https://github.com/ThHanke/ontosphere/tree/main/benchmarks/ontoauthor-mat/T5) | `owl:sameAs` | Consolidating duplicate material entries |
| [T6](https://github.com/ThHanke/ontosphere/tree/main/benchmarks/ontoauthor-mat/T6) | Unsatisfiability | Contradictory classification detection |

Each task directory contains a natural-language brief, the reference OWL 2 DL solution, SHACL shapes (`shapes.ttl`), and SPARQL competency questions. Work through T1 and T2 first — they cover the two structures that appear in most materials science measurement prototype graphs.

---

## What the exported TTL must contain

For MapToMethod to use your prototype graph, the exported TTL must include:

| Requirement | Why |
|---|---|
| At least one `owl:NamedIndividual` | These are the link targets — MapToMethod binds CSV columns to named individual IRIs |
| A `qudt:unit` annotation on quantity nodes | CSVToCSVW's unit detection aligns with QUDT; the prototype graph must agree |
| Publicly accessible URL | MapToMethod fetches the TTL over HTTP — local or intranet files are not reachable |

The named individual IRI is the **slot key**: in the mapping step you will pair your CSV column name to this IRI.

---

## Next step

Once your prototype graph TTL is at a public URL:

→ [Author a Mapping](author-a-mapping.md) — define the rules between your resource metadata (CSVW) and the prototype graph's named individual slots using MapToMethod.
