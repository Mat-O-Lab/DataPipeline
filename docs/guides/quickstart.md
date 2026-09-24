---
title: Quickstart
---

# Quickstart

Deploy the full DataStack locally with Docker Compose and process your first CSV file.

## Prerequisites

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
# Required: change all placeholder values
SECRET_KEY=<openssl rand -hex 32>
CKAN_SYSADMIN_NAME=ckan_admin
CKAN_SYSADMIN_PASSWORD=<strong_password>
CKAN_SYSADMIN_EMAIL=admin@example.com

# Deployment URL — must be reachable from your browser
CKAN_HOST=<your.domain.example.com>   # e.g. localhost or 192.168.1.10
CKAN_SITE_URL=https://${CKAN_HOST}

# Database passwords
POSTGRES_PASSWORD=<strong_password>
CKAN_DB_PASSWORD=<strong_password>
DATASTORE_READONLY_PASSWORD=<strong_password>

# Fuseki credentials
CKANINI__CKANEXT__FUSEKI__PASSWORD=<strong_password>
```

Leave all other values at their defaults for a working local stack.

✓ **Verify:** No remaining `<placeholder>` strings in `.env` (`grep '<' .env` returns empty).

---

## Step 3 — Start the stack

```bash
docker compose up -d
```

The first run pulls images and builds the CKAN container — allow 5–10 minutes. Watch logs with `docker compose logs -f ckan`.

✓ **Verify:** `docker compose ps` shows all services as `Up`. CKAN home page loads at `http://<CKAN_HOST>`.

---

## Step 4 — Create admin user and API token

1. Open `http://<CKAN_HOST>` → sign in as `ckan_admin` with the password from `.env`
2. Click your avatar → **Settings** → **API Tokens**
3. Create a token named `backgroundjobs` — copy the token value

✓ **Verify:** Token string is displayed (copy it now — it will not be shown again).

---

## Step 5 — Set `BACKGROUNDJOBS_API_TOKEN` and restart

!!! warning "Most common setup failure"
    Without this token, background jobs never execute. CSV uploads appear to succeed but no CSVW metadata or Turtle resources are created.

Open `.env` and add:

```bash
BACKGROUNDJOBS_API_TOKEN=<paste_token_here>
```

Restart CKAN:

```bash
docker compose restart ckan
```

✓ **Verify:** Upload any small CSV → within 30 seconds, the dataset should gain a second resource named `<filename>.csvw.json`.

---

## Step 6 — Create the `mappings` group

!!! note "Case-sensitive"
    The group name must be exactly `mappings` — any other capitalisation or spelling means no automatic mapping selection will occur.

1. In CKAN, go to **Groups** → **Add Group**
2. Name: `mappings` (lowercase, exact match)
3. Save

✓ **Verify:** The group appears at `http://<CKAN_HOST>/group/mappings`.

---

## Step 7 — Upload a test CSV

1. Go to **Datasets** → **Add Dataset**
2. Fill in title and organization, then **Next**
3. Upload a CSV file — any lab measurement CSV works; format must be `CSV`
4. Save

Watch what appears in the dataset's **Resources** tab over the next 60 seconds:

| Resource | Created by |
|---|---|
| `yourfile.csv` | You (upload) |
| `yourfile.csvw.json` | ckanext-csvtocsvw (automatic) |
| `yourfile.ttl` | ckanext-csvtocsvw (automatic) |
| `yourfile-joined.ttl` | ckanext-csvwmapandtransform (automatic, only if a matching mapping exists in the `mappings` group) |

✓ **Verify:** At minimum, `yourfile.csvw.json` appears. If joined Turtle does not appear, no matching mapping exists yet — see [Author a Mapping](author-a-mapping.md).

---

## Step 8 — (Optional) Trigger Fuseki and open SPARQL

Fuseki upload is a manual step — see [Capability Map — Limitations](../pipeline/capability-map.md#what-the-pipeline-cannot-do-current-limitations).

1. In the dataset view, click the **Fuseki** action button (or call the CKAN API action `fuseki_update`)
2. A SPARQL endpoint resource appears in the dataset
3. Click it to open Sparklis or YASGUI and run your first query

✓ **Verify:** SPARQL query `SELECT * WHERE { ?s ?p ?o } LIMIT 10` returns triples from your dataset.

---

## Next steps

- **Create a mapping for your data:** [Author a Mapping](author-a-mapping.md)
- **Use the microservices without CKAN:** [Standalone APIs](standalone-apis.md)
- **Understand what the pipeline can handle:** [Capability Map](../pipeline/capability-map.md)
