---
title: "feat: Build MkDocs documentation site for DataStack pipeline"
type: feat
status: active
date: 2026-09-24
origin: docs/brainstorms/documentation-site-requirements.md
---

# feat: Build MkDocs Documentation Site for DataStack Pipeline

## Overview

Set up a MkDocs + Material theme documentation site in this repo, deploy to GitHub Pages, brand it with the Fraunhofer/Mat-O-Lab color palette, scaffold all planned pages (stubs acceptable), and write the two highest-priority narrative pages. The agent sweep workflow for spec refresh is deferred to a follow-on plan.

## Problem Frame

The Mat-O-Lab DataStack pipeline is production-proven but undiscoverable. Documentation is scattered across 7+ repos with no cross-cutting narrative. All four target audiences (researchers, data engineers, ontology engineers, integrators) have unmet needs. The site must be a single updatable point of truth across all components.

*(see origin: docs/brainstorms/documentation-site-requirements.md)*

## Requirements Trace

- R1a. MkDocs + Material theme — site builds locally and serves correctly
- R1b. GitHub Pages — auto-deploys on push to main
- R2. Mat-O-Lab Fraunhofer brand identity applied (colors, logo, fonts)
- R3. Full nav structure scaffolded — all planned pages exist, stubs acceptable
- R4. Two highest-priority pages written: `index.md` (landing) and `pipeline/overview.md`
- R5. ~~Agent update workflow script~~ — **deferred to follow-on plan** (build after site ships with authored content and first manual update cycle validates the pattern)
- R6. Topic-structured navigation (four tabs: Pipeline, Guides, Reference, Components) with audience-specific entry points — each tab index page oriented to its primary audience (see Key Technical Decisions for the tab-to-audience mapping)
- R7. **Gate requirement:** Before this plan is marked complete, a follow-on content authoring task must be created with: at minimum three pages identified for next iteration, and an owner named. Prevents the stub scaffold from becoming the permanent state.

## Scope Boundaries

- Content authoring beyond the two priority pages is out of scope — stubs with `!!! info "Coming soon"` admonition are sufficient
- No per-repo docs in upstream repos (cross-link to this site from their READMEs instead — separate task)
- No automated OpenAPI rendering — link to live `/api/docs` URLs instead
- No Jupyter notebooks in the site
- No versioned docs

### Deferred to Separate Tasks

- Writing remaining guide and reference pages: ongoing content authoring effort
- Updating upstream repo READMEs to link to this site: separate PR per repo (at minimum DataStack README should link to site; see Finding 5 note)
- **Agent update sweep workflow (R5):** deferred to follow-on plan — build `update-workflow.js` after the site ships, authored content exists, and one manual update cycle has been run to validate the pattern. See Unit 5 (removed) for the full component table with spec_file / source_repo / api_url that the follow-on plan should consume.

## Context & Research

### Relevant Files in This Repo

- `docs/specs/` — 10 component spec files (source material for site content; `ontop-integration.md` is TODO)
- `docs/plan.md` — full content plan and page outline
- `docs/references.md` — all URLs, repos, deployments, publications
- `docs/brainstorms/documentation-site-requirements.md` — origin document

### External References

- MkDocs Material: https://squidfunk.github.io/mkdocs-material/
- Material Mermaid support: built-in via `pymdownx.superfences` — **do not** use `mkdocs-mermaid2` plugin (deprecated for Material users)
- GitHub Pages deploy action: `mhausenblas/mkdocs-deploy-gh-pages` or `mkdocs gh-deploy --force`
- Fraunhofer palette source: `Mat-O-Lab/ckanext-matolabtheme` and `Mat-O-Lab/OrgSite`
- Logo SVG: `Mat-O-Lab/OrgSite` → `assets/images/logos/matolab.svg`
- Favicon SVG: `Mat-O-Lab/OrgSite` → `favicon.svg`

## Key Technical Decisions

