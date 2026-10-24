# Fujitsu FUTRO S930 / D3313-E1 BIOS Recovery and Firmware Analysis

Independent technical investigation of BIOS recovery, SPI firmware structure, Fujitsu machine-specific OEM data, Realtek LAN identity, and selective firmware restoration on a Fujitsu FUTRO S930 equipped with a D3313-E1 mainboard.

> **Important:** this repository documents an individual research case. It is not an official Fujitsu recovery procedure and should not be assumed to apply unchanged to every D3313-E1 or FUTRO S930 system.

Hungarian version: [README.hu.md](README.hu.md)

---

## 1. Scope and objectives

The project began as an attempt to recover a Fujitsu FUTRO S930 / D3313-E1 system that no longer booted correctly.

The investigation gradually expanded into a firmware-forensics case study with several objectives:

- preserve the original SPI contents before modification;
- determine whether the original firmware was structurally valid;
- compare the machine-specific firmware with official Fujitsu firmware;
- identify machine-specific OEM/manufacturing data;
- understand why Fujitsu BIOS recovery restored bootability but not the original LAN identity;
- determine whether the Realtek network controller stored its permanent MAC address in EEPROM/eFuse or obtained it from platform firmware;
- restore only the smallest firmware region supported by the evidence;
- document the complete evidence chain and remaining uncertainties.

The main recovery and OEM-identity investigation is now considered successfully completed.

A separate 16 GB mSATA detection problem remains open and is intentionally treated as a different issue.

---

## 2. Evidence classification

To avoid presenting interpretation as measurement, this repository uses five evidence categories.

### `[OFFICIAL]`

Information obtained from manufacturer documentation, official firmware packages, standards documentation, upstream software documentation, or another authoritative primary source.

### `[OBSERVED]`

Information directly measured, captured, read, logged, extracted, reproduced, or independently verified on the investigated hardware or software.

### `[INFERRED]`

A conclusion supported by the available observations but not itself directly measured.

### `[REPORTED HISTORY]`

Information describing something that happened before the controlled investigation and cannot now be independently verified from preserved evidence.

This category is used for historical context without presenting recollection as a controlled measurement.

### `[OPEN QUESTION]`

A hypothesis, unresolved mechanism, or behavior requiring additional evidence or independent reproduction.

These labels are used throughout the project whenever the distinction materially affects interpretation.

---

## 3. Investigated platform

The primary investigated system is:

- **System:** Fujitsu FUTRO S930
- **Mainboard:** Fujitsu D3313-E1
- **Board identifier:** S26361-D3313-E1
- **Processor platform:** AMD embedded platform; investigated system uses an AMD GX-424CC-class configuration
- **Firmware family:** Fujitsu / American Megatrends UEFI
- **SPI flash:** Macronix MX25L6473F-class 8 MiB SPI flash device
- **LAN controller:** Realtek RTL8111/RTL8168G family
- **Primary research OS:** Debian Linux
- **External programmer:** CH341A
- **SPI software used during the investigation:** IMSProg, AsProgrammer and flashrom
- **Firmware-analysis tooling:** UEFITool and supporting binary-analysis utilities

Storage observed during the wider hardware investigation included:

- conventional SATA SSD storage;
- a separate 16 GB mSATA device whose detection problem remains unresolved.

The firmware conclusions in this repository apply primarily to this investigated specimen.

---

## 4. Initial failure state

`[REPORTED HISTORY]`

The system stopped booting normally after hands-on hardware work around the mainboard and processor area.

Considerable physical manipulation had taken place before the controlled firmware investigation began.

Because the relevant electrical state was not captured at the moment of failure, this repository does **not** claim a proven causal link between any single mechanical action and the BIOS problem.

Possible explanations considered during the investigation included:

- corrupted firmware;
- altered firmware state;
- electrical or mechanical board damage;
- CPU/socket/contact problems;
- storage-related problems;
- recovery-state changes;
- unrelated pre-existing faults.

