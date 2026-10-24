# Fujitsu FUTRO S930 / D3313-E1  
## BIOS Recovery, Firmware Analysis, and Restoration of Machine-Specific OEM Identity

### A forensic-style technical case study of firmware recovery and platform identity reconstruction on a Fujitsu D3313-E1 mainboard

> **Status:** BIOS recovery successful · machine-specific OEM identity restored · original LAN identity restored · separate mSATA issue remains under investigation

[Magyar változat / Hungarian version](README.hu.md)

---

## Abstract

This repository documents a forensic-style investigation and recovery of a Fujitsu FUTRO S930 thin client based on the Fujitsu D3313-E1 mainboard.

The system arrived in a non-working state following a reported processor repasting attempt during which the cooling assembly and rear retention plate had been displaced. Because substantial mechanical pressure may have been applied to the processor area, the initial failure could not safely be attributed to firmware. Mechanical damage, power-delivery faults, board-level failure, corrupted SPI firmware, or a combination of these possibilities all had to remain open hypotheses.

The investigation therefore followed an evidence-driven methodology rather than a conventional “reflash the BIOS and test” approach.

The process included repeated board-level electrical measurements, external SPI acquisition using a CH341A programmer, duplicate 8 MiB firmware reads with cryptographic hash comparison, comparison against official Fujitsu firmware, UEFI Firmware Volume and FFS analysis, Fujitsu recovery-mode experiments, DOS-level Realtek diagnostics, direct Realtek eFuse inspection, internal SPI acquisition under Linux, block-level firmware differencing, and finally a selective 64 KiB in-system SPI restoration.

Several significant findings emerged.

The original machine SPI image and the corresponding official Fujitsu R1.16 firmware were found to be identical in **127 of 128 64 KiB blocks**. The complete machine-specific difference was concentrated in a single 64 KiB OEM/manufacturing region at `0x440000–0x44FFFF`.

That region contained multiple machine-specific fields, including two redundant structures containing `MAC Address #0`. The factory firmware stored a template value of `FF:FF:FF:FF:FF:FF`, while the original machine dump contained the real Fujitsu-assigned MAC address.

After successful Fujitsu BIOS recovery, the system POSTed and booted again, but the network controller had lost its permanent MAC identity. DOS-level Realtek diagnostics found no valid external EEPROM, while direct Linux PGTool inspection showed an apparently unprogrammed eFuse with `Efuse Write Count = 0`.

A post-recovery internal SPI dump then revealed that the machine-specific OEM MAC field had returned to the factory-template `FF` value.

Rather than permanently programming the Realtek eFuse, the recovery-state firmware was preserved and only the original 64 KiB OEM region was restored. The selective write was independently verified at the physical flash level.

After a cold boot, the BIOS once again displayed the machine's original Fujitsu LAN address, and Linux also recovered the corresponding permanent network identity.

The evidence therefore supports a model in which, on the investigated D3313-E1 system, the Realtek controller's machine-specific MAC identity is initialized from Fujitsu platform firmware rather than stored in the controller's own programmed eFuse.

The project also demonstrates a broader firmware-repair principle:

> **Restore the smallest region that has been demonstrated to require restoration, rather than rewriting the entire firmware image by default.**

---

## Keywords

Fujitsu D3313-E1 · FUTRO S930 · BIOS recovery · UEFI · SPI flash · firmware forensics · OEM data · manufacturing data · flashrom · CH341A · MX25L6473F · Realtek RTL8168G · MAC address · eFuse · firmware recovery · forensic firmware analysis

---

# 1. Scope and objectives

This project began as an attempt to repair one apparently failed Fujitsu FUTRO S930.

It evolved into a structured firmware investigation after it became clear that recovery of basic boot functionality did not fully restore the original identity and configuration of the machine.

The central objectives became:

1. determine whether a firmware-related failure could be demonstrated without prematurely excluding mechanical or electrical causes;
2. obtain a reliable copy of the original SPI firmware;
3. compare the original machine firmware against the corresponding official Fujitsu firmware;
4. identify machine-specific regions and determine their function where possible;
5. restore POST and boot capability using the documented recovery mechanism;
6. investigate why the network controller lost its permanent MAC identity after recovery;
7. determine whether that identity originated from Realtek EEPROM/eFuse or Fujitsu platform firmware;
8. restore the original machine-specific configuration while modifying the smallest possible SPI region;
9. preserve a reproducible evidence chain that can be independently reviewed and extended.

This repository is therefore not intended to be a generic BIOS flashing tutorial.

It is a **case study in evidence-driven firmware recovery and platform identity reconstruction**.

---

# 2. Evidence classification

To avoid mixing measured facts with interpretation, observations in this project are conceptually separated into four categories.

### `[OFFICIAL]`

Information obtained from manufacturer documentation, official firmware packages, official utilities, or equivalent primary technical sources.

### `[OBSERVED]`

Information directly measured, read, logged, extracted, or reproduced on the investigated hardware.

### `[INFERRED]`

A conclusion that is strongly supported by multiple observations but is not itself a directly measured value.

### `[OPEN QUESTION]`

A hypothesis or unresolved behavior requiring additional evidence or replication.

