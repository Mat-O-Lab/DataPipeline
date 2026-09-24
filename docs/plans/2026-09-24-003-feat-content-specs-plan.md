---
title: "feat: Per-page content specification JSONs for documentation authoring"
type: feat
status: active
date: 2026-09-24
origin: docs/plans/2026-09-24-002-feat-content-authoring-plan.md
---

# feat: Per-page content specification JSONs for documentation authoring

## Overview

Create one JSON authoring brief per documentation page in `docs/specs/content/`. Each brief defines: target persona, narrative arc, exact reference URLs, required inline assets, reproducible walkthrough commands, and a quality gate (done-when checklist). These specs drive authoring — read the spec, fetch the URLs, write the page.

## Problem Frame

The content authoring plan (2026-09-24-002) defines *what* each page must achieve but does not encode the exact reference material (URLs, example files, inline snippets) needed to author it to the IOFMaterialsTutorial depth standard. Without per-page specs, each authoring session must rediscover which URLs to use, which examples are real vs. illustrative, and what the quality gate is.

*(see origin: docs/plans/2026-09-24-002-feat-content-authoring-plan.md)*

## Depth Standard

**IOFMaterialsTutorial benchmark:** Every curl command must be reproducible as-is using real public URLs. Real API responses shown at each step. No synthesized examples without explicit `# illustrative example` label.

## Scope

17 content spec JSON files in `docs/specs/content/`, one per documentation page:

| File | Page | Status |
|---|---|---|
| `index.json` | index.md | partial |
| `pipeline-overview.json` | pipeline/overview.md | partial |
| `pipeline-default-use-case.json` | pipeline/default-use-case.md | **authored** |
| `pipeline-capability-map.json` | pipeline/capability-map.md | **authored** |
| `pipeline-advanced-samm-catena-x.json` | pipeline/advanced/samm-catena-x.md | stub |
| `pipeline-advanced-idta-aas.json` | pipeline/advanced/idta-aas.md | stub |
| `guides-quickstart.json` | guides/quickstart.md | partial |
| `guides-author-a-mapping.json` | guides/author-a-mapping.md | **authored** |
| `guides-standalone-apis.json` | guides/standalone-apis.md | **authored** |
| `guides-add-a-use-case.json` | guides/add-a-use-case.md | partial |
| `reference-api-endpoints.json` | reference/api-endpoints.md | **authored** |
| `reference-configuration.json` | reference/configuration.md | partial |
| `reference-data-formats.json` | reference/data-formats.md | partial |
| `extractors-csvtocsvw.json` | extractors/csvtocsvw.md | **authored** |
| `extractors-omeroextractor.json` | extractors/omeroextractor.md | partial |
| `extractors-openbismantic.json` | extractors/openbismantic.md | partial |
| `extractors-ontop.json` | extractors/ontop.md | stub |

## Content Spec JSON Schema

Each spec file contains:

```json
{
  "page": "relative/path.md",
  "title": "...",
  "status": "authored | partial | stub",
  "primary_persona": "P1 | P2 | P3 | P4",
  "secondary_personas": [],
  "narrative_arc": {
    "enters_knowing": "...",
    "leaves_knowing": "...",
    "steps": ["..."]
  },
  "reference_material": {
    "spec_files": ["docs/specs/*.json"],
    "example_urls": {},
    "live_api_calls": []
  },
  "inline_assets_required": [
    {"type": "json|yaml|bash|turtle|mermaid|table|prose", "description": "...", "status": "done|needed|partial"}
  ],
  "walkthrough_requirements": [
    {"step": "...", "command": "...", "expected": "..."}
  ],
  "quality_gate": ["done-when checklist items"]
}
```

## Authoring Workflow (How to Use These Specs)

For each page to author or deepen:

1. **Read spec** — `docs/specs/content/<page-slug>.json`
2. **Read component specs** — all files listed in `reference_material.spec_files`
3. **Fetch reference URLs** — run `live_api_calls`, fetch `example_urls` values
4. **Follow narrative_arc.steps** in order — each step maps to a section
5. **Embed inline_assets_required** — fetch from live sources or local fixtures; label `# illustrative example` if synthesized
6. **Run walkthrough_requirements** — each command must produce `expected` output; include command + response inline
7. **Check quality_gate** before marking done
8. **Run** `mkdocs build --strict` — must exit 0

## Persona Reference

| ID | Role | Avoids | Entry points |
|---|---|---|---|
| P1 | Researcher / data manager | Docker, API jargon | index, default-use-case, capability-map |
| P2 | Data engineer / DevOps | Ontology theory | quickstart, configuration |
| P3 | Ontology engineer | Deploy instructions | author-a-mapping, api-endpoints, data-formats |
| P4 | Developer / integrator | Pipeline narrative | standalone-apis, add-a-use-case, components |

## Priority Order for Authoring

1. `guides/quickstart.md` — Must; P2 onboarding, most blocking for new deployments
2. `index.md` — Must; landing page, use case gallery + proof of production missing
3. `pipeline/advanced/samm-catena-x.md` — Should; full spec available in samm-idta-pipeline-patterns.json
4. `reference/configuration.md` — Should; all config keys in spec JSONs, just needs table format
5. `reference/data-formats.md` — Could; inline format examples needed
6. `guides/add-a-use-case.md` — Could; decision tree + configuration table
7. `extractors/omeroextractor.md` — Could; local test fixtures available
8. `pipeline/overview.md` — Could; needs master Mermaid diagram
9. All other stubs — Later

## Key Technical Decisions

- **Spec files are JSON, not Markdown** — machine-readable, queryable, diff-friendly. Content pages reference specs but do not replace them.
- **Status field tracks authoring progress** — `authored` = quality gate passed, `partial` = content roughed in, `stub` = placeholder only
- **local_test_fixtures** field in extractor specs — OmeroExtractor has real fixtures at `/home/hanke/OmeroExtractor/tests/`; use these when no public deployment exists
- **Illustrative example policy** — synthesized examples allowed only when live source unavailable; must be labeled `# illustrative example`

## Risks

| Risk | Mitigation |
|---|---|
| Live API responses become stale | Version comments in inline examples (e.g. `# CSVToCSVW v1.3.5`); update when specs sweep runs |
| Public services down during authoring | Use pre-computed examples from GitHub raw (example2-metadata.json, measurements-map.yaml, etc.) |
| OmeroExtractor no public deployment | Use local test fixtures at /home/hanke/OmeroExtractor/tests/ |
| OpenBISmantic no public deployment | Placeholder URLs with clear label; document deployment steps from spec |

## Sources

- Origin plan: `docs/plans/2026-09-24-002-feat-content-authoring-plan.md`
- Requirements: `docs/brainstorms/documentation-site-requirements.md`
- Component specs: `docs/specs/*.json` (all 10 files)
- Live instances: `https://futurecarproduction.materialsdata.space`, `https://dataportal.material-digital.de`
- Public microservices: csvtocsvw.matolab.org, maptomethod.matolab.org, rdfconverter.matolab.org
- Local repos: /home/hanke/CSVToCSVW, /home/hanke/MapToMethod, /home/hanke/RDFConverter, /home/hanke/OmeroExtractor, /home/hanke/OpenBISmantic
