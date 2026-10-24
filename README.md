# Fujitsu FUTRO S930 / D3313-E1

## BIOS Recovery, Firmware Analysis, and Restoration of Machine-Specific OEM Identity

### A forensic-style technical case study of firmware recovery and platform identity reconstruction on a Fujitsu D3313-E1 mainboard

> **Status:** BIOS recovery successful · machine-specific OEM identity restored · original LAN identity restored · separate mSATA issue remains under investigation

[Magyar változat / Hungarian version](README.hu.md)

---

## Project navigation

- [Hungarian documentation](README.hu.md)
- [Sources and prior work](SOURCES.md)
- [Legal and research notice](DISCLAIMER.md)
- [Hungarian legal notice](DISCLAIMER.hu.md)
- [License](LICENSE)
- [Citation metadata](CITATION.cff)

---

# Abstract

This repository documents a forensic-style investigation and recovery of a Fujitsu FUTRO S930 thin client based on the Fujitsu D3313-E1 mainboard.

The system arrived in a non-working state following a reported processor repasting attempt during which the cooling assembly and rear retention plate had been displaced. Because substantial mechanical pressure may have been applied to the processor area, the initial failure could not safely be attributed to firmware alone.

Mechanical damage, power-delivery faults, board-level failure, corrupted SPI firmware, or a combination of these possibilities all remained viable hypotheses.

The investigation therefore followed an evidence-driven methodology rather than a conventional “reflash the BIOS and test” approach.

The process included:

- repeated board-level electrical measurements;
- external SPI acquisition using a CH341A programmer;
- duplicate 8 MiB firmware reads;
- cryptographic hash comparison;
- comparison with official Fujitsu firmware;
- UEFI Firmware Volume and FFS analysis;
- Fujitsu recovery-mode experiments;
- DOS-level Realtek diagnostics;
- direct Realtek eFuse inspection;
- internal SPI acquisition under Linux;
- block-level firmware differencing;
- and a selective 64 KiB in-system SPI restoration.

Several significant findings emerged.

The original machine SPI image and the corresponding official Fujitsu R1.16 firmware were found to be identical in **127 of 128 64 KiB blocks**.

The complete machine-specific difference was concentrated in a single 64 KiB OEM/manufacturing region:

```text
0x440000–0x44FFFF
```

That region contained multiple machine-specific fields, including two structures containing:

```text
MAC Address #0
```

The official firmware contained the template value:

```text
FF:FF:FF:FF:FF:FF
```

while the original machine dump contained the real Fujitsu-assigned Ethernet identity.

After successful Fujitsu BIOS recovery, the system POSTed and booted again, but the network controller had lost its permanent MAC identity.

DOS-level Realtek diagnostics found no valid external EEPROM, while direct Linux PGTool inspection showed an apparently unprogrammed eFuse with:

```text
Efuse Write Count = 0
```

A post-recovery internal SPI dump then demonstrated that the machine-specific OEM MAC field had returned to the factory-template `FF` state.

Rather than permanently programming the Realtek eFuse, the recovery-state firmware was preserved and only the original 64 KiB OEM region was restored.

The selective write was independently verified at the physical flash level.

After a cold boot, BIOS once again displayed the original Fujitsu LAN identity, and Linux recovered the corresponding permanent network identity.

The evidence therefore supports a model in which, on the investigated D3313-E1 system, the Realtek controller receives its machine-specific Ethernet identity from Fujitsu platform firmware rather than from a previously programmed controller eFuse.

The broader firmware-repair principle demonstrated by the project is:

> **Restore the smallest region that has been demonstrated to require restoration rather than rewriting the entire firmware image by default.**

---

# Keywords

Fujitsu D3313-E1 · Fujitsu FUTRO S930 · BIOS recovery · UEFI · SPI flash · firmware forensics · OEM data · manufacturing data · flashrom · CH341A · MX25L6473F · Realtek RTL8168G · MAC address · eFuse · firmware recovery · forensic firmware analysis

---

# 1. Scope and objectives

This project began as an attempt to repair one apparently failed Fujitsu FUTRO S930.

It evolved into a structured firmware investigation after it became clear that restoring basic boot functionality did not automatically restore the original identity and configuration of the individual machine.

The principal objectives became:

1. determine whether a firmware-related failure could be demonstrated without prematurely excluding mechanical or electrical causes;
2. obtain a trustworthy copy of the original SPI firmware;
3. compare the machine firmware against the corresponding official Fujitsu firmware;
4. identify machine-specific regions;
5. determine their functions where possible;
6. restore POST and normal boot capability;
7. investigate the loss of permanent network identity after recovery;
8. determine whether that identity originated from Realtek EEPROM/eFuse or Fujitsu platform firmware;
9. restore the original machine-specific configuration while modifying the smallest possible SPI region;
10. preserve a reproducible evidence chain.

This repository is therefore **not intended to be a generic BIOS flashing tutorial**.

It is a:

> **case study in evidence-driven firmware recovery and platform identity reconstruction.**

---

# 2. Evidence classification

To avoid mixing measurements with interpretation, observations are separated conceptually into several categories.

### `[OFFICIAL]`

Information derived from manufacturer documentation, official firmware packages, vendor utilities, or equivalent primary technical sources.

### `[OBSERVED]`

Information directly measured, read, logged, extracted, or reproduced on the investigated hardware.

### `[INFERRED]`

A conclusion strongly supported by multiple observations but not itself directly measured.

### `[REPORTED HISTORY]`

Information concerning the state or treatment of the machine before the structured investigation began.

### `[OPEN QUESTION]`

A hypothesis or unresolved behavior requiring additional evidence or replication.