The later successful firmware recovery demonstrates that firmware state was materially involved, but it does not retrospectively prove the original physical cause.

---

## 5. External SPI acquisition

Before writing anything to the board, the original 8 MiB SPI flash was read externally.

A CH341A programmer was used together with a SOIC8 connection method.

The programmer was observed as a WCH CH341A-class device.

During Windows experimentation, driver/software combinations produced several different behaviors, including:

```text
VID_1A86
PID_5512
```

and errors such as:

```text
LIBUSB_ERROR_NOT_SUPPORTED
```

IMSProg successfully recognized the programmer and reported device revision information.

The flash device was identified as an MXIC/Macronix 64-Mbit-class SPI device corresponding to an 8 MiB image.

The purpose of this stage was preservation, not repair.

---

## 6. Duplicate acquisition and integrity verification

`[OBSERVED]`

Two independent full SPI reads were obtained before modification.

Each image size was:

```text
8,388,608 bytes
```

The independent reads were compared.

They matched bit-for-bit.

The preserved original image had SHA-256:

```text
7E138DA4B68FB7C86DBD841631D6D60937AF2EA109297A51476610C8F01E3603
```

The workflow followed the principle:

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

The untouched original image was retained separately from all later experimental and repaired images.

---

## 7. Comparison with official Fujitsu firmware

An official Fujitsu firmware package for the D3313-E1x was used as the principal clean comparison image.

The investigated official version was:

```text
V4.6.5.4 R1.16.0 for D3313-E1x
```

Relevant official firmware images included 8 MiB files such as:

```text
D3313-E1.rom
D3313-E1.103
D3313-E1.109
D3313-E1.110
D3313-E1.114
```

The main official ROM image size was:

```text
8,388,608 bytes
```

SHA-256 of the investigated official `D3313-E1.rom`:

```text
1E51B1392A291CEC24575170F5527FE541048463366D2FA541E90A93EA371F59
```

The official package also contained Fujitsu flashing and recovery components including:

```text
DOS\D3313-E1.UPC
DOS\D3313-E1.UPD
DOS\DosFlash.BAT
DOS\EfiFlash.exe
```

The investigated `DosFlash.BAT` invokes:

```text
EfiFlash.exe /AUTO /NOMCU
```

---

## 8. Major structural finding: 127 of 128 blocks matched

The 8 MiB images were divided into 64 KiB blocks.

An 8 MiB image contains:

```text
128 × 64 KiB
```

Comparison of the preserved original machine image with the official Fujitsu image produced a striking result:

```text
127 of 128 blocks matched
```

Only one 64 KiB block differed:

```text
0x440000–0x44FFFF
```

This immediately narrowed the machine-specific difference to a very small part of the complete firmware image.

`[INFERRED]`

The result strongly suggested that the main executable firmware was essentially the official Fujitsu code, while the differing 64 KiB area contained machine-specific or manufacturing-specific information.

---

## 9. UEFI Firmware Volume and FFS analysis

The images were structurally examined as UEFI firmware.

The investigation identified:

```text
8 valid Firmware Volumes
84 FFS files
0 invalid FV headers
0 invalid FFS checksums
```

Relevant Firmware Volume locations included approximately:

```text
0x400000
0x440000
0x443000
0x44A000
0x44C000
0x474000
0x48C000
0x740000
```

The machine-specific area around `0x440000` therefore sits inside a structured UEFI firmware region rather than being arbitrary unused space.

FREEFORM FFS structures identified during the analysis included GUIDs such as:

```text
0972EB29-E62F-4699-A19C-664AC3B7208B
8EEAF5CF-1C81-4FCC-AE1E-D8AD3CD15683
```

The exact semantic purpose of every structure was not assumed merely from its location.

---

## 10. OEM/manufacturing data structures

Inspection of the differing region revealed recognizable Fujitsu/OEM-oriented fields.

Examples included labels associated with:

```text
BdSn
PcId
BUID
UUID
MAC Address #0
HDCP
TPMD
OEMB
```

The region contains information that is clearly distinct from ordinary executable BIOS code.

`[OBSERVED]`

Machine-specific identifiers are embedded in the SPI image.

`[INFERRED]`

The block acts as an OEM/manufacturing identity region containing information provisioned for a particular machine during manufacturing or configuration.

Because these structures can contain sensitive identifiers, the complete personalized original firmware image is **not** published in this repository.

---

## 11. Discovery of the machine MAC field

A particularly important field was:

```text
MAC Address #0
```

In the original preserved machine firmware, this field contained the machine's Fujitsu-assigned Ethernet identity.

The actual value is intentionally not published and is represented here as:

```text
<REDACTED_FUJITSU_MAC>
```

In the corresponding official Fujitsu firmware template, the same location contained:

```text
FF:FF:FF:FF:FF:FF
```

This was one of the strongest indicators that the official image is a generic template while the machine dump contains manufacturing personalization.

---

## 12. Privacy and forensic implications of SPI dumps

A raw SPI image is not merely "BIOS code."

It may contain:

- serial numbers;
- board serial numbers;
- UUIDs;
- PC IDs;
- MAC addresses;
- manufacturing identifiers;
- HDCP-related information;
- licensing data;
- security-related settings;
- calibration or platform-specific values;
- other undocumented machine-specific structures.

For that reason:

> **A personalized firmware dump must be treated as potentially sensitive technical evidence.**

The repository intentionally publishes:

- hashes;
- offsets;
- structural findings;
- sanitized examples;
- reproducible methodology;

rather than publishing the complete machine-specific SPI dump.

---

## 13. Fujitsu BootStick investigation

Fujitsu's recovery tooling was investigated using the official package.

The BootStick process prepared a USB recovery device using a small USB flash drive.

The successful configuration used a conventional USB 2.0-class recovery device.

The resulting media used a DOS/FAT-style boot environment.

Relevant recovery components included:

```text
D3313-E1.rom
D3313-E1.UPD
EfiFlash.exe
DosFlash.BAT
```

The Fujitsu recovery media and the normal `EfiFlash` update procedure should not automatically be treated as the same mechanism.

Recovery boot behavior can involve separate bootloader/recovery logic.

---

## 14. Recovery jumper configuration

The investigated board reports the following system configuration positions:

```text
DEFAULT   J5–3
RESERVED  J1–2
RECOVERY  J5–6
```

The recovery process used the hardware recovery jumper position.

This is consistent with Fujitsu documentation describing a dedicated recovery mode for corrupted BIOS contents.

The exact jumper configuration should always be checked against the correct board documentation before attempting recovery on another system.

---

## 15. Successful recovery configuration

`[OBSERVED]`

With the appropriate Fujitsu recovery media and the recovery jumper set correctly, the board entered the recovery procedure.

During the process, audible beeps were observed.

The sequence included a pattern described during the investigation as:

```text
long – long – short – short
```

After completion and restoration of the normal jumper position:

- POST functionality returned;
- the machine booted;
- Debian could start;
- the board was no longer in the original non-booting condition.

The primary BIOS recovery therefore succeeded.

---

## 16. Post-recovery network identity failure

After successful Fujitsu recovery, a new anomaly remained.

The onboard Realtek Ethernet controller no longer exposed the original permanent MAC identity.

At BIOS/firmware level, the relevant identity appeared effectively empty.

Observed values included:

```text
00:00:00:00:00:00
```

Linux's `r8169` driver reported behavior consistent with the absence of a usable hardware MAC and generated a temporary/random address.

Observed diagnostic wording included:

```text
can't read MAC address, setting random one
```

and:

```text
Permanent address: not set
```

Network functionality could therefore exist while the original platform identity was missing.

---

## 17. Separating OS behavior from hardware state

