---
title: DataStack
---

# DataStack

Docker Compose orchestration that deploys the full Mat-O-Lab PMD-CKAN platform as a containerized stack.

**Repo:** https://github.com/Mat-O-Lab/DataStack

## Service Topology

| Service | Image | Role |
|---|---|---|
| `nginx` | nginx:1.27-alpine | Reverse proxy / TLS termination |
| `ckan` | custom build | CKAN 2.10/2.11 data portal |
| `db` | custom PostgreSQL | `ckandb` + `datastore` databases |
| `solr` | ckan/ckan-solr | Full-text search index |
| `redis` | redis:7-alpine | RQ job queue + session cache |
| `fuseki` | secoresearch/fuseki:4.9.0 | Apache Jena Fuseki triple store |
| `csvtocsvw` | ghcr.io/mat-o-lab/csvtocsvw | CSV → CSVW metadata (FastAPI) |
| `maptomethod` | ghcr.io/mat-o-lab/maptomethod | YARRRML mapping editor (FastAPI) |
| `yarrrml-parser` | ghcr.io/mat-o-lab/yarrrml-parser | YARRRML → RML conversion (Node.js) |
| `rmlmapper` | ghcr.io/mat-o-lab/rmlmapper-webapi | RML execution engine (Java, port 4000) |
| `rdfconverter` | ghcr.io/mat-o-lab/rdfconverter | Orchestrates yarrrml-parser + rmlmapper (FastAPI) |
| `sparklis` | sferre/sparklis | User-friendly SPARQL query UI |

All services share the internal bridge network `datastack_net`.  
Persistent volumes: `ckan_storage`, `pg_data`, `solr_data`, `jena_data`.

## Configuration Convention

Every `.env` variable prefixed `CKANINI__` is auto-translated to `ckan.ini` at container start:

- prefix stripped
- `__` → `.`
- lowercased

Example: `CKANINI__CKANEXT__FUSEKI__URL` → `ckanext.fuseki.url`

## Key Environment Variables

| Variable | Purpose |
|---|---|
| `CKAN__PLUGINS` | Space-separated plugin list |
| `BACKGROUNDJOBS_API_TOKEN` | CKAN API token for background-job extensions; created manually after first boot |
| `FUSEKI_JAVA_OPTS` | JVM heap (default: `-Xmx10g -Xms10g`) |
| `CKANINI__CKANEXT__CSVTOCSVW__CSVTOCSVW_URL` | Internal URL to csvtocsvw container |
| `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPTOMETHOD_URL` | Public URL (used in browser iframes) |
| `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_URL` | Internal container URL for RDFConverter |
| `CKANINI__CKANEXT__FUSEKI__URL` | Internal Fuseki URL |

## Post-Boot Setup

1. Start the stack: `docker compose up`
2. Create an admin API token at `/user/<admin>/api-tokens`
3. Paste it as `BACKGROUNDJOBS_API_TOKEN` in `.env`
4. Restart: `docker compose restart ckan`
5. Create a CKAN group named exactly `mappings` — required by ckanext-csvwmapandtransform

## Live Instances

- [futurecarproduction.materialsdata.space](https://futurecarproduction.materialsdata.space/) — Future Car Production Materials Data Space
- [dataportal.material-digital.de](https://dataportal.material-digital.de/) — Material Digital Data Portal
