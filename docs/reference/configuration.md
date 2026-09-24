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

---

## ckanext-csvtocsvw

Controls which file formats trigger CSV annotation and where the CSVToCSVW microservice is reached.

| ini key | `CKANINI__` env form | Default | Description |
|---|---|---|---|
| `ckanext.csvtocsvw.csvtocsvw_url` | `CKANINI__CKANEXT__CSVTOCSVW__CSVTOCSVW_URL` | `https://csvtocsvw.matolab.org` | Microservice URL — use internal container URL in DataStack: `http://csvtocsvw:5001` |
| `ckanext.csvtocsvw.formats` | `CKANINI__CKANEXT__CSVTOCSVW__FORMATS` | `csv txt asc tsv` | Space-separated file formats that trigger annotation |
| `ckanext.csvtocsvw.ckan_token` | `CKANINI__CKANEXT__CSVTOCSVW__CKAN_TOKEN` | `""` | Set to `${BACKGROUNDJOBS_API_TOKEN}` |
| `ckanext.csvtocsvw.ssl_verify` | `CKANINI__CKANEXT__CSVTOCSVW__SSL_VERIFY` | `True` | SSL cert verification for outbound requests |

**DataStack compose snippet:**

```bash
CKANINI__CKANEXT__CSVTOCSVW__CSVTOCSVW_URL=http://csvtocsvw:${CSVTOCSVW_APP_PORT}
CKANINI__CKANEXT__CSVTOCSVW__FORMATS="csv txt asc"
CKANINI__CKANEXT__CSVTOCSVW__SSL_VERIFY=${SSL_VERIFY}
```

---

## ckanext-csvwmapandtransform

Controls mapping discovery, selection strategy, and which RDF formats trigger transformation.

| ini key | `CKANINI__` env form | Default | Description |
|---|---|---|---|
| `ckanext.csvwmapandtransform.maptomethod_url` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPTOMETHOD_URL` | `https://maptomethod.matolab.org` | **Public URL** — used in browser iframes for mapping authoring |
| `ckanext.csvwmapandtransform.rdfconverter_url` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_URL` | `https://rdfconverter.matolab.org` | RDFConverter URL — use internal container URL in DataStack: `http://rdfconverter:5003` |
| `ckanext.csvwmapandtransform.formats` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__FORMATS` | `json json-ld turtle n3 nt hext trig longturtle xml ld+json` | Formats that trigger transformation |
| `ckanext.csvwmapandtransform.mapping_strategy` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPPING_STRATEGY` | `exact` | `exact` (production) or `best_match` (development) |
| `ckanext.csvwmapandtransform.ckan_token` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__CKAN_TOKEN` | `""` | Set to `${BACKGROUNDJOBS_API_TOKEN}` |
| `ckanext.csvwmapandtransform.rdfconverter_timeout_check` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_TIMEOUT_CHECK` | `30` | Timeout (seconds) for `/api/checkmapping` calls |
| `ckanext.csvwmapandtransform.rdfconverter_timeout_create` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_TIMEOUT_CREATE` | `120` | Timeout (seconds) for `/api/createrdf` calls — increase for large files |

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

| ini key | `CKANINI__` env form | Default | Description |
|---|---|---|---|
| `ckanext.fuseki.url` | `CKANINI__CKANEXT__FUSEKI__URL` | — | Internal Fuseki URL (e.g. `http://fuseki:3030`) |
| `ckanext.fuseki.username` | `CKANINI__CKANEXT__FUSEKI__USERNAME` | `admin` | Fuseki admin username |
| `ckanext.fuseki.password` | `CKANINI__CKANEXT__FUSEKI__PASSWORD` | — | Fuseki admin password |
| `ckanext.fuseki.sparklis_url` | `CKANINI__CKANEXT__FUSEKI__SPARKLIS_URL` | — | Sparklis UI URL; empty = use YASGUI |
| `ckanext.fuseki.formats` | `CKANINI__CKANEXT__FUSEKI__FORMATS` | `turtle text/turtle n3 nt …` | RDF formats that trigger Fuseki upload |

**DataStack compose snippet:**

```bash
CKANINI__CKANEXT__FUSEKI__URL=${CKAN_SITE_URL}/fuseki/
CKANINI__CKANEXT__FUSEKI__USERNAME=admin
CKANINI__CKANEXT__FUSEKI__PASSWORD=<strong_password>
CKANINI__CKANEXT__FUSEKI__SPARKLIS_URL=${CKAN_SITE_URL}/sparklis/
FUSEKI_JAVA_OPTS="-Xmx10g -Xms10g -DentityExpansionLimit=0"
```
