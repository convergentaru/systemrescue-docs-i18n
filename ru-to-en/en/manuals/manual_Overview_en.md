+++
title = "SystemRescue Overview"
+++

## Description

**SystemRescue** is a Linux system available on a bootable CD-ROM that allows you to recover your system and data after a failure. It also aims to provide an easy way to perform administrative tasks on your computer, such as creating and editing hard disk partitions. It contains many system utilities (parted, fsarchiver, filesystem tools, ...) and basic ones (editors, midnight commander, network tools). It aims to be very easy to use: just boot from the boot media, and you can do everything. The system kernel supports most important filesystems (ext3, ext4, xfs, btrfs, reiserfs, vfat, ntfs, iso9660) and network ones (samba and nfs).

You can use SystemRescue for many tasks:

*   First time you're using a computer when no operating system is installed. The first task is creating partitions on the hard disk and installing an operating system. With this CD-ROM you can easily create partitions using the GParted graphical partition tool, and you can install Gentoo Linux.
*   After a crash or error, you may have boot problems. For example, after installing Windows your bootloader (LILO, Grub) may have been removed from MBR. With this CD-ROM you have everything needed to reinstall Grub or Lilo.
*   For Windows users who don't have Linux installed, it provides a small Linux system with most important system tools. For example, Windows users can create a backup of their Windows system partition using Linux tools.

## Contents

Here's a brief list describing what you'll find on this rescue system:

*   New Linux kernel supporting most important filesystems and crucial hardware. Supported filesystems include: ext3, ext4, xfs, reiserfs, fat16, fat32, jfs, ntfs. The kernel supports NFS and Samba.
*   Graphical partition tools that aim to be free clones of partition-magic for Linux. You can use GParted.
*   Most important system console tools for Linux. Of course, you have GNU Parted (partition editor), Partimage or FSArchiver (disk image clone) for creating disk partition backups into a filesystem image. Filesystem tools (e2fsprogs for ext4, xfsprogs, reiserfsprogs for reiserFS, jfsutils, dosfstools for FAT, ntfsprogs for NTFS). You can use dump/restore for creating an ext4 partition backup.
*   ntfs-3g (third-generation NTFS driver) provides full read/write support for NTFS partitions from Linux.
*   Standard tools for Linux users: tar/gzip/bzip2 for file archiving. The same tools for Windows users: zip/unzip, p7zip are provided. This means you're able to backup/restore your Windows data. We added DAR (Disk Archiver). It's a program like tar but more powerful
*   **Midnight Commander** (enter `mc` in the console command line) — it's a free Norton Commander clone for Linux. With mc you can easily browse, copy, move, edit all files on your computer. If you don't know all Linux shell commands, you can start with mc.
*   You can use basic web browsers to get documentation while you're working from the rescue environment.
*   Of course, editors are important when you have problems. Nano (simple editor), vim (enhanced vi) and qemacs (emacs clone) are provided for text mode.
*   The XFCE desktop environment allows using graphical programs such as GParted and graphical text editors like Geany.

---

## Quick Reference

### Supported Filesystems
| Filesystem | Read/Write | Notes |
|------------|------------|-------|
| ext2/ext3/ext4 | ✅ Full | Native Linux filesystems |
| xfs | ✅ Full | High-performance enterprise FS |
| btrfs | ✅ Full | Copy-on-write with snapshots |
| reiserfs | ✅ Full | Journaling filesystem |
| vfat/fat32 | ✅ Full | Windows/Mac compatibility |
| ntfs | ✅ Full | Via ntfs-3g driver |
| iso9660 | ✅ Read-only | CD/DVD images |
| nfs | ✅ Full | Network filesystem |
| samba | ✅ Full | SMB/CIFS protocol |

### Key Tools Available
| Tool | Purpose | Command |
|------|---------|---------|
| GParted | Graphical partitioning | `gparted` or from XFCE menu |
| Midnight Commander | File manager (Norton Commander clone) | `mc` |
| Nano | Simple text editor | `nano filename` |
| Vim/Emacs | Advanced editors | `vim`, `qemacs` |
| Tar/Gzip/Bzip2 | Archiving | `tar -czvf archive.tar.gz dir/` |
| Partimage/FSArchiver | Disk image backup | `partimage`, `fsarchiver` |
| dump/restore | ext4 backup | `dump -f backup.dump /dev/sdXN` |

### Boot Options
*   **Live mode**: Run SystemRescue without installing
*   **Install to USB**: Install SystemRescue onto USB drive
*   **Boot from disk**: Install SystemRescue directly on hard disk

---

## Next Steps

1.  **[Downloading and burning](./manual_Downloading_and_burning_en.md)** — Get the ISO image and create bootable media
2.  **[Installing on USB](./manual_Installing_SystemRescue_on_the_disk_en.md)** — Install SystemRescue to USB flash drive
3.  **[Booting options](./manual_Booting_SystemRescue_en.md)** — Learn all available boot parameters

---

*Last updated: 2026-11-04*
