# Sources, References, Prior Work, and Research Provenance

## Fujitsu FUTRO S930 / D3313-E1 BIOS Recovery and Firmware Analysis

This document lists the external references, manufacturer documentation, software tools, community research, and locally generated research artifacts used or consulted during the investigation documented in this repository.

The purpose of this file is not merely to provide a collection of links.

Sources are classified according to their evidentiary role so that manufacturer documentation, normative specifications, software documentation, community research, and observations generated during this project are not treated as equivalent forms of evidence.

---

# 1. Source classification

The following source classes are used throughout this document.

### `[PRIMARY — MANUFACTURER]`

Documentation, firmware, software, datasheets, or support material published by the manufacturer of the relevant hardware.

Examples:

- Fujitsu
- Realtek
- Macronix
- WCH

These sources are preferred for claims about documented hardware capabilities, official firmware releases, and manufacturer-supported behavior.

---

### `[PRIMARY — STANDARD]`

Normative technical specifications maintained by a recognized standards organization.

Example:

- UEFI Forum Platform Initialization Specification

These sources are used for definitions of UEFI Firmware Volumes, Firmware File Systems, FFS files, GUID-based firmware structures, and related architecture.

---

### `[PRIMARY — SOFTWARE]`

Documentation or source code maintained by the upstream software project directly involved in the investigation.

Examples:

- flashrom
- Linux kernel
- systemd
- UEFITool
- IMSProg

These sources are used to document the behavior of tools and operating-system components used during the investigation.

---

### `[SECONDARY — TECHNICAL]`

Third-party technical documentation, mirrors, or repositories containing tools or documentation not currently available from a directly accessible manufacturer location.

These sources are useful but should not be treated as manufacturer documentation unless independently verified.

---

### `[PRIOR WORK — COMMUNITY]`

Community investigations, forum discussions, hardware documentation, reverse engineering, and previous experiments involving related Fujitsu FUTRO or D3313 systems.

These sources are valuable because they establish prior work and provide research leads.

They are **not** treated as proof that the D3313-E1 specimen investigated in this repository behaves identically.

---

### `[LOCAL EVIDENCE]`

Data generated directly during this investigation.

Examples:

- SPI dumps;
- cryptographic hashes;
- binary comparisons;
- flashrom verification results;
- Linux logs;
- BIOS observations;
- RSET8168 output;
- eFuse reads;
- multimeter measurements.

These are the primary experimental evidence for the conclusions specific to the investigated machine.

---

# 2. Fujitsu D3313 platform documentation

## 2.1. Fujitsu — Mainboard D3313, Short Description

**Classification:** `[PRIMARY — MANUFACTURER]`  
**Publisher:** Fujitsu  
**Document:** Mainboard D3313, Short description  
**Version:** 07/2015  
**Date:** 31 July 2015  
**File type:** PDF  
**Language:** multilingual

Fujitsu explicitly lists the following relevant variants among the supported products:

- D3313-A1x
- D3313-B1x
- **D3313-E1x**
- D3313-F1x
- D3313-G1x
- D3313-S family variants
- additional D3313 and D3314 variants

**Relevance to this project**

This is one of the principal manufacturer references establishing the investigated D3313-E1 as a documented member of the Fujitsu D3313 platform family.

It should be preferred over community documentation for basic board-family identification.

**Source**

https://support.ts.fujitsu.com/Search/SWP1136296.asp

---

## 2.2. Fujitsu — BIOS Handbuch D331x

**Classification:** `[PRIMARY — MANUFACTURER]`  
**Publisher:** Fujitsu  
**Document:** BIOS Handbuch D331x  
**Version:** 08/2015  
**Date:** 7 August 2015  
**Language:** German  
**File type:** PDF

The supported products explicitly include:

- D3313-A1x
- D3313-B1x
- **D3313-E1x**
- D3313-F1x
- D3313-G1x
- multiple D3313-S variants
- related D3314 variants

**Relevance**

This is the principal manufacturer BIOS reference for the D331x platform generation and is particularly important when interpreting:

- firmware setup behavior;
- platform configuration;
- BIOS update/recovery terminology;
- board-specific firmware features.

