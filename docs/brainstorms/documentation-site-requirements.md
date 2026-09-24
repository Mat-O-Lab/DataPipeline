# Documentation Site Requirements
**Date:** 2026-09-24  
**Status:** Ready for planning

---

## Problem

The Mat-O-Lab DataStack pipeline is powerful and production-proven (26 Catena-X mappings, peer-reviewed publication, live CKAN instances) but undiscoverable and unapproachable. Documentation is scattered across 7+ repos with no cross-cutting narrative. The core principle — *create complete metadata for a non-semantic resource, then transform with RML/YARRRML* — is not communicated anywhere.

## Goal

A single, navigable documentation site that:
- Explains the pipeline principle and proves it works across diverse resource types
- Serves all four audiences without forcing each to wade through content meant for others
- Lives in one place (`DataPipeline` repo) and can be updated as upstream repos evolve
- Is publishable and shareable immediately, with stubs acceptable for planned-but-not-yet-written content

---

## Decisions Made

| Decision | Choice | Rationale |
|---|---|---|
| Platform | MkDocs + Material theme | Audience tabs, Mermaid, search, GitHub Pages CI — zero infra cost |
| Hosting | GitHub Pages (this repo) | Single point of truth; auto-deploys on push to main |
| Audiences | All four (see below) | Each has distinct needs; can't prioritise one |
| v1 scope | Full nav structure, stubs OK | Maps what exists and what's coming; easier incremental contribution |
| Content strategy | Narrative-first, specs as backing material | docs/specs/ are reference; published site tells the story |

---

## Audiences and Their Needs

| Audience | Primary need | Entry point |
|---|---|---|
| Researchers / data managers | Upload data, get FAIR knowledge graph — no coding | "How it works" + quickstart |
| Data engineers / DevOps | Deploy and configure DataStack | Quickstart + configuration reference |
| Ontology engineers | Author mappings, design patterns | Mapping authoring guide + OntosphereIO workflow |
| Developers / integrators | Call microservices directly from own code | API reference + standalone usage |

---

## Site Structure

```
docs/
  index.md                        ← Landing: pipeline principle, use case gallery, quick links
  pipeline/
    overview.md                   ← Architecture diagram (Mermaid) + component table
    default-use-case.md           ← CSV → CSVW → mapping → RDF → Fuseki walkthrough
    capability-map.md             ← Resource-type matrix + two transformation paths
    advanced/
      samm-catena-x.md            ← SAMM/Catena-X + SPARQL INSERT pattern (industrial use case)
      idta-aas.md                 ← IDTA/AAS submodel single-stage pattern
  guides/
    quickstart.md                 ← Deploy DataStack, upload CSV, see result
    author-a-mapping.md           ← OntosphereIO → MapToMethod → CKAN workflow
    standalone-apis.md            ← Use microservices without CKAN
    add-a-use-case.md             ← New domain / new extractor
  reference/
    api-endpoints.md              ← Consolidated: CSVToCSVW + MapToMethod + RDFConverter
    configuration.md              ← All config keys for DataStack + 3 extensions
    data-formats.md               ← What flows between each stage
  extractors/
    csvtocsvw.md                  ← Extractor spec (from docs/specs/)
    omeroextractor.md             ← Extractor spec + DF-TEM-PAW example + publication
    openbismantic.md              ← Extractor spec (no public deployment note)
    ontop.md                      ← External SQL bridge (stub)
  components/
    maptomethod.md                ← Microservice spec
    rdfconverter.md               ← Microservice spec
    ckanext-csvtocsvw.md          ← Extension spec
    ckanext-csvwmapandtransform.md ← Extension spec
    ckanext-fuseki.md             ← Extension spec (Fuseki auto-hooks note)
```

---

## Technical Requirements

### MkDocs Setup

- **Theme:** `material` (Material for MkDocs)
- **Plugins:**
  - `search` — full-text search
  - `mermaid2` — pipeline flow diagrams
- **Navigation tabs:** top-level tabs mapping to audience entry points (Pipeline, Guides, Reference, Components)
- **Repo link:** point to `Mat-O-Lab/DataStack` as the canonical stack repo

### Branding (Mat-O-Lab / Fraunhofer identity)

Both `ckanext-matolabtheme` and `OrgSite` use the same Fraunhofer color system — apply via `docs/stylesheets/extra.css`:

```css
:root {
  --md-primary-fg-color:        #005B7F;  /* Fraunhofer blue */
  --md-primary-fg-color--light: #4A7FA5;  /* steel blue */
  --md-primary-fg-color--dark:  #1A3D5C;  /* navy */
  --md-accent-fg-color:         #F58220;  /* orange */
  --md-typeset-a-color:         #005B7F;
  --md-text-font: "Helvetica Neue", Arial, Helvetica, sans-serif;
}
[data-md-color-scheme="slate"] {
  --md-primary-fg-color:        #3A6FA8;
  --md-default-bg-color:        #222222;
  --md-default-fg-color:        #A6BBC8;
}
```