- **Mermaid via pymdownx.superfences, not mkdocs-mermaid2:** Material v9 includes Mermaid support natively; the plugin adds no value and is no longer maintained for Material users
- **Custom CSS for branding, not a custom theme:** ~20 lines in `docs/stylesheets/extra.css` override Material's CSS custom properties — no theme fork needed
- **Navigation tabs:** `navigation.tabs` + `navigation.indexes` features in Material give top-level tab navigation and section index pages. Tab-to-audience mapping: **Pipeline** → researchers/data managers; **Guides** → data engineers/DevOps; **Reference** → ontology engineers; **Components** → developers/integrators. Each tab index page should open with a one-paragraph "this section is for you if…" oriented to its audience.
- **GitHub Pages deploy via `mkdocs gh-deploy --force` in CI:** Simplest approach; pushes to `gh-pages` branch automatically
- **Update workflow as a Workflow tool script, not GitHub Actions bot:** Human-reviewed, on-demand; uses the agent infrastructure already available in this project
- **Stubs use a consistent template:** Each stub page has a frontmatter title + one-line description + `!!! info "Coming soon"` admonition so the nav renders meaningfully even before content is written

## Open Questions

### Resolved During Planning

- **Mermaid plugin choice:** Use `pymdownx.superfences` (built into Material v9), not `mkdocs-mermaid2`
- **Branding approach:** CSS custom property override + SVG asset copy — no full theme port
- **Update trigger:** On-demand Workflow script; cron scheduling deferred

### Deferred to Implementation

- Exact Mermaid diagram layout for `pipeline/overview.md` — draft in implementation, iterate visually
- Whether to enable `navigation.instant` (client-side navigation) — test during scaffolding; disable if it conflicts with Mermaid rendering
- ~~Exact tab labels~~ — **decided:** Pipeline / Guides / Reference / Components (see Key Technical Decisions for audience mapping)

## Output Structure

```
docs/
  index.md
  stylesheets/
    extra.css
  assets/
    matolab.svg
    favicon.svg
  pipeline/
    index.md
    overview.md
    default-use-case.md
    capability-map.md
    advanced/
      samm-catena-x.md
      idta-aas.md  ← also under pipeline/advanced/ (not pipeline/)
  guides/
    index.md
    quickstart.md
    author-a-mapping.md
    standalone-apis.md
    add-a-use-case.md
  reference/
    index.md
    api-endpoints.md
    configuration.md
    data-formats.md
  extractors/
    index.md
    csvtocsvw.md
    omeroextractor.md
    openbismantic.md
    ontop.md
  components/
    index.md
    datastack.md  ← Docker Compose orchestration (wrapper for docs/specs/datastack.md)
    maptomethod.md
    rdfconverter.md
    ckanext-csvtocsvw.md
    ckanext-csvwmapandtransform.md
    ckanext-fuseki.md
mkdocs.yml
requirements-docs.txt
.github/
  workflows/
    docs.yml
```

## High-Level Technical Design

> *This illustrates the intended approach and is directional guidance for review, not implementation specification.*

```
mkdocs.yml
  theme: material
    palette: Fraunhofer blue primary, orange accent
    logo: docs/assets/matolab.svg
    favicon: docs/assets/favicon.svg
    features:
      - navigation.tabs
      - navigation.indexes
      - content.code.copy
  markdown_extensions:
    - pymdownx.superfences (Mermaid blocks via ``` mermaid ```)
    - pymdownx.tabbed
    - admonition
  extra_css: [stylesheets/extra.css]

GitHub Actions (docs.yml)
  on: push to main
  steps:
    - checkout
    - setup-python 3.12
    - pip install -r requirements-docs.txt
    - mkdocs gh-deploy --force

```

## Implementation Units

- [ ] **Unit 1: MkDocs project scaffolding**

**Goal:** Working MkDocs site that builds locally and applies Mat-O-Lab branding

**Requirements:** R1a, R2

**Dependencies:** None

**Files:**
- Create: `mkdocs.yml`
- Create: `requirements-docs.txt`
- Create: `docs/stylesheets/extra.css`
- Create: `docs/assets/matolab.svg` (copy from OrgSite)
- Create: `docs/assets/favicon.svg` (copy from OrgSite)