This distinction is essential.

Firmware research frequently produces plausible explanations long before those explanations are actually proven. One purpose of this repository is to preserve the boundary between **what happened**, **what was measured**, and **what we currently believe those measurements imply**.

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
- physically identified Macronix MX25L6473F-family device;
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

Machine-specific values such as serial number, PC ID, UUID, MAC address, HDCP-related data, and other manufacturing identifiers are intentionally redacted from the public repository.

---

# 4. Initial failure state

The system was already non-functional when the structured investigation began.

According to the history supplied with the machine, a processor repasting attempt had taken place shortly before the failure.

During reassembly, the metal retention plate behind the mainboard had reportedly shifted from its correct position. The cooling assembly was subsequently pressed and repositioned several times before the correct mechanical arrangement was identified.

This is classified as:

**`[REPORTED HISTORY]`**

rather than direct experimental evidence.

The mechanical history mattered because it prevented an immediate assumption that the BIOS was responsible.

Possible failure mechanisms initially included:

- PCB cracking;
- damaged solder joints;
- damaged processor-area components;
- VRM or power-rail failure;
- short circuit;
- mechanical deformation;
- corrupted SPI flash contents;
- invalid firmware state;
- or multiple simultaneous faults.

Accessible board-level rails and test points were therefore measured with a multimeter on three separate occasions.

No single obvious electrical fault was found that could independently explain the complete failure to boot.

This did **not** prove that the board was mechanically intact.

It did, however, justify continued firmware investigation.

Hardware and firmware hypotheses therefore remained open in parallel.

---

# 5. External SPI acquisition

The initial firmware acquisition was performed using a low-cost WCH CH341A USB SPI programmer with a SOIC8 test clip.

Programmer identification:

```text
VID: 1A86
PID: 5512
Device: WCH CH341A
```

Windows-based experimentation involved several drivers and programming utilities.

USB access through `flashrom` was unreliable in this environment and produced errors including:

```text
LIBUSB_ERROR_NOT_SUPPORTED
```

IMSProg successfully identified the programmer as:

```text
WCH CH341A
Device rev 3.0.4
```

The principal practical problem was not the programmer itself but the mechanical reliability of the SOIC8 clip.

The clip could move or lose contact with the flash package.

For this reason, a single apparently successful read was explicitly considered insufficient evidence.

---

# 6. Duplicate acquisition and integrity verification

Two independent complete SPI acquisitions were performed.

Each image had a size of exactly:

```text
8,388,608 bytes
```

corresponding to the complete contents of an 8 MiB SPI flash device.

Cryptographic SHA256 hashes were calculated independently for both acquisitions.

Both images produced the same SHA256 digest:

```text
7E138DA4B68FB7C86DBD841631D6D60937AF2EA109297A51476610C8F01E3603
```

The matching digest demonstrates that the two independently acquired 8 MiB images were bit-for-bit identical.

The SHA256 value is published as a reproducibility and provenance identifier. It allows future comparisons to determine whether another copy is identical to the investigated original image without publishing or revealing the contents of the personalized firmware itself.

The raw SPI image is intentionally not distributed because it contains machine-specific manufacturing and identity data.

The significance of the duplicate acquisition is methodological.

A marginal SOIC8 connection may produce corrupted reads, but two independently acquired 8 MiB images matching bit-for-bit provide substantially stronger evidence that the physical contents of the flash were captured correctly.

This became the fundamental recovery workflow used throughout the project:

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

This principle was applied again later when the firmware was read internally from the restored system.

---

# 7. Comparison with official Fujitsu firmware

The corresponding official Fujitsu R1.16 firmware image was obtained separately.

The core research question was:

> **How does the firmware stored in one manufactured D3313-E1 differ from the generic firmware distributed by Fujitsu?**

Instead of treating the entire 8 MiB image as one opaque binary object, the images were divided into 64 KiB blocks.

An 8 MiB image contains:

```text
128 × 64 KiB blocks
```

The result was unexpectedly clear.

---

# 8. Major structural finding: 127 of 128 blocks were identical

The original machine SPI dump and the official Fujitsu R1.16 firmware matched in:

**127 of 128 blocks.**

Only one 64 KiB region differed:

```text
0x440000–0x44FFFF
```

This observation significantly changed the direction of the investigation.

The machine firmware was not broadly different from the factory image.

Instead, virtually the entire 8 MiB image matched the official Fujitsu release.

The machine-specific customization was concentrated in one small region.

A second test was then performed.

The official Fujitsu R1.16 image was copied, and only its:

```text
0x440000–0x44FFFF
```

block was replaced with the equivalent block from the original machine SPI dump.

The resulting complete 8 MiB image became bit-for-bit identical to the original machine dump.

This relationship can therefore be expressed as:

```text
Official Fujitsu R1.16 firmware
            +
original machine-specific 64 KiB OEM block
            =
complete original machine SPI image
```

For the investigated system, this is one of the strongest structural findings in the project.

---

# 9. UEFI Firmware Volume and FFS analysis

The firmware was subsequently examined at UEFI Firmware Volume and Firmware File System level.

The analyzed image contained:

```text
8 valid Firmware Volumes
84 FFS files
0 invalid FV headers
0 invalid FFS checksums
```