Firmware research frequently produces plausible explanations before those explanations are actually demonstrated.

One purpose of this repository is therefore to preserve the distinction between:

- what happened;
- what was measured;
- what was reported;
- and what the available evidence currently suggests.

---

# 3. Investigated platform

The investigated system is a:

**Fujitsu FUTRO S930**

with a:

**Fujitsu D3313-E1 mainboard**

Relevant hardware included:

- AMD GX-424CC quad-core embedded processor;
- Fujitsu D3313-E1 mainboard;
- AMI Aptio-based firmware;
- 8 MiB Macronix SPI flash;
- physically identified MX25L6473F-family flash device;
- Realtek RTL8111/8168G-family Gigabit Ethernet controller;
- AMD FCH SATA controller;
- ASMedia ASM1061/ASM1062 SATA controller;
- 240 GB Kingston SATA SSD;
- 16 GB mSATA SSD;
- Debian Linux.

The firmware version identified during the investigation was:

```text
V4.6.5.4 R1.16.0 for D3313-E1x
```

Machine-specific values including serial number, PC ID, UUID, MAC address, HDCP-related data, and other manufacturing identifiers are intentionally redacted from the public repository.

---

# 4. Initial failure state

The system was already non-functional when the structured investigation began.

According to the history supplied with the machine, a processor repasting attempt had taken place shortly before the failure.

During reassembly, the metal retention plate behind the mainboard had reportedly shifted from its correct position. The cooling assembly was subsequently pressed and repositioned several times.

This information is classified as:

```text
[REPORTED HISTORY]
```

rather than direct experimental evidence.

Because mechanical stress had occurred, the investigation could not safely assume that firmware was the sole cause.

Initial possible failure mechanisms included:

- PCB damage;
- damaged solder joints;
- processor-area component damage;
- VRM or power-rail failure;
- short circuit;
- mechanical deformation;
- corrupted SPI contents;
- invalid firmware state;
- or multiple simultaneous faults.

Accessible board-level rails and test points were measured repeatedly.

No single obvious electrical fault was identified that independently explained the complete failure to boot.

This did **not** prove that the motherboard was mechanically intact.

It did justify continuing the firmware investigation while keeping hardware-related hypotheses open.

---

# 5. External SPI acquisition

Initial firmware acquisition was performed with a low-cost WCH CH341A USB SPI programmer and a SOIC8 test clip.

Programmer identification:

```text
VID: 1A86
PID: 5512
Device: WCH CH341A
```

Windows-based testing involved multiple drivers and programming utilities.

USB access using `flashrom` was unreliable in this environment and produced errors including:

```text
LIBUSB_ERROR_NOT_SUPPORTED
```

IMSProg successfully detected:

```text
WCH CH341A
Device rev 3.0.4
```

The largest practical problem was not the programmer itself but the mechanical reliability of the SOIC8 clip.

The clip could move or lose electrical contact with the flash package.

For this reason, one apparently successful read was explicitly considered insufficient evidence.

---

# 6. Duplicate acquisition and integrity verification

Two independent complete SPI acquisitions were performed.

Each image had a size of:

```text
8,388,608 bytes
```

corresponding to an 8 MiB SPI flash.

Cryptographic SHA256 hashes were independently calculated.

Both acquisitions produced:

```text
7E138DA4B68FB7C86DBD841631D6D60937AF2EA109297A51476610C8F01E3603
```

The matching digest demonstrates that the two independently acquired images were bit-for-bit identical.

The hash is published as a provenance and reproducibility identifier.

The personalized SPI image itself is intentionally not publicly distributed because it contains machine-specific manufacturing and identity information.

The acquisition workflow became:

```text
READ
  ↓
READ AGAIN
  ↓
COMPARE
  ↓
HASH
  ↓
ANALYZE
  ↓
ONLY THEN WRITE
```

This principle was later repeated when acquiring the post-recovery firmware internally.

---

# 7. Comparison with official Fujitsu firmware

The corresponding official Fujitsu R1.16 firmware was obtained separately.

The central question was:

> **How does the firmware stored in one manufactured D3313-E1 differ from the generic firmware distributed by Fujitsu?**

Instead of treating the entire 8 MiB image as one opaque binary object, both images were divided into 64 KiB blocks.

An 8 MiB image contains:

```text
128 × 64 KiB blocks
```

The result was unexpectedly clear.

---

# 8. Major structural finding: 127 of 128 blocks matched

The original machine SPI dump and the official Fujitsu R1.16 firmware matched in:

**127 of 128 blocks.**

Only one 64 KiB region differed:

```text
0x440000–0x44FFFF
```

This fundamentally changed the direction of the investigation.

The individual machine firmware was not broadly different from the factory release.

Instead, virtually the entire 8 MiB image matched the official firmware.

Machine-specific customization was concentrated in one small region.

A second verification test was performed.

A copy of the official Fujitsu R1.16 image was created, and only:

```text
0x440000–0x44FFFF
```

was replaced with the equivalent block from the original machine dump.

The resulting complete 8 MiB image became bit-for-bit identical to the original machine SPI image.

The relationship can therefore be represented as:

```text
Official Fujitsu R1.16 firmware
            +
original machine-specific 64 KiB OEM block
            =
complete original machine SPI image
```

For the investigated system, this is one of the strongest structural findings of the project.

---

# 9. UEFI Firmware Volume and FFS analysis

The firmware was examined at UEFI Firmware Volume and Firmware File System level.

The analyzed image contained:

```text
8 valid Firmware Volumes
84 FFS files
0 invalid FV headers
0 invalid FFS checksums
```

This provided additional evidence that the acquired image was internally coherent.

