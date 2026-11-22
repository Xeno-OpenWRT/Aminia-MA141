# MA141 Hardware Documentation

This document records the hardware information identified during research on the Aminia MA141 router.

The purpose is to document the physical platform and its components for firmware analysis, bootloader research, and future rooting work.

## 1. Hardware Overview

| Component | Identified Information |
|---|---|
| Device | Aminia MA141 |
| SoC marking | RTL8685FB |
| Software-reported SoC | RTL8685SB |
| CPU architecture | MIPS |
| RAM | 128 MB |
| Flash capacity | 128 MB |
| Flash type | SPI-NAND |
| Ethernet subsystem | Realtek RTL SwCore |

**Important:** The physical chip marking is `RTL8685FB`, while the software identifies the platform as `RTL8685SB`. These identifiers are recorded separately because their relationship has not yet been established.

## 2. SoC

The MA141 uses a Realtek SoC identified physically by the marking:

`RTL8685FB`

The software and bootloader environment identify the platform as:

`RTL8685SB`

The available bootloader information and kernel image headers indicate a MIPS-based platform.

Further investigation is required to establish the exact CPU core, clock frequency, peripheral configuration, and hardware capabilities.

## 3. Memory

The identified memory configuration is:

| Memory | Capacity |
|---|---:|
| RAM | 128 MB |
| Flash | 128 MB |

The RAM is used by the bootloader and Linux system during operation.

The flash stores the bootloader, environment variables, configuration data, firmware images, and filesystem partitions.

## 4. Flash Storage

The MA141 uses SPI-NAND flash.

The firmware research archive contains NAND partition dumps and associated OOB dump files.

The identified firmware layout includes:

- Bootloader
- Primary environment
- Secondary environment
- Configuration storage
- First kernel and root filesystem
- Second kernel and root filesystem
- Root filesystem data
- Free NAND space

The device uses a dual-firmware-slot layout. More detailed partition information is documented in [NAND Flash Layout](nand-layout.md).

## 5. Ethernet

The bootloader environment identifies the Ethernet subsystem as:

`RTL SwCore`

This is recorded as the identified Ethernet subsystem. The exact switch configuration, PHY connections, register initialization sequence, and Linux driver requirements remain subjects for further research.

## 6. Hardware Information Still Under Investigation

The following information has not yet been independently established:

- Exact CPU core and revision
- CPU clock frequency
- Wireless chipset
- Wi-Fi radio configuration
- Ethernet PHY model
- Ethernet switch register configuration
- GPIO assignments
- Serial interface pinout
- RAM chip model
- Exact SPI-NAND geometry and ECC configuration

These details should only be added when supported by physical inspection, bootloader output, firmware analysis, or other reliable evidence.

## 7. Research Notes

This document is intended to evolve as more hardware information becomes available.

The MA141 research project focuses on understanding the original hardware and firmware, documenting the boot process, and investigating root access.

It is not currently an active OpenWrt porting project.

---

*Documentation status: Initial hardware identification completed. Further component-level investigation is ongoing.*
