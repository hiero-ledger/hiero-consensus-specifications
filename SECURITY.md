# Security Policy

## Supported Versions

| Version | Supported |
| --- | --- |
| `main` | ✅ |

## Reporting a Vulnerability

We take the security of the Hiero Consensus Standards (HCS) ecosystem seriously. If you discover a security issue related to:

- These specifications (ambiguities that could lead to unsafe implementations), or
- Reference implementations (Standards SDK),

please follow these steps:

1. **Do not** disclose the issue publicly until it has been addressed by the maintainers.
2. Email a detailed report to support@hashgraphonline.com.
3. Include:
   - Description of the issue
   - Steps to reproduce (if applicable)
   - Potential impact
   - Suggested fixes/mitigations (if any)

We will acknowledge receipt within 48 hours and aim to provide an initial assessment within 72 hours.

## Security Best Practices

When implementing these specifications:

1. Prefer the reference implementations in the Standards SDK where available: https://hol.org/docs/libraries/standards-sdk/overview
2. Validate all external inputs and handle errors explicitly
3. Keep secrets (keys/tokens) out of source control and logs
4. Regularly audit dependencies and update to patched versions
5. Follow the principle of least privilege for operational keys and permissions

## Security Updates

Security-related updates to the Standards SDK are tracked in its releases and changelogs. Specification clarifications that impact security SHOULD be called out clearly in PR descriptions and review notes.