Machine-specific differences were concentrated in structures beginning approximately at:

```text
0x440000
```

and:

```text
0x443000
```

Observed FREEFORM FFS GUIDs included:

```text
0972EB29-E62F-4699-A19C-664AC3B7208B
```

and:

```text
8EEAF5CF-1C81-4FCC-AE1E-D8AD3CD15683
```

The two structures were highly similar.

They appear to contain redundant or parallel representations of machine-specific manufacturing information.

The precise redundancy model is not yet fully understood.

Possible explanations include:

- primary and backup copies;
- active and standby records;
- generation-based copies;
- versioned manufacturing records;
- integrity/fallback structures;
- or another vendor-specific mechanism.

This remains an open research direction.

---

# 10. OEM/manufacturing data structures

Recognizable ASCII field names inside the machine-specific region included:

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

Some fields are straightforward to interpret.

Others remain partially or completely undocumented.

The critical architectural observation is that official Fujitsu firmware contains template or unpersonalized values in several locations, while the machine firmware contains populated manufacturing information.

This supports the model:

```text
generic vendor firmware
          +
manufacturing personalization
          =
firmware state of an individual machine
```

A public vendor firmware image should therefore not automatically be interpreted as a complete reconstruction of a specific manufactured system.

---

# 11. Discovery of the machine MAC field

Both OEM structures contained:

```text
MAC Address #0
```

followed by six bytes.

In the original machine dump, both copies contained the same machine-specific Ethernet address.

The actual value is represented publicly as:

```text
<REDACTED_FUJITSU_MAC>
```

The equivalent field in the official R1.16 firmware contained:

```text
FF FF FF FF FF FF
```

or:

```text
FF:FF:FF:FF:FF:FF
```

The official firmware therefore did not contain a generic Fujitsu MAC.

It contained an **unprogrammed manufacturing-template value**.

The individual Ethernet identity had been introduced during system manufacturing or personalization.

---

# 12. Privacy and forensic implications of SPI dumps

A raw SPI image should not automatically be treated as a generic “BIOS file.”

It may contain:

- serial numbers;
- PC IDs;
- UUIDs;
- MAC addresses;
- HDCP-related information;
- platform identifiers;
- configuration data;
- manufacturing metadata;
- and other machine-specific values.

For this reason, the public repository does **not** publish:

- the original personalized SPI dump;
- a personalized recovery ROM;
- real serial numbers;
- real PC IDs;
- full UUIDs;
- the complete machine MAC;
- HDCP-related data;
- other machine-specific manufacturing secrets.

Only sanitized examples, structural descriptions, hashes, offsets, and appropriately redacted material should be made public.

---

# 13. Fujitsu BootStick investigation

The Fujitsu BootStick mechanism was examined separately.

The utility produced a conventional FreeDOS-based boot device containing:

- MBR;
- one active FAT32 partition;
- FreeDOS boot code;
- `KERNEL.SYS`;
- `COMMAND.COM`.

A newly generated BootStick did not contain an automatic BIOS flashing command in its default `AUTOEXEC.BAT`.

This helped separate two concepts:

1. the generic boot environment created by Fujitsu BootStick;
2. the actual board-specific firmware recovery logic.

A 2 GB USB flash drive was ultimately used during the successful recovery.

A rear USB 2.0 port was selected to minimize variables.

No claim is made that this specific port is electrically shorter, uniquely required, or officially designated as a firmware-recovery port.

It was a pragmatic compatibility choice.

---

# 14. Recovery jumper configuration

The observed jumper arrangement on the investigated D3313-E1 was:

```text
5     3     1
●     ●     ●

●           ●
6    [ ]    2
      ↑
 missing pin
 position 4
```

Observed positions:

```text
DEFAULT   J5–3
RESERVED  J1–2
RECOVERY  J5–6
```

During an earlier troubleshooting phase, the board was accidentally powered several times in the RESERVED position.

No irreversible damage attributable to this state was observed on this specimen.

This must **not** be interpreted as evidence that the RESERVED position is safe.

Its function has not been fully reverse engineered, and its use is not recommended.

---

# 15. Successful recovery configuration

Successful firmware recovery was performed with a deliberately minimal hardware configuration.

Connected components included:

- mainboard;
- processor;
- one RAM module in DIMM1;
- CMOS battery;
- buzzer;
- display;
- keyboard;
- recovery USB device.

Non-essential storage devices and peripherals were removed.

The jumper was placed in the RECOVERY position.

During recovery, periodic short beeps were observed.

At apparent completion, the observed repeating pattern was:

```text
long – long – short – short
```

The machine was then:

1. powered off;
2. disconnected from power;
3. recovery media removed;
4. jumper returned to DEFAULT.

On the next normal boot, video output returned and BIOS Setup became accessible.

The previously non-working board was POSTing again.

The BIOS recovery itself had succeeded.

---

# 16. Post-recovery network identity failure

After POST was restored, a new anomaly became visible.

BIOS displayed the Realtek interface MAC as:

```text
00:00:00:00:00:00
```

Debian nevertheless booted and networking worked.

The interface negotiated a physical link, obtained network configuration, and was accessible over SSH.

Linux logs showed that the problem was not merely cosmetic.

The `r8169` driver reported:

```text
can't read MAC address, setting random one
```

and `ethtool` reported:

```text
Permanent address: not set
```

The operating system therefore did not receive a valid permanent hardware MAC.

Networking remained possible only because Linux generated a replacement address.

The computer had regained connectivity but lost its original hardware network identity.

---

# 17. Separating OS behavior from hardware state

The next question was:

> **Is this only a Linux driver problem?**

To test this, diagnostics were moved outside the normal Debian network stack.

