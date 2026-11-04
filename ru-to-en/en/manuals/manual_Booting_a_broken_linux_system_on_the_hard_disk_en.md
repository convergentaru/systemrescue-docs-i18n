+++
title = "Booting a Broken Linux System on the Hard Disk"
draft = false
aliases = ["/Sysresccd-manual-en_Booting_a_broken_linux_system_on_the_hard_disk"]
+++

## Overview

Sometimes we may have problems booting a Linux system installed on disk. For example, after some administrative tasks or after installing an operating system to your hard disk, you may lose the ability to boot Linux because **another system may have overwritten the MBR or replaced the bootloader**. In such cases, it's useful to use SystemRescue to boot an existing Linux system that is already installed on disk. And as soon as this system starts, you will be able to [repair the bootloader](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/Repairing_a_damaged_Grub.md).

Since SystemRescue version 6.1.4, it supports the `findroot` parameter in the boot command line. You can use it either by entering this parameter in the boot command line or by selecting **Boot from Linux OS installed on disk** in the predefined SystemRescue boot menu. When you start SystemRescue with this option enabled, it will attempt to find and boot any Linux system that is installed on disk. This supports Linux operating systems installed on regular filesystems, including LVM disk and dm-crypt encrypted disk. If multiple systems are found, it will show a menu so you can choose which system to boot.

As soon as you can boot your Linux operating system, you can use various commands to repair the system. This includes commands for reinstalling the bootloader on disk, updating or replacing packages, or updating configuration files.

---

## How findroot Works

### Automatic Detection Process

When `findroot` is enabled, SystemRescue performs the following steps:

1. **Scans all disks** — Identifies all block devices in the system
2. **Searches for Linux filesystems** — Looks for ext4, xfs, btrfs, and other Linux filesystems
3. **Finds boot configurations** — Searches for `/boot/grub/grub.cfg` or `/etc/fstab`
4. **Identifies kernels** — Locates kernel images in `/boot` directories
5. **Presents menu** — Shows available systems if multiple are found

### Supported Filesystems

✅ **Fully supported:**
- ext2, ext3, ext4
- xfs
- btrfs
- vfat (for /boot partitions)

✅ **With additional options:**
- LVM logical volumes
- dm-crypt encrypted partitions
- RAID arrays (software RAID 0, 1, 5, 10)

❌ **Not supported for auto-detection:**
- NTFS (Windows filesystem)
- APFS (Apple filesystem)
- ZFS (requires separate module loading)

---

## Manual Boot with findroot

### From Boot Menu

1. Start SystemRescue from boot media (USB/CD)
2. Wait for the boot menu to appear
3. Select **"Boot from Linux OS installed on disk"**
4. Choose the desired system from the list if prompted

### From Command Line

Add `findroot` to the boot parameters:

```bash
# BIOS mode
linux /sysresccd/boot/x86_64/vmlinuz findroot
initrd /sysresccd/boot/x86_64/sysresccd.img

# UEFI mode  
efi /sysresccd/boot/x86_64/sysrescue.efi findroot
```

Or with additional options:

```bash
# Load into RAM + automatic detection
linux /sysresccd/boot/x86_64/vmlinuz copytoram findroot setkmap=us
initrd /sysresccd/boot/x86_64/sysresccd.img

# Disable firewall for easier access
linux /sysresccd/boot/x86_64/vmlinuz nofirewall findroot
initrd /sysresccd/boot/x86_64/sysresccd.img
```

---

## Manual Root Finding (Advanced)

If automatic detection fails or you need more control, you can manually specify the root filesystem:

### Step 1: Identify Available Disks

```bash
# List all block devices
lsblk -f

# Check filesystem labels and UUIDs
blkid

# Detailed device information
fdisk -l
```

Example output:
```
NAME   FSTYPE LABEL            MOUNTPOINT
sda                                  
├─sda1 vfat  BOOT             /boot
├─sda2 ext4  SYSTEM           /
└─sda3 swap SWAP-1           [swap]
```

### Step 2: Mount the Root Filesystem