**Approach:**
- `mkdocs.yml` sets `site_name: Mat-O-Lab DataStack`, `repo_url` pointing to `Mat-O-Lab/DataStack` (the canonical pipeline code repo — distinct from this docs repo, Mat-O-Lab/DataPipeline, which is what GitHub Pages deploys from), and `theme.name: material`
- Enable features: `navigation.tabs`, `navigation.indexes`, `content.code.copy`, `navigation.top`
- Enable Mermaid via `pymdownx.superfences` with `mermaid` custom fence; **do not** add `mkdocs-mermaid2` to plugins
- `extra.css` overrides Material CSS custom properties with Fraunhofer palette: primary `#005B7F`, accent `#F58220`, dark-mode bg `#222222`, dark-mode text `#A6BBC8`
- `requirements-docs.txt`: `mkdocs-material>=9.0`, no additional plugins needed
- SVG assets: fetch raw from `Mat-O-Lab/OrgSite` repo (`assets/images/logos/matolab.svg`, `favicon.svg`)

**Test scenarios:**
- Happy path: `mkdocs build` completes without errors and `site/` is created
- Happy path: `mkdocs serve` renders the home page with Mat-O-Lab logo visible in the header
- Happy path: Primary nav color matches `#005B7F` Fraunhofer blue
- Edge case: Mermaid fence block in a test page renders as a diagram, not raw text

**Verification:**
- `mkdocs build` exits 0 with no warnings
- Logo and favicon visible in browser; primary color is Fraunhofer blue
- A test Mermaid block renders correctly

---

- [ ] **Unit 2: GitHub Actions CI/CD deploy**

**Goal:** Site auto-deploys to GitHub Pages on every push to main

**Requirements:** R1b

**Dependencies:** Unit 1 (mkdocs.yml must exist); GitHub Pages must be enabled on the repo with source set to `gh-pages` branch (Settings → Pages — one-time manual setup required before first deploy)

**Files:**
- Create: `.github/workflows/docs.yml`

**Approach:**
- Trigger: `on: push: branches: [main]`
- Steps: `actions/checkout@v4` → `actions/setup-python@v5` (3.12) → `pip install -r requirements-docs.txt` → `mkdocs gh-deploy --force`
- Grant `contents: write` permissions in the workflow (`mkdocs gh-deploy --force` pushes to gh-pages branch directly; `pages: write` is not needed for this approach)
- GitHub Pages source must be set to `gh-pages` branch in repo settings (note this in a comment in the workflow file)

**Test scenarios:**
- Happy path: push to main triggers workflow, `gh-pages` branch updated, site accessible at GitHub Pages URL
- Error path: build failure (e.g., broken Mermaid) causes workflow to fail and not deploy broken site

**Verification:**
- GitHub Actions workflow completes green on push to main
- Site is accessible at `https://mat-o-lab.github.io/DataPipeline/` (or equivalent Pages URL)

---

- [ ] **Unit 3: Full site structure and stub pages**

**Goal:** All planned pages exist in the nav, stubs render cleanly

**Requirements:** R3, R6

**Dependencies:** Unit 1