**Source**

https://support.ts.fujitsu.com/Search/SWP1138063.asp

---

## 2.3. Fujitsu — FUTRO S720 / S920 Operating Manual, including FUTRO S930

**Classification:** `[PRIMARY — MANUFACTURER]`  
**Publisher:** Fujitsu  
**Document:** FUTRO S720 / S920, ESPRIMO A525-L, Operating Manual  
**Version:** 11/2017  
**Publication date:** 20 December 2017  
**Language:** English  
**File type:** PDF

Although the title references S720/S920, the Fujitsu support record explicitly lists:

- FUTRO S720
- FUTRO S920
- **FUTRO S930**

among the covered products.

**Relevance**

Useful for:

- system-level hardware context;
- chassis and maintenance procedures;
- storage and expansion context;
- BIOS/update background;
- identifying the S930 as part of the documented FUTRO platform family.

**Source**

https://support.ts.fujitsu.com/Search/SWP1206702.asp

---

# 3. Related Fujitsu D3313 firmware history

## 3.1. Fujitsu D3313-S1x — BIOS R1.16.0 release information

**Classification:** `[PRIMARY — MANUFACTURER]`  
**Important limitation:** related D3313 variant, **not D3313-E1-specific evidence**.

**Firmware version**

```text
V4.6.5.4 R1.16.0 for D3313-S1x
```

**Date:** 28 January 2015

The Fujitsu release notes include several changes particularly relevant to this research:

- changed recovery-mode behavior;
- automatic BIOS recovery;
- a fix for sporadic mSATA detection failure;
- changes involving UEFI variables and other firmware functions.

**Research relevance**

This source demonstrates that:

1. BIOS recovery behavior was actively modified within the D3313 platform family;
2. mSATA detection was a documented firmware issue on at least one related D3313 variant.

It does **not** prove that the same recovery implementation or mSATA defect exists on the D3313-E1 investigated here.

This source is therefore used only as:

> **related D3313 platform firmware history**

and not as direct evidence for E1-specific behavior.

**Source**

https://support.ts.fujitsu.com/Search/SWP1123238.asp

---

# 4. UEFI architecture and firmware structure

## 4.1. UEFI Forum — Platform Initialization Specification

**Classification:** `[PRIMARY — STANDARD]`  
**Organization:** UEFI Forum

The PI Specification defines the structures relevant to our firmware analysis, including:

- Firmware Volumes (FV);
- Firmware File Systems (FFS);
- Firmware Files;
- firmware file GUIDs;
- firmware sections;
- integrity information;
- storage organization inside firmware devices.

**Research relevance**

The terminology used in this repository for:

- Firmware Volume;
- FFS;
- FREEFORM FFS;
- GUID;
- firmware file structure;

is grounded in the UEFI Platform Initialization architecture rather than being project-specific terminology.

**UEFI specifications portal**

https://uefi.org/specifications

**Platform Initialization Specification 1.9**

https://uefi.org/sites/default/files/resources/UEFI_PI_Spec_Final_Draft_1.9.pdf

**Firmware storage design discussion**

https://uefi.org/specs/PI/1.8/V3_Design_Discussion.html

---

# 5. UEFI firmware analysis tooling

## 5.1. UEFITool / UEFIExtract

**Classification:** `[PRIMARY — SOFTWARE]`  
**Project:** LongSoft / UEFITool  
**License:** BSD-2-Clause

UEFITool describes itself as a viewer and editor for firmware images conforming to UEFI Platform Initialization specifications.

The project also includes:

- UEFIExtract;
- UEFIFind;
- firmware parsing;
- integrity validation;
- tree-based representation of UEFI firmware contents.

**Research relevance**

This family of tools provides an established framework for examining:

- Firmware Volumes;
- FFS files;
- GUIDs;
- firmware sections;
- structural integrity.

The D3313 investigation used this architectural approach to interpret the internal structure of the 8 MiB firmware image.

**Repository**

https://github.com/LongSoft/UEFITool

**Releases**

https://github.com/LongSoft/UEFITool/releases

---

# 6. flashrom

## 6.1. flashrom project

**Classification:** `[PRIMARY — SOFTWARE]`

