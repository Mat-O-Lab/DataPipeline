---
title: Configuration
---

# Configuration

All configuration for the DataStack and its CKAN extensions, organized by component.

## The `CKANINI__` Convention

Every `.env` variable prefixed `CKANINI__` is automatically translated to `ckan.ini` at container startup:

- Prefix `CKANINI__` is stripped
- `__` separators become `.`
- Result is lowercased

```bash
# .env                                              # ckan.ini
CKANINI__CKANEXT__FUSEKI__URL=http://fuseki:3030   →  ckanext.fuseki.url = http://fuseki:3030
CKANINI__CKANEXT__CSVTOCSVW__FORMATS=csv txt asc   →  ckanext.csvtocsvw.formats = csv txt asc
```

This means you never edit `ckan.ini` directly — all settings are in `.env`.

---

## DataStack `.env` — Core Variables

Start from `config/example.env` (see [Quickstart](../guides/quickstart.md)). Key variables to set before first boot:

| Variable | Required | Default | Description |
|---|---|---|---|
| `SECRET_KEY` | yes | — | CKAN secret key (`openssl rand -hex 32`) |
| `CKAN_HOST` | yes | — | Public-facing hostname or IP |
| `CKAN_SITE_URL` | yes | — | Full URL (e.g. `https://${CKAN_HOST}`) |
| `CKAN_SYSADMIN_NAME` | yes | `ckan_admin` | Admin username |
| `CKAN_SYSADMIN_PASSWORD` | yes | — | Admin password |
| `BACKGROUNDJOBS_API_TOKEN` | yes (post-boot) | — | CKAN API token for background jobs; create after first boot |
| `POSTGRES_PASSWORD` | yes | — | PostgreSQL superuser password |
| `CKAN_DB_PASSWORD` | yes | — | CKAN database user password |
| `FUSEKI_JAVA_OPTS` | no | `-Xmx10g -Xms10g` | JVM heap for Fuseki — tune for available RAM |
| `CKAN__PLUGINS` | no | see example.env | Space-separated plugin list; must include `fuseki csvtocsvw csvwmapandtransform` |

!!! warning "Most common setup failure: `BACKGROUNDJOBS_API_TOKEN` not set"
    All three extensions (csvtocsvw, csvwmapandtransform, fuseki) share this token for background job authentication. Without it the stack starts normally but **no resource processing happens** — no CSV annotation, no RDF conversion, no Fuseki sync. Create an API token at `/user/<admin>/api-tokens` after first boot, paste it into `.env`, then restart CKAN.

---

## ckanext-csvtocsvw

Controls which file formats trigger CSV annotation and where the CSVToCSVW microservice is reached.

| Config key (ini form) | `CKANINI__` env var form | Default | Description |
|---|---|---|---|
| `ckanext.csvtocsvw.csvtocsvw_url` | `CKANINI__CKANEXT__CSVTOCSVW__CSVTOCSVW_URL` | `https://csvtocsvw.matolab.org` | Microservice URL — use internal container URL in DataStack: `http://csvtocsvw:5001` |
| `ckanext.csvtocsvw.formats` | `CKANINI__CKANEXT__CSVTOCSVW__FORMATS` | `csv txt asc tsv` | Space-separated file formats that trigger annotation (lowercased comparison) |
| `ckanext.csvtocsvw.ckan_token` | `CKANINI__CKANEXT__CSVTOCSVW__CKAN_TOKEN` | `""` | CKAN API token for background job authentication — set to `${BACKGROUNDJOBS_API_TOKEN}` |
| `ckanext.csvtocsvw.ssl_verify` | `CKANINI__CKANEXT__CSVTOCSVW__SSL_VERIFY` | `True` | SSL cert verification for outbound requests to CSVToCSVW |

**DataStack compose snippet:**

```bash
CKANINI__CKANEXT__CSVTOCSVW__CSVTOCSVW_URL=http://csvtocsvw:${CSVTOCSVW_APP_PORT}
CKANINI__CKANEXT__CSVTOCSVW__FORMATS="csv txt asc"
CKANINI__CKANEXT__CSVTOCSVW__SSL_VERIFY=${SSL_VERIFY}
```

---

## ckanext-csvwmapandtransform

Controls mapping discovery, selection strategy, and which RDF formats trigger transformation.

