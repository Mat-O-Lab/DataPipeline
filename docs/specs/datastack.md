# DataStack — Component Specification

**Repo:** https://github.com/Mat-O-Lab/DataStack  
**Type:** Docker Compose orchestration  
**Role:** Deploys the full Mat-O-Lab PMD-CKAN platform as a containerized stack for FAIR materials-science data management in decentralised linked-data spaces.

---

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

---

## Configuration Convention

Every `.env` variable prefixed `CKANINI__` is auto-translated to `ckan.ini` at container start:
- prefix stripped
- `__` → `.`
- lowercased

Example: `CKANINI__CKANEXT__FUSEKI__URL` → `ckanext.fuseki.url`

## Key Environment Variables

| Variable | Purpose |
|---|---|
| `CKAN__PLUGINS` | Space-separated plugin list: must include `fuseki csvtocsvw csvwmapandtransform dcat structured_data harvest ...` |
| `BACKGROUNDJOBS_API_TOKEN` | CKAN API token used by all background-job extensions; must be created manually after first boot |
| `FUSEKI_JAVA_OPTS` | JVM heap (default: `-Xmx10g -Xms10g`) |
| `CKANINI__CKANEXT__CSVTOCSVW__CSVTOCSVW_URL` | Internal URL to csvtocsvw container |
| `CKANINI__CKANEXT__CSVTOCSVW__FORMATS` | Formats that trigger CSV annotation |
| `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPTOMETHOD_URL` | Public URL (used in browser iframes) |
| `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_URL` | Internal container URL for RDFConverter |
| `CKANINI__CKANEXT__FUSEKI__URL` | Internal Fuseki URL |
| `CKANINI__CKANEXT__FUSEKI__USERNAME` / `PASSWORD` | Fuseki admin credentials |
| `CKANINI__CKANEXT__FUSEKI__SPARKLIS_URL` | Sparklis UI URL (empty = use YASGUI) |

## Post-Boot Setup

1. Start the stack (`docker compose up`)
2. Create an admin API token at `/user/<admin>/api-tokens`
3. Paste it as `BACKGROUNDJOBS_API_TOKEN` in `.env`
4. Restart (`docker compose restart ckan`)
5. Create a CKAN group named exactly `mappings` — required by ckanext-csvwmapandtransform

---

## Capabilities

- Full FAIR data portal (CKAN 2.10/2.11)
- Automated CSV → semantic data pipeline on upload (via extensions)
- SPARQL query interface (Fuseki + Sparklis/YASGUI)
- Harvestable metadata (DCAT, structured_data)
- All components deploy with a single `docker compose up`

## Limitations / Known Constraints

- `BACKGROUNDJOBS_API_TOKEN` is a manual post-boot step — stack won't auto-process resources without it
- Fuseki auto-sync hooks are commented out in ckanext-fuseki — Fuseki upload requires manual trigger via CKAN UI or API
- `rmlmapper` is hardcoded to port 4000; cannot be changed without rebuilding the image
- Data sources must be publicly accessible URLs for URL-based microservice endpoints (upload endpoints exist as fallback)
- Fuseki JVM heap is set to 10 GB by default — tune for your host

## Live Instances

- https://futurecarproduction.materialsdata.space/ — Future Car Production Materials Data Space
- https://dataportal.material-digital.de/ — Material Digital Data Portal
