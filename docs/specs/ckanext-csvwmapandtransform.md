# ckanext-csvwmapandtransform — Component Specification

**Repo:** https://github.com/Mat-O-Lab/ckanext-csvwmapandtransform  
**Type:** CKAN extension  
**Depends on:** RDFConverter microservice, CKAN group named `mappings`

---

## Purpose

After an RDF/JSON-LD resource is created or updated in CKAN, automatically discovers YAML mapping files stored in a CKAN group named `mappings`, tests each against the resource, selects the best-fit mapping, and calls RDFConverter to produce a joined Turtle RDF knowledge graph — which is then added back to the same CKAN dataset.

---

## CKAN Integration Points

Implements `IResourceController` and `IResourceUrlChange`:

| Hook | Trigger |
|---|---|
| `after_resource_create` | New resource created |
| `after_create` | Dataset/resource created |
| `after_update` | Resource updated |
| `notify` | Resource URL changed |

All route to the same `_submit_transform` handler.

**Trigger conditions (both must hold):**
1. resource `format` (lowercased) in `ckanext.csvwmapandtransform.formats`
2. resource URL does **not** contain `-joined` (prevents recursive re-processing of already-transformed outputs)

---

## Mapping Discovery

Action: `csvwmapandtransform_find_mappings`

- CKAN `package_search` with `fq=groups:mappings` — finds all packages in the `mappings` group
- Collects every resource with `format == YAML` from those packages
- Returns list of YAML URLs as mapping candidates

**Prerequisite:** A CKAN group named exactly `mappings` must exist. If missing, a warning is logged and no transformations occur.

---

## Mapping Selection

For each YAML mapping URL:
1. `POST {rdfconverter_url}/api/checkmapping` with `{mapping_url, data_url}`
2. Receives `{rules_applicable, rules_skipped}`
3. Computes `rating = rules_applicable - rules_skipped`

Selection strategy (`ckanext.csvwmapandtransform.mapping_strategy`):

| Strategy | Condition to select |
|---|---|
| `exact` (default) | `rules_skipped == 0` AND `rules_applicable > 0` |
| `best_match` | `rating > 0` (best-rated candidate wins) |

---

## RDF Conversion

Calls RDFConverter `POST /api/createrdf?return_type=turtle` with `{mapping_url, data_url}`.

The Turtle result is uploaded as a new resource to the same CKAN dataset. Filename contains `-joined` (prevents re-trigger).

---

## Registered CKAN Actions

| Action | Purpose |
|---|---|
| `csvwmapandtransform_find_mappings` | Discover mappings from `mappings` group |
| `csvwmapandtransform_transform` | Trigger transformation job |
| `csvwmapandtransform_test_mappings` | Test all mappings against a resource |
| `csvwmapandtransform_transform_status` | Check job status |
| `csvwmapandtransform_hook` | Job status callback |

---

## Configuration

| Key | Default | Description |
|---|---|---|
| `ckanext.csvwmapandtransform.maptomethod_url` | `https://maptomethod.matolab.org` | Public URL (used in browser iframes for mapping authoring) |
| `ckanext.csvwmapandtransform.rdfconverter_url` | `https://rdfconverter.matolab.org` | RDFConverter URL (use internal container URL in DataStack) |
| `ckanext.csvwmapandtransform.formats` | `json json-ld turtle n3 nt hext trig longturtle xml ld+json` | Formats that trigger transformation |
| `ckanext.csvwmapandtransform.mapping_strategy` | `exact` | `exact` or `best_match` |
| `ckanext.csvwmapandtransform.ckan_token` | `""` | CKAN API token for background jobs |
| `ckanext.csvwmapandtransform.rdfconverter_timeout_check` | `30` | Timeout (s) for check calls |
| `ckanext.csvwmapandtransform.rdfconverter_timeout_create` | `120` | Timeout (s) for create calls |
| `ckanext.csvwmapandtransform.ssl_verify` | `True` | SSL verification |

In DataStack, configure via:
```
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_URL=http://rdfconverter:5000
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPTOMETHOD_URL=https://maptomethod.matolab.org
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPPING_STRATEGY=exact
```

---

## Mapping Strategy: `exact` vs `best_match`

| | `exact` | `best_match` |
|---|---|---|
| When to use | Production — only fully-matching mappings apply | Development / exploration — best partial match applies |
| Risk | May produce no output if no exact mapping exists | May apply an imperfect mapping, producing partial graphs |
| Recommended | Default — safer for data quality | Use when iterating on new mapping designs |

---

## Capabilities

- Fully automatic — no user action needed after CSV upload (if mappings exist)
- Scales to many mappings — checks all in the `mappings` group
- Two selection strategies for different accuracy/coverage tradeoffs
- `-joined` suffix prevents recursive re-processing loops
- Timeout configuration for large datasets

## Limitations

- Requires a `mappings` CKAN group with YAML resources — no mappings = no transformation
- Data URL must be publicly accessible from the CKAN server
- `exact` strategy may silently produce no output if no mapping fully matches
- RDFConverter timeouts may need tuning for large files (default: 120s)
- `BACKGROUNDJOBS_API_TOKEN` must be set or jobs will not execute
