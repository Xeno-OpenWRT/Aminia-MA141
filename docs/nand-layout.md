# MA141 NAND Flash Layout

This document describes the NAND flash layout of the Aminia MA141 router, based on the supplied `mtds.zip` partition dump archive and the saved U-Boot environment.

The purpose is to document the physical flash regions, firmware slots, bootloader environment, raw NAND dumps, and OOB data.

**Important:** This document describes the original firmware layout. It does not provide instructions to erase, repartition, or rewrite the router's flash.

---

## 1. Flash Overview

The MA141 uses SPI-NAND flash.

The supplied archive contains:

- Bootloader partition.
- Two U-Boot environment partitions.
- Configuration partition.
- Two kernel partitions.
- Two root filesystem partitions.
- Writable root filesystem data.
- Free flash space.
- Two additional small partition dump files.
- Corresponding OOB dumps for most partitions.

The saved U-Boot environment identifies the storage interface as:

```text
spi_nand
```

The environment defines the following bootloader flash-size variables:

```text
fl_boot_sz=c0000
fl_env=c0000
fl_env2=e0000
fl_env_sz=20000
```

The offsets and sizes are hexadecimal byte values.

## 2. Physical Partition Inventory

The following table summarizes the supplied partition dump files.

| MTD | Filename | Size |
|---|---|---:|
| MTD0 | `mtd0_bootloader.bin` | 768 KiB |
| MTD1 | `mtd1_env.bin` | 128 KiB |
| MTD2 | `mtd2_env2.bin` | 128 KiB |
| MTD3 | `mtd3_config.bin` | 24 MiB |
| MTD4 | `mtd4_k0_kernel.bin` | 8 MiB |
| MTD5 | `mtd5_r0_rootfs.bin` | 20 MiB |
| MTD6 | `mtd6_k1_kernel.bin` | 8 MiB |
| MTD7 | `mtd7_r1_rootfs.bin` | 20 MiB |
| MTD8 | `mtd8_rootfs_data.bin` | 12 MiB |
| MTD9 | `mtd9_free.bin` | 32.5 MiB |
| MTD10 | `mtd10_Partition_010.bin` | 4 KiB |
| MTD11 | `mtd11_Partition_011.bin` | 4 KiB |

The table reflects the supplied dump filenames and file sizes. The exact relationship between MTD10/MTD11 and the additional Linux partition aliases in the boot arguments remains under investigation.

## 3. Flash Offsets

The bootloader environment provides explicit offsets and sizes for the main firmware regions.

All values below are hexadecimal.

| Region | Start offset | Size |
|---|---:|---:|
| Bootloader | `0x00000000` | `0x000C0000` |
| Environment 1 | `0x000C0000` | `0x00020000` |
| Environment 2 | `0x000E0000` | `0x00020000` |
| Configuration | `0x00100000` | `0x01800000` |
| Kernel Slot 0 | `0x01900000` | `0x00800000` |
| RootFS Slot 0 | `0x02100000` | `0x01400000` |
| Kernel Slot 1 | `0x03500000` | `0x00800000` |
| RootFS Slot 1 | `0x03D00000` | `0x01400000` |
| RootFS Data | `0x05100000` | `0x00C00000` |
| Free Region | `0x05D00000` | `0x02080000` |

These offsets are derived from the saved environment:

```text
fl_boot_sz=c0000

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

The environment also provides:

```text
mtdparts0=
mtdparts=spinand:768K(boot),128K(env),128K(env2),24M(config),8M(k0)ro,20M(r0)ro,8M(k1),20M(r1),12M(rootfs_data),33280K(free),4K@0ro,4K@0ro,8M@25M(linux),20M@33M(rootfs) root=31:5
```

And for the other firmware slot:

```text
mtdparts1=
mtdparts=spinand:768K(boot),128K(env),128K(env2),24M(config),8M(k0),20M(r0),8M(k1)ro,20M(r1)ro,12M(rootfs_data),33280K(free),4K@0ro,4K@0ro,8M@53M(linux),20M@61M(rootfs) root=31:7
```

The `ro` markers indicate read-only partitions in the corresponding Linux partition definitions.

The additional `@` entries are partition aliases or separately defined Linux partition regions. They should not automatically be treated as additional physical NAND regions.

## 4. Dual-Firmware Architecture

The MA141 has two firmware slots.

Each slot contains a separate kernel and root filesystem.

### Slot 0

```text
Kernel:
mtd4_k0_kernel.bin

