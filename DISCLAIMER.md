# Legal, Research and Licensing Notice

## Fujitsu FUTRO S930 / D3313 BIOS Recovery Research

This repository documents independent technical research, diagnostic work, firmware-recovery experiments, hardware observations, and related procedures concerning Fujitsu FUTRO S930 systems and D3313-series hardware.

This document defines the legal, licensing, research, safety, and third-party-material boundaries of the repository.

---

## 1. Independent research project

This repository is an independent research and documentation project.

It is not an official Fujitsu project and is not affiliated with, sponsored by, approved by, certified by, or endorsed by Fujitsu or any other hardware, software, firmware, chipset, operating-system, or tool vendor mentioned in the repository.

All product names, company names, trademarks, service marks, firmware names, software names, and logos remain the property of their respective owners.

References to third-party products are made solely for identification, interoperability, technical documentation, research, and educational purposes.

---

## 2. Purpose of the repository

The primary purposes of this repository are:

- documenting BIOS and firmware recovery procedures;
- documenting hardware-specific behavior of Fujitsu FUTRO S930 and D3313-series systems;
- recording diagnostic observations and experimental results;
- preserving reproducible research notes;
- analysing firmware structure and platform-specific configuration;
- documenting recovery of machine-specific configuration and OEM identity data;
- comparing recovery methods, tools, firmware packages, and hardware behavior;
- supporting legitimate repair, restoration, interoperability, research, education, and defensive technical analysis;
- documenting unsuccessful approaches and failure modes so that others can avoid unnecessary risk or repeatable mistakes.

The repository is intended as a technical case study and research record rather than as an official repair manual.

---

## 3. License for original research and documentation

Unless explicitly stated otherwise, the original copyrightable research and documentation created specifically for this repository is made available under the:

**Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International  
(CC BY-NC-SA 4.0)**

The complete official license text is contained in the repository's:

**[LICENSE](LICENSE)**

The official Creative Commons license terms govern the licensed material.

In practical terms, the license permits others to share and adapt covered material for non-commercial purposes, provided that the applicable attribution and ShareAlike requirements are satisfied.

This explanatory document does not replace or modify the official license.

If any statement in this document conflicts with the official CC BY-NC-SA 4.0 legal code, the official license terms control for material actually licensed under CC BY-NC-SA 4.0.

---

## 4. Scope of the Creative Commons license

The CC BY-NC-SA 4.0 license is intended to cover original copyrightable non-software material created by the repository author for this project, including, where applicable:

- written research notes;
- technical explanations;
- original documentation;
- original diagrams;
- original tables;
- original photographs;
- original measurements;
- original analysis;
- original research summaries;
- original testing methodology;
- original descriptions of procedures;
- original observations and conclusions.

The license applies only to material for which the repository author has the legal authority to grant such rights.

The presence of material in this repository does **not** automatically mean that every file is owned by the repository author or licensed under CC BY-NC-SA 4.0.

---

## 5. Software, scripts and source code

Creative Commons licenses are primarily designed for creative, educational, research, and documentation material and are not intended to serve as the primary software license for source code.

Accordingly, unless a software file or directory explicitly states otherwise:

**source code, scripts, utilities, applications, executable code, software components, build files, and other program code are not automatically licensed under CC BY-NC-SA 4.0 merely because they are present in this repository.**

Software developed specifically for this project may later be released under a separate:

- open-source license;
- source-available license;
- dual-license model;
- proprietary license;
- commercial license;
- or another software-specific licensing arrangement.

Where a file or directory contains its own license notice, that specific notice governs that material.

Until a separate software license is explicitly provided, users should not assume that software or source code contained in the repository carries the same reuse permissions as the CC-licensed research documentation.

---

## 6. Commercial use and separate licensing

The CC BY-NC-SA 4.0 license does not grant permission for uses that fall outside its NonCommercial terms.

The copyright holder may separately offer additional permissions or a separate commercial license for original material for which the copyright holder owns the necessary rights.

Possible separately negotiated permissions may include, for example:

- commercial publication;
- commercial training material;
- integration into paid products or services;
- professional consulting deliverables;
- commercial technical documentation;
- commercial databases or knowledge bases;
- commercial research products;
- commercial software or services incorporating separately licensable original material.

