# Contributing to the Fujitsu D3313-E1 BIOS Recovery Research Project

Thank you for considering a contribution to this project.

This repository documents independent technical research into BIOS recovery, firmware analysis, hardware behavior, machine-specific OEM data, and platform-identity restoration on the Fujitsu FUTRO S930 / D3313-E1 platform.

Contributions are welcome when they improve:

- technical accuracy;
- reproducibility;
- documentation;
- source attribution;
- hardware compatibility information;
- firmware-structure understanding;
- tooling;
- privacy and safety;
- or the quality of the research record.

Because this project deals with firmware, hardware identity, potentially destructive procedures, and third-party intellectual property, contributions require more care than a typical documentation repository.

---

# 1. Before contributing

Please review:

- [README.md](README.md)
- [README.hu.md](README.hu.md)
- [SOURCES.md](SOURCES.md)
- [DISCLAIMER.md](DISCLAIMER.md)
- [DISCLAIMER.hu.md](DISCLAIMER.hu.md)
- [SECURITY.md](SECURITY.md)
- [SECURITY.hu.md](SECURITY.hu.md)
- [LICENSE](LICENSE)

These documents define the project's:

- research scope;
- evidence standards;
- safety boundaries;
- privacy rules;
- licensing structure;
- third-party-material boundaries;
- and security-reporting procedures.

---

# 2. Appropriate contributions

Useful contributions may include:

- corrections to technical inaccuracies;
- reproducible measurements;
- additional D3313-series hardware observations;
- firmware-structure analysis;
- UEFI Firmware Volume or FFS findings;
- recovery-procedure observations;
- sanitized screenshots or photographs;
- documentation improvements;
- translation corrections;
- source and citation improvements;
- reproducibility testing;
- privacy improvements;
- scripts or utilities supporting the research;
- analysis of differences between D3313 revisions;
- confirmation or falsification of existing hypotheses.

Negative results are also valuable.

A well-documented failed experiment may prevent others from repeating an unsafe or unproductive procedure.

---

# 3. Evidence standards

The project distinguishes between different levels of evidence.

When contributing technical information, please make clear whether the information is:

### `[OFFICIAL]`

Derived from manufacturer documentation, official firmware, vendor utilities, standards documentation, or another authoritative primary source.

### `[OBSERVED]`

Directly measured, captured, read, logged, extracted, or reproduced on actual hardware or software.

### `[INFERRED]`

A conclusion supported by available evidence but not directly measured.

### `[REPORTED HISTORY]`

Information describing something that happened before the controlled investigation and cannot now be independently verified.

### `[OPEN QUESTION]`

A hypothesis or unresolved behavior requiring further investigation.

Do not present inference or speculation as direct observation.

---

# 4. Reproducibility

Where practical, technical claims should include enough information for another researcher to reproduce or independently evaluate the result.

Useful supporting information may include:

- exact board revision;
- hardware model;
- firmware version;
- flash-chip model;
- software/tool version;
- command used;
- relevant output;
- cryptographic hash;
- offset or address range;
- test conditions;
- whether power was connected or disconnected;
- jumper position;
- number of repeated measurements;
- photographs or screenshots;
- comparison methodology.

For firmware acquisition, duplicate independent reads and cryptographic verification are strongly preferred.

A single apparently successful SPI read should not automatically be treated as authoritative evidence.

---

# 5. Firmware contributions

## Do not upload personalized firmware images publicly

Raw firmware may contain:

- serial numbers;
- UUIDs;
- PC IDs;
- MAC addresses;
- HDCP-related information;
- OEM manufacturing data;
- licensing information;
- device-specific security information;
- other machine-specific identifiers.

Do **not** submit an unredacted personalized SPI dump in:

- a commit;
- a pull request;
- a public issue;
- a GitHub Discussion;
- a release;
- or any other public repository location.

Where firmware evidence is necessary, prefer:

- cryptographic hashes;
- sanitized excerpts;
- structural descriptions;
- redacted hexdumps;
- offsets;
- size information;
- block-level comparisons;
- reproducible extraction procedures.

If a complete personalized firmware image is genuinely required to investigate a security issue, use the private vulnerability-reporting process described in [SECURITY.md](SECURITY.md).

---

# 6. Third-party firmware and copyrighted material

Do not contribute third-party material unless you have the legal right to do so.

Examples include:

- Fujitsu firmware;
- BIOS update packages;
- vendor utilities;
- Realtek software;
- AMD software;
- AMI components;
- proprietary drivers;
- copyrighted manuals;
- third-party source code;
- commercial diagnostic tools.

The fact that third-party material is useful for research does not automatically make redistribution lawful.

Whenever possible, provide:

- an official source link;
- a citation;
- a cryptographic hash;
- a filename;
- a version number;
- instructions for obtaining the material from the rights holder.

Third-party materials are not relicensed under the repository's CC BY-NC-SA 4.0 license merely because they are discussed or referenced here.