Root filesystem:
mtd5_r0_rootfs.bin

Firmware version:
4.1.1_20201102-201102
```

### Slot 1

```text
Kernel:
mtd6_k1_kernel.bin

Root filesystem:
mtd7_r1_rootfs.bin

Firmware version:
4.1.2_20220208-220208
```

The bootloader environment records:

```text
sw_version0=4.1.1_20201102-201102
sw_version1=4.1.2_20220208-220208

sw_active=1
sw_tryactive=2
sw_commit=1
```

The saved environment therefore indicates that slot 1 is selected by the committed boot configuration.

The bootloader uses separate commands for the two slots.

```text
b0=setenv bootargs ${bootargs_base} ${mtdparts0};spi_nand read ${freeAddr} ${fl_kernel1} ${fl_kernel1_sz};bootm ${freeAddr}

b1=setenv bootargs ${bootargs_base} ${mtdparts1};spi_nand read ${freeAddr} ${fl_kernel2} ${fl_kernel2_sz};bootm ${freeAddr}
```

The first command loads the kernel from the first firmware slot, while the second loads the kernel from the second firmware slot.

The root filesystem is subsequently identified through the Linux partition arguments.

## 5. U-Boot Environment Partitions

The archive contains two environment dumps:

```text
mtd1_env.bin
mtd2_env2.bin
```

Both files are 128 KiB.

Their offsets are:

```text
Environment 1: 0x000C0000
Environment 2: 0x000E0000
```

Both supplied environment dumps contain the same principal boot configuration, including:

- Boot selection variables.
- Kernel and root filesystem offsets.
- Firmware version identifiers.
- Network configuration.
- NAND update commands.
- Serial console configuration.

The environment also contains:

```text
baudrate=115200
bootdelay=5
freeAddr=81000000
bootargs_base=console=ttyS0,115200
```

These are values recorded in the supplied dump, not necessarily universal settings for every MA141 hardware revision.

## 6. NAND OOB Dumps

The archive includes separate files with the `_OOB.bin` suffix.

Examples:

```text
mtd0_bootloader_OOB.bin
mtd1_env_OOB.bin
mtd2_env2_OOB.bin
mtd3_config_OOB.bin
mtd4_k0_kernel_OOB.bin
mtd5_r0_rootfs_OOB.bin
mtd6_k1_kernel_OOB.bin
mtd7_r1_rootfs_OOB.bin
mtd8_rootfs_data_OOB.bin
mtd9_free_OOB.bin
```

OOB means Out-of-Band data.

NAND flash commonly stores additional bytes alongside its main data pages. These additional bytes can be used for:

- Error Correction Code (ECC).
- Bad-block markers.
- NAND-specific metadata.
- Other controller or filesystem information.

The supplied archive preserves this additional data separately.

### Observed OOB Sizes

| Main dump | Main size | OOB dump size |
|---|---:|---:|
| MTD0 | 768 KiB | 792 KiB |
| MTD1 | 128 KiB | 132 KiB |
| MTD2 | 128 KiB | 132 KiB |
| MTD3 | 24 MiB | 24.75 MiB |
| MTD4 | 8 MiB | 8.25 MiB |
| MTD5 | 20 MiB | 20.625 MiB |
| MTD6 | 8 MiB | 8.25 MiB |
| MTD7 | 20 MiB | 20.625 MiB |
| MTD8 | 12 MiB | 12.375 MiB |
| MTD9 | 32.5 MiB | 33.515625 MiB |

The consistent size difference is compatible with additional OOB bytes being included in the dumps.

The exact page layout, ECC algorithm, and bad-block marker positions still need to be established.

## 7. Kernel Image Partitions

Both kernel partitions contain U-Boot legacy image headers.

The supplied files begin with:

```text
27 05 19 56
```

This is the standard U-Boot legacy image magic:

```text
0x27051956
```

### Slot 0 Kernel

```text
Filename:
mtd4_k0_kernel.bin

