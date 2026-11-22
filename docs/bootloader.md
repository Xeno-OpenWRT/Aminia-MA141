# MA141 Bootloader Documentation

This document records the bootloader information discovered during research on the Aminia MA141 router.

The focus is on the bootloader version, serial console, NAND layout variables, firmware selection, and Linux boot process.

## 1. Bootloader Identification

The captured bootloader information identifies the following:

| Item | Information |
|---|---|
| Bootloader | U-Boot |
| Version | 2011.12.NA |
| Build date | April 21, 2021 |
| Build time | 14:19:02 |
| Bootloader prompt | `VD2#` |
| Preloader | 4.0 |
| CPU frequency reported during boot | 1000 MHz |
| DDR2 frequency reported during boot | 525 MHz |

The bootloader provides access to hardware initialization, NAND operations, environment variables, and firmware boot commands.

## 2. Serial Console

The captured environment contains:

```text
baudrate=115200
bootdelay=5
```

This indicates a serial console baud rate of 115200 and a boot delay of 5 seconds.

The bootloader prompt observed in the capture is:

```text
VD2#
```

The serial console is useful for observing the startup process and examining the bootloader environment.

## 3. Memory and Boot Arguments

The environment includes:

```text
freeAddr=81000000
bootargs_base=console=ttyS0,115200
```

`freeAddr` is used as the RAM destination for reading a kernel image before booting it.

The base kernel arguments configure the Linux console to use `ttyS0` at 115200 baud.

## 4. NAND Flash Environment Variables

The following NAND-related environment variables were recovered:

| Variable | Value | Description |
|---|---:|---|
| `fl_boot_sz` | `0xc0000` | Bootloader region size |
| `fl_env` | `0xc0000` | Primary environment offset |
| `fl_env2` | `0xe0000` | Secondary environment offset |
| `fl_env_sz` | `0x20000` | Environment region size |
| `fl_cfgfs` | `0x100000` | Configuration storage offset |
| `fl_cfgfs_sz` | `0x1800000` | Configuration storage size |
| `fl_kernel1` | `0x1900000` | First kernel offset |
| `fl_kernel1_sz` | `0x800000` | First kernel region size |
| `fl_rootfs1` | `0x2100000` | First root filesystem offset |
| `fl_rootfs1_sz` | `0x1400000` | First root filesystem size |
| `fl_kernel2` | `0x3500000` | Second kernel offset |
| `fl_kernel2_sz` | `0x800000` | Second kernel region size |
| `fl_rootfs2` | `0x3d00000` | Second root filesystem offset |
| `fl_rootfs2_sz` | `0x1400000` | Second root filesystem size |

These offsets are recorded as hexadecimal values.

## 5. Dual-Firmware Architecture

The MA141 contains two firmware slots.

| Component | Slot 0 | Slot 1 |
|---|---|---|
| Kernel | MTD4 | MTD6 |
| Root filesystem | MTD5 | MTD7 |
| Kernel offset | `0x1900000` | `0x3500000` |
| Root filesystem offset | `0x2100000` | `0x3d00000` |
| Kernel region | 8 MiB | 8 MiB |
| Root filesystem region | 20 MiB | 20 MiB |

The recovered environment reports these firmware versions:

```text
sw_version0=4.1.1_20201102-201102
sw_version1=4.1.2_20220208-220208
```

The environment also contains:

```text
sw_active=1
sw_tryactive=2
sw_commit=1
```

These variables appear to participate in firmware-slot selection and update state management. Their exact state-transition behavior should be confirmed by examining the corresponding bootloader environment functions.

## 6. Firmware Boot Commands

The environment contains two important boot commands.

### Slot 0

```bash
b0=setenv bootargs ${bootargs_base} ${mtdparts0};spi_nand read ${freeAddr} ${fl_kernel1} ${fl_kernel1_sz};bootm ${freeAddr}
```

This command:

1. Sets the Linux boot arguments using `mtdparts0`.
2. Reads the first kernel region from SPI-NAND into RAM.
3. Executes the kernel image using `bootm`.

### Slot 1

```bash
b1=setenv bootargs ${bootargs_base} ${mtdparts1};spi_nand read ${freeAddr} ${fl_kernel2} ${fl_kernel2_sz};bootm ${freeAddr}
```

This command performs the equivalent operation for the second firmware slot.

## 7. Automatic Boot Selection

The recovered environment contains:

```bash
bootcmd=if itest.s ${sw_tryactive} == 2; then run boot_by_commit;else run boot_by_tryactive;fi
```

This is the automatic boot-selection entry point.

It checks the value of `sw_tryactive` and executes one of two environment functions:

- `boot_by_commit`
- `boot_by_tryactive`

The function definitions determine the actual selection and fallback behavior. Their complete contents should be documented when recovered.

## 8. MTD Partition Configuration

The bootloader defines partition layouts through `mtdparts`.

The first firmware-slot configuration includes:

```text
mtdparts=spinand:768K(boot),128K(env),128K(env2),24M(config),8M(k0)ro,20M(r0)ro,8M(k1),20M(r1),12M(rootfs_data),33280K(free),4K@0ro,4K@0ro,8M@25M(linux),20M@33M(rootfs) root=31:5
```

The alternate configuration includes:

```text
mtdparts=spinand:768K(boot),128K(env),128K(env2),24M(config),8M(k0),20M(r0),8M(k1)ro,20M(r1)ro,12M(rootfs_data),33280K(free),4K@0ro,4K@0ro,8M@53M(linux),20M@61M(rootfs) root=31:7
```

These strings describe the NAND partition layout presented to Linux. The `root=31:5` and `root=31:7` arguments select different root filesystem MTD devices.

The partition names and offsets should be interpreted alongside the actual NAND dump documentation.

See [NAND Flash Layout](nand-layout.md) for more information.

## 9. Ethernet Environment

The bootloader environment also identifies:

```text
ethact=RTL SwCore
```

This records the Ethernet subsystem name used by the bootloader.

It does not, by itself, establish the complete switch or PHY initialization sequence.

## 10. Research and Safety Notes

The bootloader exposes NAND commands, including read, erase, and write operations.

For research purposes, reading and documenting the environment is safer than modifying NAND.

**Do not execute NAND erase or write commands simply to test them.** Incorrect offsets, sizes, or image contents can damage the bootloader, firmware, or configuration data.

Before any future modification, preserve verified backups of the relevant NAND contents, including OOB data where applicable.

## 11. Outstanding Research

The following questions remain open:

- What are the complete definitions of `boot_by_commit` and `boot_by_tryactive`?
- How exactly does the bootloader decide when to switch firmware slots?
- What conditions cause a firmware slot to be marked committed?
- How does the bootloader handle a failed kernel boot?
- What are the exact NAND read and ECC behaviors?
- Which bootloader commands are available for safely inspecting hardware?

These should be answered using the captured bootloader environment and further read-only analysis.

---

*Documentation status: Initial U-Boot environment and dual-firmware boot flow documented. Further bootloader analysis ongoing.*