flashrom is an open-source utility designed to:

- detect flash devices;
- read firmware;
- write firmware;
- verify firmware;
- erase flash devices;
- access supported flash chips through internal and external programmers.

It supports, among many other interfaces:

- `internal`;
- `ch341a_spi`;
- various PCI devices;
- SPI interfaces;
- external programmers.

**Research relevance**

flashrom was central to the final phases of this project.

It was used for:

- in-system flash detection;
- complete SPI acquisition;
- explicit flash-chip selection;
- physical verification;
- ROM layout definition;
- selective region writing;
- post-write verification.

**Official site**

https://flashrom.org/

**Debian manual**

https://manpages.debian.org/trixie/flashrom/flashrom.8.en.html

---

## 6.2. flashrom ROM layouts and selective region writes

**Classification:** `[PRIMARY — SOFTWARE]`

The flashrom manual documents ROM layouts using:

```text
startaddr:endaddr imagename
```

and the use of layout regions with options such as:

```text
-l
--layout
-i
--image
```

This directly supports the method used in this project to define:

```text
0x440000:0x44ffff OEM
```

and write only the selected OEM region rather than rewriting the complete 8 MiB device.

**Documentation**

https://manpages.debian.org/trixie/flashrom/flashrom.8.en.html

---

## 6.3. flashrom verification behavior

**Classification:** `[PRIMARY — SOFTWARE]`

The flashrom documentation explains that after writing, the utility can read the flash contents again and compare them against the input image.

The project deliberately performed:

1. pre-write verification;
2. write-time verification;
3. separate post-write verification.

This exceeded the minimum required procedure and was intended to strengthen the evidence that the intended region had actually been written correctly.

**Documentation**

https://manpages.debian.org/trixie/flashrom/flashrom.8.en.html

---

## 6.4. flashrom guidance on firmware backups and machine-specific data

**Classification:** `[PRIMARY — SOFTWARE]`

The flashrom documentation explicitly warns that original firmware images may contain machine-specific information including:

- serial numbers;
- MAC addresses;
- Windows preactivation information.

This strongly supports the privacy policy used by this repository.

**Source**

https://flashrom.org/Laptops

---

## 6.5. flashrom official release archive

**Classification:** `[PRIMARY — SOFTWARE]`

The flashrom project maintains an official archive containing release tarballs and, for recent versions, signatures and SHA256 checksums.

This is useful for reproducibility when documenting exactly which tool version was used in a future experiment.

**Official archive**

https://download.flashrom.org/releases/

---

# 7. Macronix SPI flash documentation

## 7.1. Macronix MX25L6473F datasheet

**Classification:** `[PRIMARY — MANUFACTURER]`  
**Manufacturer:** Macronix International  
**Part:** MX25L6473F  
**Capacity:** 64 Mbit  
**Logical size:** 8,388,608 × 8 bits  
**Nominal family voltage:** approximately 2.65–3.6 V

The datasheet documents:

- Serial Peripheral Interface operation;
- 256-byte page programming;
- 4 KiB sector erase;
- 32 KiB block erase;
- 64 KiB block erase;
- whole-chip erase;
- status registers;
- program/erase verification behavior.

The physical flash on the investigated board was identified as belonging to this family.

**Research relevance**

The documented capacity corresponds exactly to the observed:

```text
8,388,608-byte
```

complete SPI acquisitions.

**Datasheet**

https://www.macronix.com/Lists/Datasheet/Attachments/8667/MX25L6473F%2C%203V%2C%2064Mb%2C%20v1.3.pdf

---

# 8. CH341A external SPI programmer

## 8.1. WCH CH341 driver and interface documentation

**Classification:** `[PRIMARY — MANUFACTURER]`  
**Manufacturer:** Nanjing Qinheng Microelectronics / WCH

The WCH CH341 driver page documents CH341 support for interfaces including:

- SPI;
- I²C;
- EPP;
- parallel/MEM interfaces.

This is the appropriate manufacturer driver family for the CH341A programming device used in the investigation.

**Source**

https://www.wch-ic.com/downloads/CH341PAR.EXE.html?type=en

---

