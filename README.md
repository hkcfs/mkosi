# Arch Linux Rescue System Configuration

This repository contains a configuration to create an Arch Linux-based rescue image using `mkosi`. The image is built with the `x86-64` architecture and is customized to be minimal, while including essential packages and settings for system recovery.

## Table of Contents

1. [Overview](#overview)
2. [Configuration Details](#configuration-details)
   - [Distribution](#distribution)
   - [Output](#output)
   - [Content](#content)
   - [Packages](#packages)
   - [Kernel Modules](#kernel-modules)
   - [Removed Files](#removed-files)
3. [Skeleton Structure](#skeleton-structure)
  - [/etc/pacman.conf](#etcpacmanconf)
  - [/etc/resolv.conf](#etcresolvconf)
  - [/etc/systemd/system-preset/](#etcsystemdsystem-preset)
4. [Usage](#usage)

## Overview

This project creates a rescue image for Arch Linux using the `mkosi` tool. The generated image is compressed using `zstd` and designed to boot on x86-64 systems. The configuration includes basic rescue utilities, a minimal set of packages, and customization of kernel modules for an efficient recovery environment.

## Configuration Details

The configuration file for `mkosi` is split into several sections:

### Distribution

The `[Distribution]` section specifies the base distribution and architecture for the image.

```ini
[Distribution]
Distribution=arch
Architecture=x86-64
```

- **Distribution**: Arch Linux is used as the base OS.
- **Architecture**: The image is built for 64-bit (`x86-64`) systems.

### Output

The `[Output]` section configures how the output image is built and compressed.

```ini
[Output]
ImageId=archlinux-rescue
CompressOutput=zstd
CompressLevel=19
Format=uki
#OutputDirectory=/tmp/mkosi
```

- **ImageId**: The name of the output image is `archlinux-rescue`.
- **CompressOutput**: The image is compressed using the `zstd` compression algorithm.
- **CompressLevel**: Compression level is set to the maximum (19) for smaller image size.
- **Format**: The output format is `uki` (Unified Kernel Image).

### Content

The `[Content]` section defines various system settings, including locale, timezone, and kernel parameters.

```ini
[Content]
Keymap=us
MakeInitrd=no
Timezone=Europe/Amsterdam
Locale=en_US.UTF-8
RootPassword=root
Hostname=archlinux-rescue
Bootloader=none
Bootable=false
```

- **Keymap**: Set to `us` for the US keyboard layout.
- **MakeInitrd**: `no`, as an initramfs is not required.
- **Timezone**: The timezone is set to `Europe/Amsterdam`.
- **Locale**: Locale is set to `en_US.UTF-8`.
- **RootPassword**: The root password is set to `root` (change this in production environments).
- **Hostname**: The system hostname is `archlinux-rescue`.
- **Bootloader**: No bootloader is included, assuming the image is used as a recovery environment.

### Kernel Command Line

The following kernel parameters are used to optimize system performance and disable unnecessary features:

```ini
KernelCommandLine=rw loglevel=0 console=tty0 udev.log_level=0 \
    kvm-intel.nested=1 mitigations=off nowatchdog msr.allow_writes=on \
    pcie_aspm=force module.sig_unenforce cryptomgr.notests \
    no_timer_check noreplace-smp page_alloc.shuffle=1 \
    rcupdate.rcu_expedited=1 tsc=reliable
```

- `loglevel=0`: Minimizes logging.
- `mitigations=off`: Disables certain hardware security mitigations for performance. (HUGE SECURITY RISK)
- `kvm-intel.nested=1`: Enables nested virtualization.
- Additional parameters aim to improve performance and compatibility in virtualized environments.

### Kernel Modules

The configuration includes a selective list of kernel modules to be included or excluded from the image.

```ini
KernelModulesInclude=
    i915
    btrfs
    fs/
    hid/
    input/
    usb/
    dm-.*
    crypto/
    tpm/
    virtio
```

- **Included Modules**: Specific drivers and kernel modules are included, such as:
  - `i915`: Intel integrated graphics driver.
  - `btrfs`: Filesystem support for Btrfs.
  - `usb`, `hid`, `input`: Support for USB and input devices.
  - `crypto`, `tpm`: Cryptographic and TPM modules for secure environments. (WINDOWS 11 would love it)

```ini
KernelModulesExclude=.*
```

- **Excluded Modules**: All kernel modules are excluded by default (`.*`), except those explicitly included above.

### Removed Files

Certain unnecessary files are removed from the image to minimize size.

```ini
RemoveFiles=
    /usr/share/locale
    /var/lib/pacman/sync/*
```

- **Locale files** and **Pacman sync cache** are removed to reduce image size.

### Packages

The `[Packages]` section lists the software to be included in the image:

```ini
Packages=
    base
    intel-ucode
    linux-clear
    ntfs-3g
    linux-firmware
    networkmanager
    dhcpcd
    neovim
    btop
    btrfs-progs
    fastfetch
    less
    arch-install-scripts
```

- **Base**: The minimal Arch Linux base package.
- **intel-ucode**: Microcode updates for Intel CPUs.
- **linux-clear**: Clear Linux kernel, optimized for performance.
- **ntfs-3g**: NTFS filesystem support.
- **Network Utilities**: Includes `networkmanager` and `dhcpcd` for network configuration.
- **Tools**: Utilities like `neovim` (text editor), `btop` (system monitoring), `fastfetch` (system info), and `less` (file pager).
- **Filesystem Tools**: `btrfs-progs` for managing Btrfs.
- **arch-install-scripts**: Scripts for Arch Linux installation.

## Skeleton Structure

The `/mkosi.skeleton/etc/` directory contains essential configuration files that are copied into the resulting disk image before the building of the image.

### `/etc/pacman.conf`

```ini
[options]
HoldPkg     = pacman glibc
Architecture = auto

[multilib]
Include = /etc/pacman.d/mirrorlist

[chaotic-aur]
SigLevel = Optional TrustAll
Include = /etc/pacman.d/chaotic-mirrorlist
```

`multilib` & the `chaotic-aur` have been enabled

### `/etc/resolv.conf`

```bash
nameserver 8.8.8.8
nameserver 8.8.4.4
```

Google’s public DNS servers are used.

### `/etc/systemd/system-preset/`

This directory contains systemd preset files, which define which system services should be enabled or disabled by default.

#### `10-rescue-disable-unneeded.preset`

This preset file disables unneeded services when entering rescue mode:

```ini
disable *
enable rescue.service
```

#### `10-rescue-networking.preset`

This preset ensures that networking is enabled in rescue mode:

```ini
enable systemd-networkd.service
enable systemd-resolved.service
```

#### `10-timesync.preset`

This preset ensures that time synchronization is enabled:

```ini
enable systemd-timesyncd.service

## Usage

To generate the image using `mkosi`, run the following command:

```bash
mkosi -f
```

This will create a compressed rescue image based on the configuration in the `mkosi` file.
The output image will be found in the `mkosi.output` directory