# ckanext-fuseki — Component Specification

**Repo:** https://github.com/Mat-O-Lab/ckanext-fuseki  
**Type:** CKAN extension  
**Depends on:** Apache Jena Fuseki container

---

## Purpose

Integrates Apache Jena Fuseki as a triple store mirror of CKAN datasets. For each RDF resource in a dataset, creates a Fuseki dataset (named by CKAN dataset UUID), uploads resources as named graphs, and creates a `SPARQL` resource link in CKAN pointing to a CKAN-proxied SPARQL endpoint.

---

## CKAN Integration Points

Implements `IResourceController` (with `inherit=True`).

**Current state:** The automatic `after_create`/`after_update` hooks are **commented out** in `plugin.py`. Fuseki sync is triggered **manually** via the `fuseki_update` CKAN action — accessible from the dataset UI.

**Trigger condition:** resource `format` (lowercased) must be in `ckanext.fuseki.formats`.

---

## Dataset UUID Matching

Fuseki dataset name = CKAN package (dataset) UUID directly. No secondary mapping table. A Fuseki dataset named after the CKAN UUID is created on first sync and reused on updates.

---

## Named Graph URIs

Each resource's download URL is used as its named graph URI.

With reasoning enabled, two named graphs exist per resource:
- Raw graph: `{resource_url}`
- Inference graph: `urn:x-arq:InferenceGraph:{resource_url}`

---

## Upload Flow (`fuseki_update` action)

1. Delete existing Fuseki dataset for the package UUID (if present)
2. Poll for confirmed deletion (`verify_dataset_deleted`, up to 10 retries)
3. Re-create dataset via Fuseki Assembly API (Turtle assembler POSTed to `$/datasets`)
   - Configures TDB2 (persistent or in-memory)
   - Sets `unionDefaultGraph=true`
   - Optional reasoning: OWL/RDFS variants (`Reasoners` enum)
4. Upload each RDF resource via Graph Store Protocol `PUT` to `{dataset_url}/data?graph={named_graph_uri}`
   - `PUT` replaces content — prevents accumulation of stale triples
5. Clean up orphaned named graphs (for removed resources)
6. Create or patch a CKAN resource named `SPARQL` with URL pointing to the CKAN proxy endpoint

---

## SPARQL Link Format

Controlled by `ckanext.fuseki.sparklis_url`:

| Setting | SPARQL link format |
|---|---|
| `sparklis_url` set | `{sparklis_url}?title={dataset_name}&endpoint={sparql_proxy_url}` |
| `sparklis_url` empty | Built-in YASGUI page at `/dataset/{id}/fuseki/sparql` |

The SPARQL proxy endpoint: `/dataset/{id}/fuseki/$/sparql` — CKAN authenticates before forwarding to Fuseki. Users never hit Fuseki directly.

---

## Registered CKAN Actions

| Action | Purpose |
|---|---|
| `fuseki_update` | Upload/sync all RDF resources of a dataset to Fuseki |
| `fuseki_delete` | Delete the Fuseki dataset for a CKAN dataset |

---

## Configuration

| Key | Default | Description |
|---|---|---|
| `ckanext.fuseki.url` | `http://fuseki:3030/` | Fuseki base URL (internal container) |
| `ckanext.fuseki.internal_url` | `""` | Optional direct URL bypassing nginx |
| `ckanext.fuseki.username` | `admin` | Fuseki admin username |
| `ckanext.fuseki.password` | `admin` | Fuseki admin password |
| `ckanext.fuseki.ckan_token` | `""` | CKAN API token for background jobs |
| `ckanext.fuseki.formats` | `turtle text/turtle n3 nt hext trig longturtle xml json-ld ld+json jsonld` | Formats that trigger upload |
| `ckanext.fuseki.sparklis_url` | `""` | Sparklis UI URL (empty = YASGUI) |
| `ckanext.fuseki.ssl_verify` | `True` | SSL verification |

In DataStack, configure via:
```
CKANINI__CKANEXT__FUSEKI__URL=http://fuseki:3030/
CKANINI__CKANEXT__FUSEKI__USERNAME=admin
CKANINI__CKANEXT__FUSEKI__PASSWORD=<password>
CKANINI__CKANEXT__FUSEKI__SPARKLIS_URL=http://sparklis:8080/
```

---

## Capabilities

- CKAN UUID → Fuseki dataset: stable, predictable naming, no mapping table needed
- Named graph per resource: tracks provenance by resource URL
- Union default graph: cross-resource SPARQL queries work out of the box
- Optional OWL/RDFS reasoning: inference graphs created automatically
- CKAN proxy: users access SPARQL without direct Fuseki credentials
- Sparklis integration: user-friendly faceted SPARQL UI as an alternative to YASGUI
- Stale-triple prevention via `PUT` (replace, not append)

## Limitations

- Auto-hooks are currently **commented out** — Fuseki sync requires manual trigger via CKAN UI or API; it does not happen automatically on resource upload
- Delete-and-recreate strategy means Fuseki is briefly empty during large dataset re-syncs
- Resource URLs must be publicly accessible for Fuseki to fetch the RDF content
- Reasoning adds inference triples — can significantly increase storage for large ontologies
- `BACKGROUNDJOBS_API_TOKEN` must be set or jobs will not execute
