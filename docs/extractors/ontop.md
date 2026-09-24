---
title: Ontop
---

# Ontop

[Ontop](https://ontop-vkg.org/) is an external SPARQL-to-SQL bridge that exposes relational database contents as a SPARQL endpoint using OBDA (Ontology-Based Data Access) and R2RML mapping rules. It is a potential Stage 1 extractor for SQL database sources in the Mat-O-Lab pipeline.

**Upstream:** https://github.com/ontop/ontop  
**Status:** External tool — no Mat-O-Lab integration built yet

---

## What it does

Ontop sits in front of a relational database and translates SPARQL queries into SQL at runtime. Data stays in the database — Ontop presents a virtual RDF view defined by an R2RML mapping file.

```
SQL database (PostgreSQL, MySQL, SQLite, …)
  + R2RML or OBDA mapping file
  → Ontop SPARQL endpoint
  → Virtual RDF graph (queryable, not materialized)
```

This differs from the other extractors: Ontop does not produce a static JSON-LD or Turtle file to feed downstream. Instead it presents a live SPARQL endpoint. Integration with MapToMethod and RDFConverter would require materializing the output (e.g. via `CONSTRUCT` query).

---

## Integration path (potential)

A rough integration approach for the Mat-O-Lab pipeline:

1. Deploy Ontop against your SQL database with an R2RML mapping
2. Use Ontop's SPARQL endpoint to run a `CONSTRUCT` query that exports the relevant data as Turtle
3. Upload the Turtle to CKAN as a resource
4. ckanext-csvwmapandtransform picks it up and applies a YARRRML mapping via RDFConverter

No automated CKAN extension exists for this path — it requires a manual or scripted export step.

---

## Example — R2RML mapping (illustrative)

```turtle
@prefix rr: <http://www.w3.org/ns/r2rml#> .
@prefix ex: <https://example.org/> .

<#TriplesMap1>
    rr:logicalTable [ rr:tableName "measurements" ] ;
    rr:subjectMap [
        rr:template "https://example.org/measurement/{id}" ;
        rr:class ex:Measurement
    ] ;
    rr:predicateObjectMap [
        rr:predicate ex:hasTensileStrength ;
        rr:objectMap [ rr:column "tensile_strength_mpa" ; rr:datatype xsd:double ]
    ] .
```

See [R2RML W3C Recommendation](https://www.w3.org/TR/r2rml/) for full mapping syntax, and the [Ontop tutorial](https://ontop-vkg.org/tutorial/) for deployment.

---

## When to use Ontop vs a CSV export

| Situation | Approach |
|---|---|
| Data changes frequently; materialization is expensive | Ontop virtual endpoint — query on demand |
| One-off export or small dataset | Export to CSV → CSVToCSVW |
| Pipeline needs a static Turtle resource in CKAN | Export from Ontop via CONSTRUCT → upload to CKAN |

---

## Limitations

- No Mat-O-Lab CKAN extension — requires manual or scripted integration
- Virtual endpoint; downstream tools (MapToMethod, RDFConverter) need a static URL to a file, not a SPARQL endpoint
- R2RML/OBDA mapping authoring has a steeper learning curve than YARRRML
- Ontop is SPARQL-over-SQL only — no support for NoSQL or document stores