```bash
# Create mount point
mkdir -p /mnt/root

# Mount root partition (replace with actual device)
mount /dev/sda2 /mnt/root

# If using LVM, first activate volumes
vgchange -ay
lvscan

# Then mount logical volume
mount /dev/vg0/root /mnt/root
```

### Step 3: Mount Additional Partitions

```bash
# Mount /boot if not already mounted
mount /dev/sda1 /mnt/root/boot

# Mount /home if separate partition
mount /dev/sda4 /mnt/root/home

# Mount /var if separate partition
mount /dev/sda5 /mnt/root/var

# Mount /etc if separate partition (rare)
mount /dev/sda6 /mnt/root/etc
```

### Step 4: Bind Mount Necessary Directories

```bash
# Bind mount virtual filesystems
mount --bind /dev /mnt/root/dev
mount --bind /proc /mnt/root/proc
mount --bind /sys /mnt/root/sys

# For chroot environment
mount --rbind /dev /mnt/root/dev
mount --rbind /proc /mnt/root/proc
mount --rbind /sys /mnt/root/sys
```

### Step 5: Boot into the System

```bash
# Method 1: Direct boot with manual root
linux /path/to/kernel root=/dev/sda2 findroot=manual
initrd /path/to/initramfs.img

# Method 2: After manual mounting, use chroot
arch-chroot /mnt/root

# Method 3: Use dracut to regenerate initramfs
dracut --force --regenerate-all
```

---

## Repairing the Bootloader

Once you have booted into the broken system (either automatically or manually), you can repair the bootloader.

### GRUB2 Repair

```bash
# Identify GRUB version and location
which grub-install
grub-probe --target-bootdevice

# Install GRUB to MBR (BIOS)
grub-install /dev/sda

# Install GRUB to EFI partition (UEFI)
mount /dev/sda1 /mnt/boot  # Replace with your EFI partition
efibootmgr -c -d /dev/sda -s /boot/grub/grub.cfg

# Update GRUB configuration
update-grub
grub2-mkconfig -o /boot/grub/grub.cfg

# Verify installation
grub-probe --target-bootdevice
```

### Chroot Method (Recommended)

```bash
# Mount the broken system
mount /dev/sda2 /mnt/root
mount /dev/sda1 /mnt/root/boot  # if separate /boot
mount --bind /dev /mnt/root/dev
mount --bind /proc /mnt/root/proc
mount --bind /sys /mnt/root/sys

# Chroot into the system
arch-chroot /mnt/root

# Repair GRUB
grub-install /dev/sda
update-grub

# Exit chroot and reboot
exit
reboot
```

---

## Common Issues and Solutions

### "No Linux systems found"

**Problem:** Automatic detection finds no Linux installations.

**Solutions:**
1. Check if `/boot` is accessible:
   ```bash
   ls /mnt/root/boot/
   ```
2. Verify fstab entries:
   ```bash
   cat /mnt/root/etc/fstab
   ```
3. Manually specify root device:
   ```bash
   linux ... root=/dev/sda2 findroot=manual
   ```

### "Filesystem check required"

**Problem:** Filesystem needs fsck before mounting.

```bash
# Check and repair ext4 filesystem
fsck -y /dev/sda2

# Check XFS filesystem
xfs_repair /dev/sda2

# Check BTRFS filesystem
btrfs check --repair /dev/sda2
```

### "Cannot find kernel"

**Problem:** Kernel images are missing or in unexpected location.

```bash
# Search for kernel images
find /mnt/root/boot -name "vmlinuz*" -o -name "Image*"

# Check for alternative locations
ls -la /mnt/root/boot/
ls -la /mnt/root/lib/modules/

# Regenerate initramfs if needed
dracut --force
```

### LVM Issues

**Problem:** Cannot access LVM logical volumes.

```bash
# Activate all volume groups
vgchange -ay

# Scan for LVs
lvscan

# Check VG status
vgs
lvs

# Mount specific LV
mount /dev/vg0/root /mnt/root
```

