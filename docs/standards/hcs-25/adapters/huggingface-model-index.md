# HCS-25 (Adapter): Hugging Face Model Index (Informative)

## Purpose

Expose Hugging Face-derived model-index and popularity signals as a normalized component.

## Contribution

- Adapter id: `huggingface-model-index`
- Contribution mode: `conditional`
- Suggested weight: `0.8`
- Typical applicability: models where a stable mapping to a Hugging Face model id exists

## Inputs (reference)

See `../signals/huggingface-model-index.md`.

## Output components

- `huggingface-model-index.score` in `[0,100]`

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:82BHmpsiwqfeUTsazhWGkxj8UaV1q7qwVcXBZMS7PfDGkibnteEajdgCsd5qQYAm4f`