This provided additional evidence that the acquired image was internally coherent UEFI firmware.

The machine-specific differences were concentrated in two small firmware structures beginning approximately at:

```text
0x440000
```

and:

```text
0x443000
```

Each contained a FREEFORM FFS object.

The first observed FFS GUID was:

```text
0972EB29-E62F-4699-A19C-664AC3B7208B
```

The second:

```text
8EEAF5CF-1C81-4FCC-AE1E-D8AD3CD15683
```

The two structures were highly similar.

They appear to contain redundant or parallel representations of machine-specific manufacturing data.

The precise redundancy model is not yet fully understood.

Possible explanations include:

- primary and backup copies;
- active and standby records;
- generation counters;
- versioned manufacturing records;
- or another integrity/fallback mechanism.

This remains an open research direction.

---

# 10. OEM/manufacturing data structures

Several recognizable ASCII field names were identified inside the machine-specific region.

Observed labels included:

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

Some of these fields are straightforward to interpret.

Others remain partially or completely undocumented.

The key observation was that the official Fujitsu firmware contains unpersonalized or template values in several of these fields, while the original machine dump contains populated machine-specific information.

This supports a broader architectural model:

```text
generic vendor firmware
          +
manufacturing personalization
          =
firmware state of an individual machine
```

The public vendor image should therefore not automatically be interpreted as a complete reconstruction of one specific manufactured system.

---

# 11. Discovery of the machine MAC field

Both OEM structures contained the field:

```text
MAC Address #0
```

followed by six bytes.

In the original machine dump, both copies contained the same machine-specific Ethernet address.

The real address is intentionally represented in this repository as:

```text
<REDACTED_FUJITSU_MAC>
```

The corresponding field in the official Fujitsu R1.16 image contained:

```text
FF FF FF FF FF FF
```

or equivalently:

```text
FF:FF:FF:FF:FF:FF
```

This was an important result.

The official firmware did not contain a generic Fujitsu MAC.

It contained an **unprogrammed manufacturing template value**.

The machine-specific Ethernet identity had therefore been introduced during manufacturing or personalization.

---

# 12. Privacy and forensic implications of raw SPI images

This finding has implications beyond the repair itself.

A raw SPI dump should not be treated as a generic “BIOS file.”

It may contain:

- serial numbers;
- PC IDs;
- UUIDs;
- MAC addresses;
- HDCP-related information;
- platform identifiers;
- configuration data;
- manufacturing metadata;
- and potentially other sensitive fields whose functions have not yet been fully identified.

For that reason, this repository does **not** publish:

- the original personalized SPI dump;
- a personalized recovery ROM;
- real serial numbers;
- real PC IDs;
- full UUIDs;
- the full machine MAC;
- HDCP-related data;
- other machine-specific manufacturing values.

Only sanitized examples, offsets, structural descriptions, and redacted screenshots should be made public.

---

# 13. Fujitsu BootStick analysis

The Fujitsu BootStick mechanism was examined separately.

The utility produced a conventional FreeDOS-based boot device containing:

- an MBR;
- one active FAT32 partition;
- FreeDOS boot code;
- `KERNEL.SYS`;
- `COMMAND.COM`.

A newly created BootStick did not contain an automatic BIOS flashing command in its default `AUTOEXEC.BAT`.

This distinction was useful because it separated two concepts that can easily be conflated:

1. the generic boot environment generated by Fujitsu BootStick;
2. the actual firmware recovery logic associated with the board and BIOS package.

A 2 GB USB flash drive was ultimately used for the successful recovery.

A simple rear USB 2.0 port was selected to minimize variables.

No claim is made that this port is electrically shorter, uniquely required, or officially designated as the recovery port.

It was a pragmatic compatibility decision.

---

# 14. Recovery jumper configuration

The observed jumper layout on the investigated D3313-E1 was:

```text
5     3     1
●     ●     ●

●           ●
6    [ ]    2
      ↑
 missing pin
 position 4
```

Observed operating positions:

```text
DEFAULT   J5–3
RESERVED  J1–2
RECOVERY  J5–6
```

During an earlier troubleshooting stage, the board was accidentally powered several times in the RESERVED position.

No irreversible damage attributable to this state was observed on the investigated board.

This should **not** be interpreted as evidence that the RESERVED setting is safe.

Its actual function has not been reverse engineered and its use is not recommended.

---

# 15. Successful recovery configuration

The successful firmware recovery was performed using a deliberately minimal hardware configuration.

Connected components included:

- mainboard;
- processor;
- one RAM module installed in DIMM1;
- CMOS battery;
- buzzer;
- display;
- keyboard;
- recovery USB device.

Non-essential storage devices and peripherals were removed.

The jumper was placed in the RECOVERY position.

During the procedure, periodic short beeps were observed.

At the apparent completion of the process, a repeating pattern of:

```text
long – long – short – short
```

was heard.

The machine was then:

1. powered off;
2. disconnected from power;
3. recovery media removed;
4. jumper returned to DEFAULT.

On the next normal boot, video output returned and BIOS Setup became accessible.

The previously non-working board was POSTing again.

