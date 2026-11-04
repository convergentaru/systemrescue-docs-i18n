+++
title = "Installing SystemRescue on Disk"
+++

## Overview

This page explains how to install SystemRescue on a local disk so it can be booted without using removable media. These instructions apply to SystemRescue version 6.0.0 and later.

---

## Introduction

There's a simple way to boot SystemRescue from a local disk. This is convenient if you frequently want to boot into SystemRescue, as it will be faster and there's no need to insert any media before restarting the computer. This method allows you to use SystemRescue just like with CDROM or USB versions. It also allows you to boot into SystemRescue if your computer doesn't have a CD/DVD drive or USB port.

You can use another operating system to handle the installation of SystemRescue and its activation. Thus, this method is recommended if you frequently use SystemRescue and want to avoid having to insert a device every time.

Installation is performed by copying SystemRescue files from media to the filesystem of a local disk where your operating system is installed. This approach does not require repartitioning your disk. Using grub4dos, you can even install SystemRescue on a Windows NTFS partition. All you need to do is install the files that are on the SystemRescue disk onto an existing partition on your local disk, which can be either a Linux partition (ext4, xfs, ...) or a Windows partition (NTFS). Thus, you can boot into SystemRescue from your Windows partition, and using the `copytoram` option, you can even use it for troubleshooting Windows itself, or `ntfs-3g` to work with the filesystem on which it's installed.

This installation has two steps: first you need to copy the main SystemRescue files onto your disk partition. Then you need to configure your bootloader. The installation process requires a partition with sufficient space to copy the SystemRescue files. You should have a working Grub2 bootloader if you want to install SystemRescue files on a Linux partition.

---

## Approach 1: Using Grub2 and loopback.cfg

Grub2 provides a feature for booting from an ISO image that's stored on a local disk. If you place a copy of `systemrescue-x.y.z.iso` on a filesystem that Grub2 can read, you'll be able to boot into SystemRescue directly from the ISO image stored on your disk. This is very convenient if you frequently update SystemRescue and want to boot into it directly from Grub2, or when you want to configure SystemRescue on a working system as a backup boot option. Thus, even if the normal operating system becomes unusable, as long as GRUB and the corresponding hard disk partition are in working condition, you can boot into SystemRescue.

