# Disclaimer

## Research and educational purpose

This repository documents an independent technical investigation of privately controlled hardware.

The material is provided for:

- technical research;
- firmware analysis;
- hardware preservation;
- documentation;
- reproducibility;
- education;
- and legitimate repair of hardware under the researcher's control.

It is not intended to serve as an official Fujitsu repair procedure, manufacturer service manual, certified engineering document, or guaranteed recovery method.

---

## Independent project

This project is independent and is not affiliated with, endorsed by, sponsored by, or produced by:

- Fujitsu;
- Kontron;
- Realtek;
- Macronix;
- WCH;
- AMI;
- or any other manufacturer, standards body, software project, or community referenced in this repository.

Product names, trademarks, model names, software names, and company names remain the property of their respective owners.

They are used solely for technical identification, documentation, attribution, and research reference.

---

## Experimental scope

The findings documented in this repository are based primarily on one investigated Fujitsu FUTRO S930 system equipped with a Fujitsu D3313-E1 mainboard.

Unless explicitly stated otherwise, observations from this machine should not automatically be generalized to:

- every D3313-E1 mainboard;
- every FUTRO S930 system;
- other D3313 variants;
- other Fujitsu systems;
- other Realtek controller revisions;
- other firmware revisions.

Hardware revisions, firmware versions, manufacturing configuration, component substitutions, and runtime state may materially change system behavior.

---

## Evidence classification

Where relevant, this project distinguishes between:

```text
[OFFICIAL]
[OBSERVED]
[INFERRED]
[OPEN QUESTION]
```

### `[OFFICIAL]`

Information obtained from manufacturer documentation, official firmware packages, standards, upstream software documentation, or equivalent primary sources.

### `[OBSERVED]`

Information directly measured, read, logged, extracted, reproduced, or verified on the investigated hardware.

### `[INFERRED]`

A conclusion strongly supported by multiple observations but not itself directly measured.

### `[OPEN QUESTION]`

A hypothesis, unresolved mechanism, or behavior requiring further evidence or independent reproduction.

This distinction is intended to prevent interpretation from being presented as measurement.

---

## No warranty

All information, procedures, commands, offsets, firmware observations, scripts, diagrams, configuration examples, research notes, and other material are provided:

> **AS IS**

without warranty of any kind, express or implied.

No guarantee is made that:

- a documented procedure will work on another system;
- another D3313 board will use the same firmware layout;
- offsets will remain valid across firmware revisions;
- recovery procedures will succeed;
- a particular flash device will behave identically;
- hardware will remain functional;
- machine-specific data will be preserved;
- a recovery attempt will be reversible;
- or a system restored using these methods will be suitable for production use.

---

## Risk of permanent hardware damage

Firmware modification and board-level repair can permanently damage hardware.

Possible consequences include, but are not limited to:

- an unbootable system;
- corrupted SPI firmware;
- damaged flash storage;
- damaged mainboard components;
- damaged power-delivery circuitry;
- damaged peripheral controllers;
- loss of firmware configuration;
- loss of board-specific data;
- loss of recovery capability.

Incorrect voltage, wiring, programmer configuration, clip orientation, flash-chip selection, or write operation may permanently damage a device.

Anyone attempting similar work is responsible for understanding the electrical and firmware characteristics of their own hardware.

---

## Risk of data and identity loss

Firmware devices may contain machine-specific information that is not present in generic manufacturer firmware images.

Examples may include:

- serial numbers;
- UUIDs;
- PC IDs;
- MAC addresses;
- OEM/manufacturing data;
- licensing information;
- platform-specific configuration;
- calibration values;
- cryptographic or security-related material;
- other device-specific metadata.

A generic firmware image may restore bootability without restoring the complete identity or configuration of a particular machine.

This repository therefore strongly recommends preserving verified copies of original firmware before attempting any modification.

---

## Backup methodology

A single apparently successful firmware read should not automatically be considered a trustworthy backup.

