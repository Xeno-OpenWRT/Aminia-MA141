# MA141 Firmware Analysis

This document records the firmware structure and binary analysis of the Aminia MA141 router.

The research is based on the NAND partition dumps, kernel images, root filesystem images, and bootloader environment recovered from the device.

The goal is to understand how the original firmware is organized and identify potential paths toward gaining root access.

## 1. Firmware Overview

The MA141 uses a dual-firmware-slot architecture.

Two firmware versions were identified in the recovered bootloader environment:

| Firmware | Version | Kernel | Root filesystem |
|---|---|---|---|
| Slot 0 | `4.1.1_20201102-201102` | MTD4 | MTD5 |
| Slot 1 | `4.1.2_20220208-220208` | MTD6 | MTD7 |

Each slot contains a separate kernel and root filesystem.

This arrangement allows the bootloader to select between two firmware installations.

## 2. Firmware Partition Files

The recovered NAND archive contains the following important partition dumps:

| File | Description |
|---|---|
| `mtd4_k0_kernel.bin` | First firmware kernel |
| `mtd5_r0_rootfs.bin` | First firmware root filesystem |
| `mtd6_k1_kernel.bin` | Second firmware kernel |
| `mtd7_r1_rootfs.bin` | Second firmware root filesystem |
| `mtd6_k1_kernel_payload.lzma` | Extracted LZMA kernel payload |

Associated OOB dump files are also present in the archive.

The partition names reflect the firmware layout identified in the bootloader environment.

## 3. Kernel Image Format

The recovered kernel images use the legacy U-Boot image format.

The image header begins with the following magic bytes:

```text
27 05 19 56
```

This corresponds to the U-Boot legacy image magic:

```text
0x27051956
```

The recovered image headers identify:

| Property | Identified value |
|---|---|
| Image format | U-Boot legacy image (uImage) |
| Architecture | MIPS |
| Compression | LZMA |
| Load address | `0x80000000` |
| Entry point | `0x80000000` |

The reported kernel image sizes are:

| Kernel | Reported image size |
|---|---:|
| Slot 0 | 3,249,852 bytes |
| Slot 1 | 3,330,651 bytes |

The kernel partitions are larger than the actual image payloads, leaving unused space within their allocated regions.

## 4. Root Filesystem Format

The recovered root filesystem images begin with:

```text
68 73 71 73
```

This is the SquashFS magic:

```text
hsqs
```

The identified filesystem properties are:

| Property | Identified value |
|---|---|
| Filesystem | SquashFS |
| Version | 4.0 |
| Compression | XZ |
| Block size | 131,072 bytes |

SquashFS is a compressed, read-only filesystem commonly used in embedded Linux firmware.

This makes it suitable for storing the router's base operating system and applications.

## 5. Firmware Boot Process

The bootloader environment reveals the general firmware boot sequence:

1. The bootloader initializes the hardware.
2. The bootloader evaluates its firmware-selection variables.
3. It selects a firmware slot.
4. The corresponding kernel is read from SPI-NAND into RAM.
5. Linux is started using the `bootm` command.
6. Linux receives the appropriate MTD partition configuration.
7. Linux mounts the selected root filesystem.

The two firmware slots use different root filesystem MTD devices:

- Slot 0: `root=31:5`
- Slot 1: `root=31:7`

The exact root filesystem mounting behavior should also be verified against the kernel command line and Linux startup scripts.

## 6. Firmware Versions and Update Architecture

The recovered environment contains:

```text
sw_version0=4.1.1_20201102-201102
sw_version1=4.1.2_20220208-220208
sw_active=1
sw_tryactive=2
sw_commit=1
```

These variables indicate that the firmware system maintains information about both firmware versions and the active or attempted firmware slot.

The exact update, rollback, and commit mechanisms require further investigation.

See [Bootloader Documentation](bootloader.md) for the recovered boot commands and selection logic.

## 7. Rooting and Security Research

Understanding the firmware binaries is an important step toward investigating root access.

Potential research areas include:

- Inspecting Linux startup scripts.
- Identifying default users and groups.
- Examining `/etc/passwd` and `/etc/shadow`.
- Finding SSH and Telnet configuration.
- Investigating web administration services.
- Identifying privileged binaries and scripts.
- Examining update mechanisms.
- Checking for exposed debugging interfaces.
- Identifying firmware integrity and signature checks.

The extracted root filesystem should be inspected before making assumptions about available accounts or services.

**Important:** Do not publish configuration files containing passwords, private keys, device-specific credentials, or other sensitive information.

## 8. Further Analysis

Future firmware analysis should investigate:

- The complete contents of both root filesystems.
- Differences between firmware versions 4.1.1 and 4.1.2.
- Installed Linux kernel version.
- Available system utilities.
- Default login accounts.
- SSH and Telnet availability.
- Web interface authentication.
- Startup scripts and service configuration.
- Firmware update and recovery mechanisms.

Comparing both firmware versions may reveal security fixes, changed authentication mechanisms, or newly introduced functionality.

## 9. Research Scope

This repository documents the original MA141 firmware and its security characteristics.

The current objective is firmware understanding, reverse engineering, and investigating root access on hardware owned by the researcher.

This is not an active native OpenWrt porting project.

---

*Documentation status: Initial kernel and root filesystem formats identified. Detailed filesystem and security analysis ongoing.*