Such permission requires a separate agreement with the relevant copyright holder.

The availability of separate commercial terms does not change the CC BY-NC-SA 4.0 rights already granted for material distributed under that license.

A Creative Commons license already validly granted to a recipient is not revoked merely because the copyright holder later offers the same original material under different or commercial terms.

Commercial licensing inquiries concerning original repository material may be directed to the repository owner through the GitHub account associated with this project.

---

## 7. Third-party firmware, BIOS images and vendor materials

This distinction is particularly important for a firmware-recovery project.

Firmware, BIOS code, vendor software, OEM utilities, update packages, drivers, documentation, binary components, microcode, embedded modules, trademarks, logos, and other third-party materials may be protected by copyright, trademark, patent, contract, or other rights belonging to their respective owners.

Such third-party material is **not relicensed under CC BY-NC-SA 4.0 by this repository** merely because it is:

- discussed;
- analysed;
- referenced;
- linked;
- identified;
- compared;
- shown in screenshots;
- included in an analytical description;
- extracted during legitimate research;
- or otherwise mentioned in the repository.

In particular, Fujitsu firmware and BIOS components remain subject to whatever rights and terms apply to those materials.

The repository's CC BY-NC-SA 4.0 license applies only to material for which the repository author possesses sufficient rights to grant that license.

---

## 8. Firmware dumps and device-specific data

A firmware dump created from a physical device may contain several different categories of information, potentially including:

- third-party copyrighted firmware code;
- OEM configuration data;
- platform configuration;
- serial identifiers;
- network identifiers;
- machine-specific values;
- calibration information;
- device-specific settings;
- or other data originating from the hardware or its manufacturer.

The act of creating or analysing a dump does not transfer ownership of third-party intellectual property contained within it.

Consequently, a firmware dump should not automatically be treated as an original work of the repository author or as CC BY-NC-SA 4.0 material.

Where firmware images, extracts, hashes, structural information, or related artifacts are discussed, users remain responsible for determining what rights apply to the underlying material.

Machine-specific identifiers should be handled carefully before public distribution.

---

## 9. Attribution

When CC BY-NC-SA 4.0 material from this repository is shared or adapted, attribution should identify the original repository/project and its author or repository owner in a reasonable manner consistent with the license.

Where reasonably practicable, attribution should include:

- the repository or project name;
- the repository owner or credited author;
- a link to the original repository;
- a reference to **CC BY-NC-SA 4.0**;
- a link to or copy of the applicable license;
- and an indication of whether modifications were made.

Attribution must not imply endorsement, sponsorship, certification, partnership, or official affiliation.

---

## 10. Research integrity

The repository aims to distinguish clearly between:

- directly observed facts;
- measured results;
- reproducible procedures;
- interpretations;
- hypotheses;
- external information;
- and conclusions that remain uncertain.

Where possible, research claims should be supported by:

- measurements;
- firmware hashes;
- photographs;
- logs;
- tool output;
- reproducible procedures;
- primary documentation;
- or clearly identified external sources.

An unsuccessful experiment may remain documented if it provides useful technical information.

Historical notes may therefore describe procedures that are no longer recommended.

Readers should consider the context and current status of a procedure before reproducing it.

---

## 11. Safety and hardware risk

Firmware programming, BIOS modification, direct flash access, soldering, desoldering, hot-air rework, voltage measurement, board-level diagnostics, and similar procedures can permanently damage hardware.

Possible consequences include:

- an unbootable system;
- corrupted firmware;
- destroyed flash memory;
- damaged motherboard components;
- damaged programmers or test equipment;
- data loss;
- electrical damage;
- overheating;
- burns or other physical injury.

Any procedure described in this repository should be performed only by a person who understands the associated risks.

Backups and non-destructive diagnostic methods should be preferred whenever possible.

Power should be disconnected whenever required by the relevant procedure and electrical conditions.

The repository is not a substitute for appropriate technical competence, electrical safety practices, manufacturer documentation, or professional repair services.

---

## 12. No warranty

Material in this repository is provided for research, documentation, educational, repair, interoperability, and informational purposes.