## 8.2. IMSProg

**Classification:** `[PRIMARY — SOFTWARE]`  
**Project:** IMSProg  
**Author/project maintainer:** Mikhail Medvedev / bigbigmdm

IMSProg is an open-source programmer application supporting devices including:

- WCH CH341A;
- CH347;
- FT232H;

and SPI/I²C/MicroWire EEPROM and flash devices.

The documentation explicitly includes:

- read;
- write;
- erase;
- verification;
- SPI NOR support;
- CH341A operation.

**Research relevance**

IMSProg successfully recognized the CH341A during the initial external SPI work when the Windows flashrom/libusb path proved unreliable.

Observed device identification:

```text
WCH CH341A
Device rev 3.0.4
```

**Repository**

https://github.com/bigbigmdm/IMSProg

**README**

https://github.com/bigbigmdm/IMSProg/blob/main/README.md

**Releases**

https://github.com/bigbigmdm/IMSProg/releases

---

# 9. Realtek RTL8111/RTL8168 family

## 9.1. Realtek PCIe Ethernet software and diagnostic downloads

**Classification:** `[PRIMARY — MANUFACTURER]`  
**Manufacturer:** Realtek

Realtek's official PCIe Ethernet controller software page covers the relevant RTL8111/RTL8168 families and provides multiple drivers and diagnostic tools.

Importantly, the page includes:

```text
DOS Diagnostic program (RSET8168)
Version 1.24
```

**Research relevance**

RSET8168 v1.24 was used directly during this investigation.

**Official Realtek page**

https://www.realtek.com/Download/List?cate_id=584

---

## 9.2. RSET8168 v1.24

**Classification:** `[PRIMARY — MANUFACTURER]`

Realtek RSET8168 was used from FreeDOS to inspect the Ethernet controller independently of Linux.

Observed findings on the investigated hardware included:

```text
Ethernet Address: 00 00 00 00 00 00
```

and during EEPROM diagnostics:

```text
No Valid External EEPROM!!
Data from EFuse
EFuse Fail
```

These outputs are classified as:

**`[LOCAL EVIDENCE]`**

The source listed here establishes the provenance of the diagnostic utility itself.

**Source**

https://www.realtek.com/Download/List?cate_id=584

---

# 10. Realtek EEPROM/eFuse programming and inspection

## 10.1. redchenjs/rtnicpg — Realtek NIC PG Program

**Classification:** `[SECONDARY — TECHNICAL]`

Repository description:

> Realtek NIC PG Program (EFUSE / EEPROM)

The repository contains:

- `rtnicpg`;
- `pgdrv`;
- `pgload.sh`;
- numerous controller-specific configuration files;
- EEPROM/eFuse read/programming support.

Of particular relevance to this project is:

```text
8168GEF.cfg
```

for RTL8168G-family eFuse operation.

The documented workflow includes unloading the normal NIC driver, loading `pgdrv.ko`, and accessing the controller through the PG utility.

**Repository**

https://github.com/redchenjs/rtnicpg

**Important qualification**

This is not being cited as an official Realtek distribution page.

It is used as the provenance of the particular PGTool-compatible package used during the investigation.

---

## 10.2. Local compatibility patch for Linux 6.12

The `pgdrv` source in the package used an older kernel API.

Original code:

```c
vma->vm_flags |= VM_IO;
```

Local compatibility change:

```c
vm_flags_set(vma, VM_IO);
```

This modification allowed the module to build against the Debian 6.12 kernel used during the investigation.

This patch is:

**`[LOCAL EVIDENCE / LOCAL MODIFICATION]`**

and should not be attributed to the original upstream package.

---

# 11. Linux r8169 behavior

## 11.1. Linux kernel — r8169 MAC initialization

**Classification:** `[PRIMARY — SOFTWARE]`  
**Project:** Linux kernel

The r8169 driver source code demonstrates that the driver attempts several methods to obtain a valid MAC address.

If no valid address can be obtained, the driver falls back to a generated address and logs:

```text
can't read MAC address, setting random one
```

This is the exact class of message observed on the investigated machine after BIOS recovery.

**Research relevance**

