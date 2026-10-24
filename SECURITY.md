# Security Policy

## Scope

This repository documents independent firmware-recovery, hardware-analysis, and security-related technical research concerning the Fujitsu FUTRO S930 / D3313-E1 platform.

This security policy applies to security issues involving:

- files maintained directly in this repository;
- scripts, utilities, or software developed specifically for this project;
- repository configuration;
- GitHub Actions or other automation added to this repository;
- accidental disclosure of secrets, credentials, tokens, or sensitive machine-specific data;
- vulnerabilities introduced by original project code;
- unsafe project behavior that could unexpectedly modify data or hardware beyond its documented purpose.

This policy does **not** make this repository the security contact for Fujitsu, Realtek, AMD, AMI, flashrom, Linux, Debian, or any other third-party project, manufacturer, product, firmware, or service mentioned in the documentation.

Security vulnerabilities affecting third-party products should normally be reported to the relevant vendor or upstream project.

---

## Supported versions

This project is primarily maintained as a research repository rather than as a versioned software product.

Security corrections are applied to the current state of the default branch:

| Version / branch | Security support |
| --- | --- |
| `main` | Supported |
| Historical commits | Not actively maintained |
| Forks and third-party copies | Not maintained by this project |

Where software components are later released with their own version numbers or release branches, this policy may be updated accordingly.

---

## Reporting a vulnerability

Please **do not open a public GitHub issue** for a vulnerability if the report contains:

- an exploitable security flaw;
- credentials or secrets;
- API keys or access tokens;
- private machine-specific identifiers;
- unredacted firmware data;
- personal information;
- information that could materially increase the risk of exploitation before a fix is available.

Use **GitHub Private Vulnerability Reporting** for this repository instead.

From the repository's **Security** section, use the private vulnerability-reporting function to submit the report directly to the repository maintainer.

If private reporting is unavailable for any reason, contact the repository owner through the contact information published in the repository metadata.

---

## What to include in a report

A useful vulnerability report should contain, where applicable:

- a concise description of the issue;
- the affected file, component, script, or workflow;
- the relevant commit or version;
- steps required to reproduce the issue;
- expected behavior;
- observed behavior;
- potential security impact;
- proof-of-concept information, if necessary to demonstrate the issue;
- suggested remediation, if available.

Please remove or redact unnecessary:

- passwords;
- authentication tokens;
- private keys;
- serial numbers;
- UUIDs;
- full MAC addresses;
- personalized firmware images;
- HDCP or comparable machine-specific secrets;
- personal information.

Only provide sensitive information when it is genuinely required to understand or reproduce the vulnerability.

---

## Responsible disclosure

Please allow a reasonable opportunity to investigate and, where appropriate, correct a reported vulnerability before publishing technical exploitation details.

Reports will be assessed based on:

- reproducibility;
- technical impact;
- affected project components;
- likelihood of exploitation;
- potential harm to users or hardware;
- and whether the issue is within the scope of this repository.

There is no guaranteed response-time or remediation-time service level for this independently maintained research project.

A report may be closed or redirected when it concerns a third-party product rather than code or material maintained by this repository.

---

## Research and authorization

This repository contains material related to:

- firmware analysis;
- BIOS recovery;
- SPI flash access;
- hardware diagnostics;
- reverse engineering;
- network-controller configuration;
- low-level system analysis.

Security research must be performed only on:

- hardware you own;
- systems you are authorized to test;
- data you are legally permitted to examine;
- or environments for which explicit permission has been granted.

This project does not authorize testing, access, modification, monitoring, interference, or exploitation of third-party systems without permission.

---

## Hardware safety

Some techniques documented in this repository may cause irreversible hardware or firmware damage if used incorrectly.

Examples include:

- SPI flash writes;
- BIOS modification;
- EEPROM programming;
- OTP/eFuse programming;
- soldering or desoldering;
- hot-air rework;
- board-level electrical testing.

A security report concerning an unsafe procedure should clearly distinguish between:

- an inherent and clearly documented hardware risk;
- incorrect use;
- missing documentation;
- and an unexpected project defect.

Irreversible storage such as OTP or eFuse memory requires particular caution.

---

## Sensitive firmware and machine-specific data

Firmware images may contain information that should not be publicly disclosed, including:

- serial numbers;
- UUIDs;
- PC IDs;
- MAC addresses;
- manufacturing data;
- licensing information;
- HDCP-related information;
- security configuration;
- device-specific identifiers.

Please do not attach an unredacted personalized firmware image to a public issue.

If such material is required to demonstrate a security issue, use the repository's private vulnerability-reporting channel and minimize the disclosed data.

---

## Secrets and credentials

Never intentionally commit:

- passwords;
- private keys;
- access tokens;
- API keys;
- session credentials;
- recovery codes;
- cloud credentials;
- signing secrets.

GitHub secret scanning and push protection may detect some exposed secrets, but automated detection is not a substitute for careful review.

If a credential is accidentally committed, assume that it may have been exposed and rotate or revoke it immediately.

Removing a secret from the latest commit does not necessarily remove it from repository history.

---

## Public issues that are appropriate

Public GitHub issues remain appropriate for non-sensitive matters such as:

- documentation errors;
- broken links;
- reproducibility problems that do not expose sensitive data;
- hardware compatibility observations;
- feature requests;
- clarification requests;
- non-security bugs that do not create a material security risk.

When uncertain whether something should be public, use private vulnerability reporting first.

---

## Third-party vulnerabilities

A vulnerability discovered in a third-party component mentioned by this repository should generally be reported directly to the responsible vendor or upstream maintainer.

Examples include vulnerabilities in:

- Fujitsu firmware or utilities;
- Realtek software or hardware;
- AMD components;
- AMI firmware;
- Debian or Linux;
- flashrom;
- CH341A software;
- other third-party tools.

This repository may document publicly available vulnerability information when relevant to the research, but it does not act as the authoritative security contact for those projects.

---

## Disclosure and publication

Where a vulnerability affects code maintained by this repository, security information may later be published after:

1. the issue has been reasonably investigated;
2. sensitive information has been removed;
3. an appropriate fix or mitigation has been prepared where possible;
4. disclosure no longer creates unnecessary additional risk.

Security advisories, release notes, repository documentation, or commit history may be used to document resolved issues.

---

## Legal and licensing information

This security policy does not replace the repository's legal, research, safety, or licensing notices.

See:

- [DISCLAIMER.md](DISCLAIMER.md)
- [DISCLAIMER.hu.md](DISCLAIMER.hu.md)
- [LICENSE](LICENSE)
- [README.md](README.md)
- [README.hu.md](README.hu.md)

Third-party rights and licenses remain with their respective rights holders.

---

## Contact

Preferred security-reporting channel:

**GitHub Private Vulnerability Reporting for this repository**

Repository:

`https://github.com/benczepeter001/fujitsu-d3313-bios-recovery`

For non-sensitive questions, normal GitHub issues may be used.

---

Copyright © 2026 Bencze Péter and respective contributors for their original contributions.