The Linux-generated MAC address was not treated as proof of a permanent controller identity.

The investigation distinguished between:

1. the runtime address currently used by the operating system;
2. a permanent controller-programmed address;
3. the Fujitsu machine-specific value stored in platform firmware.

The absence of a permanent address in Linux diagnostics showed that the issue was deeper than a normal OS network configuration problem.

Changing the Linux interface configuration would not reconstruct the original Fujitsu provisioning mechanism.

---

## 18. Realtek RSET8168 diagnostics

Realtek DOS diagnostics were used to examine the controller.

RSET8168 v1.24 produced messages including:

```text
No Valid External EEPROM!!
```

The tool also indicated data associated with:

```text
Data from EFuse
```

and reported:

```text
EFuse Fail
```

These results raised the question of whether the controller's permanent identity had originally been stored in:

- external EEPROM;
- internal eFuse;
- platform firmware;
- or a combination of mechanisms.

The result was not interpreted in isolation.

---

## 19. EEPROM diagnostic result

Further Realtek-oriented tooling was used to investigate the external EEPROM path.

The adapter reported behavior such as:

```text
Not Support This Adapter
```

for the relevant EEPROM operation.

No evidence was found that a conventional populated external EEPROM contained the original Fujitsu MAC identity.

`[OBSERVED]`

The investigated system did not expose a usable external EEPROM containing the original permanent address through the tested diagnostic path.

---

## 20. Direct Realtek eFuse inspection

Because eFuse programming can be irreversible, the controller's eFuse state was inspected before any write was considered.

The investigation used Realtek PGTool-compatible diagnostics to inspect the controller state.

The goal was explicitly:

> **read first, never program irreversible storage without evidence.**

The eFuse data appeared blank/unprogrammed in the relevant areas.

---

## 21. Critical eFuse result

The key result was:

```text
Efuse Write Count = 0
```

Relevant eFuse data was observed as:

```text
FF
```

rather than a previously programmed machine MAC.

`[OBSERVED]`

There was no evidence that the investigated controller's eFuse had ever been programmed with the original Fujitsu Ethernet identity.

This was a decisive result.

---

## 22. Why the Realtek eFuse was not programmed

Programming the original MAC into the Realtek eFuse would have created a new permanent state.

But the evidence indicated:

- the original machine SPI image contained the Fujitsu MAC;
- the official Fujitsu firmware contained `FF:FF:FF:FF:FF:FF` in that field;
- the Realtek eFuse write count was `0`;
- the controller therefore did not appear to have received its original identity from a programmed eFuse.

`[INFERRED]`

Programming the eFuse would not have reproduced the machine's original manufacturing mechanism.

For that reason, eFuse programming was deliberately **not** performed.

---

## 23. Internal SPI access under Linux

After the board was bootable again, the SPI flash was accessed from the running system using flashrom.

The relevant programmer mode was:

```text
flashrom -p internal
```

flashrom detected the AMD platform/FCH access path and identified the Macronix SPI flash.

This provided a safer and more reproducible route for controlled comparison and later selective writing than repeated physical clip attachment.

The externally preserved original remained the authoritative pre-repair evidence.

---

## 24. Decisive post-recovery OEM finding

A full post-recovery SPI image was acquired and compared against both:

- the original preserved machine dump;
- the official Fujitsu firmware.

The post-recovery image contained:

```text
FF FF FF FF FF FF
```

in the OEM MAC field where the original machine image contained:

```text
<REDACTED_FUJITSU_MAC>
```

This directly demonstrated that Fujitsu recovery had restored bootable firmware while leaving the machine-specific MAC field in its generic/template state.

The recovery procedure therefore did **not** fully reconstruct machine personalization.

---

## 25. Implications for the Fujitsu recovery mechanism

The combined evidence supports the following interpretation:

