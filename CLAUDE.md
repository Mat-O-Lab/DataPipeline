# DataPipeline — Claude Instructions

## Build & quality gate

```bash
mkdocs build --strict
```

No errors or warnings = passing. Run after every edit before committing.

## Spec file maintenance field

Content spec files (`docs/specs/content/*.json`) may include a `maintenance` field listing external dependencies that can go stale:

```json
"maintenance": {
  "screenshots": [
    { "file": "docs/assets/ui-foo.png", "url": "https://...", "used_in": "guides/standalone-apis.md" }
  ],
  "live_files": [
    { "url": "https://...", "used_in": "page.md", "note": "what the snippet shows" }
  ],
  "live_urls": [
    { "url": "https://...", "used_in": "page.md", "note": "search link / portal query" }
  ]
}
```

An agent reviewing docs should check these URLs still return the expected content before marking a page as current.

---

## Periodic maintenance

These items need human review when the referenced services or data change:

### UI screenshots (docs/assets/ui-*.png)

Screenshots were captured live on 2026-09-25. Retake when the service UIs change:

| File | Service URL | Used in |
|---|---|---|
| `docs/assets/ui-csvtocsvw.png` | https://csvtocsvw.matolab.org/ | `guides/standalone-apis.md` |
| `docs/assets/ui-maptomethod.png` | https://maptomethod.matolab.org/ | `guides/standalone-apis.md` |
| `docs/assets/ui-rdfconverter.png` | https://rdfconverter.matolab.org/ | `guides/standalone-apis.md` |

To retake: use Playwright (`mcp__playwright__browser_navigate` + `mcp__playwright__browser_take_screenshot`) or any browser screenshot tool. Save as PNG to `docs/assets/` and commit.

### Live portal search links (docs/index.md, docs/pipeline/semantic-foundation.md)

Search query links point to live CKAN portals. Verify tags/queries still return results if portal datasets change significantly:

- https://futurecarproduction.materialsdata.space/dataset?q=SAMM
- https://dataportal.material-digital.de/dataset?q=tensile+tests

### Real file examples

Several pages embed snippets fetched from live public URLs. If upstream repos change these files the snippets may become stale:

| Page | Source file |
|---|---|
| `guides/author-a-mapping.md` | `BAMresearch/DF-TEM-PAW/detection_runs-map.yaml` |
| `pipeline/default-use-case.md` | `Mat-O-Lab/CSVToCSVW/examples/example2-metadata.json` |
| `pipeline/default-use-case.md` | `BAMresearch/DF-TEM-PAW/detection_runs-joined.ttl` |
| `pipeline/advanced/samm-catena-x.md` | `futurecarproduction.materialsdata.space` YARRRML + SPARQL |
| `pipeline/advanced/idta-aas.md` | `futurecarproduction.materialsdata.space` AAS YARRRML |
| `extractors/csvtocsvw.md` | Nasrabadi et al. (2023) paper figures §5.1 |
