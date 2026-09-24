---
title: Ontop (SQL Bridge)
---

# Ontop (SQL Bridge)

!!! info "External tool — not maintained by Mat-O-Lab"
    Ontop is developed and maintained by the Ontop community. This page is
    informational only. See the [Ontop official documentation](https://ontop-vkg.org/)
    for setup, configuration, and deployment guides.

## What Ontop does

[Ontop](https://ontop-vkg.org/) is an Ontology-Based Data Access (OBDA) bridge
that exposes a relational database as a live SPARQL endpoint without any ETL or
data movement. You write an [R2RML](https://www.w3.org/TR/r2rml/) (or Ontop
OBDA) mapping that describes how database columns map to RDF predicates; Ontop
translates each incoming SPARQL query into SQL at runtime and returns a virtual
RDF graph. The data stays in the database — nothing is materialized unless you
explicitly request it.

## How it could connect to the DataStack

An Ontop SPARQL endpoint could act as a Stage 1 data source for a Mat-O-Lab
pipeline: a `CONSTRUCT` query issued against the endpoint would return a
JSON-LD or Turtle document that a downstream MapToMethod mapping could consume,
following the same path as any other RDF resource uploaded to CKAN. No
automated CKAN extension or ckanext-csvwmapandtransform connector exists for
this path today.

## Further reading

- [Ontop official documentation](https://ontop-vkg.org/)
- [W3C R2RML Recommendation](https://www.w3.org/TR/r2rml/)

---

No Mat-O-Lab integration has been built yet — contributions welcome.