Logo and favicon: copy from `Mat-O-Lab/OrgSite` (`assets/images/logos/matolab.svg`, `favicon.svg`). Reference in `mkdocs.yml`:

```yaml
theme:
  logo: assets/matolab.svg
  favicon: assets/favicon.svg
```

No full theme port needed — design token extraction + SVG reuse is sufficient.

### GitHub Actions CI

- Trigger: push to `main`
- Build: `mkdocs build`
- Deploy: `mkdocs gh-deploy` → GitHub Pages
- Python deps: `mkdocs-material`, `mkdocs-mermaid2-plugin`

### Diagrams

Every pipeline overview page must include a Mermaid diagram. The master diagram shows all services + data flows + automation vs. manual steps. Use it on `index.md` and `pipeline/overview.md`.

### Status Badges

Components that are incomplete or partially available must carry an explicit status note:
- ckanext-fuseki: "auto-sync hooks currently manual — requires triggering `fuseki_update` via CKAN UI or API"
- OpenBISmantic: "no active public deployment — requires own openBIS instance"
- OmeroExtractor: "early stage (v0.0.4)"

---

## Content Priorities (writing order)

1. `index.md` — landing page with pipeline principle + use case gallery (CSV, OMERO, SAMM/Catena-X)
2. `pipeline/overview.md` — master Mermaid diagram + component table
3. `pipeline/default-use-case.md` — concrete CSV walkthrough using live instance examples
4. `guides/quickstart.md` — deploy DataStack end to end
5. `guides/author-a-mapping.md` — OntosphereIO → MapToMethod workflow
6. `pipeline/capability-map.md` — resource-type matrix
7. `pipeline/advanced/samm-catena-x.md` — SAMM two-stage pattern (PA6GF30 example)
8. All reference pages — generated largely from docs/specs/
9. Remaining guides + extractor pages — fill in as time allows

---

## Key Content Decisions

**Ontology-agnostic story:** Show it through the use case gallery on the landing page — same pipeline steps, three different domains (lab CSV, microscopy, automotive supply chain), three different ontologies. The infrastructure doesn't change.

**SAMM/Catena-X placement:** `pipeline/advanced/` — not buried, but not the headline. The default CSV use case is the on-ramp. SAMM is the proof that the pipeline scales to industrial standards.

**Incomplete components:** Document what exists honestly. Status notes are better than silence or omission.

**OntosphereIO:** Mention on `guides/author-a-mapping.md` as the recommended pattern authoring tool. Do not recommend draw.io or reference IOFMaterialsTutorial as current guidance.

**IOFMaterialsTutorial:** Reference as "historical context" with a clear "tooling is outdated" note if linked at all.

---

## Keeping Docs Current

**Approach: agent-based update sweep, run on demand or on schedule.**

A Workflow script (`docs/update-workflow.js`) fans out one agent per component. Each agent:
1. Fetches the source repo README + any release notes (GitHub API)
2. Fetches the live OpenAPI JSON from the public deployment (if available)
3. Reads the existing `docs/specs/<component>.md`
4. Diffs what changed — new endpoints, changed config keys, new capabilities, version bumps
5. Rewrites/enriches the spec file with accurate current info
6. Reports what changed and what it flagged for human review

Also runs agents on the live CKAN instances to find new example datasets worth documenting.

**Trigger options:**
- Manual: `claude /update-docs` (Claude Code slash command or Workflow invocation)
- Scheduled: Claude Code cron job (weekly or on release)
- On-demand when a source repo releases a new version

**Human review:** Agent outputs a summary of what changed. A human approves before committing. No auto-push to main.

**Result:** The `docs/specs/` directory stays as the single point of truth, always enriched from primary sources, never stale by more than one sweep interval.

## Out of Scope (v1)

- Per-repo docs in each upstream repo (cross-link to this site instead)
- Automated OpenAPI spec rendering (link to live `/api/docs` URLs instead)
- Jupyter notebooks in the doc site
- Versioned docs (one version to start)
- Translations

---

## Success Criteria

- A new user can deploy DataStack and see an automated CSV → SPARQL result by following the quickstart
- An ontology engineer can author a new mapping using OntosphereIO + MapToMethod without prior knowledge of YARRRML
- A developer can call CSVToCSVW, MapToMethod, and RDFConverter standalone using only the API reference
- The site is publicly accessible via GitHub Pages and linked from the DataStack repo README