The BIOS recovery itself had therefore succeeded.

---

# 16. Unexpected post-recovery network identity failure

After POST had been restored, a new anomaly became visible.

The BIOS displayed an all-zero MAC address for the Realtek network interface:

```text
00:00:00:00:00:00
```

Debian nevertheless booted and networking functioned.

The interface negotiated a physical link, received network configuration, and could be accessed over SSH.

At first glance this could have been interpreted as a cosmetic BIOS display problem.

Linux logs demonstrated otherwise.

The `r8169` driver reported:

```text
can't read MAC address, setting random one
```

`ethtool` reported:

```text
Permanent address: not set
```

The operating system therefore did not receive a valid permanent hardware MAC.

Networking continued to work only because Linux generated a replacement address, later stabilized by systemd's MAC policy.

The machine had regained network connectivity but had lost its original hardware network identity.

---

# 17. Separating operating-system behavior from hardware state

A necessary next question was:

> **Is this only a Linux driver problem?**

That hypothesis was tested by moving diagnostics outside the normal Linux network stack.

A DOS-based Realtek diagnostic utility was selected so that the controller could be inspected independently of Debian and `r8169`.

---

# 18. Realtek RSET8168 diagnostics

Realtek's DOS diagnostic utility:

```text
RSET8168 v1.24
```

was booted from a FreeDOS USB device.

The utility directly examined the network controller.

Its status screen reported:

```text
Ethernet Address: 00 00 00 00 00 00
```

The MAC register view also showed zero values.

This was a major evidentiary step.

The missing MAC was not created by the Debian network stack.

The controller already lacked a valid machine identity outside Linux.

---

# 19. EEPROM diagnostic result

The RSET8168 EEPROM diagnostic test reported:

```text
No Valid External EEPROM!!
```

followed by:

```text
Data from EFuse
```

and finally:

```text
EFuse Fail
```

This result suggested a new hypothesis.

If the controller had no valid external EEPROM, perhaps the MAC address was supposed to exist in the RTL8168G internal eFuse.

If the eFuse had been damaged, erased, or lost during recovery, direct Realtek programming might have been a possible repair.

At this stage that explanation was plausible but unproven.

---

# 20. Direct Realtek eFuse inspection

To test the eFuse hypothesis directly, a Linux Realtek PGTool-compatible utility was used.

The toolset included:

```text
8168GEF.cfg
```

for the RTL8168G-family eFuse configuration.

Its custom kernel module used an obsolete Linux API and initially failed to compile on the Debian 6.12 kernel.

The original code:

```c
vma->vm_flags |= VM_IO;
```

was replaced with:

```c
vm_flags_set(vma, VM_IO);
```

After this compatibility modification, the module compiled and loaded successfully.

The normal `r8169` driver was unloaded and the Realtek controller was accessed through the PGTool driver.

---

# 21. Critical eFuse result

A read-only eFuse acquisition showed that the readable region contained only:

```text
FF
```

values.

More importantly, the tool reported:

```text
Efuse Write Count = 0
```

and showed essentially the full eFuse capacity still available.

This distinction is critical.

An eFuse containing only `FF` values could theoretically represent erased or absent data.

However, a reported write count of zero strongly suggests that the region had **never been programmed**.

The EEPROM read path also reported:

```text
Not Support This Adapter
```

The combined evidence therefore argued against the idea that the original Fujitsu MAC had been stored permanently in Realtek EEPROM or eFuse.

A new model became significantly more plausible:

> **The Fujitsu firmware may initialize the Realtek controller from machine-specific platform data stored in the mainboard SPI flash.**

At this point the model remained an inference.

A direct comparison of the post-recovery physical SPI contents was required.

---

# 22. Why the Realtek eFuse was not programmed as a workaround

The PGTool utility supported programming a node ID/MAC into Realtek non-volatile storage.

The original machine MAC was already known.

The eFuse appeared blank.

Programming it could therefore have provided a fast workaround.

That option was deliberately rejected.

The reasons were methodological and technical.

eFuse programming is:

- persistent;
- not equivalent to ordinary rewritable flash;
- constrained by finite programmable capacity;
- potentially irreversible;
- and not demonstrated to be part of Fujitsu's original platform design.

The objective of the project was not simply:

> “make the NIC have a permanent address again.”

The objective was:

> **restore the mechanism used by the original machine.**

The Realtek eFuse path therefore remained a fallback only.

---

# 23. Internal SPI access under Linux

At this stage the external SOIC8 clip cable had become unusable.

Rather than immediately repairing the external programmer setup, internal flash access was tested from the now-booting Debian system.

Using:

```text
flashrom -p internal
```

the utility detected the:

```text
AMD FCH
```

chipset.

Multiple compatible Macronix 8 MiB flash definitions were proposed.

Based on the physical device identification, the explicit flash definition:

```text
MX25L6473F
```

was selected.

Two complete internal 8 MiB reads were performed.

Their SHA256 hashes were identical.

The same evidentiary principle used during the original external acquisition had therefore been reproduced:

```text
two independent reads
        +
matching cryptographic hashes
        =
trusted post-recovery SPI baseline
```

A reliable image of the **physical flash state after Fujitsu recovery** was now available.