To the maximum extent permitted by applicable law, it is provided **AS IS** and **AS AVAILABLE**, without warranties or representations of any kind.

No guarantee is made that:

- a documented procedure will work on another device;
- hardware revisions will behave identically;
- a firmware image is appropriate for another machine;
- an external source remains accurate or available;
- a particular tool behaves identically across versions;
- or a procedure is free from errors or risks.

Users must independently verify procedures before applying them to hardware, software, firmware, networks, or valuable data.

---

## 13. Limitation of liability

To the maximum extent permitted by applicable law, the repository author, contributors, and copyright holders shall not be liable for damage or loss resulting from the use of information or procedures contained in this repository.

This may include, without limitation:

- hardware damage;
- firmware corruption;
- loss of data;
- loss of device configuration;
- loss of availability;
- economic loss;
- loss of revenue;
- loss of warranty coverage;
- or other direct or indirect consequences.

Use of the material is at the user's own risk.

Nothing in this section limits liability where such limitation is prohibited by applicable law.

---

## 14. Lawful and authorized use

Material relating to firmware, reverse engineering, diagnostics, networking, security, digital forensics, hardware programming, or system analysis is provided for legitimate purposes such as:

- research;
- repair;
- restoration;
- education;
- interoperability;
- defensive analysis;
- documentation;
- preservation;
- and authorized testing.

Nothing in this repository grants authorization to access, alter, monitor, interfere with, bypass protections on, or analyse systems, devices, accounts, communications, networks, or data where the user does not have the necessary legal right or permission.

Users are responsible for compliance with applicable:

- laws;
- regulations;
- licenses;
- contracts;
- privacy requirements;
- authorization requirements;
- and third-party rights.

---

## 15. External sources and citations

External documentation, articles, forum posts, source code, technical discussions, vendor documents, and other references remain subject to their own licenses and legal terms.

A citation or link does not imply ownership of the referenced material.

Where material from an external source is reproduced under an applicable quotation right, license, permission, exception, or limitation, the underlying rights remain with the original rights holder.

Users wishing to reuse such third-party material should consult the original source and applicable terms.

---

## 16. Contributions

Contributors retain copyright in their own contributions unless they explicitly agree otherwise.

A contribution should only contain material that the contributor has the right to submit.

Acceptance of a contribution into the repository does not automatically transfer copyright ownership to the repository owner.

Where a contribution is intended to become part of CC BY-NC-SA 4.0 documentation, contributors should ensure that they have the authority to provide that material under compatible terms.

Software contributions may be subject to a separate software license once such a license is established for the relevant part of the project.

---

## 17. Separate and future licensing

The copyright holder reserves the right, for material for which they own the necessary rights, to:

- distribute future versions under different terms;
- grant additional permissions;
- enter into separate commercial agreements;
- offer dual licensing;
- offer software under a separate software-specific license;
- stop distributing future copies;
- or publish additional material under another license.

These choices do not retroactively revoke rights already validly granted under an applicable Creative Commons license.

Third-party material cannot be relicensed by the repository owner unless the repository owner has the necessary rights to do so.

---

## 18. Priority of applicable terms

For original material expressly covered by CC BY-NC-SA 4.0, the official Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International Public License is authoritative.

For software or files carrying a separate license notice, that separate license governs.

For third-party material, the rights and terms of the relevant third-party rights holder govern.

This document is intended to clarify the structure and scope of the repository and does not replace the applicable licenses themselves.

---

## 19. Contact

Questions concerning the research, attribution, permission to reuse original material, or possible commercial licensing may be submitted to the repository owner through the GitHub account associated with this repository.

---

**Repository licensing structure**

- **Original research and documentation:** CC BY-NC-SA 4.0, unless otherwise stated
- **Official license text:** `LICENSE`
- **Software/source code:** separate license where explicitly provided
- **Third-party firmware and materials:** rights remain with their respective owners
- **Commercial licensing of eligible original material:** available only by separate agreement where offered

---

Copyright © 2026 repository author and respective contributors for their original contributions.

Third-party copyrights, trademarks, firmware, software, documentation, and other protected materials remain the property of their respective rights holders.