---

# 7. Privacy requirements

Before contributing screenshots, photographs, logs, firmware excerpts, or diagnostic output, check for sensitive information.

Remove or redact unnecessary:

- serial numbers;
- UUIDs;
- full MAC addresses;
- PC IDs;
- usernames;
- local filesystem paths;
- email addresses;
- IP addresses where unnecessary;
- authentication tokens;
- API keys;
- QR codes;
- barcodes;
- EXIF location data;
- HDCP or other machine-specific secrets.

If a machine identifier is technically relevant but should not be published, use a placeholder such as:

```text
<REDACTED_FUJITSU_MAC>
```

or:

```text
<REDACTED_SERIAL_NUMBER>
```

---

# 8. Security-sensitive findings

Do not open a public issue for an exploitable vulnerability or accidental credential disclosure.

Use GitHub Private Vulnerability Reporting as described in:

[SECURITY.md](SECURITY.md)

Examples of information that should normally be reported privately first:

- exploitable project-code vulnerabilities;
- exposed credentials;
- authentication tokens;
- private keys;
- sensitive personalized firmware;
- vulnerabilities where immediate public disclosure would materially increase risk.

---

# 9. Hardware safety

Some procedures documented by this project can permanently damage hardware.

Examples include:

- SPI flash programming;
- BIOS modification;
- EEPROM writes;
- OTP/eFuse programming;
- soldering;
- desoldering;
- hot-air rework;
- direct voltage injection;
- board-level electrical testing.

When contributing a procedure that can modify hardware or firmware, clearly document:

- prerequisites;
- power state;
- voltage requirements;
- irreversible steps;
- backup requirements;
- expected result;
- failure modes;
- recovery path, if one exists.

Irreversible operations should be clearly identified.

Do not describe a destructive or irreversible operation as routine when a less invasive method is available.

---

# 10. Authorized research only

Contributions involving security analysis, reverse engineering, networking, system access, or digital forensics must concern systems and data that the contributor is authorized to examine.

Do not submit material derived from unauthorized:

- system access;
- account access;
- network interception;
- credential use;
- data extraction;
- monitoring;
- exploitation.

This project documents legitimate repair, interoperability, preservation, defensive analysis, and authorized research.

---

# 11. Documentation style

Documentation should prioritize accuracy over certainty.

Prefer wording such as:

> The observed result suggests...

when the mechanism has not been directly demonstrated.

Prefer:

> On the investigated D3313-E1 specimen...

instead of:

> All D3313-E1 boards...

unless multiple systems or authoritative documentation justify the broader statement.

Where practical:

- separate observation from interpretation;
- preserve exact command output when technically important;
- identify uncertainty;
- identify failed experiments;
- specify hardware revision;
- cite primary sources;
- avoid unsupported generalization.

---

# 12. English and Hungarian documentation

The project maintains major documentation in both English and Hungarian where practical.

Canonical public-facing pairs currently include:

- `README.md` / `README.hu.md`
- `DISCLAIMER.md` / `DISCLAIMER.hu.md`
- `SECURITY.md` / `SECURITY.hu.md`
- `CONTRIBUTING.md` / `CONTRIBUTING.hu.md`

A contribution does not have to include both languages when the contributor cannot reasonably provide an accurate translation.

However, substantial changes to one language version should be clearly identified so the corresponding version can later be synchronized.

Machine translation may be used as a starting point, but technical terminology and meaning should be reviewed manually before publication.

---

# 13. Issues

A public GitHub issue is appropriate for:

- documentation errors;
- broken links;
- questions about reproducibility;
- non-sensitive hardware observations;
- compatibility reports;
- proposed research directions;
- feature requests;
- non-security bugs.

Before opening a new issue, check whether the topic is already documented or discussed.

A useful issue title should identify the subject clearly.

Example:

```text
D3313-E1 R1.18: OEM block differs from documented 0x440000–0x44FFFF region
```

Rather than:

```text
BIOS problem
```

---

# 14. Pull requests

Pull requests should be focused and reviewable.

Prefer one coherent change per pull request where practical.

A pull request should explain:

- what changed;
- why the change is needed;
- how the result was verified;
- which hardware or firmware version was involved;
- whether sensitive information was removed;
- whether third-party material is included;
- whether documentation in another language also requires updating.

Large firmware-analysis contributions should include a concise summary of the evidence chain.

---

# 15. Commit messages

Use concise, descriptive commit messages.

Good examples:

```text
Document D3313-E1 R1.18 OEM region comparison
```

```text
Correct RTL8168G eFuse interpretation
```

```text
Add second-board SPI block comparison
```

```text
Update Hungarian recovery documentation
```

Avoid vague messages such as:

```text
fix
```

```text
update
```

```text
changes
```

when a more informative description is possible.

---

# 16. Documentation licensing

Unless explicitly stated otherwise, eligible original research and documentation in this repository is distributed under:

**Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International  
(CC BY-NC-SA 4.0)**

By submitting original documentation, research text, original diagrams, original photographs, original measurements, or similar copyrightable material for inclusion in CC BY-NC-SA 4.0-licensed project documentation, you agree that the accepted contribution may be distributed as part of that documentation under **CC BY-NC-SA 4.0**, unless another arrangement is explicitly agreed before acceptance.

You must have the legal right to submit the material.

You retain copyright in your original contribution unless you explicitly agree otherwise.

Submitting a contribution does not transfer ownership of your copyright to the repository owner.

See:

- [LICENSE](LICENSE)
- [DISCLAIMER.md](DISCLAIMER.md)

---

# 17. Software and source-code licensing

Creative Commons licenses are not intended to serve as the primary software license for source code.

The repository's `LICENSE` therefore does **not automatically mean that all source code is CC BY-NC-SA 4.0 licensed**.

Where a software component, script, or directory carries a separate software license, that license governs the relevant code.

Until a specific software license is established for a new code component:

> **Please open an issue before submitting substantial new source code.**

This allows the applicable license to be agreed before code is incorporated into the project.

Do not assume that submitting source code automatically places it under the repository's documentation license.

---

# 18. Contributor rights and representations

By submitting a contribution, you represent that:

- you created the contribution yourself, **or**
- you have sufficient rights and permission to submit it;
- the contribution does not knowingly contain material you are prohibited from redistributing;
- sensitive information has been removed where appropriate;
- third-party material is identified;
- applicable licenses are respected.

Do not copy code, documentation, photographs, firmware, diagrams, or other protected content from another source without appropriate rights or permission.

---

# 19. Attribution

Significant external research, prior work, documentation, and technical sources should be attributed.

Relevant references may be added to:

[SOURCES.md](SOURCES.md)

Where possible, prefer:

1. primary manufacturer documentation;
2. standards documentation;
3. upstream software documentation;
4. original research sources;
5. community discussions as supporting evidence.

Community reports can be valuable but should not automatically be treated as equivalent to primary technical documentation.

---

# 20. Research disagreements

Technical disagreement is welcome when supported by evidence.

If a contribution challenges an existing conclusion, provide the contradictory observation rather than merely stating that the conclusion is wrong.

For example:

> On a second D3313-E1 board running R1.18, two independent SPI reads place the populated MAC structure at a different offset.

is more useful than:

> The documented offset is wrong.

Existing conclusions should be revised when stronger evidence becomes available.

---

# 21. Experimental work

Experiments that intentionally alter firmware state should, where practical, begin from a recoverable state.

Recommended precautions include:

- verified original SPI backup;
- multiple independent reads;
- cryptographic hashes;
- documented flash-chip identification;
- known recovery procedure;
- minimal write scope;
- post-write verification.

When testing a hypothesis, prefer changing the smallest possible region necessary to answer the research question.

---

# 22. Repository baseline

The first stable documented research baseline is tagged:

```text
v1.0.0
```

Contributors performing experimental work are encouraged to identify the baseline or commit from which their observations derive.

This helps distinguish later experimental state from the documented recovery baseline.

---

# 23. What may be rejected

A contribution may be rejected if it:

- contains unredacted sensitive information;
- redistributes third-party material without sufficient rights;
- contains unsupported technical claims presented as fact;
- cannot be reasonably reviewed;
- introduces unsafe instructions without adequate warning;
- encourages unauthorized access or testing;
- conflicts with the repository's legal or security policies;
- duplicates existing material without adding value;
- attempts to relicense third-party content;
- contains malicious code;
- intentionally weakens privacy or security protections.

Rejection does not necessarily mean the underlying research question is unimportant.

It may simply require a safer, better documented, or legally appropriate form.

---

# 24. Suggested contribution workflow

For a straightforward documentation correction:

```text
Fork / branch
   ↓
make the change
   ↓
review for sensitive data
   ↓
commit
   ↓
open pull request
```

For substantial firmware or hardware research:

```text
preserve original evidence
        ↓
document hardware and firmware versions
        ↓
perform reproducible measurements
        ↓
verify results
        ↓
remove sensitive information
        ↓
separate observation from inference
        ↓
cite relevant prior work
        ↓
submit focused pull request
```

For security-sensitive findings:

```text
do not publish publicly
        ↓
use GitHub Private Vulnerability Reporting
```

---

# 25. Questions

For ordinary contribution questions, open a public GitHub issue.

For security-sensitive matters, use the private vulnerability-reporting process.

For legal, licensing, or reuse questions, consult:

- [LICENSE](LICENSE)
- [DISCLAIMER.md](DISCLAIMER.md)
- [DISCLAIMER.hu.md](DISCLAIMER.hu.md)

---

Thank you for helping improve the technical accuracy, reproducibility, safety, and long-term value of this research project.

---

Copyright © 2026 Bencze Péter and respective contributors for their original contributions.