This source explains the Linux-side behavior after the platform failed to provide a usable permanent MAC.

It does **not** identify the cause of the missing platform MAC; that conclusion comes from the local firmware and hardware evidence.

**Source example — Linux kernel source**

https://kernel.googlesource.com/pub/scm/linux/kernel/git/torvalds/linux/+/f91a3aa6bce480fe6e08df540129f4a923222419/drivers/net/ethernet/realtek/r8169_main.c

---

# 12. systemd persistent MAC policy

## 12.1. systemd.link — MACAddressPolicy

**Classification:** `[PRIMARY — SOFTWARE]`  
**Project:** systemd

The systemd documentation defines:

```text
MACAddressPolicy=persistent
```

If a device does not provide a persistent hardware address suitable for use by the kernel, systemd may generate a machine/device-specific address designed to remain stable across boots.

This explains why the recovered Debian system could remain network-accessible with a stable-looking MAC even though:

```text
Permanent address: not set
```

was reported for the physical interface.

**Source**

https://www.freedesktop.org/software/systemd/man/latest/systemd.link.html

---

# 13. Prior Fujitsu FUTRO / D3313 community research

The following sources are included as **prior work**.

They are not used as manufacturer-level evidence.

---

## 13.1. ParkyTowers — Fujitsu Futro S920

**Classification:** `[PRIOR WORK — COMMUNITY]`

ParkyTowers documents the FUTRO S920 and the D3313 platform and includes later community contributions involving:

- D3313 hardware;
- PCIe behavior;
- Fujitsu EditCMOS;
- hidden/platform settings;
- community testing.

The site also explicitly discusses the restricted Fujitsu EditCMOS utility and subsequent community work.

**Research relevance**

This source establishes that the D3313 platform had already attracted detailed community investigation before this project.

Our work should therefore not claim to be the first D3313 firmware investigation.

**Source**

https://www.parkytowers.me.uk/thin/Futro/s920/

---

## 13.2. ServeTheHome — Fujitsu Futro S920 technical discussion

**Classification:** `[PRIOR WORK — COMMUNITY]`

A long-running ServeTheHome thread documents extensive experimentation on FUTRO S920/D3313 systems.

Relevant areas include:

- PCIe configuration;
- EditCMOS;
- D3313 setup IDs;
- SATA and storage;
- platform configuration.

One particularly relevant community finding lists:

```text
0x015b    mSataSupport
```

among D3313 setup identifiers.

**Important limitation**

This does **not** prove that the unresolved mSATA problem on the investigated D3313-E1 is caused by this setting.

It is retained only as a research lead and evidence of prior D3313 configuration research.

**Source**

https://forums.servethehome.com/index.php?threads/fujitsu-futro-s920-thin-client-as-opnsense-firewall.31087/

A relevant later page of the discussion:

https://forums.servethehome.com/index.php?threads/fujitsu-futro-s920-thin-client-as-opnsense-firewall.31087/page-13

---

## 13.3. R3NE07 — Fujitsu FUTRO S740 documentation

**Classification:** `[PRIOR WORK — COMMUNITY]`

Although the FUTRO S740 is a later and different platform, this repository contains a documented Fujitsu recovery workflow involving:

- a recovery jumper;
- BIOS update media;
- recovery beeps;
- returning the jumper to standard/default position.

**Research relevance**

This is useful prior work for Fujitsu recovery methodology.

It is not used as evidence that the S930/D3313-E1 recovery implementation is identical.

**Repository**

https://github.com/R3NE07/Futro-S740

**Recovery documentation**

https://github.com/R3NE07/Futro-S740/blob/main/README.md

---

## 13.4. BIOS-Mods — FUTRO S920 / D3313 firmware modification discussion

**Classification:** `[PRIOR WORK — COMMUNITY]`

The BIOS-Mods community has previously investigated D3313-based FUTRO firmware in connection with:

- IOMMU;
- VRAM configuration;
- hidden firmware options;
- AMI firmware modification.

The discussion references D3313 variants and official R1.16.0 firmware.

**Research relevance**

This confirms that BIOS analysis/modification of the D3313 platform predates this project.

The contribution of the present work is therefore not generic D3313 BIOS modification, but the specific documented chain involving:

- machine-specific OEM data;
- recovery-state comparison;
- NIC identity;
- blank Realtek eFuse;
- selective OEM restoration.

**Source**

https://www.bios-mods.com/forum/Thread-Request-Fujitsu-Futro-S920-Thin-Client-Enable-IOMMU-Fix-Reserved-VRAM

---

# 14. Source-use rules for this repository

To avoid overstating evidence, this project follows these citation rules.

## 14.1. Fujitsu E1 claims

A claim specific to the D3313-E1 should preferably be supported by:

1. Fujitsu documentation explicitly listing D3313-E1x; or
2. local observations from the investigated board.

Documentation for D3313-S, A, B, G, or other variants should not be silently generalized to E1.

---

## 14.2. Related D3313 variants

Sources concerning another D3313 variant may be used to establish:

- platform-family context;
- historical firmware behavior;
- research hypotheses;
- possible areas for further investigation.

They should be described explicitly as:

> related-platform evidence

rather than E1-specific proof.

---

## 14.3. Community findings

Forum posts and community projects may be used for:

- prior-work acknowledgement;
- hypotheses;
- reproduction attempts;
- discovering documentation;
- identifying possible setup variables.

They should not override contradictory manufacturer documentation or direct observations.

---

## 14.4. Local measurements

Claims concerning the specific machine investigated here should be based primarily on:

- captured firmware;
- cryptographic hashes;
- direct hardware diagnostics;
- logs;
- screenshots;
- repeatable commands;
- verified flash contents.

---

# 15. Research artifacts and cryptographic provenance

The following artifacts were generated or used during the investigation.

Personalized firmware images are **not publicly distributed**.

Their cryptographic digests are published to provide provenance and reproducibility.

---

## 15.1. Original external SPI dump #1

**Artifact**

```text
FUTRO_D3313_ORIGINAL_1.bin
```

**Size**

```text
8,388,608 bytes
```

**SHA256**

```text
7E138DA4B68FB7C86DBD841631D6D60937AF2EA109297A51476610C8F01E3603
```

**Acquisition**

External CH341A / SOIC8 acquisition.

**Publication status**

Raw image withheld because it contains machine-specific manufacturing data.

---

## 15.2. Original external SPI dump #2

**Artifact**

```text
FUTRO_D3313_ORIGINAL_2.bin
```

**Size**

```text
8,388,608 bytes
```

**SHA256**

```text
7E138DA4B68FB7C86DBD841631D6D60937AF2EA109297A51476610C8F01E3603
```

**Result**

Bit-for-bit identical to original dump #1.

**Research significance**

The matching hashes form the integrity baseline for the original firmware acquisition.

---

## 15.3. Official Fujitsu R1.16 comparison ROM

**Classification:** reference firmware image

**SHA256**

```text
1E51B1392A291CEC24575170F5527FE541048463366D2FA541E90A93EA371F59
```

**Research purpose**

Used for block-level comparison with the original machine SPI image.

**Observed result**

```text
127 / 128 × 64 KiB blocks identical
```

Only:

```text
0x440000–0x44FFFF
```

differed.

Replacing the factory block in that range with the original machine block reproduced the complete original machine SPI image.

---

## 15.4. Post-recovery internal SPI dump #1

**Artifact**

```text
D3313_AFTER_RECOVERY_1.bin
```

**SHA256**

```text
4669F17C9F6B65E0682151C97384702357BA19358FFF797C2EFF8236540EAEC0
```

**Acquisition**

Linux internal flashrom read using explicit MX25L6473F chip selection.

---

## 15.5. Post-recovery internal SPI dump #2

**Artifact**

```text
D3313_AFTER_RECOVERY_2.bin
```

**SHA256**

```text
4669F17C9F6B65E0682151C97384702357BA19358FFF797C2EFF8236540EAEC0
```

**Result**

Bit-for-bit identical to post-recovery dump #1.

This established the trusted post-recovery physical SPI baseline.

---

## 15.6. Selective OEM restoration candidate

**Artifact**

```text
D3313_REPAIRED_CANDIDATE.bin
```

**SHA256**