---

# 24. Decisive finding: post-recovery OEM MAC state

The two:

```text
MAC Address #0
```

structures were located again in the post-recovery SPI dump.

Both structures were still present.

However, the six-byte values associated with them were now:

```text
FF FF FF FF FF FF
```

This was exactly the same value present in the official Fujitsu R1.16 factory image.

The three states could therefore be compared directly.

### Original machine SPI

```text
MAC Address #0 → <REDACTED_FUJITSU_MAC>
```

### Official Fujitsu R1.16 image

```text
MAC Address #0 → FF:FF:FF:FF:FF:FF
```

### Physical SPI after recovery

```text
MAC Address #0 → FF:FF:FF:FF:FF:FF
```

This observation connected several previously separate symptoms.

It explained:

- the all-zero MAC shown by BIOS;
- the zero Ethernet address shown by RSET8168;
- the Linux `Permanent address: not set` condition;
- the kernel's generated replacement MAC;
- and the apparently blank Realtek eFuse.

The controller had not necessarily “lost” an internally stored MAC.

The Fujitsu platform firmware no longer contained the machine-specific value required to initialize the controller.

---

# 25. Implications for the Fujitsu recovery mechanism

Careful interpretation is necessary.

The evidence does **not** justify the oversimplified statement:

> “Fujitsu recovery simply overwrites the OEM region with the factory image.”

What is directly established is narrower and more interesting.

The recovery source firmware contained the original personalized data.

Yet after recovery, the physical SPI contained factory-template MAC values in the relevant OEM structures.

Therefore, the recovery process was **not a simple bit-for-bit full-chip copy of the supplied 8 MiB source image**.

Some additional recovery logic must exist.

Possible models include:

- selective Firmware Volume writing;
- manufacturing-region regeneration;
- omission of particular personalized structures;
- special treatment of OEM regions;
- reconstruction from an internal recovery component;
- or another board-specific recovery policy.

The precise algorithm remains an:

**`[OPEN QUESTION]`**

and is one of the strongest directions for future research.

---

# 26. Why the complete old SPI image was not restored

A complete original 8 MiB firmware dump was available.

The simplest possible repair would therefore have been to write the entire historical image back to the flash.

That approach was deliberately rejected.

Comparison of the original and post-recovery SPI images showed differences in several 64 KiB blocks, including blocks:

```text
2
54
64
65
66
67
68
```

Large differences existed in areas associated with mutable firmware state.

This is not inherently suspicious.

A running UEFI system may legitimately change:

- boot variables;
- NVRAM records;
- counters;
- runtime configuration;
- device state;
- recovery state.

Writing the complete historical image would therefore have reverted unrelated runtime state along with the machine identity.

The guiding principle became:

> **Preserve the currently working firmware state and restore only the data demonstrated to be missing.**

---

# 27. Construction of the repair candidate

The trusted post-recovery SPI image was used as the base.

Only the region:

```text
0x440000–0x44FFFF
```

was replaced with the equivalent 64 KiB block from the original pre-recovery machine dump.

The resulting image was designated:

```text
D3313_REPAIRED_CANDIDATE.bin
```

A byte-level comparison between the candidate and the current post-recovery image showed:

```text
3526 differing bytes
```

All differences were confined to:

```text
one 64 KiB block
```

namely the identified OEM/manufacturing region.

This was the desired state.

The working recovery firmware, including unrelated runtime changes, remained intact.

Only the personalized OEM region differed.

---

# 28. Pre-write physical verification

A flashrom layout was defined for the OEM region:

```text
0x440000:0x44ffff OEM
```

Before any write was attempted, the repair candidate was verified against the physical flash.

The verification reported:

```text
3526 differing bytes
```

This result exactly matched the independent file-level comparison.

Two separate methods therefore agreed:

```text
file-level binary comparison  → 3526 bytes
physical flash verification   → 3526 bytes
```

This agreement significantly reduced the risk of accidentally writing the wrong region or using the wrong candidate image.

Only after this consistency check was the physical write performed.

---

# 29. Selective 64 KiB SPI restoration

The complete 8 MiB flash was **not** rewritten.

Only:

```text
0x440000–0x44FFFF
```

was selected.

flashrom reported:

```text
Erase/write done from 440000 to 44ffff
```

followed by:

```text
Verifying flash... VERIFIED.
```

A separate verification operation was then executed after the write.

It again returned:

```text
VERIFIED
```

This independent second verification was intentional.

The success message immediately following a flash operation was not considered sufficient on its own.

The physical flash was explicitly re-read and compared against the intended repair state.

---

# 30. Restoration result

The machine was fully powered down and then cold booted.

BIOS Setup now displayed the original Fujitsu LAN MAC address again.

In this repository the real value is represented as:

```text
<REDACTED_FUJITSU_MAC>
```

After Debian booted, the network interface also used the restored original hardware identity.

The previous:

```text
Permanent address: not set
```

condition was no longer present.

This completed the evidence chain.

---

# 31. Resulting evidence chain

The final sequence can be summarized as follows:

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
Fujitsu OEM MAC field now contains FF template
        ↓
only original 64 KiB OEM block restored
        ↓
physical write independently verified
        ↓
