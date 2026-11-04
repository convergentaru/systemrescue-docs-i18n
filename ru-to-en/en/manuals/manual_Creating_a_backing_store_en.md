+++
title = "Creating a Backing Store"
+++

## Overview

SystemRescue is based on a read-only compressed filesystem. All system files are stored in a large squashfs image file. Consequently, **changes you make to the system are not persisted**, and they're lost upon reboot (except for what you do on other filesystems that you might have mounted).

The root filesystem in SystemRescue is overlayfs since version 6.0. Therefore, **all changes made to system files are allowed and stored in RAM**. This allows you to modify a system file, for example, you can replace a program with your own version, or you can make changes to application configurations such as Firefox.

Users who want to persist their changes in system files can create a custom SystemRescue carrier. This is very convenient when you want to add new programs to the system, but it's not comfortable if you frequently need to change files in the system. You don't want to create a new customized version every day.

This is why SystemRescue provides the backing-store feature. Backing-store is a filesystem saved on a USB flash drive or hard disk that contains all modified system files. Changes are saved to backing-store every time you edit a file, when you create a new directory, or when the system writes or deletes a file for any other reason. Consequently, this allows you to persist your configuration changes: you can add bookmarks and extensions in Firefox, and they'll still be there when you reboot SystemRescue, as long as the same backing-store is loaded.

Backing-stores are incompatible between SystemRescueCd-5.x and SystemRescue from version 6.0 onwards, since they use different filesystem merge layers (aufs vs overlayfs).

---

## Using Backing-Store with SystemRescue-6.x and Later

You need a Linux filesystem where SystemRescue is allowed to store its changes. This can be any Linux filesystem such as ext4, xfs, or btrfs. You can either create a special filesystem for storing these changes, or you can reuse an existing filesystem since changes will be isolated in a directory. You should provide SystemRescue with an option in the boot command line or in [YAML configuration file](./manual_Configuring_SystemRescue_en.md) so it can identify the filesystem where to store these changes. The recommended method is to specify the filesystem label, but this isn't the only way.

Then you need to run SystemRescue with the `cow_label=xxxx` option, where `xxxx` is the label of the Linux filesystem where you want changes to be saved. For example, you use `cow_label=boot` if your `/boot` filesystem is labeled as `boot`. This is similar to `cow_device=/dev/disk/by-label/boot`.

How to set a filesystem label depends on the filesystem used. For ext4 use `tune2fs -L <newlabel> <device>`, for xfs use `xfs_admin -L <newlabel> <device>` and `btrfs filesystem label <device> <newlabel>` for btrfs.

By default, changes will be saved in a directory named `persistent_${archisolabel}/${arch}`. So, for example, `persistent_RESCUE904/x86_64`. You can override this default with the `cow_directory=xxxx` option.

See [Booting SystemRescue](./manual_Booting_SystemRescue_en.md) for more detailed information about boot options supported in SystemRescue.

---

## How Backing-Store Works

### Overlayfs Architecture

```
┌─────────────────────────────────────┐
│         SquashFS (Read-only)        │
│   /usr, /bin, /etc (original files) │
└─────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────┐
│       OverlayFS Merge Layer         │
│     Combines upper and lower dirs   │
└─────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────┐
│    Persistent Directory (RAM/USB)    │
│  /persistent/RESCUE904/x86_64        │
│  - Modified system files            │
│  - New configuration files          │
│  - Browser bookmarks/extensions      │
└─────────────────────────────────────┘
```

### What Gets Persisted

✅ **Persisted with backing-store:**
- Modified configuration files (`/etc/fstab`, `/etc/ssh/sshd_config`)
- Application settings (Firefox bookmarks, browser profiles)
- Installed packages (if using pacman in persistent mode)
- Created directories and files
- Network configurations
- SSH authorized keys

❌ **Not persisted:**
- Temporary files (`/tmp/*`)
- Cache files (`/var/cache/*`)
- Session data
- Files in read-only mounted partitions

---

## Setting Up a Persistent Storage

### Method 1: Using Existing Partition (Recommended)

```bash
# Identify available partitions
lsblk

# Check filesystem labels
blkid

# Label an unused partition for persistence
# Example for ext4:
tune2fs -L PERSISTENT /dev/sdb2

# Or for xfs:
xfs_admin -L PERSISTENT /dev/sdb2

# Or for btrfs:
btrfs filesystem label /dev/sdb2 PERSISTENT
```

### Method 2: Creating New Partition

```bash
# Create new partition (example)
fdisk /dev/sdb
# Create new partition, set type to Linux (83)
# Write changes and exit

# Format as ext4
mkfs.ext4 /dev/sdb2

# Label it
tune2fs -L PERSISTENT /dev/sdb2

# Mount to verify
mkdir /mnt/persistent
mount /dev/sdb2 /mnt/persistent
ls /mnt/persistent
```

### Method 3: Using USB Flash Drive

```bash
# Identify USB drive
lsblk

# Format USB as ext4 (⚠️ wipes all data!)
mkfs.ext4 /dev/sdX1

# Label it
tune2fs -L PERSISTENT_USB /dev/sdX1

# Mount to verify
mkdir /mnt/persistent_usb
mount /dev/sdX1 /mnt/persistent_usb
```

---

## Using Backing-Store at Boot Time