A DOS-based Realtek diagnostic utility was used to inspect the controller independently.

---

# 18. Realtek RSET8168 diagnostics

Realtek's DOS utility:

```text
RSET8168 v1.24
```

was booted from FreeDOS.

Its status display reported:

```text
Ethernet Address: 00 00 00 00 00 00
```

The MAC register view also contained zero values.

The missing identity was therefore already absent outside Linux.

---

# 19. EEPROM diagnostic result

RSET8168 reported:

```text
No Valid External EEPROM!!
```

followed by:

```text
Data from EFuse
```

and:

```text
EFuse Fail
```

A new hypothesis became plausible.

If no valid external EEPROM existed, perhaps the system normally stored the Ethernet identity in the RTL8168G internal eFuse.

If so, direct Realtek programming might have provided a repair route.

At this stage, however, that explanation was still unproven.

---

# 20. Direct Realtek eFuse inspection

A Linux Realtek PGTool-compatible tool was used to inspect the controller directly.

The toolset included:

```text
8168GEF.cfg
```

for the RTL8168G-family configuration.

Its custom kernel module used an obsolete Linux API.

The original code:

```c
vma->vm_flags |= VM_IO;
```

was changed for compatibility with the Debian 6.12 kernel to:

```c
vm_flags_set(vma, VM_IO);
```

After this modification, the module compiled and loaded.

The normal `r8169` driver was unloaded and the controller was accessed through the PGTool driver.

---

# 21. Critical eFuse result

The readable eFuse area contained only:

```text
FF
```

values.

More importantly, the tool reported:

```text
Efuse Write Count = 0
```

while essentially the complete programmable eFuse capacity remained available.

This distinction is important.

A region containing only `FF` values might theoretically represent erased or absent data.

A reported write count of zero, however, strongly suggests that the region had **never been programmed**.

The EEPROM path also reported:

```text
Not Support This Adapter
```

Taken together, the evidence argued strongly against the idea that the original Fujitsu MAC had previously been stored permanently in Realtek EEPROM or eFuse.

A new model became substantially more plausible:

> **The Fujitsu firmware initializes the Realtek controller from machine-specific platform data stored in the motherboard SPI flash.**

A direct comparison with the physical post-recovery SPI contents was still required.

---

# 22. Why the Realtek eFuse was not programmed

The PGTool environment supported programming a node ID/MAC into non-volatile controller storage.

The original machine MAC was known.

The eFuse appeared blank.

Programming it could therefore have produced a fast workaround.

That option was deliberately rejected.

eFuse programming is:

- persistent;
- different from ordinary rewritable flash;
- constrained by finite programmable capacity;
- potentially irreversible;
- and was not demonstrated to be part of the original Fujitsu design.

The goal was not simply:

> “make the NIC have a permanent MAC again.”

The goal was:

> **restore the mechanism used by the original machine.**

Realtek eFuse programming therefore remained a fallback rather than the chosen repair.

---

# 23. Internal SPI access under Linux

By this stage, the external SOIC8 clip cable had become unusable.

Because the computer was now booting, internal SPI access was tested under Debian.

Using:

```text
flashrom -p internal
```

the utility identified the:

```text
AMD FCH
```

chipset.

Several compatible Macronix definitions were proposed.

Based on the physically identified flash package, the explicit device:

```text
MX25L6473F
```

was selected.

Two complete internal 8 MiB acquisitions were performed.

Their SHA256 hashes matched.

The acquisition principle was therefore reproduced:

```text
two independent reads
        +
matching cryptographic hashes
        =
trusted post-recovery SPI baseline
```

A reliable image of the physical post-recovery flash state was now available.

---

# 24. Decisive post-recovery OEM finding

The two:

```text
MAC Address #0
```

structures were located again in the post-recovery SPI dump.

Both were still structurally present.

Their associated six-byte values, however, were now:

```text
FF FF FF FF FF FF
```

exactly matching the official Fujitsu R1.16 template.

The three relevant states could therefore be compared.

### Original machine SPI

```text
MAC Address #0 → <REDACTED_FUJITSU_MAC>
```

### Official Fujitsu R1.16

```text
MAC Address #0 → FF:FF:FF:FF:FF:FF
```

### Physical SPI after Fujitsu recovery

```text
MAC Address #0 → FF:FF:FF:FF:FF:FF
```

This observation connected the previously separate symptoms:

- all-zero BIOS MAC;
- zero Ethernet address in RSET8168;
- Linux `Permanent address: not set`;
- generated Linux replacement MAC;
- apparently unused Realtek eFuse.

The controller had not necessarily lost an internally stored MAC.

The Fujitsu platform firmware no longer contained the machine-specific value required to initialize it.

---

# 25. Implications for the Fujitsu recovery mechanism

Care is necessary when interpreting the result.

The evidence does **not** establish the simplified claim:

> “Fujitsu recovery simply overwrites the OEM region with the factory image.”

The actual finding is narrower.

The recovery source firmware contained personalized data.

Nevertheless, after recovery the physical SPI contained factory-template values in the relevant OEM structures.

Therefore the recovery process was **not a simple bit-for-bit full-chip copy of the supplied 8 MiB source image**.

Some additional recovery behavior exists.

Possible models include:

- selective Firmware Volume writing;
- manufacturing-region regeneration;
- omission of selected personalized structures;
- special treatment of OEM regions;
- reconstruction from an internal component;
- or another board-specific policy.

The precise implementation remains:

```text
[OPEN QUESTION]
```

---

# 26. Why the complete historical SPI image was not restored

A complete original 8 MiB dump was available.

The simplest apparent repair would therefore have been to write the entire historical image back to flash.

That approach was deliberately rejected.