1. Fujitsu recovery successfully restores a functional firmware state.
2. The official image contains a generic OEM template.
3. Machine-specific values are not necessarily reconstructed by the recovery process.
4. The onboard Realtek controller on the investigated specimen did not have the original MAC permanently stored in its eFuse.
5. The platform firmware's OEM/manufacturing region supplies the machine-specific Ethernet identity.

`[INFERRED]`

On the investigated D3313-E1, the original permanent LAN identity is platform-firmware-derived rather than the result of a previously programmed Realtek eFuse.

This conclusion is specific to the investigated specimen unless independently reproduced.

---

## 26. Why the complete historical SPI image was not restored

The simplest repair would have been to write the entire original 8 MiB dump back to the SPI flash.

That option was deliberately rejected.

The system had already undergone:

- recovery;
- boot-state changes;
- firmware runtime changes;
- NVRAM changes;
- additional diagnostics.

Rewriting the complete historical image would also restore unrelated historical state.

The investigation instead followed the principle:

> **Restore only the smallest region demonstrated to contain the required machine-specific information.**

This reduces unnecessary modification and produces a stronger causal test.

---

## 27. Construction of the repair candidate

A repaired candidate image was constructed from the current post-recovery firmware state.

Only the original machine-specific OEM region was transplanted from the preserved pre-recovery image:

```text
0x440000–0x44FFFF
```

The resulting candidate was saved separately, including under a working name such as:

```text
D3313_REPAIRED_CANDIDATE.bin
```

Comparison showed approximately:

```text
3526 differing bytes
```

between the post-recovery image and the candidate, all associated with the intended OEM restoration rather than a complete 8 MiB rollback.

---

## 28. Pre-write physical verification

Before the selective write, several conditions were checked:

- the machine was bootable;
- the original SPI backup remained preserved;
- the current SPI image had been read;
- the candidate was built from the current image plus the original OEM block;
- the target address range was confirmed;
- the intended write region was limited to 64 KiB;
- the write mechanism supported post-write verification.

The aim was to minimize both firmware scope and physical intervention.

---

## 29. Selective 64 KiB restoration

A flashrom layout was used to define the OEM region:

```text
0x440000:0x44ffff OEM
```

Only that region was written.

flashrom reported the write/erase operation for the intended range:

```text
Erase/write done from 440000 to 44ffff
```

Post-write verification completed successfully:

```text
Verifying flash... VERIFIED.
```

The rest of the firmware remained in the recovered/current state.

---

## 30. Restoration result

After the selective write, the system was shut down and cold-booted.

`[OBSERVED]`

The original Fujitsu machine-specific Ethernet identity returned.

The restored identity was visible through the platform/network stack as the machine's permanent LAN identity rather than as a Linux-generated random address.

Bootability remained intact.

The selective 64 KiB restoration therefore restored the missing identity without reverting the entire 8 MiB flash image.

---

## 31. Resulting evidence chain

The complete evidence chain is:

```text
Original machine SPI
        ↓
OEM region contains original Fujitsu MAC
        ↓
Official Fujitsu firmware contains FF:FF:FF:FF:FF:FF
        ↓
Fujitsu recovery restores bootability
        ↓
Post-recovery OEM field remains FF:FF:FF:FF:FF:FF
        ↓
Linux/Realtek cannot obtain permanent MAC
        ↓
RSET8168 finds no valid external EEPROM
        ↓
Realtek eFuse inspection shows:
Efuse Write Count = 0
        ↓
eFuse is not programmed
        ↓
Restore only original 0x440000–0x44FFFF OEM region
        ↓
flashrom verification succeeds
        ↓
cold boot
        ↓
original Fujitsu LAN identity returns
```

This chain is substantially stronger than inferring the mechanism from any single diagnostic tool.

---

## 32. What the investigation demonstrated

For the investigated D3313-E1 specimen, the evidence demonstrates that:

- the externally acquired original SPI image was reproducible;
- the original 8 MiB firmware was overwhelmingly identical to official Fujitsu firmware;
- 127 of 128 64 KiB blocks matched;
- machine personalization was concentrated in the `0x440000–0x44FFFF` region;
- that region contained the original Fujitsu-assigned MAC identity;
- the official firmware contained a generic `FF:FF:FF:FF:FF:FF` template value;
- Fujitsu recovery restored bootability but did not restore that personalized MAC value;
- the investigated Realtek eFuse showed `Efuse Write Count = 0`;
- the original MAC was therefore not recovered by programming Realtek eFuse;
- selectively restoring the original 64 KiB OEM region restored the original permanent LAN identity.

---

## 33. What the investigation does not demonstrate

This repository does **not** demonstrate that:

- every D3313-E1 has the same exact personalization layout;
- every FUTRO S930 stores its MAC through the same mechanism;
- every Realtek RTL8111/8168G controller in this platform family has an unprogrammed eFuse;
- every Fujitsu recovery package handles personalization identically;
- every firmware revision uses the same offsets;
- every OEM field in the 64 KiB region has been semantically decoded;
- the original physical cause of the BIOS failure is proven;
- writing this region is universally safe on other systems.

These require independent reproduction.

---

## 34. Separate unresolved issue: 16 GB mSATA

The investigated machine also has a separate 16 GB mSATA-related problem.

Symptoms include failure of the device to appear reliably or to establish a normal SATA link under the investigated configuration.

This issue is **not considered part of the completed BIOS/OEM identity recovery**.

Possible causes remain under investigation, including:

- device failure;
- connector/contact problems;
- power delivery;
- SATA routing;
- controller configuration;
- board-level hardware damage.

`[OPEN QUESTION]`

The mSATA issue remains unresolved.

---

## 35. Separate hardware incident: movable SATA adapter

During the wider hardware work, a SATA adapter/connector assembly was mechanically movable and could change position.

This incident is documented because mechanical instability can complicate interpretation of storage failures.

However:

> **No direct causal relationship between this adapter behavior and the firmware/OEM recovery result has been demonstrated.**

It should therefore be treated as a separate hardware observation.

---

## 36. Minimum necessary modification

The most important methodological result was not merely that the machine was repaired.

It was that repair succeeded by modifying only:

```text
64 KiB
```

out of:

```text
8 MiB
```

of SPI firmware.

This supports a general firmware-recovery principle:

> **When evidence identifies the required machine-specific state, preserve the known-good current firmware and restore only the smallest necessary region.**

This approach reduces unnecessary changes to:

- NVRAM;
- boot variables;
- firmware runtime state;
- unrelated configuration;
- unknown structures;
- recovery-generated changes.

---

## 37. Reproducibility

A comparable investigation should preserve a reproducible evidence chain.

Recommended minimum procedure:

```text
1. Identify exact board revision
2. Identify exact SPI flash
3. Read full flash
4. Read full flash again
5. Compare both reads
6. Compute cryptographic hashes
7. Preserve originals unchanged
8. Obtain official firmware from an authoritative source
9. Compare at block level
10. Analyze structural differences
11. Separate observation from inference
12. Avoid irreversible writes without evidence
13. Read current firmware immediately before repair
14. Modify only the minimum justified region
15. Verify after write
16. Cold boot and independently validate the result
```

Independent reproduction on additional D3313-E1 boards would materially strengthen the broader conclusions.

---

## 38. Limitations

The investigation has important limitations.

It primarily concerns one physical machine.

Some observations were made after the system had already undergone:

- previous troubleshooting;
- recovery attempts;
- hardware handling;
- firmware operations;
- OS-level changes.

Not every historical state was preserved.

Some mechanisms are inferred from converging evidence rather than directly documented by Fujitsu.

Vendor diagnostic tools may also have undocumented behavior.

For those reasons, the repository distinguishes carefully between direct observation and interpretation.

---

## 39. Future research directions

Useful future work includes:

1. reproduce the OEM-region comparison on additional D3313-E1 boards;
2. compare D3313-A/B/E/G/S variants where legally and practically possible;
3. map the complete structure of the `0x440000–0x44FFFF` region;
4. determine how Fujitsu manufacturing tools provision the OEM block;
5. investigate the relationship between `BdSn`, `PcId`, `BUID`, `UUID`, MAC, HDCP and TPM-related structures;
6. compare multiple official firmware revisions;
7. determine whether recovery behavior changed between Fujitsu BIOS releases;
8. investigate the complete Realtek initialization path;
9. develop safe scripts for block-level comparison and sanitized reporting;
10. resolve the separate 16 GB mSATA problem.

---

## 40. Broader firmware-forensics significance

The case demonstrates an important distinction:

> **A generic manufacturer firmware image and a machine's complete firmware state are not necessarily the same thing.**

A functioning platform may depend on several layers:

```text
generic executable firmware
        +
manufacturing personalization
        +
machine identity
        +
runtime NVRAM/configuration
        +
licensing/security data
        +
possible calibration/platform-specific state
```

A recovery process may restore only some of these layers.

This has implications for:

- firmware forensics;
- hardware preservation;
- board replacement;
- legitimate repair;
- machine-identity preservation;
- reverse engineering;
- safe selective recovery.

---

## 41. Safety considerations

Firmware and board-level work can permanently damage hardware.

Recommended precautions include:

- obtain multiple independent SPI reads;
- compare them before writing;
- compute cryptographic hashes;
- preserve original evidence unchanged;
- confirm the exact flash chip;
- confirm voltage and programmer configuration;
- use the smallest possible write scope;
- verify every write;
- treat OTP/eFuse operations as potentially irreversible.

The Realtek eFuse was intentionally **not** programmed in this investigation because the evidence did not support it as the original provisioning mechanism.

See:

- [DISCLAIMER.md](DISCLAIMER.md)
- [DISCLAIMER.hu.md](DISCLAIMER.hu.md)

---

## 42. Repository privacy policy

The public repository should not contain unredacted machine-specific evidence unnecessarily.

Do not publicly publish:

- personalized complete SPI images;
- machine serial numbers;
- board serial numbers where unnecessary;
- PC IDs;
- UUIDs;
- full machine-specific MAC addresses;
- HDCP-related data;
- private manufacturing data;
- credentials;
- local personal filesystem paths;
- unrelated personal information.

Public evidence should use:

- sanitized excerpts;
- hashes;
- offsets;
- redacted values;
- structural descriptions.

Screenshots and photographs must also be reviewed for:

- usernames;
- taskbars;
- IP addresses;
- filenames;
- background windows;
- stickers;
- QR/barcodes;
- EXIF/GPS metadata;
- other identifying information.

---

## 43. Repository structure

The project currently uses the following core files:

```text
README.md
README.hu.md

LICENSE

CITATION.cff

SOURCES.md

DISCLAIMER.md
DISCLAIMER.hu.md

SECURITY.md
SECURITY.hu.md

CONTRIBUTING.md
CONTRIBUTING.hu.md

.github/
└── ISSUE_TEMPLATE/
```

Raw personalized firmware is intentionally not part of the public repository.

Media is also intentionally omitted unless a future item can be published without exposing unnecessary identifying information.

---

## 44. Prior work and attribution

This investigation builds on existing work rather than claiming invention of the underlying tools or techniques.

Relevant prior work includes:

- Fujitsu technical and firmware documentation;
- Fujitsu BIOS Flash Tools documentation;
- official D3313 firmware packages;
- flashrom;
- UEFITool;
- IMSProg;
- Realtek diagnostic utilities;
- community D3313/FUTRO research;
- firmware-analysis and recovery techniques developed by other researchers.

Detailed references are maintained in:

[SOURCES.md](SOURCES.md)

Third-party authors retain their respective copyrights and licenses.

---

## 45. Research contribution

The contribution of this repository is the documented combination and verification of the evidence in one controlled D3313-E1 case.

