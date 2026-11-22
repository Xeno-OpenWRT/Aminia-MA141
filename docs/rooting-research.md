# MA141 Rooting Research

This document tracks research into obtaining root access on the Aminia MA141 router.

The objective is to understand the device's authentication mechanisms, available interfaces, firmware security, and potential methods of obtaining administrative access.

All research is performed on hardware owned by the researcher.

## 1. Research Objectives

The main objectives are:

- Investigate available administrative accounts.
- Identify root and restricted shell access.
- Examine SSH and Telnet availability.
- Analyze the web administration interface.
- Inspect Linux users, groups, and permissions.
- Investigate firmware update mechanisms.
- Understand bootloader access and recovery options.
- Document confirmed root-access methods.

## 2. Known Platform Information

| Component | Information |
|---|---|
| Device | Aminia MA141 |
| Physical SoC marking | `RTL8685FB` |
| Software-reported SoC | `RTL8685SB` |
| Architecture | MIPS |
| RAM | 128 MB |
| Flash | 128 MB |
| Flash type | SPI-NAND |
| Firmware architecture | Dual-slot |
| Root filesystem | SquashFS 4.0 |

See [Hardware Documentation](hardware.md), [NAND Flash Layout](nand-layout.md), and [Firmware Analysis](firmware-analysis.md).

## 3. Access Interfaces to Investigate

### Serial Console

The bootloader serial console has been observed.

Known settings:

```text
baudrate=115200
bootdelay=5
```

The bootloader prompt is:

```text
VD2#
```

Further research should establish what access is available after Linux starts and whether the serial console provides an authenticated shell.

### SSH

Investigate whether the router runs an SSH server.

Questions to answer:

- Is SSH enabled?
- Which SSH server implementation is installed?
- Which accounts can authenticate?
- Is password authentication enabled?
- Are there restrictions on administrative logins?

### Telnet

Investigate whether Telnet is available.

Questions to answer:

- Is the Telnet service enabled?
- Does it provide a login prompt?
- Which accounts are accepted?
- Is access restricted to the local network?

### Web Administration Interface

The web interface may expose information about the firmware and system configuration.

Research areas include:

- Authentication mechanisms.
- Session management.
- Administrative permissions.
- Firmware upload functionality.
- Diagnostic and debugging endpoints.
- Configuration export and import.

## 4. Linux Account Investigation

The extracted root filesystem should be inspected for account information.

Important files include:

```text
/etc/passwd
/etc/shadow
/etc/group
/etc/sudoers
/etc/inittab
/etc/profile
```

These files can help identify:

- Available accounts.
- User IDs and group IDs.
- Login shells.
- Privileged groups.
- Authentication configuration.
- Startup behavior.

Do not assume that an account named `root` is remotely accessible simply because it exists in `/etc/passwd`.

## 5. Firmware Security Investigation

The firmware should be examined for security mechanisms that could affect root access.

Areas of interest:

- Password storage and authentication.
- Privileged services.
- File and directory permissions.
- SUID and SGID binaries.
- Writable startup scripts.
- Firmware update validation.
- Image signatures and checksums.
- Recovery and rollback mechanisms.

Any potential vulnerability should be verified before being documented as exploitable.

## 6. Bootloader and Recovery Research

The U-Boot environment provides useful information about firmware selection and NAND storage.

Research should investigate:

- Available bootloader commands.
- Environment variable persistence.
- Firmware-slot selection.
- Recovery behavior.
- Boot failure handling.
- Read-only NAND inspection methods.

The presence of NAND write commands does not establish that modifying firmware is necessary to obtain root access.

Preserve the original bootloader, environment, firmware partitions, and OOB data before attempting any modifications.

## 7. Findings and Evidence

Use this section to record verified discoveries.

| Finding | Status | Evidence |
|---|---|---|
| Serial bootloader access | Confirmed | Captured U-Boot console |
| Dual firmware slots | Confirmed | NAND layout and bootloader environment |
| Root shell access | Not yet documented | Requires verification |
| SSH root login | Unknown | Requires investigation |
| Telnet root login | Unknown | Requires investigation |
| Web interface privilege escalation | Unknown | Requires investigation |
| Firmware update security | Unknown | Requires investigation |

A finding should only be marked confirmed when there is sufficient evidence.

## 8. Research Log

Record important experiments here.

Suggested format:

```text
Date:
Research area:
Device state:
Method:
Commands or tools:
Observed result:
Conclusion:
Backup available:
```

Keep logs factual. Include error messages and unexpected behavior because failed experiments can also provide useful information.

## 9. Safety and Backup Policy

Before modifying firmware or system configuration:

1. Preserve original firmware dumps.
2. Preserve NAND OOB data where available.
3. Keep copies of bootloader and environment partitions.
4. Record the original partition layout.
5. Avoid unnecessary NAND erase or write operations.
6. Verify recovery options before making changes.

Never publish device-specific passwords, private keys, serial numbers, or sensitive configuration data.

## 10. Current Research Status

The MA141's hardware, NAND layout, U-Boot environment, and firmware structure have been partially documented.

Root access methods have not yet been established in this document.

The next stage is to inspect the extracted root filesystem and identify its services, accounts, authentication configuration, and privilege boundaries.

---

*Documentation status: Rooting research framework established. Confirmed access methods remain to be documented.*