Comparing the original and post-recovery images showed differences in several 64 KiB blocks, including:

```text
2
54
64
65
66
67
68
```

Large differences existed in regions associated with mutable firmware state.

A running UEFI environment may legitimately modify:

- boot variables;
- NVRAM records;
- counters;
- runtime configuration;
- device state;
- recovery state.

A full historical rewrite would therefore have reverted unrelated runtime state along with the missing machine identity.

The guiding principle became:

> **Preserve the currently working firmware and restore only the data demonstrated to be missing.**

---

# 27. Construction of the repair candidate

The trusted post-recovery SPI image was used as the base.

Only:

```text
0x440000–0x44FFFF
```

was replaced with the equivalent 64 KiB block from the original pre-recovery machine dump.

The resulting candidate was designated:

```text
D3313_REPAIRED_CANDIDATE.bin
```

A byte-level comparison against the current post-recovery image showed:

```text
3526 differing bytes
```

All differences were confined to:

```text
one 64 KiB block
```

namely the identified OEM/manufacturing region.

The working recovery firmware remained intact.

Only the personalized OEM region differed.

---

# 28. Pre-write physical verification

A `flashrom` layout was defined for the OEM region:

```text
0x440000:0x44ffff OEM
```

Before writing, the candidate was verified against the physical flash.

The verification reported:

```text
3526 differing bytes
```

This exactly matched the independent file-level comparison.

Two separate methods therefore agreed:

```text
file-level binary comparison  → 3526 bytes
physical flash verification   → 3526 bytes
```

Only after this consistency check was the physical write performed.

---

# 29. Selective 64 KiB restoration

The complete 8 MiB SPI flash was **not** rewritten.

Only:

```text
0x440000–0x44FFFF
```

was selected.

`flashrom` reported:

```text
Erase/write done from 440000 to 44ffff
```

followed by:

```text
Verifying flash... VERIFIED.
```

A separate verification operation was then performed.

It again returned:

```text
VERIFIED
```

This second verification was intentional.

The success message immediately following a flash write was not considered sufficient by itself.

The physical device was explicitly checked against the intended repair state.

---

# 30. Restoration result

The machine was fully powered down and cold booted.

BIOS Setup again displayed the original Fujitsu LAN identity.

The value remains represented publicly as:

```text
<REDACTED_FUJITSU_MAC>
```

After Debian booted, Linux also used the restored original hardware identity.

The previous condition:

```text
Permanent address: not set
```

was no longer present.

The evidence chain was complete.

---

# 31. Resulting evidence chain

```text
Original machine SPI
        ↓
machine-specific MAC present in Fujitsu OEM region
        ↓
official Fujitsu firmware
        ↓
same field contains FF:FF:FF:FF:FF:FF template
        ↓
Fujitsu recovery restores POST
        ↓
BIOS MAC becomes 00:00:00:00:00:00
        ↓
Linux reports no permanent MAC
        ↓
RSET8168 confirms zero address outside Linux
        ↓
no valid external EEPROM found
        ↓
Realtek eFuse read directly
        ↓
blank FF state + Efuse Write Count = 0
        ↓
post-recovery SPI read internally
        ↓
Fujitsu OEM MAC field contains FF template
        ↓
original 64 KiB OEM block restored
        ↓
physical write independently verified
        ↓
cold boot
        ↓
original Fujitsu MAC returns in BIOS
        ↓
Linux permanent identity restored
```

Taken together, these observations strongly support the interpretation:

> **On the investigated D3313-E1, the Realtek RTL8168G-family controller receives its machine-specific Ethernet identity from Fujitsu platform firmware during initialization rather than from a previously programmed controller eFuse.**

This conclusion applies to the investigated specimen.

Replication on additional D3313 systems is required before generalizing the result to the entire platform family.

---

# 32. What the investigation demonstrated

The available evidence supports the following findings:

1. the non-working D3313-E1 could be returned to POST through Fujitsu recovery;
2. duplicate independent SPI acquisition with cryptographic verification created a trustworthy baseline;
3. original machine firmware and official Fujitsu R1.16 were identical in 127 of 128 64 KiB blocks;
4. the principal machine-specific personalization was concentrated in one 64 KiB OEM/manufacturing region;
5. that region contained multiple machine-specific fields;
6. the official firmware used an `FF` template value for the LAN MAC;
7. the original machine dump contained the actual Fujitsu MAC in the same field;
8. after recovery, the physical SPI returned to the factory-template MAC state;
9. the Realtek controller exposed no valid external EEPROM;
10. its eFuse appeared never to have been programmed;
11. the investigated machine's LAN identity therefore did not originate from programmed Realtek eFuse storage;
12. the complete post-recovery SPI could be acquired internally under Linux;
13. the exact missing OEM region could be isolated;
14. only that 64 KiB region could be restored;
15. the physical write could be independently verified;
16. restoring the Fujitsu OEM region restored the original LAN identity in both BIOS and Linux.

---

# 33. What the investigation does not demonstrate

This project does **not** establish that:

- every D3313-E1 behaves identically;
- every FUTRO S930 stores network identity in exactly the same way;
- all Fujitsu recovery procedures remove personalized MAC information;
- the complete recovery algorithm has been reverse engineered;
- the RESERVED jumper position is safe;
- the selected USB port is an official recovery requirement;
- every RTL8168G on this platform uses an unprogrammed eFuse;
- the unresolved mSATA problem is firmware-related;
- or the underlying firmware-analysis methods originated with this project.

Claims are intentionally limited to the available evidence.

---

# 34. Separate unresolved issue: 16 GB mSATA

After OEM identity restoration was completed, the installed 16 GB mSATA SSD was investigated separately.