cold boot
        ↓
original Fujitsu MAC returns in BIOS
        ↓
Linux permanent identity restored
```

Taken together, these observations provide strong support for the following interpretation:

> **On the investigated D3313-E1, the Realtek RTL8168G-family controller receives its machine-specific Ethernet identity from Fujitsu platform firmware during initialization rather than from a previously programmed controller eFuse.**

This conclusion applies to the investigated hardware.

Replication on additional D3313 systems is required before generalizing it to the entire platform family.

---

# 32. What the investigation demonstrated

The available evidence supports the following findings:

1. the non-working D3313-E1 board could be returned to POST through Fujitsu recovery;
2. duplicate independent SPI acquisition with cryptographic verification produced a reliable firmware baseline;
3. the original machine image and official Fujitsu R1.16 firmware were identical in 127 of 128 64 KiB blocks;
4. the principal machine-specific personalization was concentrated in one 64 KiB OEM/manufacturing region;
5. that region contained multiple machine-specific fields;
6. the official image used an `FF` template value for the LAN MAC field;
7. the original machine dump contained the real Fujitsu MAC in the same field;
8. after recovery, the physical SPI returned to the factory-template MAC state;
9. the Realtek controller did not expose a valid external EEPROM;
10. its eFuse appeared never to have been programmed;
11. therefore the investigated machine's LAN identity did not originate from programmed Realtek eFuse storage;
12. the complete SPI contents could be acquired internally using Linux `flashrom`;
13. the exact missing OEM region could be isolated;
14. only that 64 KiB region could be restored;
15. the physical write could be independently verified;
16. restoring the Fujitsu OEM region restored the original LAN identity in both BIOS and Linux.

---

# 33. What the investigation does not demonstrate

This project does **not** establish that:

- every D3313-E1 behaves identically;
- every FUTRO S930 stores its network identity using exactly the same mechanism;
- all Fujitsu recovery procedures replace personalized MAC data;
- the recovery algorithm has been completely reverse engineered;
- the RESERVED jumper position is safe;
- the selected USB port is an official recovery requirement;
- every RTL8168G on this platform has an unused eFuse;
- the unresolved mSATA problem is definitely hardware-related;
- or the recovery and firmware-analysis techniques themselves originated with this project.

The strongest claims in this repository are intentionally limited to what was directly observed on the investigated system.

---

# 34. Separate unresolved issue: 16 GB mSATA

After the OEM identity restoration was completed, the installed 16 GB mSATA SSD was investigated separately.

Linux successfully detects two SATA controllers:

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
- leaves the relevant SATA paths in `SATA link down` state.

Restoring the original Fujitsu OEM block did not alter this behavior.

There is therefore currently no evidence that the mSATA issue is part of the same firmware failure.

Possible causes include:

- failed mSATA SSD;
- connector/contact failure;
- missing socket power;
- damaged SATA signal path;
- board-level hardware damage;
- or another configuration mechanism not yet identified.

This remains an:

**`[OPEN QUESTION]`**

and is intentionally documented separately from the completed firmware recovery.

---

# 35. Separate hardware incident: movable SATA adapter

During testing, the adapter carrying the Kingston SATA SSD moved while the machine was operating.

Linux immediately logged errors including:

- ATA bus errors;
- BadCRC;
- handshake/link failures;
- I/O errors.

The filesystem was consequently capable of entering a read-only state.

After physically stabilizing the adapter and rebooting, those errors disappeared.

This incident should not be confused with the unresolved mSATA problem.

It does, however, demonstrate that the current improvised adapter mounting is unsuitable for permanent operation.

The planned mechanical correction is the use of appropriate:

```text
M2 nylon/plastic standoffs
approximately 6 mm
```

with proper fasteners.

---

# 36. Methodological principle: minimum necessary modification

One general principle emerged from the recovery process:

> **Do not rewrite more firmware than the evidence demonstrates must be changed.**

In this case:

- a complete original SPI image was available;
- a complete working post-recovery image was available;
- the exact personalized region had been identified;
- only 3526 bytes differed inside one 64 KiB block;
- the rest of the current working firmware could therefore be preserved.

The final operation modified only:

```text
64 KiB out of an 8 MiB flash
```

This approach reduces unnecessary changes to:

- NVRAM;
- boot variables;
- runtime state;
- recovery-generated data;
- unknown firmware structures.

The same principle may be useful in other firmware-repair workflows.

---

# 37. Reproducibility

The strongest findings should ideally be reproduced on additional hardware.

A particularly valuable replication experiment would use a second D3313-E1 and compare:

- location of the OEM/manufacturing block;
- FFS GUIDs;
- structure of the duplicated records;
- factory template values;
- original machine MAC storage;
- Realtek eFuse state;
- behavior of Fujitsu recovery;
- post-recovery OEM state;
- result of selective OEM restoration.

Replication would allow the findings to be classified more confidently as:

- unique to one specimen;
- characteristic of D3313-E1;
- or common across a broader D3313 platform family.

---

# 38. Limitations

This study has several important limitations.

### Single primary specimen

The complete investigation was performed on one D3313-E1 system.

This prevents population-level generalization.

### Incomplete recovery algorithm knowledge

The observable result of the Fujitsu recovery procedure is documented, but the internal recovery implementation has not yet been reverse engineered.

### Partially undocumented OEM fields

Several machine-specific field names have been identified, but many of their exact functions remain unknown.

### No destructive redundancy testing

The two apparently redundant OEM structures have not been intentionally corrupted to determine fallback behavior.

### mSATA issue unresolved

The non-detected 16 GB mSATA has not yet been independently cross-tested in another system or external adapter.

### Privacy restrictions

The original personalized firmware cannot be published in full because it contains machine-specific information.

These limitations are documented explicitly so that absence of evidence is not mistaken for a negative finding.

---

# 39. Future research directions

The recovered system provides a platform for several additional investigations.

## 39.1. Fujitsu recovery write-map reconstruction

One of the most significant open questions is why personalized data present in the recovery source image did not remain present in the physical SPI after recovery.

Controlled images could be created with non-critical marker changes in selected regions.

After recovery, the physical SPI could be reacquired and compared.

This could reveal:

- which regions are copied;
- which are ignored;
- which are reconstructed;
- which are preserved;
- and whether special manufacturing rules exist.

The result could become a formal **D3313 recovery write map**.

---

## 39.2. Complete reverse engineering of the OEM/manufacturing structure

The 64 KiB region appears to contain considerably more information than the LAN MAC alone.

Future work could compare anonymized dumps from multiple systems to determine:

- invariant fields;
- model-dependent fields;
- board-specific fields;
- per-machine identifiers;
- counters;
- integrity fields;
- feature flags;
- security-related fields.

This could eventually produce a documented specification of the D3313 manufacturing data format.

---

## 39.3. Redundant OEM structure behavior

The two related FFS structures require further investigation.

Questions include:

- Is one copy primary?
- Is one a backup?
- Is there a generation number?
- Is one selected using a validity flag?
- Can firmware reconstruct one from the other?
- What happens if only one record contains a valid MAC?

A spare test platform would allow controlled experiments without risking the recovered board.

---

## 39.4. Identification of the UEFI module responsible for NIC initialization

The restored OEM block caused the correct Realtek MAC identity to return.

The next logical question is:

> **Which firmware component reads the Fujitsu OEM data and writes it into the Realtek controller?**

Possible future work includes:

- DXE module analysis;
- PCI initialization tracing;
- string and GUID correlation;
- MMIO register tracing;
- emulator-assisted analysis;
- boot-time instrumentation.

Identifying the responsible module would convert the current behavioral evidence into a detailed implementation model.

---

## 39.5. Other hardware initialized from OEM data

The MAC result raises a broader question.

If the network controller receives machine-specific configuration from the OEM region, other hardware may do so as well.

Potential targets include:

- SATA/mSATA behavior;
- TPM configuration;
- HDCP information;
- board feature enablement;
- licensing;
- PCIe device configuration;
- platform identity.

The unresolved mSATA issue is therefore worth keeping conceptually separate while still considering OEM-dependent configuration as a testable hypothesis.

---

## 39.6. Comparison of D3313 E, B, and S variants

The D3313 family exists in several variants.

A cross-platform comparison could determine:

- whether the OEM FFS GUIDs are shared;
- whether the manufacturing region occupies the same address range;
- whether the field structure is compatible;
- whether recovery behavior is similar;
- which documentation can safely be used across variants.

This could help distinguish platform architecture from model-specific implementation.

---

## 39.7. Automated firmware differencing

Much of the current analysis can be automated.

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

Such tooling could make similar recovery work faster and less error-prone.

---

## 39.8. Firmware privacy sanitizer

The project demonstrates that SPI images can contain information that should not casually be published.

A dedicated sanitizer could detect or redact:

- MAC addresses;
- serial numbers;
- UUIDs;
- PC IDs;
- manufacturing identifiers;
- known credential-like structures.

A privacy-aware firmware sharing tool would be useful beyond the D3313 platform.

---

## 39.9. Recovery acoustic signal analysis

The recovery procedure produced distinct beep sequences.

Recording the complete acoustic sequence with accurate timing may allow correlation with:

- device detection;
- erase;
- programming;
- verification;
- success;
- failure.

This could be valuable for machines where no video output is available during firmware recovery.

---

## 39.10. Hardware-versus-firmware fault isolation methodology

The original failure was particularly useful as a case study because mechanical damage and firmware corruption were both plausible.

The resulting diagnostic sequence was:

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

This sequence may be useful as a general framework when the origin of an embedded-system failure is uncertain.

---

# 40. Broader firmware-forensics significance

One of the broader lessons of this case is that:

> **“factory BIOS image” and “the complete firmware state of an individual machine” are not necessarily the same thing.**

A real machine may contain several layers:

```text
generic vendor firmware
machine-specific manufacturing data
runtime NVRAM
hardware identity
licensing information
security configuration
platform-specific calibration
```

Writing a generic vendor image may therefore restore boot functionality while still removing characteristics that belong to the individual system.

In this case, the visible lost characteristic was the LAN identity.

On other systems, similar personalized data may include:

- system serial numbers;
- UUIDs;
- Windows/OEM licensing information;
- calibration data;
- TPM configuration;
- board-specific feature flags;
- network identifiers.

Firmware repair should therefore distinguish **bootability** from **complete machine-state restoration**.

---

# 41. Safety considerations

SPI flash modification can render a system unbootable.

The methodology used here strongly favors:

- multiple independent reads;
- cryptographic hashes;
- immutable archival copies of original dumps;
- exact flash-device identification;
- pre-write verification;
- smallest-possible write scope;
- independent post-write verification.

OTP/eFuse programming should be treated differently from ordinary flash programming.

Before modifying irreversible or limited-write storage, the operator should understand:

- what data is being programmed;
- whether the operation can be reversed;
- whether the target actually used that storage mechanism originally;
- how much programmable capacity remains;
- whether a less invasive recovery method exists.

---

# 42. Repository privacy policy

The public repository must not contain:

- original personalized raw SPI images;
- complete machine serial numbers;
- PC IDs;
- UUIDs;
- full machine MAC addresses;
- HDCP or other machine-specific manufacturing secrets;
- personal operating-system usernames;
- private local filesystem paths;
- EXIF/GPS metadata;
- readable device labels containing identifying data;
- QR codes or barcodes containing machine identity.

All screenshots, videos, audio recordings, and firmware excerpts should undergo a privacy review before publication.

Raw media should not be committed directly.

---

# 43. Proposed repository structure

The repository is intended to evolve toward a structure similar to:

```text
/
├── README.md
├── README.hu.md
├── LICENSE
├── CITATION.cff
├── SOURCES.md
├── DISCLAIMER.md
├── docs/
│   ├── en/
│   └── hu/
├── scripts/
├── examples/
└── media/
    └── public/