Where possible, a safer workflow is:

```text
READ
  ↓
READ AGAIN
  ↓
COMPARE
  ↓
HASH
  ↓
ARCHIVE
  ↓
ANALYZE
  ↓
ONLY THEN WRITE
```

Independent reads should be compared and cryptographically hashed.

Original dumps should be preserved unchanged.

Modified firmware images should be stored separately from original evidence.

---

## Irreversible and limited-write storage

Special caution applies to:

- eFuse;
- OTP memory;
- one-time-programmable configuration;
- security fuses;
- hardware identity fuses;
- other limited-write or irreversible storage.

Programming such storage can be permanent.

The investigation documented here deliberately avoided programming the Realtek eFuse after evidence suggested that doing so would not reproduce the original Fujitsu platform mechanism.

Users should not modify irreversible storage unless they understand:

- what is being programmed;
- how the storage behaves electrically;
- whether the operation can be reversed;
- whether the target platform originally used that storage mechanism;
- whether a less invasive recovery method exists.

---

## Minimum-change principle

This project generally follows the methodological principle:

> **Modify only the smallest firmware region that the evidence demonstrates must be changed.**

This approach is intended to reduce unnecessary modification of unrelated firmware state, including:

- runtime NVRAM;
- boot variables;
- recovery state;
- manufacturing data;
- counters;
- unknown firmware structures.

This principle reduces unnecessary intervention but does not eliminate risk.

---

## Privacy and machine-specific information

Raw firmware images should be treated as potentially sensitive technical evidence.

They may contain:

- serial numbers;
- MAC addresses;
- UUIDs;
- PC IDs;
- manufacturing identifiers;
- HDCP-related information;
- licensing information;
- platform configuration;
- security-related metadata;
- unknown machine-specific structures.

For this reason, personalized raw firmware images from the investigated system are not intentionally distributed through this repository.

---

## Cryptographic hashes and provenance

Cryptographic hashes may be published for:

- provenance;
- reproducibility;
- artifact identification;
- future comparison.

A cryptographic digest can identify whether another binary is bit-for-bit identical to a research artifact without disclosing the artifact's contents.

Publication of a hash does not mean publication of the underlying firmware image.

---

## Screenshots, photographs, logs, and media

Images, screenshots, recordings, logs, and terminal output may also contain identifying information.

Examples include:

- usernames;
- local filesystem paths;
- IP addresses;
- MAC addresses;
- serial numbers;
- UUIDs;
- device labels;
- stickers;
- QR codes;
- barcodes;
- EXIF metadata;
- GPS metadata;
- background objects or documents.

All public evidence should therefore be reviewed and sanitized before publication.

Raw media should not automatically be assumed safe for public distribution.

---

## Accidental disclosure

Despite careful review, accidental disclosure of machine-specific or personal information may still occur.

If such material is identified, responsible notification is encouraged so that the repository can be corrected.

Because Git repositories preserve history and public repositories may be cloned, forked, archived, or indexed, removal from the current repository does not guarantee that previously published information disappears from all copies.

---

## Third-party software and tools

This project references third-party software including, among others:

- flashrom;
- UEFITool;
- IMSProg;
- Linux;
- systemd;
- Realtek diagnostic utilities;
- Realtek PGTool-compatible tooling;
- community-developed utilities.

Those projects retain their own:

- copyrights;
- licenses;
- documentation;
- warranty terms;
- security policies.

Their inclusion or citation here does not imply ownership, endorsement, or redistribution rights by this project.

Users should consult the original project or manufacturer for authoritative licensing and usage information.

---

## Third-party firmware and proprietary material

This repository does not intend to redistribute proprietary manufacturer firmware except where redistribution is explicitly permitted.

Official firmware packages are referenced for:

- identification;
- comparison;
- research context;
- reproducibility.

Machine-specific firmware dumps from the investigated system are treated as research evidence and are not publicly distributed by default.