| Config key (ini form) | `CKANINI__` env var form | Default | Description |
|---|---|---|---|
| `ckanext.csvwmapandtransform.maptomethod_url` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPTOMETHOD_URL` | `https://maptomethod.matolab.org` | **Public URL** — used in browser iframes for mapping authoring; must be publicly accessible |
| `ckanext.csvwmapandtransform.rdfconverter_url` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_URL` | `https://rdfconverter.matolab.org` | RDFConverter URL — use internal container URL in DataStack: `http://rdfconverter:5003` |
| `ckanext.csvwmapandtransform.formats` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__FORMATS` | `json json-ld turtle n3 nt hext trig longturtle xml ld+json` | Resource formats that trigger transformation |
| `ckanext.csvwmapandtransform.mapping_strategy` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPPING_STRATEGY` | `exact` | `exact` (production — only fully-matching mappings apply) or `best_match` (development — best partial match applies) |
| `ckanext.csvwmapandtransform.ckan_token` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__CKAN_TOKEN` | `""` | CKAN API token for background job authentication — set to `${BACKGROUNDJOBS_API_TOKEN}` |
| `ckanext.csvwmapandtransform.rdfconverter_timeout_check` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_TIMEOUT_CHECK` | `30` | Timeout (seconds) for `/api/checkmapping` calls |
| `ckanext.csvwmapandtransform.rdfconverter_timeout_create` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_TIMEOUT_CREATE` | `120` | Timeout (seconds) for `/api/createrdf` calls — increase for large files |
| `ckanext.csvwmapandtransform.ssl_verify` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__SSL_VERIFY` | `True` | SSL verification for outbound requests to RDFConverter |
| `ckanext.csvwmapandtransform.db_url` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__DB_URL` | inherits `sqlalchemy.url` | Database URL for job state tracking — inherits the CKAN main database URL (`sqlalchemy.url`) by default; only override to point at a separate database |

**DataStack compose snippet:**

```bash
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPTOMETHOD_URL=${CKAN_SITE_URL}/maptomethod
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_URL=http://rdfconverter:${RDFCONVERTER_APP_PORT}
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPPING_STRATEGY=exact
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__SSL_VERIFY=${SSL_VERIFY}
```

---

## ckanext-fuseki

Controls the Fuseki triplestore connection and the SPARQL query UI.

!!! note "Auto-sync hooks are disabled"
    `auto_hooks_state = DISABLED` — the `after_create` and `after_update` hooks are commented out in `plugin.py`. Fuseki sync does **not** happen automatically when resources are uploaded. Trigger it manually via the `fuseki_update` CKAN action (available from the dataset UI or API).

| Config key (ini form) | `CKANINI__` env var form | Default | Description |
|---|---|---|---|
| `ckanext.fuseki.url` | `CKANINI__CKANEXT__FUSEKI__URL` | `http://fuseki:3030/` | Fuseki base URL — use internal container URL in DataStack |
| `ckanext.fuseki.internal_url` | `CKANINI__CKANEXT__FUSEKI__INTERNAL_URL` | `""` | Optional direct URL bypassing the nginx reverse proxy |
| `ckanext.fuseki.username` | `CKANINI__CKANEXT__FUSEKI__USERNAME` | `admin` | Fuseki admin username |
| `ckanext.fuseki.password` | `CKANINI__CKANEXT__FUSEKI__PASSWORD` | `admin` | Fuseki admin password — **change before deploying** |
| `ckanext.fuseki.ckan_token` | `CKANINI__CKANEXT__FUSEKI__CKAN_TOKEN` | `""` | CKAN API token for background job authentication — set to `${BACKGROUNDJOBS_API_TOKEN}` |
| `ckanext.fuseki.formats` | `CKANINI__CKANEXT__FUSEKI__FORMATS` | `turtle text/turtle n3 nt hext trig longturtle xml json-ld ld+json jsonld` | Resource formats that trigger Fuseki upload |
| `ckanext.fuseki.sparklis_url` | `CKANINI__CKANEXT__FUSEKI__SPARKLIS_URL` | `""` | Sparklis UI URL — empty = use built-in YASGUI at `/dataset/{id}/fuseki/sparql` |
| `ckanext.fuseki.ssl_verify` | `CKANINI__CKANEXT__FUSEKI__SSL_VERIFY` | `True` | SSL verification for outbound requests |

**DataStack compose snippet:**

```bash
CKANINI__CKANEXT__FUSEKI__URL=${CKAN_SITE_URL}/fuseki/
CKANINI__CKANEXT__FUSEKI__USERNAME=admin
CKANINI__CKANEXT__FUSEKI__PASSWORD=<strong_password>
CKANINI__CKANEXT__FUSEKI__SPARKLIS_URL=${CKAN_SITE_URL}/sparklis/
FUSEKI_JAVA_OPTS="-Xmx10g -Xms10g -DentityExpansionLimit=0"
```
