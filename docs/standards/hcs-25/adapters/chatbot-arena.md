# HCS-25 (Adapter): Chatbot Arena (Informative)

## Purpose

Expose an independent preference-style model signal as a normalized component.

## Contribution

- Adapter id: `chatbot-arena`
- Contribution mode: `scoped`
- Suggested weight: `6`
- Typical applicability: model catalogs where an external preference leaderboard is available

## Inputs (reference)

See `../signals/chatbot-arena.md`.

## Output components

- `chatbot-arena.score` in `[0,100]`

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:A4ESM82SuJswkaEjd2ntyMyhY6o3HKMuW2JzRFVg7dNGdmKSwtX2DpbptjW7YrnEvs`