For this, SystemRescue implements the `loopback.cfg` agreement, as [documented here](https://supergrubdisk.org/wiki/Loopback.cfg).

### grml-rescueboot (Simplest Method)

The easiest way to use this is through the [grml-rescueboot](https://aur.archlinux.org/packages/grml-rescueboot) package, which is available for Arch via AUR and Debian/Ubuntu. Install it, place the SystemRescue ISO image in `/boot/grml/`, and run `update-grub`. Next time you enter your system's GRUB menu, you'll see a new entry. Select it, and a submenu with standard SystemRescue boot options will appear. Pressing `Esc` will return you to the previous menu.

If you don't have enough space on `/boot/` to fit the ISO image, possibly because it's on a small partition, then **grml-rescueboot** can be configured to search for ISO images elsewhere. See the `ISO_LOCATION` parameter in the file `/etc/default/grml-rescueboot`.

### Custom Configuration

Or, if you're comfortable with GRUB configuration language, a menu entry similar to the following should work. Make sure you edit the `iso_path=` line to reflect the name and location of your SystemRescue ISO image file:

```bash
menuentry "SystemRescue" {
  iso_path="/boot/iso/systemrescue.iso"
  export iso_path
  search --set=root --file $iso_path
  # Support for booting recent GRUB versions on UEFI systems
  rmmod tpm
  loopback loop $iso_path
  set root=(loop)
  # Path below is on ISO SystemRescue, not on hard disk
  configfile /boot/grub/loopback.cfg
}
```

This can typically be added to `/boot/grub/custom.cfg` or `/etc/grub.d/40_custom`.

**Important Notes:**

1. It's recommended to remove the version number from the ISO filename so you don't have to update Grub configuration every time you download a new ISO version
2. If you edit `40_custom`, you typically need to run `update-grub`, `grub-mkconfig -o /boot/grub/grub.cfg`, or `grub2-mkconfig -o /boot/grub2/grub.cfg` (depending on your Linux distribution) afterward
3. You may need to load some GRUB modules (e.g., `lvm`) for GRUB to be able to read the disk location containing the ISO image file
4. The ISO path relative to its containing filesystem may differ from the path you see on the running system. For example, if `/boot/` is on a separate partition, then the correct path above would be `/iso/systemrescue.iso`, without the `/boot/` prefix) More information about installing and configuring GRUB can be found [here](https://wiki.archlinux.org/title/GRUB).

### Boot Process Steps

It's important to understand the main steps of the boot process to configure it correctly:

* Grub2 searches across all available partitions for a filesystem containing a file named `/boot/iso/systemrescue.iso`
* The path to this filesystem is then saved in the `root` variable
* The iso file is opened via the loopback mechanism
* The `root` variable is now set to use the loopback device
* The configuration file `/boot/grub/loopback.cfg`, which is contained inside SystemRescue, is loaded and executed
* This configuration file will use the `iso_path` variable to determine the correct `archiso_param` variable
* Then the standard grub configuration file `/boot/grub/grubsrcd.cfg` from SystemRescue is loaded and executed

---

## Approach 2: Extracting Contents from ISO Image

This approach works with Linux with Grub bootloader and also works with Windows with Grub4dos.

It involves copying the main SystemRescue files onto a filesystem on disk and configuring Grub or Grub4dos so it can boot.

### Step 1: Copy Files from ISO Image to Disk

First, mount the ISO image with `mount` on Linux
(e.g.: `mount -o loop systemrescue-x.y.z.iso /mnt/cdrom`) or you can use
a program like Daemon-Tools on Windows if you didn't burn the disc), to get the main files.

You should copy the entire `/sysresccd/` directory from the ISO to the root folder of the partition it's installed on. If you have a separate `/boot` partition, then this directory should be copied to `/boot/sysresccd`. You should preserve the directory structure as it is on the original ISO.

### Step 2: Update Bootloader Configuration (if installing on Linux)

Now you need to update the bootloader. This section describes how to update grub. If you're using an NTFS partition, please read the next section instead.

We'll need to add a few lines to the bootloader manager configuration file
(typically located in `/etc/grub.d/` for Grub2). You should adapt
the configuration given there.

Here's an example Grub2 configuration. Create a new file such as
`/etc/grub.d/25_sysresccd` so its configuration comes after your
standard operating system configuration. The `search` directive is very important,
as it provides a way for grub to find the filesystem that contains
the SystemRescue files.

In the following example, the filesystem is identified by its label, which
is `boot`, but you can also identify the filesystem by its UUID if you prefer.
After Grub finds the device with this label, it will set it as its
root filesystem. This can be confusing, as what grub considers the root filesystem,
is what Linux will consider the boot filesystem if `/boot` is on a separate
filesystem. The path to SystemRescue files are relative to the root of that filesystem, which may differ from the Linux path.

The option `archisolabel=boot` indicates that SystemRescue will try to find
its files on a filesystem labeled as `boot`, just like grub. The option
`archisobasedir=sysresccd` then indicates that SystemRescue needs to search
for its files in `/sysresccd` on that filesystem. The option `copytoram`
is recommended so the boot filesystem can be unmounted after the boot process completes,
which allows you to make changes on the disk where SystemRescue is installed.

```bash
#!/bin/sh
exec tail -n +3 $0

menuentry 'SystemRescue' {
  load_video
  insmod gzio
  insmod part_gpt
  insmod part_msdos
  insmod ext2
  search --no-floppy --label boot --set=root
  echo   'Loading Linux kernel ...'
  linux  /sysresccd/boot/x86_64/vmlinuz archisobasedir=sysresccd archisolabel=boot copytoram setkmap=us
  echo   'Loading initramfs ...'
  initrd /sysresccd/boot/x86_64/sysresccd.img
}
```

After creating the configuration file, you typically need to run a command such as `grub-mkconfig -o /boot/grub/grub.cfg` to generate the final grub configuration. Make sure the final grub configuration contains the expected entry for SystemRescue before rebooting.

### Step 2: Update Grub4dos Bootloader Manager (if installing on Windows)

Now you need to update your bootloader using grub4dos, which is a port of grub for Windows. This section describes how to install the grub4dos bootloader if you've installed SystemRescue files on an NTFS partition running Windows. If you're using a Linux partition, please read the previous section instead.

One of the interesting things you can do with sysresccd ntfs installation
is Windows troubleshooting when it has problems. Thus, you can
mount the Windows partition with ntfs-3g and fix your Windows (replace registry backup, ...).
The only problem is that you cannot mount the Windows disk read-write with ntfs-3g
because it was already mounted read-only during the boot process. The solution to this problem is to use the `copytoram` option during boot.
When this option is enabled, sysresccd will cache its own files
(found on the ntfs disk) in memory during the boot process, and the ntfs disk
will be unmounted. Thus, it allows you to mount it again with ntfs-3g.

Thus, you need to add `copytoram` to the `menu.lst` boot options if you want to be able to mount your Windows disk with ntfs-3g after booting from the ntfs disk itself.

Installation is really simple. In this mini-tutorial I assume that Windows
is installed on an NTFS disk (Disk-C) and that you've copied the main SystemRescue files to `C:\sysresccd`

You need to download grub4dos, extract the zip file to a temporary directory, and
copy `grldr` to `C:\`. This installation was tested using the `grldr`
file provided with `grub4dos-0.4.3-2007-08-27.zip`, but it should work with any recent version.

Along with `grldr`, you need to copy `grldr.mbr` (part of the archive)
to the root of the Windows boot partition. Then enter a few
commands in the command window (run `cmd.exe` with administrator account).

In the command window (`C:>` is a fictional replacement for cmd prompt)
do the following:

```bash
C:> bcdedit /create /d "SystemRescue [GRUB4DOS]" /application bootsector
```

You get back a boot entry `{id}` — use it (copy/paste or type in) in the next steps:

```bash
C:> bcdedit /set {id} device boot
C:> bcdedit /set {id} path \grldr.mbr
C:> bcdedit /displayorder {id} /addlast
```

You need to create `C:\menu.lst`, which is the configuration file for `grub4dos`. Here's an example. It corresponds to 64-bit architecture (amd64/x86_64). If you're using a 32-bit version, you need to replace all instances of `x86_64` with `i686`:

```bash
# This is a sample menu.lst file for SystemRescue
title    SystemRescue from the NTFS disk
root     (hd0,0)
kernel   /sysresccd/boot/x86_64/vmlinuz archisobasedir=sysresccd archisolabel=windows copytoram setkmap=us
initrd   /sysresccd/boot/x86_64/sysresccd.img
```

In `menu.lst`, you need to update `archisolabel` to match the label of the NTFS filesystem containing `sysresccd`, and also you should update
the `setkmap` parameter to match your keyboard layout. In this
example, files are on an NTFS partition that is the first partition of the first hard disk.
Then the grub device name is `(hd0,0)`. This would be
`(hd1,0)` for the first partition of the 2nd hard disk, `(hd0,1)` for the second
partition of the first hard disk, ... You need to replace `root (hd0,0)` with
the grub device name of your NTFS partition.

---

## Quick Reference: Installation Methods

| Method | Platform | Difficulty | Use Case |
|--------|----------|------------|----------|
| loopback.cfg + grml-rescueboot | Linux | Easy ⭐⭐⭐ | Frequent updates, backup boot |
| Custom Grub2 config | Linux | Medium ⭐⭐ | Full control, custom layouts |
| Extracted files + Grub2 | Linux | Harder ⭐ | No ISO space, Windows bootloader |
| grub4dos on NTFS | Windows | Medium ⭐⭐ | Windows troubleshooting |

---

## Troubleshooting

### Common Issues

**"No bootable device found"**
- Check that the partition label matches in your configuration
- Verify GRUB modules are loaded (try `insmod lvm` if using LVM)
- Ensure `copytoram` is set if you need read-write access

**SystemRescue boots but can't mount partitions**
- Check filesystem type detection: `blkid`
- For NTFS, ensure ntfs-3g is loaded: `modprobe ntfs-3g`
- Try mounting manually: `mount -t ntfs-3g /dev/sdXN /mnt`

**GRUB menu doesn't show SystemRescue entry**
- Run `update-grub` after creating configuration
- Check `/etc/grub.d/25_sysresccd` exists and has correct permissions
- Verify the partition label is correct: `blkid`

### Partition Size Requirements

| Component | Minimum Space | Recommended |
|-----------|---------------|-------------|
| ISO Image | 2 GB | 4+ GB |
| Extracted Files | 1.5 GB | 3+ GB |
| Working Directory | 500 MB | 1+ GB |

---

## Next Steps

1.  **[Booting options](./manual_Booting_SystemRescue_en.md)** — Learn all available boot parameters
2.  **[Getting started](./manual_Starting_to_use_the_system_en.md)** — First steps after installation
3.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Configure network access

---

## Related Documentation

*   [Grub2 Installation Guide](https://www.gnu.org/software/grub/manual/grub/)
*   [grml-rescueboot Package](https://aur.archlinux.org/packages/grml-rescueboot)
*   [Grub4dos for Windows](http://www.riverbankcomputing.com/static/Docs/PyQt5/)

---

*Last updated: 2026-11-04*
