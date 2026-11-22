# Aminia-MA141-Research

This repository documents my research into the Aminia MA141 router.

The goal of this project is to understand the device's hardware, firmware, bootloader, NAND flash layout, software architecture, and available methods of obtaining unrestricted access.

This is a firmware research and reverse-engineering project. It is **not an attempt to port OpenWrt to the MA141**. The focus is on documenting what the router contains, understanding how its original firmware works, and preserving useful discoveries for other researchers.

## Device Overview

```text
Manufacturer: Aminia
Model: MA141
Category: AC2100 Dual-Band Wireless Router

SoC: Realtek RTL8685FB
Software-reported SoC: RTL8685SB
CPU Architecture: MIPS
RAM: 128 MB
Flash: 128 MB
Flash Type: SPI-NAND

Ethernet: Realtek RTL SwCore
Bootloader: U-Boot-derived
Firmware: Vendor firmware 4.1.x
Root Filesystem: SquashFS
```

## Technical Documentation
- [Hardware Documentation](docs/hardware.md)
- [NAND Flash Layout](docs/nand-layout.md)
- [Bootloader Documentation](docs/bootloader.md)
- [Firmware Analysis](docs/firmware-analysis.md)
- [Rooting Research](docs/rooting-research.md)


## Hardware Research

### Processor and Platform

The kernel images extracted from the NAND dumps identify their architecture as:

```text
Linux/MIPS
```

The bootloader contains DDR initialization and calibration routines, including references to DDR2 and DDR3 initialization.

The exact processor model, revision, clock frequency, and installed RAM capacity still require confirmation.

### Ethernet Controller

The saved boot environment identifies the active Ethernet controller as:

```text
ethact=RTL SwCore
```

The environment also contains switch initialization variables and register-write commands.

Relevant variables:

```text
fx1000_init=mw bb000084 00000048
sgmii_init=mw bb000084 00000044
```

These are recorded for research purposes. Their precise hardware meaning and register behavior have not yet been fully established.

### Flash Storage

The MA141 uses SPI-NAND.

The extracted firmware contains separate bootloader, environment, configuration, kernel, root filesystem, and writable-data partitions.

The NAND dump collection also includes OOB (Out-of-Band) variants of most partitions.

## Firmware Information

Two firmware versions have been identified in the saved boot environment and kernel/rootfs images.

| Slot | Firmware Version | Build Date |
|---|---|---|
| Slot 0 | 4.1.1 | 2020-11-02 |
| Slot 1 | 4.1.2 | 2022-02-08 |

The environment contains:

```text
sw_version0=4.1.1_20201102-201102
sw_version1=4.1.2_20220208-220208
```

The newer firmware is represented by the second kernel and root filesystem pair.

## Bootloader Research

The bootloader is U-Boot-derived and provides serial console access.

The saved environment identifies the serial baud rate:

```text
baudrate=115200
```

The standard console configuration is:

```text
bootargs_base=console=ttyS0,115200
```

### Bootloader Environment

The environment contains several important variables:

```text
bootdelay=5
freeAddr=81000000
ipaddr=192.168.1.3
serverip=192.168.1.7
ethact=RTL SwCore
sw_active=1
sw_tryactive=2
sw_commit=1
```

These values come from the supplied environment dump. Network addresses and MAC addresses should be treated as dump-specific values rather than universal defaults.

### Boot Selection

The bootloader uses separate commands for the two firmware slots:

```text
b0=setenv bootargs ${bootargs_base} ${mtdparts0};spi_nand read ${freeAddr} ${fl_kernel1} ${fl_kernel1_sz};bootm ${freeAddr}

b1=setenv bootargs ${bootargs_base} ${mtdparts1};spi_nand read ${freeAddr} ${fl_kernel2} ${fl_kernel2_sz};bootm ${freeAddr}
```

The saved environment also contains:

```text
bootcmd=if itest.s ${sw_tryactive} == 2; then run boot_by_commit;else run boot_by_tryactive;fi
```

This demonstrates that the firmware uses a dual-slot boot-selection mechanism.

### Firmware Update Commands

The environment contains commands for transferring firmware over TFTP and updating individual NAND regions.

Examples of command names found in the environment:

```text
upb
upk
upk1
upr
upr1
upt
upv
yk
yr
yu
```

**WARNING:** Some of these commands erase and rewrite NAND flash, including bootloader regions. They are documented for analysis only. Do not execute them casually.

## NAND Flash Layout

The following layout is derived from the saved U-Boot environment and the supplied partition dumps.

| MTD | Name | Size |
|---|---|---:|
| MTD0 | Bootloader | 768 KiB |
| MTD1 | Environment | 128 KiB |
| MTD2 | Environment Backup | 128 KiB |
| MTD3 | Configuration | 24 MiB |
| MTD4 | Kernel Slot 0 | 8 MiB |
| MTD5 | RootFS Slot 0 | 20 MiB |
| MTD6 | Kernel Slot 1 | 8 MiB |
| MTD7 | RootFS Slot 1 | 20 MiB |
| MTD8 | RootFS Data | 12 MiB |
| MTD9 | Free / Unallocated Region | 32,512 KiB |
| MTD10 | Additional Partition | 4 KiB |
| MTD11 | Additional Partition | 4 KiB |

The boot environment defines the following important offsets:

```text
fl_boot_sz=c0000

fl_env=c0000
fl_env2=e0000
fl_env_sz=20000

fl_cfgfs=100000
fl_cfgfs_sz=1800000

fl_kernel1=1900000
fl_kernel1_sz=800000

fl_rootfs1=2100000
fl_rootfs1_sz=1400000

fl_kernel2=3500000
fl_kernel2_sz=800000

fl_rootfs2=3d00000
fl_rootfs2_sz=1400000
```

The environment also defines additional Linux partition aliases. These should be distinguished from the physical MTD dump files when interpreting the partition table.

## Firmware Image Analysis

### Kernel Images

Both kernel images use the classic U-Boot legacy image format.

The extracted metadata identifies:

```text
Image Type: Linux Kernel Image
Architecture: MIPS
Compression: LZMA
Load Address: 0x80000000
Entry Point: 0x80000000
```

The images have different build dates and CRC values.

### Root Filesystems

Both root filesystem images use SquashFS version 4.0.

Identified filesystem properties:

```text
Filesystem: SquashFS
Version: 4.0
Compression: XZ
Block Size: 131072 bytes
```

The filesystem timestamps correspond to the two firmware builds:

```text
Slot 0: November 2, 2020
Slot 1: February 8, 2022
```

### OOB Data

The NAND archive includes separate OOB dumps.

OOB data is important because NAND flash stores additional metadata outside the main data area. It can contain ECC information, bad-block markers, and other NAND-specific information.

The exact OOB layout and ECC scheme still need further investigation.

## Repository Research Areas

### 1. Hardware Identification

- Identify the exact Realtek SoC.
- Identify the installed RAM.
- Identify the wireless chipset.
- Document Ethernet and switch hardware.
- Identify the SPI-NAND manufacturer and part number.
- Document PCB components and UART connections.

### 2. Bootloader Analysis

- Identify the exact bootloader version.
- Document available commands.
- Analyze environment variables.
- Understand the dual-slot boot mechanism.
- Document recovery and firmware update behavior.
- Investigate safe RAM-only testing possibilities.

### 3. NAND Research

- Preserve original partition dumps.
- Calculate and publish SHA-256 checksums.
- Understand physical NAND offsets.
- Investigate OOB layout and ECC.
- Identify bad-block handling.
- Compare the two firmware slots.

### 4. Firmware Analysis

- Extract both SquashFS filesystems.
- Compare firmware 4.1.1 and 4.1.2.
- Identify startup scripts and system services.
- Inspect installed utilities and shared libraries.
- Identify authentication and account-management mechanisms.
- Document vendor-specific configuration tools.

### 5. Root Access Research

- Investigate available shell access.
- Identify restricted accounts and privilege boundaries.
- Examine SSH and Telnet services.
- Analyze local authentication mechanisms.
- Document verified root-access methods.
- Keep untested vulnerabilities and theories separate from confirmed findings.

### 6. Network and Wireless Research

- Identify switch initialization procedures.
- Investigate Ethernet register configuration.
- Identify wireless drivers and firmware.
- Document WAN/LAN configuration.
- Investigate VLAN and hardware NAT behavior.

## Research Files

The current NAND archive contains:

```text
mtd0_bootloader.bin
mtd1_env.bin
mtd2_env2.bin
mtd3_config.bin
mtd4_k0_kernel.bin
mtd5_r0_rootfs.bin
mtd6_k1_kernel.bin
mtd7_r1_rootfs.bin
mtd8_rootfs_data.bin
mtd9_free.bin
mtd10_Partition_010.bin
mtd11_Partition_011.bin
```

Additional files include OOB dumps and an extracted LZMA kernel payload.

The original dumps should be preserved separately from modified or processed files.

## Research Status

| Area | Status |
|---|---|
| Firmware identification | Identified |
| Dual-slot firmware layout | Confirmed |
| U-Boot environment extraction | Confirmed |
| Kernel architecture | MIPS confirmed |
| Kernel image format | Identified |
| Root filesystem format | Identified |
| SPI-NAND partition layout | Partially documented |
| Realtek Ethernet identification | RTL SwCore identified |
| Exact SoC identification | Pending |
| Wireless chipset identification | Pending |
| OOB and ECC analysis | Pending |
| Root access investigation | In progress |
| Full bootloader documentation | In progress |

## Safety and Preservation

This repository is intended for research on hardware owned by the researcher or used with permission.

- Always preserve original firmware and NAND dumps.
- Verify checksums before analyzing or modifying files.
- Do not erase or rewrite NAND without a verified recovery method.
- Never assume that firmware images from different hardware revisions are interchangeable.
- Treat configuration partitions as potentially sensitive.
- Sanitize MAC addresses, serial numbers, credentials, private keys, and ISP-specific configuration before publishing dumps.
- Clearly label experimental and unverified procedures.

## Disclaimer

This is an independent community research project.

It is not affiliated with Aminia, the router manufacturer, or any ISP.

All hardware and firmware findings are based on available device dumps, observed behavior, and ongoing reverse engineering. Some information may be incomplete or subject to correction as further research is performed.

## Author

**Xeno-OpenWRT**

Researching router firmware, bootloaders, embedded Linux systems, and hardware security.

The objective is to make undocumented router hardware easier to understand through open research and technical documentation.
