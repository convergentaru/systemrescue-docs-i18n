+++
title = "Secure Deletion of Data"
draft = false
aliases = ["/Sysresccd-manual_en_Secure_Deletion_of_Data"]
+++

## Introduction

Secure deletion of data is not as simple as you might think. When you delete a file using standard operating system commands (such as "rm" in UNIX or "del" in DOS or emptying the Recycle Bin in Windows), the operating system does NOT delete the file — the file contents remain on your hard disk.

Most operating systems only remove references to files when they are asked to delete a file. The file — which you think has disappeared forever — remains on the disk until another file is created over it (until another file overwrites the space on the disk where the "deleted" file still stores), and even after this it may be possible to recover data by studying magnetic fields on the disk surface using forensics equipment.

Before a file is overwritten by a new file, anyone can easily extract data, for example using an undelete utility for disks. And even after this some people (e.g., three-letter agencies) with special equipment can recover your data at least partially.

Everyone has sensitive data that they want to keep secret. For example financial data, private emails, traces of your internet browsing habits, etc. I have heard of cases where people sold their old computers or hard drives, and the buyer recovered their financial business data.

The only way to make recovery of your sensitive data practically impossible is to overwrite ("erase" or "shred") the data with several specific patterns.

**WARNING:** Use of erasure or shredding tools relies on a very important assumption: that the filesystem overwrites data in place. This is traditional way of doing things, but many modern filesystem designs do not satisfy this assumption, e.g., Ext4, XFS, ReiserFS, etc.

See http://www.die.net/doc/linux/man/man1/shred.1.html for additional information. In this case the solution may be to erase/shred the entire device (partition) where sensitive data was stored to ensure that data is truly overwritten.

SystemRescue provides several tools that are able to make data recovery practically impossible — I say practically impossible, because no one can guarantee you that e.g., NSA or FBI cannot recover at least part of this data, but using these tools makes it more difficult.

**WARNING:** On the other hand you will not be able to recover any data deleted by these tools. Be careful. We will not take responsibility for data loss.

If you want absolute security, use encryption e.g., LOOP-AES http://loop-aes.sourceforge.net/
Encrypt your home directory or create an encrypted partition or container to store your data there.

---

## Main Tools

* **SHRED** from GNU coreutils see https://www.gnu.org/software/coreutils/coreutils.html can use shred for secure deletion of simple files, but also entire partitions or hard drives. Shred by default uses 25 overwriting passes, you can increase and decrease the number of overwriting passes. Therefore shred is faster than wipe (see below).

For example, secure deletion of all data on first IDE hard drive:

```bash
shred -v /dev/hda
```

* **WIPE** from Sourceforge see http://wipe.sourceforge.net Similar to shred, you can use wipe for secure deletion of simple files, but also entire partitions or hard drives. Wipe by default uses 35 overwriting passes. Wipe is slower than shred because it uses more overwriting passes by default and therefore is more secure.

For example, secure deletion of a Windows 98 Swap file with mounted (FAT) windows partition using 35 overwriting passes:

```bash
wipe -D /mnt/windows/win386.swp
```

---

## Other Tools

There are other tools on SystemRescue that you can use similarly for especially overwriting devices, e.g.

* **nwipe:** select disks to erase in text UI; several algorithms to choose from; fork of Darik's Boot and Nuke (DBAN)
* **dd:** if=/dev/zero or /dev/urandom, of=device
* **dd_rescue:** works similarly like "dd"
* **badblocks:** with -w option for 4-pass static overwriting

For additional information look in the man pages.

---

## Tool Comparison

| Tool | Passes (Default) | Speed | Security | Best For |
|------|------------------|-------|----------|----------|
| `shred` | 25 | Fast | High | Most use cases |
| `wipe` | 35 | Medium | Very High | Maximum security |
| `nwipe` | Configurable | Varies | Configurable | Interactive selection |
| `dd` | 1 | Very Fast | Low | Quick wipe |
| `badblocks` | 4 (static) | Slow | Medium | Verification |

---

## Usage Examples

### Secure File Deletion

```bash
# Delete a single file with 3 passes
shred -v -n 3 -z /tmp/sensitive-file.txt

# Delete with verification (check after each pass)
shred -vV -n 5 /home/user/secret.doc

# Delete and verify with zero-fill pass at end
shred -vuz -n 3 /var/log/important.log
```

### Secure Partition Wipe

```bash
# WARNING: This will destroy ALL data on the partition!
shred -v -n 5 /dev/sda2

# Wipe with zero-fill pass at end
wipe -D -n 5 -z /dev/sdb1
```

