---
title: Configuration
---

# Configuration

All configuration for the DataStack and its CKAN extensions, organized by component. Use this page as a lookup reference — find the variable you need, copy the `CKANINI__` env var form into your `.env`, and restart CKAN.

## The `CKANINI__` Convention

Every `.env` variable prefixed `CKANINI__` is automatically translated to `ckan.ini` at container startup. You never edit `ckan.ini` directly — all settings live in `.env`.

The three transformation rules:

1. The `CKANINI__` prefix is stripped
2. Each `__` (double underscore) becomes `.` (dot)
3. The result is lowercased

```bash
# .env line                                             →  ckan.ini key
CKANINI__CKANEXT__FUSEKI__URL=http://fuseki:3030/      →  ckanext.fuseki.url = http://fuseki:3030/
CKANINI__CKANEXT__CSVTOCSVW__FORMATS=csv txt asc       →  ckanext.csvtocsvw.formats = csv txt asc
```

This applies to every `CKANINI__`-prefixed line regardless of which extension it configures.

---

## DataStack `.env` — Core Variables

Start from `config/example.env` (see [Quickstart](../guides/quickstart.md)). Set all **Required** variables before the first `docker compose up`.

| Variable | Type | Required | Default | Example | Description |
|---|---|---|---|---|---|
| `SECRET_KEY` | string | yes | — | `openssl rand -hex 32` output | CKAN session secret — generate once, never change |
| `CKAN_HOST` | string | yes | — | `myportal.example.org` | Public-facing hostname or IP — used to build `CKAN_SITE_URL` |
| `CKAN_SITE_URL` | URL | yes | — | `https://${CKAN_HOST}` | Full public URL of the portal |
| `CKAN_SYSADMIN_NAME` | string | yes | `ckan_admin` | `admin` | Admin username |
| `CKAN_SYSADMIN_PASSWORD` | string | yes | — | `changeme123` | Admin portal password — change before deploying |
| `POSTGRES_PASSWORD` | string | yes | — | `changeme456` | PostgreSQL superuser password |
| `CKAN_DB_PASSWORD` | string | yes | — | `changeme789` | CKAN database user password |
| `BACKGROUNDJOBS_API_TOKEN` | string | yes (post-boot) | — | paste token from `/user/<admin>/api-tokens` | CKAN API token for all background-job extensions — create after first boot |
| `FUSEKI_JAVA_OPTS` | string | no | `-Xmx10g -Xms10g` | `-Xmx6g -Xms6g` | JVM heap for Fuseki — tune for available host RAM |
| `CKAN__PLUGINS` | string | no | see example.env | `fuseki csvtocsvw csvwmapandtransform` | Space-separated plugin list — must include the three Mat-O-Lab extensions |

!!! warning "Most common setup failure: `BACKGROUNDJOBS_API_TOKEN` not set"
    All three extensions (csvtocsvw, csvwmapandtransform, fuseki) share this single token for background job authentication. Without it, the stack starts normally but **no resource processing happens** — no CSV annotation, no conversion, no Fuseki sync.

    **Post-boot steps:**

    1. Start the stack: `docker compose up -d`
    2. Log in as admin and go to `/user/<admin>/api-tokens`
    3. Create a new API token and copy it
    4. Paste it into `.env` as `BACKGROUNDJOBS_API_TOKEN=<token>`
    5. Restart CKAN: `docker compose restart ckan`

---

## ckanext-csvtocsvw

Controls which file formats trigger CSV annotation and how to reach the CSVToCSVW microservice.

| Config key (ini form) | `CKANINI__` env var form | Type | Required | Default | DataStack value | Description |
|---|---|---|---|---|---|---|
| `ckanext.csvtocsvw.csvtocsvw_url` | `CKANINI__CKANEXT__CSVTOCSVW__CSVTOCSVW_URL` | URL | yes | `https://csvtocsvw.matolab.org` | `http://csvtocsvw:5000` | Internal container URL to the CSVToCSVW microservice |
| `ckanext.csvtocsvw.formats` | `CKANINI__CKANEXT__CSVTOCSVW__FORMATS` | string | no | `csv txt asc tsv` | `csv txt asc` | Space-separated file formats that trigger annotation (case-insensitive) |
| `ckanext.csvtocsvw.ckan_token` | `CKANINI__CKANEXT__CSVTOCSVW__CKAN_TOKEN` | string | yes | `""` | `${BACKGROUNDJOBS_API_TOKEN}` | CKAN API token for background job auth — set to `${BACKGROUNDJOBS_API_TOKEN}` |
| `ckanext.csvtocsvw.ssl_verify` | `CKANINI__CKANEXT__CSVTOCSVW__SSL_VERIFY` | boolean | no | `True` | `${SSL_VERIFY}` | SSL certificate verification for outbound requests |