**Files:**
- Create: all `.md` files listed in Output Structure above that are not yet created (wrapper pages for extractors/* and components/* — these are new files at the output paths; they link to or summarize the authoritative content in `docs/specs/`)
- Modify: `mkdocs.yml` — add complete `nav:` block
- Note: `docs/specs/` files remain authoritative. The wrapper pages at `docs/extractors/` and `docs/components/` surface that content in the published nav. Also add `exclude_docs` patterns (or frontmatter `search: exclude`) to suppress `docs/plan.md`, `docs/references.md`, `docs/brainstorms/`, `docs/plans/`, and `docs/specs/` from MkDocs build output — these are internal files not intended for the public site.

**Approach:**
- Each stub page: frontmatter with `title:` + one-sentence description + `!!! info "Coming soon"` admonition block so it renders meaningfully
- `nav:` in `mkdocs.yml` uses four top-level tabs mapping to audience entry points:
  - **Pipeline** → index, overview, default-use-case, capability-map, advanced/*
  - **Guides** → index, quickstart, author-a-mapping, standalone-apis, add-a-use-case
  - **Reference** → index, api-endpoints, configuration, data-formats
  - **Components** → extractors/* + components/*
- Index pages for each tab section serve as the audience entry point with a brief "what's in this section" intro

**Test scenarios:**
- Happy path: `mkdocs build` produces all pages in `site/` with no broken nav links
- Happy path: all four top-level tabs visible and clickable
- Edge case: pages with special characters in titles render correctly in the nav

**Verification:**
- `mkdocs build` exits 0; all nav entries resolve to real pages
- No 404s when clicking through the nav in `mkdocs serve`

---

- [ ] **Unit 4: Landing page and pipeline overview**

**Goal:** The two highest-priority narrative pages are written with real content

**Requirements:** R4

**Dependencies:** Unit 3 (page files exist)

**Files:**
- Modify: `docs/index.md`
- Modify: `docs/pipeline/overview.md`

**Approach:**

`docs/index.md` — landing page:
- Opens with the core principle (block quote): *"Create complete, consistent metadata for a non-semantic resource — then use semantic technologies (RML/YARRRML) to transform that further."*
- Use case gallery: comparison table — rows = pipeline steps (Metadata creation → YARRRML mapping → RDF output → SPARQL endpoint), columns = three resource types (CSV lab data / OMERO microscopy / SAMM automotive). Scannable for technical audiences; demonstrates ontology-agnostic nature without card-grid AI slop risk.
- Quick links to each audience entry point (tabs) — this block must appear before the fold on mobile viewports, since `navigation.tabs` collapses into the hamburger menu on narrow screens; the quick-links section is the primary audience-orientation element for mobile users
- Links to live instances: futurecarproduction.materialsdata.space, dataportal.material-digital.de
- Reference to peer-reviewed publication (doi:10.1007/s40192-023-00331-5)

`docs/pipeline/overview.md` — architecture page:
- Mermaid diagram showing full service topology: all containers in DataStack + data flows + which steps are automated vs manual. Immediately after the diagram, add a one-line note: "See the component table below for a text description of this topology." — text fallback for screen readers and high-contrast rendering environments.
- Component table (same data as `docs/specs/datastack.md` but in narrative context)
- Two transformation paths side-by-side: YARRRML/RML direct vs two-stage YARRRML + SPARQL INSERT
- Resource type entry-point table linking to extractor pages

Source material: `docs/specs/datastack.md`, `docs/specs/samm-idta-pipeline-patterns.md`, `docs/references.md`

**Test scenarios:**
- Happy path: Mermaid diagram on overview.md renders without errors
- Happy path: all links in index.md resolve (live instance URLs open, publication DOI resolves)
- Edge case: page renders correctly on mobile viewport width (Material responsive by default)

**Verification:**
- Both pages render in `mkdocs serve` with no broken links or Mermaid errors
- Mermaid diagram is readable and correctly represents the pipeline topology
- Core principle quote is visually prominent on the landing page

---

## System-Wide Impact

- **Interaction graph:** GitHub Actions on push to main — if docs build fails, CI fails and nothing deploys; this blocks main if a broken Mermaid block is merged
- **Error propagation:** Build failures surface in Actions log; local `mkdocs build` catches them before push
- **Unchanged invariants:** `docs/specs/` files remain the authoritative spec source; the MkDocs site consumes but does not replace them
- **API surface parity:** None — this is a static site, no API surface introduced

## Risks & Dependencies

| Risk | Mitigation |
|---|---|
| Mermaid rendering breaks silently in CI | Test a Mermaid block during Unit 1; add to verification checklist |
| GitHub Pages not enabled on repo | Note in CI workflow comments; one-time manual setup required |
| `navigation.instant` conflicts with Mermaid JS initialization | Test during scaffold; disable if needed (noted in Unit 1 deferred questions) |
| OrgSite SVG assets change or become unavailable | Download once, commit to this repo — no runtime dependency on OrgSite |
| Update workflow agents produce conflicting edits to the same spec file | Deferred risk — addressed in follow-on plan when update-workflow.js is built |

## Documentation / Operational Notes

- GitHub Pages source must be set to `gh-pages` branch in repo settings before first deploy
- `BACKGROUNDJOBS_API_TOKEN` note from DataStack does not apply here — this is a static site
- Update workflow (R5) is deferred to a separate follow-on plan; the component table with spec_file / source_repo / api_url mappings is preserved in the Deferred section above

## Sources & References

- **Origin document:** [docs/brainstorms/documentation-site-requirements.md](docs/brainstorms/documentation-site-requirements.md)
- Spec files: `docs/specs/` (10 files)
- Content plan: `docs/plan.md`
- All external URLs: `docs/references.md`
- Fraunhofer palette: `Mat-O-Lab/ckanext-matolabtheme`, `Mat-O-Lab/OrgSite`
- MkDocs Material docs: https://squidfunk.github.io/mkdocs-material/