Linux successfully detects:

```text
AMD FCH AHCI
ASMedia ASM1061/ASM1062 AHCI
```

The 240 GB Kingston SATA SSD is detected reliably and negotiates:

```text
6.0 Gbps
```

The 16 GB mSATA device, however:

- does not appear in `lsblk`;
- never reaches ATA device identification;
- produces no recognized block device;
- leaves the relevant SATA path in `SATA link down` state.

Restoring the original Fujitsu OEM block did not alter this behavior.

There is therefore currently no evidence that the mSATA issue is part of the recovered firmware problem.

Possible causes include:

- failed mSATA SSD;
- connector/contact failure;
- missing socket power;
- damaged SATA signal path;
- board-level damage;
- another configuration mechanism not yet identified.

This remains:

```text
[OPEN QUESTION]
```

and is intentionally documented separately from the completed firmware recovery.

---

# 35. Separate hardware incident: movable SATA adapter

During testing, the adapter carrying the Kingston SATA SSD moved while the machine was operating.

Linux immediately logged:

- ATA bus errors;
- BadCRC;
- handshake/link failures;
- I/O errors.

The filesystem was consequently capable of becoming read-only.

After physically stabilizing the adapter and rebooting, the errors disappeared.

This incident is distinct from the unresolved mSATA issue.

It does demonstrate that the improvised adapter mounting is unsuitable for permanent operation.

The intended mechanical solution is appropriate:

```text
M2 nylon/plastic standoffs
approximately 6 mm
```

with proper fasteners.

---

# 36. Minimum necessary modification

A central methodological principle emerged:

> **Do not rewrite more firmware than the evidence demonstrates must be changed.**

In this case:

- a complete original SPI image existed;
- a complete working post-recovery image existed;
- the personalized region had been identified;
- only 3526 bytes differed within one 64 KiB block;
- the rest of the working firmware could be preserved.

The final operation modified:

```text
64 KiB out of an 8 MiB flash
```

This reduced unnecessary modifications to:

- NVRAM;
- boot variables;
- runtime state;
- recovery-generated data;
- unknown firmware structures.

The same principle may be useful in other firmware-repair investigations.

---

# 37. Reproducibility

The strongest findings should ideally be reproduced on additional hardware.

A valuable replication experiment would use a second D3313-E1 and compare:

- OEM/manufacturing block location;
- FFS GUIDs;
- duplicated structure layout;
- template values;
- machine MAC storage;
- Realtek eFuse state;
- Fujitsu recovery behavior;
- post-recovery OEM state;
- selective OEM restoration result.

Replication could determine whether these observations are:

- unique to one specimen;
- characteristic of D3313-E1;
- or common across a wider D3313 family.

---

# 38. Limitations

## 38.1. Single primary specimen

The complete recovery investigation was performed on one D3313-E1 system.

Population-level generalization is therefore not justified.

## 38.2. Recovery algorithm not fully reverse engineered

The observable result of Fujitsu recovery is documented, but its complete internal implementation remains unknown.

## 38.3. Partially undocumented OEM fields

Several machine-specific field names were identified, but many exact functions remain unknown.

## 38.4. No destructive redundancy testing

The apparently redundant OEM structures were not intentionally damaged to determine fallback behavior.

## 38.5. mSATA issue unresolved

The non-detected 16 GB mSATA SSD has not yet been conclusively cross-tested.

## 38.6. Privacy restrictions

The complete personalized firmware cannot responsibly be published because it contains machine-specific information.

---

# 39. Future research directions

## 39.1. Fujitsu recovery write-map reconstruction

One of the strongest open questions is why personalized data present in the recovery source did not remain in physical SPI after recovery.

Controlled test images could potentially reveal:

- which regions are copied;
- which are ignored;
- which are reconstructed;
- which are preserved;
- whether manufacturing-specific rules exist.

A formal **D3313 recovery write map** could eventually result.

---

## 39.2. Reverse engineering the OEM/manufacturing structure

The 64 KiB region appears to contain substantially more than the LAN MAC.

Comparing anonymized dumps from multiple machines could distinguish:

- invariant fields;
- model-dependent fields;
- board-specific fields;
- machine identifiers;
- counters;
- integrity fields;
- feature flags;
- security-related fields.

---

## 39.3. Redundant OEM structure behavior

Questions remain concerning the two related FFS structures:

- Is one primary?
- Is one a backup?
- Is there a generation counter?
- Is one selected using a validity flag?
- Can firmware reconstruct one from the other?
- What occurs if only one record is valid?

A spare platform would enable controlled experiments without risking the recovered machine.

---

## 39.4. UEFI module responsible for NIC initialization

Restoration of the OEM block restored the Realtek identity.

The next question is:

> **Which firmware component reads Fujitsu OEM data and initializes the Realtek controller?**

Possible future approaches include:

- DXE module analysis;
- PCI initialization tracing;
- GUID/string correlation;
- MMIO register tracing;
- emulator-assisted analysis;
- boot-time instrumentation.

---

## 39.5. Other hardware initialized from OEM data

The MAC result raises a broader architectural question.

If network identity is initialized from the OEM region, other features may be as well.

Potential targets include:

- SATA/mSATA behavior;
- TPM configuration;
- HDCP information;
- feature enablement;
- licensing data;
- PCIe device configuration;
- platform identity.

---

## 39.6. Comparison of D3313 E, B, and S variants

The D3313 family exists in multiple variants.

Cross-platform comparison could determine:

- whether the OEM FFS GUIDs are shared;
- whether the manufacturing region occupies the same offsets;
- whether field structures are compatible;
- whether recovery behavior is similar;
- which documentation can safely be generalized.