**DataStack `.env` snippet:**

```bash
CKANINI__CKANEXT__CSVTOCSVW__CSVTOCSVW_URL=http://csvtocsvw:5000
CKANINI__CKANEXT__CSVTOCSVW__CKAN_TOKEN=${BACKGROUNDJOBS_API_TOKEN}
CKANINI__CKANEXT__CSVTOCSVW__FORMATS="csv txt asc"
CKANINI__CKANEXT__CSVTOCSVW__SSL_VERIFY=${SSL_VERIFY}
```

---

## ckanext-csvwmapandtransform

Controls mapping discovery, selection strategy, and timeouts for the RDF conversion step.

| Config key (ini form) | `CKANINI__` env var form | Type | Required | Default | DataStack value | Description |
|---|---|---|---|---|---|---|
| `ckanext.csvwmapandtransform.maptomethod_url` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPTOMETHOD_URL` | URL | yes | `https://maptomethod.matolab.org` | `${CKAN_SITE_URL}/maptomethod` | **Public URL** — loaded in browser iframes for mapping authoring; must be reachable from user browsers |
| `ckanext.csvwmapandtransform.rdfconverter_url` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_URL` | URL | yes | `https://rdfconverter.matolab.org` | `http://rdfconverter:5000` | Internal container URL to the RDFConverter microservice |
| `ckanext.csvwmapandtransform.formats` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__FORMATS` | string | no | `json json-ld turtle n3 nt hext trig longturtle xml ld+json` | — | Resource formats that trigger the transformation pipeline |
| `ckanext.csvwmapandtransform.mapping_strategy` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPPING_STRATEGY` | string | no | `exact` | `exact` | `exact` — only fully-matching mappings apply (use in production); `best_match` — best partial match applies (use in development) |
| `ckanext.csvwmapandtransform.ckan_token` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__CKAN_TOKEN` | string | yes | `""` | `${BACKGROUNDJOBS_API_TOKEN}` | CKAN API token for background job auth — set to `${BACKGROUNDJOBS_API_TOKEN}` |
| `ckanext.csvwmapandtransform.rdfconverter_timeout_check` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_TIMEOUT_CHECK` | integer (seconds) | no | `30` | `30` | Timeout for mapping compatibility checks — increase if checks are timing out |
| `ckanext.csvwmapandtransform.rdfconverter_timeout_create` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_TIMEOUT_CREATE` | integer (seconds) | no | `120` | `120` | Timeout for the conversion job — increase for large files |
| `ckanext.csvwmapandtransform.ssl_verify` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__SSL_VERIFY` | boolean | no | `True` | `${SSL_VERIFY}` | SSL certificate verification for outbound requests |
| `ckanext.csvwmapandtransform.db_url` | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__DB_URL` | URL | no | inherits `sqlalchemy.url` | — | Database URL for job state tracking — inherits the CKAN main database connection by default; only override to point at a separate database |

!!! note "Prerequisite: `mappings` CKAN group"
    This extension discovers mapping files by searching a CKAN group named exactly `mappings` (case-sensitive). Create this group from the CKAN UI after first boot. Without it, no mappings will be found and no conversion will run.

**DataStack `.env` snippet:**

```bash
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPTOMETHOD_URL=${CKAN_SITE_URL}/maptomethod
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_URL=http://rdfconverter:5000
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__CKAN_TOKEN=${BACKGROUNDJOBS_API_TOKEN}
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPPING_STRATEGY=exact
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__SSL_VERIFY=${SSL_VERIFY}
```

---

## ckanext-fuseki

Controls the connection to the Fuseki storage service and the SPARQL query interface shown to users.

!!! warning "Automatic sync is disabled by default"
    `auto_hooks_state = DISABLED` — the hooks that would automatically sync resources to Fuseki on upload are commented out. Fuseki sync does **not** happen automatically. Trigger it manually: open the dataset in the CKAN UI and use the Fuseki sync button, or call the `fuseki_update` API action directly.

| Config key (ini form) | `CKANINI__` env var form | Type | Required | Default | DataStack value | Description |
|---|---|---|---|---|---|---|
| `ckanext.fuseki.url` | `CKANINI__CKANEXT__FUSEKI__URL` | URL | yes | `http://fuseki:3030/` | `http://fuseki:3030/` | Internal Fuseki base URL — CKAN connects to Fuseki via this URL inside the Docker network |
| `ckanext.fuseki.internal_url` | `CKANINI__CKANEXT__FUSEKI__INTERNAL_URL` | URL | no | `""` | — | Optional direct URL bypassing the nginx reverse proxy — leave empty unless you have a specific network setup |
| `ckanext.fuseki.username` | `CKANINI__CKANEXT__FUSEKI__USERNAME` | string | yes | `admin` | `admin` | Fuseki admin username |
| `ckanext.fuseki.password` | `CKANINI__CKANEXT__FUSEKI__PASSWORD` | string | yes | `admin` | — | Fuseki admin password — **change this before deploying** |
| `ckanext.fuseki.ckan_token` | `CKANINI__CKANEXT__FUSEKI__CKAN_TOKEN` | string | yes | `""` | `${BACKGROUNDJOBS_API_TOKEN}` | CKAN API token for background job auth — set to `${BACKGROUNDJOBS_API_TOKEN}` |
| `ckanext.fuseki.formats` | `CKANINI__CKANEXT__FUSEKI__FORMATS` | string | no | `turtle text/turtle n3 nt hext trig longturtle xml json-ld ld+json jsonld` | — | Resource formats that trigger Fuseki upload |
| `ckanext.fuseki.sparklis_url` | `CKANINI__CKANEXT__FUSEKI__SPARKLIS_URL` | URL | no | `""` | `${CKAN_SITE_URL}/sparklis/` | Sparklis query UI URL — if set, users are sent to Sparklis; if empty, the built-in YASGUI is used at `/dataset/{id}/fuseki/sparql` |
| `ckanext.fuseki.ssl_verify` | `CKANINI__CKANEXT__FUSEKI__SSL_VERIFY` | boolean | no | `True` | `${SSL_VERIFY}` | SSL certificate verification for outbound requests |