```text
6C7641DA8F0AAA914F08F0639A0466B7E8C023CD29E85A0A13EAEE5669FB6E69
```

**Construction**

The current post-recovery firmware image was retained.

Only:

```text
0x440000–0x44FFFF
```

was replaced with the equivalent block from the original machine SPI dump.

---

## 15.7. Candidate-vs-current difference

The repaired candidate differed from the post-recovery SPI by:

```text
3526 bytes
```

All differences were confined to:

```text
block 68
0x440000–0x44FFFF
```

No other region was changed in the candidate.

---

# 16. Relevant block-level comparison results

Comparison of the original pre-recovery image and the post-recovery image showed differing 64 KiB blocks including:

```text
Block 2
Block 54
Block 64
Block 65
Block 66
Block 67
Block 68
```

These broader differences were one reason the complete historical 8 MiB image was **not** written back wholesale.

The repair was instead limited to the machine-specific OEM region.

---

# 17. OEM/manufacturing structures observed locally

The following labels were observed inside the identified machine-specific region:

```text
BdSn
PcId
BUID
UUID
MAC Address #0
HDCP
TPMD
OEMB
JPND
MMCD
CRID
SDEL
SLPD
HDLS
IBSD
AESD
```

The exact meaning of several fields remains unresolved.

These labels are:

**`[LOCAL EVIDENCE]`**

rather than claims derived from external documentation.

---

# 18. OEM FFS structures

Two highly similar machine-specific structures were identified around:

```text
0x440000
0x443000
```

Observed GUIDs:

```text
0972EB29-E62F-4699-A19C-664AC3B7208B
```

and:

```text
8EEAF5CF-1C81-4FCC-AE1E-D8AD3CD15683
```

The exact redundancy model remains unresolved.

Possible interpretations include:

- primary/backup copies;
- active/inactive generations;
- versioned records;
- another OEM integrity mechanism.

Until experimentally determined, this remains:

**`[OPEN QUESTION]`**

---

# 19. Local Realtek evidence

The following results were observed directly on the investigated system.

## BIOS after recovery

```text
MAC: 00:00:00:00:00:00
```

---

## Linux r8169

Observed message:

```text
can't read MAC address, setting random one
```

Observed permanent-address state:

```text
Permanent address: not set
```

---

## Realtek RSET8168

Observed:

```text
Ethernet Address: 00 00 00 00 00 00
```

EEPROM diagnostic:

```text
No Valid External EEPROM!!
Data from EFuse
EFuse Fail
```

---

## Realtek PGTool-compatible read

Observed controller family:

```text
RTL8168G
```

Observed eFuse state:

```text
FF...
```

Observed counter:

```text
Efuse Write Count = 0
```

The eFuse remained unmodified during this project.

No permanent MAC was programmed into it.

---

# 20. Local post-recovery OEM comparison

Original machine SPI:

```text
MAC Address #0 → <REDACTED_FUJITSU_MAC>
```

Official Fujitsu comparison firmware:

```text
MAC Address #0 → FF:FF:FF:FF:FF:FF
```

Physical SPI after recovery:

```text
MAC Address #0 → FF:FF:FF:FF:FF:FF
```

After restoring the original OEM block, the BIOS again displayed the original machine-specific Fujitsu LAN address.

This sequence forms one of the central experimental results of the project.

---

# 21. Artifact publication policy

## Research artifact policy

Cryptographic hashes are published for:

- provenance;
- reproducibility;
- identity comparison;
- future independent verification.

Personalized firmware images themselves are not distributed because they contain machine-specific manufacturing and identity information.

A SHA256 digest can be used to determine whether another binary is identical to a research artifact without publishing the artifact itself.

---

# 22. Privacy policy for evidence

Before publication, all evidence must be checked for:

- serial numbers;
- MAC addresses;
- UUIDs;
- PC IDs;
- HDCP values;
- operating-system usernames;
- home directory names;
- private paths;
- IP addresses where relevant;
- QR codes;
- barcodes;
- device stickers;
- EXIF;
- GPS metadata.

Raw screenshots and photographs should not automatically be considered safe for publication.

---

# 23. Attribution statement