In particular, the project documents:

- independently verified original SPI acquisition;
- block-level comparison against official Fujitsu firmware;
- localization of machine-specific data to one 64 KiB region;
- identification of the OEM MAC field;
- observation of Fujitsu recovery behavior;
- investigation of Realtek EEPROM/eFuse state;
- deliberate avoidance of unsupported irreversible eFuse programming;
- controlled construction of a repaired candidate;
- selective internal restoration of only the OEM block;
- successful return of the original permanent LAN identity.

The value lies in the complete evidence chain rather than in any individual tool or command.

---

## 46. Conclusion

The BIOS/OEM recovery investigation achieved its primary objective.

The Fujitsu FUTRO S930 / D3313-E1 returned to a bootable state.

The machine-specific Ethernet identity that was lost after generic recovery was traced to the Fujitsu OEM/manufacturing region of the SPI firmware.

The investigated Realtek controller showed no evidence of a previously programmed eFuse identity.

Restoring only:

```text
0x440000–0x44FFFF
```

from the preserved original firmware restored the original LAN identity while leaving the rest of the recovered firmware state intact.

The investigation therefore supports the conclusion that, on this specimen, the permanent network identity is provisioned through Fujitsu platform firmware rather than a previously programmed Realtek eFuse.

---

## 47. Current research status

### BIOS / firmware recovery

# **SUCCESSFULLY COMPLETED**

Completed milestones:

- original SPI preserved;
- duplicate reads verified;
- official firmware compared;
- UEFI structure analyzed;
- OEM region localized;
- Fujitsu recovery completed;
- post-recovery identity problem reproduced;
- Realtek EEPROM/eFuse investigated;
- eFuse left untouched;
- OEM repair candidate built;
- selective 64 KiB write completed;
- flash verification succeeded;
- original permanent LAN identity restored.

### 16 GB mSATA investigation

# **OPEN**

The mSATA issue is treated as a separate hardware/storage investigation and does not change the completed BIOS/OEM recovery result.

---

## 48. License

Eligible original research text, documentation, diagrams, analysis, and other original project material are licensed under:

**Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International  
(CC BY-NC-SA 4.0)**

See:

[LICENSE](LICENSE)

The repository license does **not** automatically relicense:

- Fujitsu firmware;
- Realtek software;
- vendor utilities;
- third-party documentation;
- third-party code;
- trademarks;
- other third-party material.

Where software source code is later added, it may use a separate software license.

---

## 49. Citation

Citation metadata is provided in:

[CITATION.cff](CITATION.cff)

When referencing this work, please cite the repository and the relevant version, release, or commit where practical.

Stable research baseline:

```text
v1.0.0
```

Repository:

```text
https://github.com/benczepeter001/fujitsu-d3313-bios-recovery
```

---

## 50. Legal and research notice

This is an independent research project.

It is not affiliated with, endorsed by, sponsored by, or produced by Fujitsu, Realtek, AMD, AMI, Macronix, WCH, or other referenced manufacturers or projects.

Third-party product names and trademarks are used only for identification and technical documentation.

Firmware modification can:

- render hardware unbootable;
- destroy machine-specific data;
- damage flash memory or other components;
- produce irreversible configuration changes.

Only work on hardware and data that you own or are explicitly authorized to examine.

For detailed legal, safety, privacy, security, and reuse information, see:

- [DISCLAIMER.md](DISCLAIMER.md)
- [DISCLAIMER.hu.md](DISCLAIMER.hu.md)
- [SECURITY.md](SECURITY.md)
- [SECURITY.hu.md](SECURITY.hu.md)
- [CONTRIBUTING.md](CONTRIBUTING.md)
- [LICENSE](LICENSE)

For technical corrections, reproduction results, or research contact:

**Bencze Péter**  
**Email:** bencze.peter86@proton.me

---

Copyright © 2026 Bencze Péter and respective contributors for their original contributions.
