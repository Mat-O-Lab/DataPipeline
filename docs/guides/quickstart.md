---
title: Quickstart
---

# Quickstart

Deploy the full DataStack locally with Docker Compose and process your first CSV file end-to-end.

## Prerequisites

!!! info "What you need before starting"
    - Docker Compose v2 (`docker compose version` ≥ 2.0)
    - Git
    - A public-facing URL or LAN IP address (CKAN needs a reachable base URL for resource links)
    - 10 GB RAM available for Fuseki (configurable via `FUSEKI_JAVA_OPTS`)

---

## Step 1 — Clone and initialise

```bash
git clone https://github.com/Mat-O-Lab/DataStack.git
cd DataStack
git submodule update --init
```

✓ **Verify:** `ls ckan-docker/` shows Docker build context files.

---

## Step 2 — Configure `.env`

```bash
cp config/example.env .env
```

Open `.env` and set these variables before first boot — the stack will not work correctly without them:

```bash
# Secrets — generate with: openssl rand -hex 32
SECRET_KEY=<random_string>

# Admin account
CKAN_SYSADMIN_NAME=ckan_admin
CKAN_SYSADMIN_PASSWORD=<strong_password>
CKAN_SYSADMIN_EMAIL=admin@example.com

# Deployment URL — must be reachable from your browser
CKAN_HOST=<your.domain.example.com>
CKAN_SITE_URL=https://${CKAN_HOST}

# Database passwords
POSTGRES_PASSWORD=<strong_password>
CKAN_DB_PASSWORD=<strong_password>
DATASTORE_READONLY_PASSWORD=<strong_password>

# Extension: CSVToCSVW — internal container URL (default is correct for docker compose)
CKANINI__CKANEXT__CSVTOCSVW__CSVTOCSVW_URL=http://csvtocsvw:5001

# Extension: CSVW Map & Transform — internal RDFConverter URL and public MapToMethod URL
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__RDFCONVERTER_URL=http://rdfconverter:5003
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPTOMETHOD_URL=${CKAN_SITE_URL}/maptomethod

# Fuseki credentials
CKANINI__CKANEXT__FUSEKI__PASSWORD=<strong_password>

# Background jobs token — leave empty now; you will fill this in after first boot (Step 5)
BACKGROUNDJOBS_API_TOKEN=
```

Leave all other values at their defaults for a working local stack.

✓ **Verify:** `grep '<' .env` returns only the variables you intentionally left as placeholders. `BACKGROUNDJOBS_API_TOKEN=` is expected to be empty at this stage.

---

## Step 3 — Start the stack

```bash
docker compose up -d
```

The first run pulls images and builds the CKAN container — allow 5–10 minutes. Watch the startup sequence with:

```bash
docker compose logs -f ckan
```

Services start in this order: **db** and **solr** initialise first, then **redis**, then **ckan** waits for both to be healthy before starting its own init. The pipeline microservices (`csvtocsvw`, `rdfconverter`, `maptomethod`, etc.) start in parallel. You will see `ckan | INFO  [ckan.config.middleware] ...` lines once CKAN is ready.

✓ **Verify:** `docker compose ps` shows all services as `Up` (not `Restarting`). CKAN home page loads at `http://<CKAN_HOST>`.

---

## Step 4 — Create admin user and API token

1. Open `http://<CKAN_HOST>` → sign in as `ckan_admin` with the password you set in `.env`
2. Navigate directly to **`/user/ckan_admin/api-tokens`** (or: click your avatar → **Settings** → **API Tokens**)
3. Enter a name (e.g. `backgroundjobs`) and click **Create API Token**
4. Copy the token value immediately — it will not be shown again

✓ **Verify:** Token string is displayed in the UI. Copy it before navigating away.

---

## Step 5 — Set `BACKGROUNDJOBS_API_TOKEN` and restart

!!! warning "Most common setup failure"
    Without `BACKGROUNDJOBS_API_TOKEN`, all background job extensions (`ckanext-csvtocsvw`, `ckanext-csvwmapandtransform`, `ckanext-fuseki`) run but never execute jobs. CSV uploads appear to succeed, but no CSVW metadata, no Turtle resources, and no Fuseki sync are ever created — silently.

Open `.env` and paste the token you copied:

```bash
BACKGROUNDJOBS_API_TOKEN=<paste_token_here>
```

Restart CKAN to pick up the new value:

```bash
docker compose restart ckan
```

✓ **Verify:** Upload any small CSV → within 30 seconds, the dataset should gain a second resource named `<filename>.csvw.json`. If it does not appear, confirm `BACKGROUNDJOBS_API_TOKEN` is set (not empty) and that you restarted CKAN.

---

## Step 6 — Create the `mappings` group

!!! note "Case-sensitive group name"
    The group name must be exactly `mappings` — all lowercase, no spaces. Any other capitalisation or spelling means `ckanext-csvwmapandtransform` will never find a matching mapping, and joined Turtle resources will never be created.

1. In CKAN, go to **Groups** → **Add Group**
2. **Name:** `mappings` (lowercase, exact match)
3. Leave all other fields at their defaults and click **Create Group**

✓ **Verify:** The group page loads at `http://<CKAN_HOST>/group/mappings`.

---

## Step 7 — Upload a test CSV

Download a sample file:

```bash
curl -L -o example2.csv \
  https://github.com/Mat-O-Lab/CSVToCSVW/raw/main/examples/example2.csv
```

Then in CKAN:

1. Go to **Datasets** → **Add Dataset**
2. Fill in a title and select your organisation, then click **Next**
3. Click **Upload** and select `example2.csv` — set format to `CSV`
4. Click **Finish**

Watch the **Resources** tab over the next 60 seconds:

| Resource | Created by |
|---|---|
| `example2.csv` | You (upload) |
| `example2.csvw.json` | ckanext-csvtocsvw (automatic) |
| `example2.ttl` | ckanext-csvtocsvw (automatic) |
| `example2-joined.ttl` | ckanext-csvwmapandtransform (automatic, only if a matching mapping exists in the `mappings` group) |

✓ **Verify:** At minimum, `example2.csvw.json` appears within 30–60 seconds. If joined Turtle does not appear, no matching mapping exists yet — see [Author a Mapping](author-a-mapping.md).

---

## Step 8 — (Optional) Trigger Fuseki and open SPARQL

Fuseki upload is a manual step (auto-sync hooks are commented out by default).

1. In the dataset view, click the **Fuseki** action button, or call the CKAN API action `fuseki_update`
2. A SPARQL endpoint resource appears in the dataset
3. Click the Sparklis link to open the faceted SPARQL UI, or use the built-in YASGUI interface

Run a quick sanity query:

```sparql
SELECT * WHERE { ?s ?p ?o } LIMIT 10
```

✓ **Verify:** The query returns triples from your dataset in the Sparklis or YASGUI interface.

---

## Next steps

- **Create a mapping for your data:** [Author a Mapping](author-a-mapping.md)
- **Use the microservices without CKAN:** [Standalone APIs](standalone-apis.md)
- **Understand what the pipeline can handle:** [Capability Map](../pipeline/capability-map.md)
