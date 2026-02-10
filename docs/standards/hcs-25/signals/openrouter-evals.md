# HCS-25 (Signal): OpenRouter Category Benchmarks (Informative)

## Purpose

Fetch OpenRouter category rankings and compute a coverage-aware benchmark score for models.

## Applicability

Applied to model catalog entries in registries such as `openrouter` and `near-ai`.

## Stored fields (example schema)

Stored in `subject.metadata.additional`:

| Field | Type | Meaning |
| --- | --- | --- |
| `openrouterEvalScore` | number \| null | Normalized score `[0,100]` |
| `openrouterEvalStatus` | `ok` \| `missing` \| `low-coverage` | Coverage status |
| `openrouterEvalUpdatedAt` | ISO timestamp | Refresh time |
| `openrouterEvalSources` | string[] | Source identifiers |
| `openrouterEvalMetricsCount` | number | Count of benchmark entries used |
| `openrouterEvalCategoryCount` | number | Category count represented |
| `openrouterEvalCoverageWeight` | number | Coverage proxy (e.g., volume) |
| `openrouterEvalCategories` | string[] | Top categories for explainability |

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:A4ESM82SuJswkaEjd2ntyMyhY6o3HKMuW2JzRFVg7dNGdmKSwtX2DpbptjW7YrnEvs`
