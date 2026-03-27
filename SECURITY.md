# Hiero, an LF Decentralized Trust Project Security Policy

## About this document

This document defines how security vulnerability reporting is handled in Hiero, an [LF Decentralized Trust Project](https://www.lfdecentralizedtrust.org/). The approach aligns with the [LF Decentralized Trust Security Policy](https://lf-decentralized-trust.github.io/governance/governing-documents/SECURITY-POLICY). Please review that document to understand the basis of the security reporting for Hiero.

This vulnerability policy borrows heavily from the recommendations of the OpenSSF Vulnerability Disclosure working group. For up-to-date information on the latest recommendations related to vulnerability disclosures, please visit the [GitHub of that working group](https://github.com/ossf).

If you are already familiar with the security policies of Hiero, and ready to report a vulnerability, please jump to [Report Intakes](#report-intakes).

## Outline

This document has the following sections:

- [About this document](#about-this-document)
- [Outline](#outline)
- [What Is a Vulnerability Disclosure Policy?](#what-is-a-vulnerability-disclosure-policy)
- [Security Team](#security-team)
- [Report Intakes](#report-intakes)
- [CNA/CVE Reporting](#cnacve-reporting)
- [Embargo List](#embargo-list)
- [GitHub Security Advisories](#github-security-advisories)
- [Private Patch Deployment Infrastructure](#private-patch-deployment-infrastructure)

## What Is a Vulnerability Disclosure Policy?

No piece of software is perfect. All software (at least, all software of a certain size and complexity) has bugs. In open source development, members of the community or the public find bugs and report them to the project. A vulnerability disclosure policy explains how this process functions from the perspective of the project.

This vulnerability disclosure policy explains the rules and guidelines for Hiero. It is intended to act as both a reference for outsiders–including both bug reporters and those looking for information on the project’s security practices–as well as a set of rules that maintainers and contributors have agreed to follow.

## Security Team

The current Hiero security team is:

| Name                     | Email ID               | Discord ID | Area/Specialty            |
|--------------------------|------------------------|------------|---------------------------|
| Hart Mongomery (LFDT)    | <>                     | <>         | Temp Security Advisor     |
| Jessica Gonzalez (LFDT)  | <>                     | <>         | Community Architect       |
| Alex Popowycz            | <>                     | <>         | Security Advisor          |
| Roger Barker             | <>                     | <>         | Project Maintainer        |
| Richard Bair             | <>                     | <>         | Project Maintainer        |
| Nathan Klick             | <>                     | <>         | Security Advisor          |

The security team for Hiero must include at least three project Maintainers that agree to carry out the following duties and responsibilities. Members are added and removed from the team via approved Pull Requests to this repository. For additional background into the role of the security team, see the [People Infrastructure](https://lf-decentralized-trust.github.io/governance/governing-documents/SECURITY-POLICY#people-infrastructure) section of the LF Decentralized Trust Security Policy.

### Responsibilities:

1. Acknowledge the receipt of vulnerability reports to the reporter within 2 business days.
2. Assess the issue. Engage with the reporter to ask any outstanding questions about the report and how to reproduce it. If the report was received by email and may be a security vulnerability, open a GitHub Security Advisory on the repository to manage the report. If the report is not considered a vulnerability, then the reporter should be informed and this process can be halted. If the report is a regular bug (but not a security vulnerability), the reporter should be informed (if necessary) of the regular process for reporting issues.
3. Some issues may require more time and resources to correct. If a particular report is complex, discuss an embargo period with the reporter during which time the report will not be publicly disclosed. The embargo period should be negotiated with the reporter.
4. If necessary, create a private patch development infrastructure for the issue by emailing the [LF Decentralized Trust Community Architects](mailto:community-architects@lfdecentralizedtrust.org).
5. Request a CVE for the issue (see the [CNA/CVE Reporting](#cnacve-reporting) section).
6. Decide a date for the public release of the vulnerability report, the date the embargo period ends.
7. If applicable, notify members of the embargo list of the vulnerability, upcoming patch and release, as described above.
8. Publish a new (software) release in which the vulnerability is addressed.
9. Publicly disclose the issue within 48 hours after the release via a GitHub security advisory (see the [(GitHub) Security Advisories](#github-security-advisories) section for details).

## Report Intakes

Do not publicly disclose vulnerablities.

For LFDT or Community Projects

Hiero has the following ways to submit security vulnerabilities. While the security team members will do their best to respond to bugs disclosed in all possible ways, it is encouraged for bug finders to report through the following approved channels:

Open a GitHub security vulnerability report: Open a draft security advisory on the “Security” tab of this GitHub repository. See [GitHub Security Advisories](https://docs.github.com/en/code-security/security-advisories) to learn more about the security infrastructure in GitHub.

For Hedera repositories details on the bug bounty program can be found at: [Hedera Bug Bounty Program](https://hedera.com/bounty).

For general questions, excluding security reports, please email the [Hiero Security Team](mailto:hiero@lists.lfdecentralizedtrust.org).

## CNA/CVE Reporting

Hiero maintains a list of Common Vulnerabilities and Exposures (CVE) and uses GitHub as its CVE numbering authority (CNA) for issuing CVEs.

## Embargo List

Hiero maintains a private embargo list. If you wish to be added to the embargo list, please email the [Hiero Security Team](mailto:hiero@lists.lfdecentralizedtrust.org), including the project name (Hiero) and reason for being added to the embargo list. Requests will be assessed by the Hiero security team in conjunction with the appropriate LF Decentralized Trust Staff, and a decision will be made to accommodate or not the request.

For more information about the embargo list, please see the Embargo List section of the [LF Decentralized Trust Security Policy](https://lf-decentralized-trust.github.io/governance/governing-documents/security.html#embargo-list).

## GitHub Security Advisories

Hiero uses [GitHub Security Advisories](https://docs.github.com/en/code-security/security-advisories) to manage the public disclosure of security vulnerabilities.

## Private Patch Deployment Infrastructure

In creating patches and new releases that address security vulnerabilities, Hiero uses the private development features of GitHub for security vulnerabilities. [GitHub](https://docs.github.com/en/code-security) has extensive documentation about these features.