### Encrypted Partitions (dm-crypt)

**Problem:** Cannot access encrypted root partition.

```bash
# Open encrypted partition
cryptsetup open /dev/sda3 cryptroot

# Activate volume group if LVM
vgchange -ay /dev/mapper/cryptroot

# Mount logical volume
mount /dev/mapper/vg0-root /mnt/root

# Or use findroot with encryption info
linux ... root=/dev/mapper/cryptroot findroot
```

---

## Quick Reference Table

| Task | Command | Notes |
|------|---------|-------|
| List devices | `lsblk -f` | Shows filesystems and UUIDs |
| Check labels | `blkid` | Displays device labels |
| Auto-detect Linux | `findroot` | Menu selection or boot param |
| Manual root mount | `mount /dev/sdXN /mnt/root` | Replace with actual device |
| Bind mounts | `mount --bind /dev /mnt/dev` | Required for chroot |
| Chroot | `arch-chroot /mnt/root` | Enter broken system |
| Install GRUB (BIOS) | `grub-install /dev/sda` | MBR installation |
| Install GRUB (UEFI) | `efibootmgr -c ...` | EFI boot entry |
| Regenerate GRUB | `update-grub` | Update bootloader config |
| Activate LVM | `vgchange -ay` | Activate volume groups |
| Open crypt | `cryptsetup open ...` | Decrypt encrypted partition |

---

## Best Practices

### ✅ Recommended

1. **Backup before repairs** — Create disk image before making changes
2. **Use auto-detection first** — Let `findroot` find systems automatically
3. **Verify partitions** — Check fstab and /boot before chroot
4. **Test after repair** — Boot from media to verify fix worked

### ❌ Not Recommended

1. **Editing MBR directly** — Use grub-install instead
2. **Skipping LVM activation** — Will fail on LVM systems
3. **Mounting with wrong options** — May cause filesystem corruption
4. **Ignoring fsck warnings** — Can lead to data loss

---

## Security Considerations

### ⚠️ Root Access

When repairing a system, you'll have root access:

1. **Don't modify user data** — Only repair bootloader
2. **Verify changes** — Check what files are being modified
3. **Test in safe mode** — Boot with recovery options first

### Password Requirements

If the target system has a root password:

```bash
# Set or verify root password
passwd

# Or use encrypted password at boot
linux ... rootcryptpass=$6$... findroot
```

---

## Troubleshooting Guide

### System Won't Boot After Repair

**Symptoms:** Same error as before repair

**Steps:**
1. Reboot from SystemRescue media
2. Check if GRUB was installed to correct device:
   ```bash
   ls -l /dev/sda*  # Should show new entries
   ```
3. Verify EFI entry (UEFI systems):
   ```bash
   efibootmgr
   ```
4. Check bootloader configuration:
   ```bash
   cat /boot/grub/grub.cfg
   ```

### GRUB Errors

**"Error 21: Cannot find a usable device"**

```bash
# Verify device exists and is writable
lsblk
fdisk -l /dev/sda

# Check if partition table is readable
parted /dev/sda print
```

**"No bootable device found"**

```bash
# Ensure kernel exists in /boot
ls /mnt/root/boot/vmlinuz-*

# Check initramfs integrity
ls -lh /mnt/root/boot/initramfs-*
```

---

## Next Steps

1.  **[Repairing GRUB](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/Repairing_a_damaged_Grub.md)** — Detailed bootloader repair guide
2.  **[LVM troubleshooting](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/disk-partitioning_Introduction-to-disk-partitioning.md)** — LVM-specific repairs
3.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Remote access for backups

---

## Additional Resources

*   [GRUB Manual](https://www.gnu.org/software/grub/manual/grub.html)
*   [LVM Documentation](https://man7.org/linux/man-pages/man8/lvm.8.html)
*   [dm-crypt Setup](https://cryptography.io/en/latest/hazmat/primitives/symmetric_ciphers/)
*   [SystemRescue Persistence](./manual_Creating_a_backing_store_en.md)

---

*Last updated: 2026-11-04*
