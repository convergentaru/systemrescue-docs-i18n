+++
title = "Mounting NTFS Filesystems"
+++

## Introduction

NTFS (New Technology File System) is the default filesystem used in recent versions of Windows, and the Windows system disk is based on NTFS format.

Linux has offered full read support for NTFS filesystems for a long time.
However, until recently, write support for NTFS that came with the kernel
was missing. To get access to full read/write support for NTFS disks,
you had to use [ntfs-3g](https://wiki.archlinux.org/title/NTFS-3G), which is a userspace program,
and consequently, it was very slow.

SystemRescue version 9.00 comes with Linux-5.15, which brings
[kernel-supported ntfs3 filesystem driver](https://www.kernel.org/doc/html/latest/filesystems/ntfs3.html).
This modern kernel module provides full read/write support for NTFS on Linux.
This is the recommended way to access NTFS disks from SystemRescue. At this stage
ntfs-3g is still the default NTFS driver, so you should explicitly specify that
you want to use ntfs3 to take advantage of this new driver.

If you don't know the device name that contains the NTFS filesystem,
you should use `lsblk` and `blkid` or GParted to identify the disk. In
the following examples we will assume that the device name is `/dev/xxx`, please replace
this with the actual device name.

---

## Full Read/Write Support

If you need full NTFS write support, you should use a mount command such as:

```bash
# Create mount point
mkdir -p /mnt/windows

# Mount NTFS partition with full read/write support
mount -t ntfs3 /dev/sdXN /mnt/windows

# Example with specific options
mount -t ntfs3 -o umask=022 /dev/sdb1 /mnt/windows
```

### ntfs3 Mount Options

| Option | Description | Example |
|--------|-------------|---------|
| `umask=` | Set file permissions | `umask=022` (rw-r--r--) |
| `uid=` | Set owner user ID | `uid=1000` |
| `gid=` | Set group ID | `gid=1000` |
| `flush=` | Flush size in KB | `flush=64` |
| `nodbflags` | Don't use database flags | `nodbflags` |

---

## Read-Only Support

If you only want read support on NTFS disks, you can specify the `-o ro` option
to restrict access to the filesystem). This read-only support is
more secure (you cannot modify or corrupt your data) and can be used for example if
you just want to backup several files, or read a document.

```bash
# Create mount point
mkdir -p /mnt/windows

# Mount NTFS partition in read-only mode
mount -t ntfs3 -o ro /dev/sdXN /mnt/windows
```

### When to Use Read-Only Mode

✅ **Use read-only when:**
- Creating backups of Windows partitions
- Viewing files without modifying them
- The filesystem might be corrupted
- Working with potentially damaged disks
- Forensic examination

❌ **Don't use read-only when:**
- You need to copy files to the NTFS partition
- Modifying or editing files on Windows disk
- Installing software on Windows system

---

## Identifying NTFS Partitions

### Using lsblk

```bash
# List all block devices with filesystem info
lsblk -f

# Output example:
# NAME   FSTYPE LABEL MOUNTPOINT
# sda                                  
# └─sda1 vfat  USB     /media/usb
# └─sda2 ext4  system  /
# └─sda3 ntfs  DATA   
```

### Using blkid

```bash
# Show filesystem labels and types
blkid

# Output example:
# /dev/sda1: LABEL="USB" TYPE="vfat" UUID="..."
# /dev/sda3: LABEL="DATA" TYPE="ntfs" UUID="..."
```

### Using GParted (Graphical)

1. Run `gparted` from menu or command line
2. Select the device from dropdown
3. Look for partitions with NTFS filesystem type (green icon)
4. Note the device name (e.g., `/dev/sdb1`)

---

## Common Mount Scenarios

### Scenario 1: Backup Windows Partition

```bash
# Create backup directory structure
mkdir -p /mnt/windows/{C:,Programs,Backups}

# Mount in read-only mode for safety
mount -t ntfs3 -o ro /dev/sda3 /mnt/windows/C:

# Copy files to backup
rsync -av /mnt/windows/C:/Documents/ /home/user/backups/

# Unmount when done
umount /mnt/windows/C:
```

### Scenario 2: Access Windows Share

```bash
# Create mount point
mkdir -p /mnt/winshare

# Mount with proper permissions
mount -t ntfs3 -o uid=1000,gid=1000,umask=022 /dev/sdb1 /mnt/winshare

# Access files
cd /mnt/winshare
ls -la

# Unmount
umount /mnt/winshare
```

### Scenario 3: Multiple NTFS Partitions

```bash
# Mount multiple partitions
mkdir -p /mnt/windows/{C:,D:,E:}

mount -t ntfs3 -o ro /dev/sda3 /mnt/windows/C:
mount -t ntfs3 -o ro /dev/sda4 /mnt/windows/D:
mount -t ntfs3 -o ro /dev/sda5 /mnt/windows/E:

# Access all partitions
ls /mnt/windows/
```

---

## Troubleshooting NTFS Mount Issues

### "ntfs-3g module not found"

Ensure ntfs-3g is installed:
```bash
# Check if installed
which ntfs-3g

# Install if missing (in SystemRescue)
pacman -S ntfs-3g
```

### "Invalid argument" error

The partition might be in an inconsistent state:
```bash
# Unmount first (if possible)
umount /dev/sdXN

# Check filesystem status
ntfsfix /dev/sdXN

# Try mounting again
mount -t ntfs3 /dev/sdXN /mnt/windows
```

### "Resource busy" error

Something is using the partition:
```bash
# Find processes using the device
lsof +D /dev/sdXN
fuser -v /dev/sdXN

# Kill processes if needed
kill <PID>
```

### "I/O error" on mount

The disk might have errors:
```bash
# Check SMART status
smartctl -a /dev/sdX

# Run filesystem check (⚠️ may take data off partition)
ntfsfix /dev/sdXN
```

---

## Comparison: ntfs3 vs ntfs-3g

| Feature | ntfs3 (kernel) | ntfs-3g (userspace) |
|---------|----------------|---------------------|
| Speed | ⭐⭐⭐ Fast | ⭐ Slow |
| Memory | Low | High |
| Write support | ✅ Full | ✅ Full |
| Stability | Good | Very stable |
| Availability | SysRescue 9.00+ | All versions |

**Recommendation:** Use `ntfs3` on SystemRescue 9.00+ for best performance.

---

## Quick Reference: NTFS Mount Commands

| Task | Command |
|------|---------|
| Full read/write | `mount -t ntfs3 /dev/sdXN /mnt` |
| Read-only | `mount -t ntfs3 -o ro /dev/sdXN /mnt` |
| With permissions | `mount -t ntfs3 -o uid=1000,gid=1000 /dev/sdXN /mnt` |
| Check device | `lsblk -f` |
| Check label/type | `blkid` |
| Unmount | `umount /mnt` |

---

## Security Considerations

### ⚠️ Data Safety Warnings

**1. Always verify the device before mounting:**
```bash
# Double-check device name to avoid accidental data loss
lsblk -f
sudo blkid | grep ntfs
```

**2. Use read-only mode when unsure:**
```bash
# Safer option for inspection and backup
mount -t ntfs3 -o ro /dev/sdXN /mnt/windows
```

**3. Backup before making changes:**
- Always create a disk image before modifying NTFS partitions
- Use `dd` or `rsync` to backup data first

### ⚠️ Data Corruption Risks

The ntfs3 driver provides write support but may not handle all edge cases:
- **Cross-platform corruption**: Windows and Linux may use the filesystem differently, potentially causing corruption
- **Journaling differences**: NTFS journaling works best with Windows' native tools
- **Recommendation**: Use read-only mode unless you're certain about compatibility

### ⚠️ Permissions Issues

When mounting with write access:
- Set appropriate `umask` to restrict permissions (e.g., `umask=022`)
- Specify explicit `uid` and `gid` for proper ownership
- Consider using `nofail` option in fstab to prevent boot failures

### ⚠️ Unmount Properly

**Always unmount before removing hardware:**
```bash
# Check if mounted
df -h | grep ntfs

# Unmount (don't use Ctrl+C on hardware removal!)
umount /mnt/windows
```

⚠️ **Warning**: Never pull USB drives or eject while Windows is still accessing the filesystem. Wait for all processes to complete.

---

## Next Steps

1.  **[Backup tools](./manual_Backup_and_transfer_your_data_using_rsync_en.md)** — Backup NTFS partitions
2.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Access Windows shares via network
3.  **[Mounting other filesystems](./manual_Overview_en.md)** — ext4, xfs, btrfs support

---

## Additional Resources

*   [NTFS-3G Documentation](https://tuxera.com/kb/ntfs-3g-faq/)
*   [Linux NTFS Driver (ntfs3)](https://github.com/ntfs-3g/ntfs-3)
*   [Windows File System Recovery](https://docs.microsoft.com/en-us/windows-server/storage/file-server/recovering-from-fs-corruption)

---

*Last updated: 2026-11-04*