This project does **not** claim that BIOS recovery, SPI programming, UEFI analysis, D3313 configuration research, flashrom region writing, or Realtek diagnostic techniques were invented here.

Earlier work by:

- Fujitsu engineers and documentation authors;
- UEFI Forum contributors;
- flashrom developers;
- Linux kernel developers;
- systemd developers;
- Macronix;
- Realtek;
- WCH;
- UEFITool developers;
- IMSProg developers;
- FUTRO/D3313 community researchers;

made substantial parts of this investigation possible.

The contribution of this repository is the documented, evidence-driven application and combination of these methods on a specific Fujitsu D3313-E1 system, including the resulting machine-specific observations.

---

# 24. Principal original observations of this project

Subject to replication on additional hardware, the most significant locally derived findings are:

1. the original D3313-E1 machine firmware and the corresponding Fujitsu R1.16 comparison image matched in 127 of 128 64 KiB blocks;

2. replacing only the `0x440000–0x44FFFF` factory block with the original machine block reproduced the complete original SPI image;

3. machine-specific manufacturing data was concentrated in this region;

4. two related OEM structures contained the machine-specific MAC;

5. the generic comparison image contained `FF:FF:FF:FF:FF:FF` in the corresponding MAC field;

6. after recovery, the physical SPI also contained the `FF` template value;

7. the Realtek controller had no valid external EEPROM according to RSET8168;

8. its eFuse appeared unprogrammed and reported `Efuse Write Count = 0`;

9. restoring only the original Fujitsu OEM block restored the original LAN identity;

10. permanent Realtek eFuse programming was therefore unnecessary;

11. the Fujitsu recovery process observed here did not behave as a simple bit-for-bit full-chip copy of the supplied firmware image;

12. the complete recovery-state SPI could be read and selectively repaired from the running Linux system using flashrom.

---

# 25. Open research questions

The following questions remain intentionally unresolved:

- How exactly does Fujitsu recovery select or reconstruct firmware regions?
- Why were personalized OEM fields not preserved despite being present in the recovery source image?
- What is the exact role of the two related OEM FFS structures?
- Which UEFI module initializes the Realtek MAC?
- Which additional OEM fields influence hardware configuration?
- Is the same structure present on other D3313-E1 boards?
- Is the structure shared with D3313-A/B/G/S variants?
- Does the unresolved mSATA problem involve firmware configuration, hardware failure, or the storage device itself?
- Is `mSataSupport` relevant on the E1 implementation?
- Can the recovery beep pattern be mapped to individual recovery stages?

---

# 26. Reproducibility priority

The most valuable next external contribution would be an independently obtained, privacy-safe comparison from another D3313-E1 board.

Particularly useful data would include:

- BIOS version;
- flash size;
- hashes of privately retained original dumps;
- location of OEM structures;
- FFS GUIDs;
- whether `MAC Address #0` exists at the same offsets;
- whether two copies are present;
- eFuse state;
- recovery behavior;
- post-recovery OEM state.

Contributors should **not** publicly upload personalized raw firmware images without first auditing them for machine-specific information.

---

# 27. Citation notes

When citing this project, distinguish between:

### External technical facts

These should cite the original manufacturer, standard, or upstream software documentation.

### Findings from the investigated D3313-E1

These may cite this repository as the experimental source.

### Community prior work

These should credit the original community author/thread rather than attributing the finding to this project.

A dedicated `CITATION.cff` file is planned so that this repository can be cited consistently in later technical or academic work.

---

# 28. Link longevity and archival note

Manufacturer and community URLs can change over time.

Where possible, this repository prefers:

1. manufacturer support pages;
2. upstream project repositories;
3. official standards pages;
4. stable project documentation;
5. community sources only where no stronger primary source exists.

If an important source disappears, an archived reference may later be added, but archived copies should not silently replace attribution to the original publisher.

---

# 29. Last source review

**Source list reviewed:** 24 October 2026

The list should be updated whenever:

- a stronger primary source becomes available;
- a link becomes unavailable;
- a new D3313-E1 reproduction is published;
- a currently open firmware structure is identified;
- a community hypothesis becomes experimentally verified or disproven.
