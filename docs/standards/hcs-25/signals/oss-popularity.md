# HCS-25 (Signal): OSS Popularity (Informative)

## Purpose

Collect open-source adoption signals for software artifacts (e.g., MCP servers), combining GitHub repository popularity with package download counts.

## Applicability

Applied to MCP agents (protocol `mcp`), where repository/package metadata can be inferred or provided.

## Stored fields (example schema)

Stored in `subject.metadata.additional`:

| Field | Type | Meaning |
| --- | --- | --- |
| `githubRepo` | string \| null | Normalized `owner/repo` |
| `githubStars` | number \| null | Star count |
| `githubStarsUpdatedAt` | ISO timestamp | GitHub refresh time |
| `packageRegistry` | `npm` \| `pypi` \| null | Registry selection |
| `packageName` | string \| null | Package name |
| `packageIdentityUpdatedAt` | ISO timestamp | Refresh time for inferred package identity |
| `packageRepositoryUpdatedAt` | ISO timestamp | Refresh time for inferred repository identity (e.g., GitHub mapping) |
| `packageDownloadCount` | number \| null | Optional unified download count used by some pipelines (legacy compatibility) |
| `npmDownloads30d` | number \| null | 30-day npm downloads |
| `pypiDownloads30d` | number \| null | 30-day PyPI downloads |
| `npmDownloadsUpdatedAt` / `pypiDownloadsUpdatedAt` | ISO timestamp | Download refresh time |

## Notes (reference)

The reference implementation uses:

- GitHub API for stars,
- npmjs.org API for npm downloads, and
- pypistats.org for PyPI downloads.

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:792bH6RDGeeJZsdsTYGLD4cNrhXip64o3mEzbZZE4cqnbhf2MDGSA8uegko9ezK5wo;uid=mcp-pulsemcp-klavis-strata;registry=pulsemcp;proto=mcp;nativeId=https://www.pulsemcp.com/servers/klavis-strata`