### Command Line Options

Boot SystemRescue with:

```bash
# Using filesystem label (recommended)
cow_label=PERSISTENT

# Using device path
cow_device=/dev/sdb2

# Using directory name
cow_directory=/persistent

# Combining options
cow_label=PERSISTENT cow_spacesize=512M
```

### YAML Configuration

Add to your [sysrescuecfg.yaml](./manual_Configuring_SystemRescue_en.md):

```yaml
global:
  cow_label: PERSISTENT
  
autorun:
  # Auto-mount persistent storage on boot
  - mount | grep -q PERSISTENT || mount -t ext4 /dev/disk/by-label/PERSISTENT /mnt/persistent
```

### Recommended Boot Command

```bash
# Full example with multiple options
linux /sysresccd/boot/x86_64/vmlinuz \
    cow_label=PERSISTENT \
    cow_spacesize=512M \
    copytoram \
    setkmap=us \
    nofirewall

initrd /sysresccd/boot/x86_64/sysresccd.img
```

---

## Backing-Store Directory Structure

When backing-store is active, changes are stored in:

```
/persistent/<LABEL>/<ARCHITECTURE>/
├── usr/
│   ├── bin/          # Modified binaries
│   ├── lib/          # Modified libraries
│   └── share/        # Modified shared files
├── etc/              # Modified configuration files
├── var/              # Modified variable data
├── tmp/              # Temporary persistent files
└── home/             # User home directory changes
```

### Viewing Persistent Changes

```bash
# List all persistent changes
ls -la /persistent/RESCUE904/x86_64/

# View modified configuration files
cat /persistent/RESCUE904/x86_64/etc/fstab

# Check available space
du -sh /persistent/RESCUE904/x86_64/*
```

---

## Backing-Store Options

| Option | Description | Default | Example |
|--------|-------------|---------|---------|
| `cow_label=` | Filesystem label for persistence | None | `cow_label=boot` |
| `cow_device=` | Device path for persistence | None | `cow_device=/dev/sdb2` |
| `cow_directory=` | Directory name for changes | `persistent_<label>/<arch>` | `cow_directory=/backup` |
| `cow_spacesize=` | Copy-on-Write area size | 25% of RAM | `cow_spacesize=512M` |

### cow_spacesize Explained

The Copy-on-Write area is stored in tmpfs (RAM). Default is 25% of system memory.

```bash
# Small persistent storage (512MB)
cow_spacesize=512M

# Large persistent storage (4GB)
cow_spacesize=4G

# Use all available RAM (not recommended)
cow_spacesize=auto
```

---

## Troubleshooting Backing-Store Issues

### "No space left on device"

The persistent storage is full:
```bash
# Check available space
df -h /persistent/

# Clean up old files
rm -rf /persistent/*/*

# Or increase size
cow_spacesize=2G
```

### "Filesystem label not found"

The label doesn't exist or changed:
```bash
# Check actual label
blkid | grep PERSISTENT

# Update boot option with correct label
cow_label=ACTUAL_LABEL
```

### Changes Not Persisting

Overlayfs might not be working correctly:
```bash
# Verify overlayfs is active
mount | grep overlay

# Check persistent directory exists
ls -la /persistent/

# After reboot, verify changes persisted
cat /etc/fstab  # Should show your modifications
```

### Performance Issues

Large cow_spacesize can slow down boot:
```bash
# Reduce persistent storage size
cow_spacesize=256M

# Or use SSD for better performance
cow_device=/dev/nvme0n1p2
```

---

## Best Practices

### ✅ Recommended

1. **Use an existing partition** — Don't create new partitions just for backing-store
2. **Set appropriate size** — 512MB-2GB is usually sufficient
3. **Label your filesystem** — Makes mounting easier and more reliable
4. **Test persistence** — Make changes, reboot, verify they persist

### ❌ Not Recommended

1. **Using /boot partition** — May conflict with boot loader
2. **Very large sizes** — Wastes RAM unnecessarily
3. **No size limit** — Can fill up available memory
4. **Modifying critical system files** — Some changes may break the system

---

## Quick Reference

| Task | Command |
|------|---------|
| Check labels | `blkid` |
| Set label (ext4) | `tune2fs -L LABEL /dev/sdXN` |
| Set label (xfs) | `xfs_admin -L LABEL /dev/sdXN` |
| Set label (btrfs) | `btrfs filesystem label /dev/sdXN LABEL` |
| Boot with persistence | `cow_label=LABEL` |
| Check persistent dir | `ls -la /persistent/` |
| View changes | `du -sh /persistent/*` |

---

## Next Steps

1.  **[Configuring SystemRescue](./manual_Configuring_SystemRescue_en.md)** — YAML configuration guide
2.  **[Running custom scripts](./manual_Run_your_own_scripts_with_autorun_en.md)** — Automation with backing-store
3.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Network setup in persistent mode

---

## Additional Resources

*   [OverlayFS Documentation](https://man7.org/linux/man-pages/man5/overlayfs.5.html)
*   [SystemRescue Persistence Guide](https://www.system-rescue.org/manual/Creating_a_backing_store/)
*   [tmpfs Limits](https://www.kernel.org/doc/html/latest/filesystems/tmpfs.html)

---

*Last updated: 2026-11-04*
