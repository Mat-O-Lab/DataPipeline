---
title: Ontop (SQL Bridge)
---

# Ontop (SQL Bridge)

!!! info "External tool — not maintained by Mat-O-Lab"
    Ontop is developed and maintained independently by the Ontop community.
    This page is informational only. For setup, configuration, and deployment
    see the [Ontop official documentation](https://ontop-vkg.org/).

## What problem it solves

If your lab stores data in a relational database — an ELN, a LIMS, or any
system that keeps records in tables and rows — you already have structured,
queryable data. The challenge is that those tables use your institution's
column names and internal identifiers, not the shared vocabulary of your
scientific community.

Ontop bridges that gap. It connects directly to your existing database and
makes its contents queryable using the shared scientific terms your community
has agreed on, **without copying or moving the data**.

## The "virtual knowledge graph" idea

Think of it as a live, read-only window onto your database: the underlying
tables stay exactly where they are, but the view through that window labels
every column using community vocabulary — the same terms another lab,
another instrument, or another tool in the pipeline would recognise.

You define a mapping that says, for example, "the column `sample_id` in
the `measurements` table corresponds to the community concept for a
*specimen identifier*." After that, anyone — or any software — querying
through Ontop sees your data expressed in those shared terms, even though
the database itself has not changed.

This approach is called **Ontology-Based Data Access (OBDA)**. Nothing is
materialised or exported unless you explicitly request it; the translation
happens on the fly for every query.

## What the mapping looks like

The mapping is a configuration file (written in a format called
[R2RML](https://www.w3.org/TR/r2rml/), or Ontop's own OBDA format) that
pairs each table and column in your database with its counterpart in the
shared vocabulary. Writing that mapping requires knowing both your database
schema and the community ontology you are mapping to — it is typically a
joint effort between a domain expert and someone familiar with the
ontology.

The Ontop documentation covers mapping authoring in detail.

## How Ontop could connect to the DataStack

Once Ontop is running, it exposes a query endpoint. A downstream Mat-O-Lab
pipeline could issue a query against that endpoint, receive the result as
structured data, and feed it into the normal mapping and transformation
steps — the same path any other structured data file takes through CKAN.

**No automated connector between Ontop and Mat-O-Lab has been built yet.**
This integration path is technically feasible but not supported out of the
box. Contributions are welcome.

## Is Ontop right for your setup?

Ontop is a good fit if:

- Your data already lives in a relational database (not flat files).
- You want to avoid exporting or duplicating that data.
- You have — or can find — a collaborator who knows the relevant ontology for
  your domain.

If your data starts as CSV or spreadsheet files, the
[CSV-to-CSVW extractor](csvtocsvw.md) is a simpler starting point.

## Further reading

- [Ontop official documentation](https://ontop-vkg.org/)
- [W3C R2RML Recommendation](https://www.w3.org/TR/r2rml/)