**DataStack `.env` snippet:**

```bash
CKANINI__CKANEXT__FUSEKI__URL=http://fuseki:3030/
CKANINI__CKANEXT__FUSEKI__USERNAME=admin
CKANINI__CKANEXT__FUSEKI__PASSWORD=<strong_password>
CKANINI__CKANEXT__FUSEKI__CKAN_TOKEN=${BACKGROUNDJOBS_API_TOKEN}
CKANINI__CKANEXT__FUSEKI__SPARKLIS_URL=${CKAN_SITE_URL}/sparklis/
FUSEKI_JAVA_OPTS="-Xmx10g -Xms10g -DentityExpansionLimit=0"
```

---

## Cross-service wiring

Three configuration keys must be consistent across services:

| What must match | Variable in each extension | Why |
|---|---|---|
| `BACKGROUNDJOBS_API_TOKEN` | `CKANINI__CKANEXT__CSVTOCSVW__CKAN_TOKEN`, `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__CKAN_TOKEN`, `CKANINI__CKANEXT__FUSEKI__CKAN_TOKEN` | All three extensions authenticate background jobs with the same CKAN API token |
| `SSL_VERIFY` | `CKANINI__CKANEXT__CSVTOCSVW__SSL_VERIFY`, `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__SSL_VERIFY`, `CKANINI__CKANEXT__FUSEKI__SSL_VERIFY` | SSL verification must be set consistently or you will get mixed TLS failures |
| `sqlalchemy.url` (CKAN main DB) | `CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__DB_URL` | csvwmapandtransform's db_url defaults to the CKAN main database connection — only set it explicitly if you want a separate database for job state |

In the DataStack `example.env`, these are set once using shell variable expansion so they stay in sync automatically.
