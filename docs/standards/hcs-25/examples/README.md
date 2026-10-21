# HCS-25 (Appendix): Composition Graph Examples

Worked composition graphs in the normative representation defined in
[HCS-25 § Graph Configuration](../../hcs-25.md#graph-configuration).

Each file is a complete graph object: it validates against
[§ Validation](../../hcs-25.md#validation), and its `id` is the content-derived identifier
specified in [§ Graph Identity](../../hcs-25.md#graph-identity) — the SHA-256 digest of the
graph's canonical form with `id` omitted. Recomputing the identifier from any of these files
MUST reproduce the value it carries.

Each file also carries a [`metadata`](../../hcs-25.md#metadata) object with a `title` and a
`description`. That object is informative and is excluded from the canonical form, so the
labels can be corrected or translated without changing the graph's identifier.

| Example | Demonstrates | Spec reference |
| --- | --- | --- |
| [`parity.json`](./parity.json) | A single `weightedMean` node reproducing the v1.0 composite exactly | [Test Vector 3](../../hcs-25.md#test-vector-3-a-composition-graph-reproduces-the-composite-parity) |
| [`safety-cap.json`](./safety-cap.json) | `cap` — a weak dimension governing rather than diluting | [Test Vector 4](../../hcs-25.md#test-vector-4-safety-cap-a-dimension-governs-rather-than-dilutes) |
| [`corroboration-gate.json`](./corroboration-gate.json) | `gate` — one dimension conditioning the whole score | [Test Vector 5](../../hcs-25.md#test-vector-5-gate-with-a-stale-then-unavailable-condition) |
| [`constant-floor.json`](./constant-floor.json) | `constant` + `max` — a genuine *lower* bound, which `cap` does not provide | [Operator Catalog](../../hcs-25.md#operator-catalog) |

## Why `constant-floor.json` uses `max`

`cap` imposes an **upper** bound. A lower bound is expressed with `max` against a
`constant` node — `max(quality, 40)` never returns less than `40`. Naming matters here:
`cap(20, 60)` is `20`, not `60`.

## Identifiers

| Example | `id` |
| --- | --- |
| `parity.json` | `sha256:76218d4122ea6c5e94005cc0087ea50f82526760ee89fd798e9fbad9bc669c98` |
| `safety-cap.json` | `sha256:ba1d6f33f1087bced364cc111f8792103ef5a83c63215f5e8f4492dc7ee5e876` |
| `corroboration-gate.json` | `sha256:7c6fffd0bdf4214cea1abb0e0971e738fde9d222e7bd4c37f0687ee131751ea4` |
| `constant-floor.json` | `sha256:2a01de555c5ee98a7af2cf41726dc91dd0a418c21c4d852e0f806fb6a2e43fc8` |

These are stable: the canonical form sorts object keys and omits both `id` and every
`metadata` object, so reformatting a file — reindenting it, reordering its keys — or
rewording a title does not change its identifier, while changing an operator, an input, or
a parameter does.