Image size:
3,249,852 bytes

Load address:
0x80000000

Entry point:
0x80000000

Architecture:
MIPS

Compression:
LZMA
```

### Slot 1 Kernel

```text
Filename:
mtd6_k1_kernel.bin

Image size:
3,330,651 bytes

Load address:
0x80000000

Entry point:
0x80000000

Architecture:
MIPS

Compression:
LZMA
```

The kernel partitions are 8 MiB each, so their actual image payloads occupy less than the full partition capacity.

The archive also contains:

```text
mtd6_k1_kernel_payload.lzma
```

This is an extracted compressed payload associated with the second kernel image.

## 8. Root Filesystem Partitions

Both root filesystem images begin with:

```text
hsqs
```

This is the SquashFS filesystem magic.

Files:

```text
mtd5_r0_rootfs.bin
mtd7_r1_rootfs.bin
```

The supplied root filesystem images are 20 MiB each.

Both should be analyzed independently because they belong to different firmware versions.

The filesystem contents may contain:

- Linux system utilities.
- Vendor management applications.
- Startup scripts.
- Network configuration.
- Wireless configuration.
- Authentication components.
- Kernel modules.
- Firmware-specific services.

The root filesystem is one of the most useful areas for further firmware and rooting research.

## 9. Important Bootloader Variables

Some useful variables from the supplied environment:

```text
baudrate=115200
bootdelay=5
freeAddr=81000000

fl_boot_sz=c0000
fl_env=c0000
fl_env2=e0000
fl_env_sz=20000

sw_active=1
sw_tryactive=2
sw_commit=1

sw_version0=4.1.1_20201102-201102
sw_version1=4.1.2_20220208-220208

ethact=RTL SwCore
```

The environment also contains TFTP-based firmware update commands.

These commands are documented as evidence of the bootloader's capabilities. Their actual behavior should be investigated without executing destructive operations.

## 10. Flash Safety

**WARNING: NAND flash operations can permanently damage the router or destroy its recovery capability.**

The supplied environment includes commands that erase and rewrite flash regions.

Examples include:

```text
spi_nand erase
spi_nand write
spi_nand write.raw
```

The environment also defines update shortcuts for:

- Bootloader replacement.
- Kernel replacement.
- Root filesystem replacement.
- Firmware image updates.

These commands are included in the research archive for documentation only.

Do not run them as experiments.

Original partition dumps should always be preserved before any further investigation.

## 11. Research Status

| Research item | Status |
|---|---|
| NAND partition dump collection | Available |
| Main partition sizes | Identified |
| Main flash offsets | Extracted from environment |
| Dual-slot firmware layout | Confirmed |
| Boot selection variables | Extracted |
| Kernel image format | Identified |
| Kernel architecture | MIPS confirmed |
| Root filesystem format | SquashFS identified |
| OOB dump collection | Available |
| Exact NAND chip model | Pending |
| NAND page and block geometry | Pending |
| ECC implementation | Pending |
| Bad-block marker layout | Pending |
| MTD10/MTD11 mapping | Pending |

## 12. Original Evidence

The information in this document is based on the following supplied files:

```text
mtds.zip

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

The corresponding OOB files are also included in the original archive.

## Disclaimer

This is an independent hardware and firmware research project.

The findings are based on available NAND dumps and extracted bootloader environment data. Some hardware details and interpretations remain unverified and may be updated as further research is performed.

No affiliation with Aminia or the router manufacturer is claimed.