---

## 39.7. Automated firmware differencing

Much of the analysis can potentially be automated.

A future utility could perform:

```text
image-size verification
SHA256 hashing
block-level comparison
FV validation
FFS validation
OEM string detection
machine-specific field classification
privacy-risk detection
flashrom layout generation
```

Such tooling could make similar investigations faster and less error-prone.

---

## 39.8. Firmware privacy sanitizer

Because raw SPI images may contain machine identity, a dedicated sanitizer could detect or redact:

- MAC addresses;
- serial numbers;
- UUIDs;
- PC IDs;
- manufacturing identifiers;
- other known sensitive structures.

A privacy-aware firmware-sharing tool could be useful beyond the D3313 family.

---

## 39.9. Recovery acoustic signal analysis

Recovery produced identifiable beep sequences.

Recording complete sequences with accurate timing could potentially correlate acoustic patterns with:

- device detection;
- erase;
- programming;
- verification;
- success;
- failure.

This could be valuable where no video output is available.

---

## 39.10. Hardware-versus-firmware fault isolation

The original failure was useful as a diagnostic case because both mechanical damage and firmware corruption were plausible.

The resulting methodology was approximately:

```text
physical inspection
        ↓
multimeter measurements
        ↓
external SPI acquisition
        ↓
duplicate-read verification
        ↓
factory comparison
        ↓
UEFI structural analysis
        ↓
recovery
        ↓
OS-level diagnostics
        ↓
DOS hardware diagnostics
        ↓
direct controller NVM inspection
        ↓
internal SPI acquisition
        ↓
minimal selective restoration
```

This sequence may be useful as a general framework where the origin of an embedded-system failure is uncertain.

---

# 40. Broader firmware-forensics significance

One broader lesson of the case is:

> **“factory BIOS image” and “complete firmware state of an individual machine” are not necessarily the same thing.**

A manufactured system may contain multiple layers:

```text
generic vendor firmware
machine-specific manufacturing data
runtime NVRAM
hardware identity
licensing information
security configuration
platform-specific calibration
```

Writing a generic vendor image may restore boot functionality while still removing characteristics that belong to the individual machine.

In this case, the visible lost characteristic was the LAN identity.

On other systems, machine-specific data might include:

- serial numbers;
- UUIDs;
- OEM licensing information;
- calibration data;
- TPM configuration;
- board-specific feature flags;
- network identities.

Firmware repair should therefore distinguish:

**bootability**

from:

**complete machine-state restoration**.

---

# 41. Safety considerations

SPI flash modification can render hardware unbootable.

The methodology used here strongly favors:

- multiple independent reads;
- cryptographic hashes;
- immutable archival copies of original dumps;
- exact flash-device identification;
- pre-write verification;
- smallest-possible write scope;
- independent post-write verification.

OTP/eFuse programming should be treated differently from ordinary rewritable flash.

Before modifying irreversible or limited-write storage, the operator should understand:

- what data is being programmed;
- whether the operation can be reversed;
- whether the target originally used that mechanism;
- available programming capacity;
- whether a less invasive recovery path exists.

For the complete legal and safety notice, see:

[DISCLAIMER.md](DISCLAIMER.md)

Hungarian version:

[DISCLAIMER.hu.md](DISCLAIMER.hu.md)

---

# 42. Repository privacy policy

The public repository must not contain unredacted:

- personalized raw SPI images;
- complete machine serial numbers;
- PC IDs;
- UUIDs;
- full machine MAC addresses;
- HDCP or comparable machine-specific secrets;
- private operating-system usernames;
- private local filesystem paths;
- unnecessary EXIF/GPS metadata;
- readable labels containing identifying information;
- QR codes or barcodes containing device identity.

Screenshots, photographs, videos, audio recordings, firmware excerpts, and log files should undergo privacy review before publication.

Raw personalized firmware should not be committed directly.

---

# 43. Repository structure

The project is intended to evolve toward a structure such as:

```text
/
├── README.md
├── README.hu.md
├── LICENSE
├── CITATION.cff
├── SOURCES.md
├── DISCLAIMER.md
├── DISCLAIMER.hu.md
├── docs/
│   ├── en/
│   └── hu/
├── scripts/
├── examples/
└── media/
    └── public/
```

Personalized firmware images must not be placed in the public tree.

---

# 44. Prior work and attribution

This project does **not** claim invention of the underlying firmware-recovery, UEFI-analysis, SPI-programming, or Realtek diagnostic techniques.

The work builds upon manufacturer documentation, open-source tooling, and prior community research.

Relevant categories include:

- Fujitsu D3313 documentation;
- Fujitsu BIOS and recovery documentation;
- Kontron documentation for related D3313 platforms;
- the `flashrom` project;
- Realtek RSET8168;
- Realtek PGTool documentation and tooling;
- community work on Fujitsu FUTRO systems;
- prior D3313 firmware-analysis projects;
- general UEFI reverse-engineering and repair research.

Direct references and attribution are collected in:

[SOURCES.md](SOURCES.md)

The contribution of this project is the:

> **documented combination, verification, and application of these methods to one D3313-E1 recovery case together with the resulting platform-specific observations.**

---

# 45. Research contribution

The principal contribution is not merely that one computer was repaired.

The project establishes a documented chain connecting:

```text
official firmware
        ↕
personalized machine firmware
        ↕
manufacturing data
        ↕
firmware recovery
        ↕
loss of platform identity
        ↕
controller-level diagnostics
        ↕
direct NVM inspection
        ↕
post-recovery physical SPI state
        ↕
minimal selective restoration
        ↕
restored machine identity
```

Potentially reusable observations include:

- a concrete location for D3313-E1 machine-specific data;
- evidence that the official firmware contains personalization templates;
- evidence that the investigated Realtek eFuse was never programmed;
- evidence that platform firmware can provide NIC identity;
- evidence that Fujitsu recovery is not a simple raw full-chip copy;
- a demonstrated minimum-region restoration method.

These observations may be useful to researchers studying:

- D3313 firmware architecture;
- embedded-system personalization;
- firmware identity storage;
- recovery mechanisms;
- EEPROM-less PCIe initialization;
- BIOS forensics;
- industrial and legacy system preservation.

---

# 46. Conclusion

At the beginning of the investigation, it was not possible to determine whether the FUTRO S930 had:

- mechanical board damage;
- processor-area damage;
- power-delivery failure;
- SPI firmware corruption;
- or multiple simultaneous problems.

The final state is substantially better understood.

The system now:

- POSTs successfully;
- exposes a functional BIOS Setup;
- boots Debian Linux;
- operates over Ethernet;
- contains the restored machine-specific Fujitsu OEM region;
- initializes the original Fujitsu LAN identity.

The post-recovery MAC failure was **not** repaired by programming Realtek eFuse storage.

Direct inspection indicated that the eFuse apparently had never been programmed.

Instead, the decisive machine identity was located in the Fujitsu SPI/OEM region.

The final recovery therefore consisted of:

```text
trusted post-recovery SPI acquisition
        ↓
comparison against original firmware
        ↓
isolation of one 64 KiB machine-specific region
        ↓
pre-write physical verification
        ↓
selective in-system flash write
        ↓
independent post-write verification
        ↓
cold boot
        ↓
restoration of original platform identity
```

The separate 16 GB mSATA detection problem remains open and is not currently considered part of the completed BIOS/OEM recovery.

The BIOS recovery and restoration of Fujitsu machine-specific OEM identity can therefore be considered:

# **SUCCESSFULLY COMPLETED**

The next phase is no longer primarily a repair task.

The recovered platform, accumulated measurements, firmware comparisons, and reproducible evidence chain provide a basis for further research into:

- D3313 firmware architecture;
- manufacturing-data structures;
- machine personalization;
- Fujitsu recovery behavior;
- hardware initialization from platform firmware.

---

# 47. Current research status

```text
BIOS recovery                         COMPLETE
POST restoration                      COMPLETE
Debian boot                            COMPLETE
Internal SPI read                      COMPLETE
OEM region identification              COMPLETE
Original LAN identity restoration      COMPLETE
Selective SPI write verification       COMPLETE
Realtek eFuse investigation            COMPLETE
Fujitsu recovery algorithm             OPEN
OEM structure full reverse engineering OPEN
Second-board reproduction              OPEN
16 GB mSATA investigation              OPEN
```

---

# 48. License

Unless explicitly stated otherwise, original copyrightable research and documentation created specifically for this repository is made available under:

**Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International  
(CC BY-NC-SA 4.0)**

See the full official license:

[LICENSE](LICENSE)

In summary, covered material may be shared and adapted for non-commercial purposes subject to the attribution and ShareAlike requirements of the license.

This summary does **not** replace the official license text.

### Important scope distinction

The repository may also contain or discuss:

- third-party firmware;
- BIOS components;
- vendor utilities;
- third-party documentation;
- trademarks;
- third-party software;
- source code or scripts.

Such material is **not automatically relicensed under CC BY-NC-SA 4.0** merely because it appears in or is referenced by this repository.

Software and source code may use separate licensing terms where explicitly provided.

Third-party material remains subject to the rights and terms of its respective rights holders.

For the full licensing boundaries, legal notice, research-purpose statement, safety provisions, and third-party-rights clarification, see:

[DISCLAIMER.md](DISCLAIMER.md)

Hungarian version:

[DISCLAIMER.hu.md](DISCLAIMER.hu.md)

Separate commercial licensing of eligible original material may be offered by the copyright holder under individually agreed terms.

---

# 49. Citation

If this project, its measurements, structural findings, methodology, or documentation contributes to your research, publication, repair work, technical analysis, or derivative research, please cite the project.

Machine-readable citation metadata is provided in:

[CITATION.cff](CITATION.cff)

Preferred project title:

> **Fujitsu FUTRO S930 / D3313-E1: BIOS Recovery, Firmware Analysis, and Restoration of Machine-Specific OEM Identity**

Author:

**Bencze Péter**

Repository:

```text
https://github.com/benczepeter001/fujitsu-d3313-bios-recovery
```

License for eligible original research/documentation:

```text
CC-BY-NC-SA-4.0
```

---

# 50. Legal and research notice

This is an independent research project.

It is not an official Fujitsu project and is not affiliated with, sponsored by, certified by, approved by, or endorsed by Fujitsu or other vendors mentioned in the repository.

All third-party product names, trademarks, firmware, software, documentation, and other protected material remain the property of their respective rights holders.

Firmware modification, SPI programming, hardware rework, and low-level diagnostic procedures can permanently damage hardware or destroy machine-specific data.

Anyone reproducing similar work is responsible for:

- verifying their own backups;
- understanding the applicable hardware;
- understanding the risks involved;
- complying with applicable laws and licenses;
- working only on systems and data they are authorized to examine.

The authoritative detailed notice is:

[DISCLAIMER.md](DISCLAIMER.md)

The corresponding Hungarian notice is:

[DISCLAIMER.hu.md](DISCLAIMER.hu.md)

---

Copyright © 2026 Bencze Péter and respective contributors for their original contributions.

Third-party copyrights, trademarks, firmware, software, documentation, and other protected materials remain with their respective rights holders.