Users are responsible for obtaining firmware and software through lawful sources and complying with applicable license terms.

---

## Copyright and attribution

This project does not claim authorship of techniques, utilities, firmware formats, standards, or prior research created by others.

Relevant prior work, manufacturer documentation, software projects, and community research are credited in:

```text
SOURCES.md
```

The project's contribution is the documented application, combination, verification, and interpretation of those methods in the investigated D3313-E1 case.

---

## Responsible use

The procedures documented here are intended for hardware:

- owned by the researcher;
- controlled by the researcher;
- or legitimately entrusted to the researcher for analysis or repair.

Nothing in this repository should be interpreted as authorization to:

- modify systems without permission;
- interfere with third-party hardware;
- bypass access controls without authorization;
- obtain confidential information without authorization;
- alter hardware identity for deceptive or unlawful purposes.

The project concerns legitimate firmware recovery, preservation, and technical analysis.

---

## Security research context

Firmware-research techniques can overlap with computer-security techniques.

Their inclusion here is intended to support:

- defensive research;
- repair;
- hardware preservation;
- interoperability;
- diagnostics;
- understanding of platform behavior.

Users remain responsible for ensuring that their activities comply with applicable law, contractual obligations, and authorization boundaries.

---

## No safety certification

The procedures documented here are not safety-certified.

The repository does not certify:

- electrical safety;
- electromagnetic compliance;
- production suitability;
- regulatory compliance;
- long-term hardware reliability.

Special care is required when working with:

- powered mainboards;
- external programmers;
- exposed conductors;
- modified power supplies;
- storage devices;
- soldering or hot-air equipment.

---

## No professional engineering certification

This repository is a technical research record.

It should not be interpreted as:

- professional engineering certification;
- manufacturer service authorization;
- official repair documentation;
- safety certification;
- warranty guidance;
- legal advice.

For regulated, safety-critical, commercial, or production systems, independent professional review may be required.

---

## Experimental contradictions

A future result that contradicts a conclusion in this repository should not automatically be treated as an error by the person reproducing the work.

Possible explanations may include:

- hardware revision differences;
- firmware revision differences;
- manufacturing variation;
- undocumented configuration;
- component substitution;
- an incorrect current interpretation.

Contradictory results are valuable research evidence.

---

## Corrections and independent reproduction

Technical corrections, independent reproductions, negative results, contradictory findings, and additional evidence are welcome.

Where new evidence changes an interpretation, the repository may be updated.

The Git version history should preserve the evolution of the research record.

---

## Scope of conclusions

The strongest conclusions in this repository apply to the specific investigated D3313-E1 system unless replication demonstrates otherwise.

For example, evidence from the investigated machine supports conclusions about:

- its original SPI image;
- its OEM region;
- its Realtek eFuse state;
- its recovery-state firmware;
- its selective restoration result.

These observations should not be silently generalized to all systems in the same product family.

---

## External links

This repository contains links to:

- manufacturer documentation;
- standards;
- software projects;
- technical documentation;
- community forums;
- prior research.

External content may change, move, or disappear.

The presence of a link does not imply endorsement of every statement on the linked site.

Community sources are intentionally distinguished from manufacturer documentation and direct local evidence.

---

## Repository evolution

This repository may later include additional:

- screenshots;
- diagrams;
- sanitized binary excerpts;
- scripts;
- reproducibility tools;
- recovery documentation;
- research notes;
- independent reproductions.

The same privacy, attribution, evidence-classification, and safety principles should apply to future additions.

---

## Contact

For technical corrections, reproduction results, or responsible reports concerning accidentally published information, contact:

**Bencze Péter**  
**Email:** bencze.peter86@proton.me

---

## Final notice

The existence of a successful recovery on the investigated system does not make the procedure universally safe.

Anyone reproducing this work should understand the hardware, preserve independently verified backups, and treat every firmware write as a potentially destructive operation.

Use the material in this repository only within your own authorization, technical competence, and risk tolerance.