```

Personalized firmware images should never be placed in the public tree.

---

# 44. Prior work and attribution

This project does **not** claim invention of the underlying firmware-recovery, UEFI-analysis, SPI-programming, or Realtek diagnostic techniques.

The work builds upon existing manufacturer documentation, open-source tooling, and prior community research.

Relevant categories of prior work include:

- Fujitsu D3313 documentation;
- Fujitsu BIOS and recovery documentation;
- later Kontron documentation for related D3313 platforms;
- the `flashrom` project;
- Realtek RSET8168;
- Realtek PGTool documentation and tooling;
- community work on Fujitsu FUTRO systems;
- prior D3313 modification and firmware-analysis projects;
- general UEFI reverse-engineering and repair research.

A dedicated:

```text
SOURCES.md
```

will contain explicit author attribution and direct source references.

The purpose of this project is not to claim that these methods were developed here.

Its contribution is the **documented combination, verification, and application of those methods to one D3313-E1 recovery case, together with the resulting platform-specific observations**.

---

# 45. Research contribution of this case study

The most relevant contribution of this work is not simply that one computer was repaired.

The project provides a documented chain connecting:

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

This chain provides several potentially reusable observations:

- a concrete location for D3313-E1 machine-specific data;
- evidence that the factory firmware contains a personalization template;
- evidence that the investigated NIC eFuse was never programmed;
- evidence that the platform firmware can supply the NIC's identity;
- evidence that Fujitsu recovery does not behave as a simple raw full-chip image copy;
- a demonstrated minimal-region recovery method.

These observations may be useful to researchers investigating:

- Fujitsu D3313 firmware architecture;
- embedded-system personalization;
- firmware identity storage;
- firmware recovery mechanisms;
- EEPROM-less PCIe device initialization;
- BIOS forensics;
- legacy industrial system preservation.

---

# 46. Conclusion

At the beginning of the investigation it was impossible to determine whether the FUTRO S930 had:

- mechanical board damage;
- processor-area damage;
- a power-delivery fault;
- SPI firmware corruption;
- or a combination of several failures.

The final state is substantially better understood.

The system now:

- POSTs successfully;
- exposes a functional BIOS Setup;
- boots Debian Linux;
- operates over Ethernet;
- contains the restored machine-specific Fujitsu OEM region;
- and again initializes the original Fujitsu LAN identity.

The post-recovery MAC failure was not resolved by programming the Realtek eFuse.

Direct inspection indicated that the eFuse had apparently never been programmed.

Instead, the decisive information was located in the machine-specific Fujitsu SPI/OEM region.

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

The unresolved 16 GB mSATA detection problem remains documented separately and is not currently treated as part of the completed firmware recovery.

The BIOS recovery and restoration of Fujitsu machine-specific OEM identity can therefore be considered:

# **SUCCESSFULLY COMPLETED**

The next phase is no longer primarily a repair task.

The recovered platform, accumulated measurements, firmware comparisons, and reproducible evidence chain now provide a basis for **further research into the D3313 firmware architecture, manufacturing data model, and recovery mechanism**.

---

## Disclaimer

This repository documents experimental firmware and hardware work performed on privately controlled hardware.

Firmware modification, SPI programming, recovery-jumper use, and board-level work can permanently damage hardware or destroy machine-specific data.

The material is provided for research, documentation, preservation, and educational purposes.

Anyone attempting similar procedures is responsible for maintaining their own verified backups and for understanding the risks associated with their hardware.

---

## Current research status

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