### Complete Drive Destruction

```bash
# WARNING: This will destroy ALL data on the entire drive!
# Make sure you have the correct device name!
shred -v -n 25 /dev/sda

# Or use nwipe for more control
nwipe -d /dev/sda
```

### Using dd for Quick Wipe

```bash
# Fill with zeros (slow but simple)
dd if=/dev/zero of=/dev/sdb1 bs=1M status=progress

# Fill with random data (faster, more secure)
dd if=/dev/urandom of=/dev/sdb1 bs=1M status=progress

# Three-pass: zeros, random, zeros
dd if=/dev/zero of=/dev/sdb1 bs=1M status=progress
dd if=/dev/urandom of=/dev/sdb1 bs=1M status=progress
dd if=/dev/zero of=/dev/sdb1 bs=1M status=progress
```

### Using badblocks for Verification

```bash
# Test and mark bad blocks (write test)
badblocks -v -w /dev/sda1

# Verify wipe with badblocks
badblocks -v -s -o result.txt /dev/sdb1
```

---

## Testing Deletion Tools

To see how tools work and check if all sectors have been overwritten e.g., floppy, you can use VCHE — virtual console hex editor. In our example we will securely erase all data from a floppy disk.

First enter the following command:

```bash
shred -v -n 1 /dev/fd0
```

Shred will overwrite the floppy with one random pass.

Then run:

```bash
vche-raw /dev/fd0
```

The floppy should be filled with random values.

Then we enter the following command:

```bash
shred -v -n 1 -z /dev/fd0
```

The `-z` option will make an additional pass with zero values.

And run VCHE again

```bash
vche-raw /dev/fd0
```

The floppy should be filled with zero values.

---

## Advanced Techniques

### Multiple Pass Patterns

Different pass patterns provide varying levels of security:

**Single Pass (Fastest, Least Secure):**
```bash
dd if=/dev/zero of=/dev/sdX1 bs=1M status=progress
```

**Three-Pass DoD Standard:**
```bash
# Pass 1: Random data
dd if=/dev/urandom of=/dev/sdX1 bs=1M status=progress
# Pass 2: Zeros
dd if=/dev/zero of=/dev/sdX1 bs=1M status=progress
# Pass 3: Random data
dd if=/dev/urandom of=/dev/sdX1 bs=1M status=progress
```

**Five-Pass DoD Standard:**
```bash
shred -v -n 5 /dev/sdX1
```

**Seven-Pass NIST Standard:**
```bash
# Equivalent to shred -n 7 with specific patterns
shred -v -n 7 /dev/sdX1
```

### Using nwipe for Interactive Selection

```bash
# Run nwipe interactively
nwipe

# Or specify devices in command line
nwipe -d /dev/sda1 /dev/sda2 /dev/sdb1

# Use specific algorithm
nwipe -a doD3565 /dev/sdX1  # DoD 35-pass standard
```

### Secure Swap File Deletion

```bash
# Find swap files
find /mnt/windows -name "*.swp" -o -name "swap*"

# Securely delete each swap file
for swapfile in $(find /mnt/windows -name "*.swp"); do
    shred -vuz -n 5 "$swapfile"
done

# Unmount and verify
umount /mnt/windows
ls /mnt/windows/*.swp  # Should be empty
```

---

## Best Practices

### ✅ Recommended

1. **Verify device names** — Double-check `/dev/sdX` before wiping
2. **Test on small media first** — Try floppy or USB before hard drive
3. **Use appropriate passes** — More passes = more secure but slower
4. **Document what you wipe** — Keep records of deleted data
5. **Verify deletion** — Use tools like `badblocks` to confirm

### ❌ Not Recommended

1. **Wiping live partitions** — Never wipe mounted filesystems
2. **Using wrong device** — Always verify `/dev/sdX` is correct
3. **Skipping verification** — Always test before production use
4. **Assuming 100% security** — No tool can guarantee absolute security

---

## Security Considerations

### ⚠️ Data Loss Warning

These tools permanently destroy data:

1. **Backup first** — Copy important data before wiping
2. **Verify device** — Double-check device names multiple times
3. **Use unmounted devices** — Never wipe mounted partitions
4. **Test on non-critical media** — Practice on USB or CD-R first

### Encryption Alternative

For maximum security, use encryption:

```bash
# Create encrypted partition with cryptsetup
cryptsetup luksFormat /dev/sdb1
cryptsetup open /dev/sdb1 myencrypted
mkfs.ext4 /dev/mapper/myencrypted
mount /dev/mapper/myencrypted /mnt/secure

# All data is now encrypted at rest
```

---

## Troubleshooting

### "No space left on device"

**Problem:** Overwriting fills up tmpfs.

```bash
# Increase available space
dd if=/dev/zero of=/tmp/wipe-tempfile bs=1M count=1024
shred -v /tmp/wipe-tempfile

# Or use direct device writing
shred -v -n 3 /dev/sdX1
```

### "Device is busy"

**Problem:** Device is mounted or in use.

```bash
# Unmount the device
umount /dev/sdX1

# Check what's using it
lsof +D /dev/sdX1
fuser -v /dev/sdX1

# Kill processes if necessary
kill <PID>
```

### "I/O error"

**Problem:** Disk has bad sectors or is failing.

```bash
# Check disk health
smartctl -a /dev/sdX

# Test with badblocks
badblocks -v /dev/sdX1

# Consider replacing failing drive
```

### Verification Failed

**Problem:** Some sectors not overwritten.

```bash
# Check for bad blocks
badblocks -v -s /dev/sdX1 > badblocks.txt

# Review results
cat badblocks.txt | grep -c "Failed"

# Retry with different tool
nwipe -a doD3565 /dev/sdX1
```

---

## Quick Reference Table

| Command | Purpose | Example |
|---------|---------|---------|
| `shred` | Secure file/device deletion | `shred -v -n 5 /dev/sda2` |
| `wipe` | Alternative secure deletion | `wipe -D -n 3 /dev/sdb1` |
| `nwipe` | Interactive wipe tool | `nwipe -d /dev/sda1` |
| `dd` | Raw device writing | `dd if=/dev/zero of=/dev/sda1` |
| `badblocks` | Verification/testing | `badblocks -v /dev/sdb1` |

---

## Common Scenarios

### Scenario 1: Selling Old Computer

```bash
# 1. Backup all important data
rsync -av /home/user/ /backup/

# 2. Document current partitions
lsblk -f > partition-info.txt

# 3. Securely wipe entire drive
shred -v -n 7 /dev/sda

# 4. Verify wipe completed
badblocks -v /dev/sda

# 5. Reinstall OS if needed
```

### Scenario 2: Removing Sensitive Files

```bash
# Find sensitive files
find /home/user -name "*.key" -o -name "password*"

# Securely delete each file
for file in $(find /home/user -name "*.key"); do
    shred -vuz "$file"
done

# Clear shell history
shred -uf ~/.bash_history
shred -uf ~/.zsh_history
```

### Scenario 3: Preparing Drive for Donor PC

```bash
# 1. Identify all data partitions
lsblk -f

# 2. Wipe only user data, keep boot partition
shred -vuz /dev/sda2  # User data
shred -vuz /dev/sda3  # Home partition

# 3. Keep system partition intact
# /dev/sda1 (boot) - leave untouched

# 4. Verify
badblocks -s /dev/sda2
```

---

## Performance Considerations

### Speed vs Security Trade-off

More passes = more secure but slower:

| Passes | Time (1TB HDD) | Security Level |
|--------|----------------|----------------|
| 1 | ~5 minutes | Low |
| 3 | ~15 minutes | Medium |
| 5 | ~25 minutes | High |
| 7 | ~35 minutes | Very High |
| 25 | ~2.5 hours | Maximum |

### SSD Considerations

For SSDs, traditional shredding is not recommended:

```bash
# SSD secure erase (faster than shredding)
echo 1 > /sys/class/device/sda/force_secure_erase

# Or use vendor-specific tool
hdparm --user-master u --security-log --security-disable /dev/sda
```

---

## Next Steps

1.  **[Backup tools](./manual_Backup_and_transfer_your_data_using_rsync_en.md)** — Data backup before deletion
2.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Remote secure deletion
3.  **[Disk encryption](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/scripts/SRM)** — Protect data with encryption

---

## Additional Resources

*   [GNU shred Manual](https://www.gnu.org/software/coreutils/shred/)
*   [Wipe Tool](http://wipe.sourceforge.net/)
*   [nwipe (DBAN fork)](https://sourceforge.net/projects/nwipe/)
*   [NIST 800-88 Guide](https://csrc.nist.gov/publications/detail/sp/800-88/rev-1/final)
*   [DoD 5220.22-M Standard](https://www.dtic.mil/docs/citations/ADA476193/)

---

*Last updated: 2026-11-04*
