+++
title = "Backup and Transfer Your Data using rsync"
draft = false
aliases = ["/Sysresccd-manual_ru_Backup_and_transfer_your_data_using_rsync"]
+++

## Overview

[Rsync](https://rsync.samba.org/documentation.html) is a file synchronization program with open source code. Designed to maintain directory mirrors, it's a **very advanced tool** that can be used for creating backups or copying files to another disk or host in case of hardware or software disaster.

Advantages of rsync compared to alternative transfer methods include:

* **efficiency**: if version of file already exists at destination, only changed **parts** are transferred. This significantly reduces resource requirements and execution time, especially with large files. Uses intelligent duplicate detection algorithm that minimizes network load.
* **reliability**: in environments with slow connections or random hardware failures (especially disks that become non-working after overheating), rsync can resume interrupted transfers
* **flexibility**: options include exclusion lists, "dry run" for checking options, comparison settings for detecting changes, various link handling methods and much more
* works on Linux, Unix and under MS Windows with cygwin.

Rsync has several disadvantages:

* Most noticeable: since it's so flexible, there are many options, some of which complement each other, some contradict. Full [rsync](https://rsync.samba.org/documentation.html) documentation is extensive and should be read several times.
* Target filesystem must support attributes. During archiving attributes are preserved in archive, and archive can be on any filesystem.

Rsync comes with SystemRescue, and this documentation is intended to provide useful information and examples for use during recovery process. After understanding this you will find rsync useful for increasing system reliability during normal operation. See "Using rsync regularly to minimize disaster impact".

---

## Basic Usage

Rsync can run as a server (when started with --daemon) or as a client for creating copies of files on local machine or over network to another host. Recommended options include:

* `--progress` displays activity for monitoring progress
* `--archive` preserves basic attributes (permissions, ownership, type such as symbolic links.)
* `--xattrs` preserves extended attributes (same as `-X`)
* `--acls` preserves access control lists (same as `-A`)
* `--hard-links` preserves hard links (doesn't handle as files) (same as `-H`)
* `--compress` compresses data on the fly (no effect at destination) (same as `-z`)

### Copying Files Locally on Same Computer

Only one rsync process is used for local backups. Here's an example that preserves all attributes (including extended) and ACL:

```bash
rsync --archive --xattrs --acls --hard-links --progress --compress \
      /home/mydir/data/ /backups/data-20080810/
```

### Remote Backup Offline

If rsync is installed as a daemon (listening on tcp/873 port) at example.com, a client running on myhost can either pull files from the daemon or push files to the daemon.

Here's an example where client can send directory to remote host (preserving attributes, ACL and hard links (using single-letter options):

```bash
rsync -aXAHz --progress /home/mydata/ example.com::mybackups/data-20080810/
```

Client can also download files from remote host:

```bash
rsync -aXAHz --progress example.com::mybackups/data-20080810/ /home/mydata/
```

Note the two colons between remote host and path when using rsync protocol.

### Remote Backup via SSH

If tcp/873 port is blocked (by firewall) or you need to encrypt connection, you can use rsync via ssh by specifying `-e` on client side.

Here's an example where client sends to remote host:

```bash
rsync -aAXHz --progress -e ssh /home/mydata/ remote.com:/backups/data-20080810/
```

or receives:

```bash
rsync -aAXHz --progress -e ssh remote.com:/backups/data-20080810/ /home/mydata/
```

Note only one colon between remote host address and remote path when rsync connects via ssh.

---

## Resuming Interrupted Transfers

When restoring large amount of data, either large files or thousands of small ones, using scp, ftp or http is impractical for resuming interrupted transfer. Manual restart of transfer is difficult. If process was interrupted at 99% of large file, transfer should be restarted from beginning, all data transferred again, and this may mean that transfer will never finish. Even with `wget -c` can get corrupted file.

This is also important in cases when hard disk works only while it cools down. Rsync can handle this well, saving files while disk works. After disk overheats and becomes unreliable, rsync will abort. Shut down system and wait an hour or two for disk to cool. Turn on power and rsync can continue saving files **where it left off**.

### Transferring Thousands of Small Files

Only files that are not up-to-date at destination will be transferred. First comparing source and destination files only those that differ will be transferred. This eases resuming after interruption and reduces execution time and reduces bandwidth requirements for remote transfers.

### Transferring Files with Slow Connections

Rsync is capable of **resuming transfers** of files if connection was lost, using `--partial` and `--inplace`. This is especially important for large files.

* `--partial` forces partially transferred files to persist. By default rsync removes partial file on abort. If connection is lost, rsync will stop. When connection is restored and rsync resumes, data must be transferred again since rsync will start from beginning of file.
* `--inplace` forces rsync to use original filename at destination. By default rsync transfers data to files with temporary name during transfer and renames files when file is finished. Without `--inplace`, on restart rsync will create new temporary files and must restart transfer from beginning of file.
* `--update` should NOT be used with `--inplace`, since partial destination file will be considered same as source file, and skipped.

Rsync uses very efficient algorithm for comparing source file with destination file and transfers only different parts of file. For transferring multiple versions of same files regularly, copy old version (which you already transferred) to new destination file, and rsync will skip all common parts.

With these options rsync can be interrupted, then resumed. Data that was transferred is preserved.

Here's an example good command for copying directory with large files to remote host:

```bash
rsync --archive --partial --inplace --progress --compress \
       /home/bigfiles/ MYMIRROR::mybackups/bigfiles/
```

Check rsync updates at http://rsync.samba.org/

---

## Important Rsync Options

Rsync has many options. Here are some that apply for recovering a broken system.

### Preserving Attributes with `--archive`

This is very important option. Preserves file attributes (permissions, timestamps and type). As a result, symbolic link will be copied as link, without `--archive` **contents** of source file is copied at destination. Hard links, extended attributes (xattr), and ACLs (Access Control Lists) are not preserved with this option, see `-HAX`

### Showing Progress During Transfer with `--progress`

This option will show progress on current transfer

### Listing Files with `--verbose`

This option shows more details and can be used multiple times for increased logging

### Not Crossing Filesystem Boundaries with `--one-file-system`

Process only one filesystem. Important when processing a filesystem (e.g.: `/`) with mounted volumes. By default processes filesystems with mount points in source specification.

### Compressing File Data During Transfer with `--compress`

Rsync can compress data that is being transferred to reduce network activity. Destination file will be same as original. Use this for remote transfers when synchronizing files that are not compressed and will have high compression ratio (e.g.: large text files, CD images or raw disk partitions).

This option is inefficient for files that are already compressed: zip, gz, bz2, tar.gz, jpeg, pdf, ... This will cause significant increase in CPU usage on both source and destination systems.

### Watching Progress on Large Files with `--inplace` and `--partial`

Use these options for transferring large files and ensure that transfer will resume at restart point in case of connection failure. See sections about transferring large files for more details.

### Excluding Files with `--exclude=pattern`

Use this option to exclude files or directories from transfer. For example temporary files or cache directories.

### Removing Destination Files with `--delete`

When synchronizing existing destination, files in **destination** directory will be deleted if they are not in source directory (files were deleted). By **default** files no longer in source **remain at destination**. Consider scenario when files are versioned by date and deleted when new is created. By default won't delete old versions, and they will accumulate. This will lead to destination not being a "mirror" of source, and space required for destination will increase with each run. Processing one run with this option will delete all accumulated files.

### Skipping Files Newer at Destination with `--update`

Tells this run to update existing destination. Use this option if files were modified in **destination** directory. For example, if you're migrating data from old server to new, and people have already started working on new server. Use `--update` to prevent overwriting changes with older version.

### Removing Synchronized Files from Source with `--remove-source-files`

This option is useful for moving data to destination. By default, rsync makes a copy, leaving original untouched. With `--remove-source-files`, source files should be deleted if transfer was successful.

### Comparing Files Based on Checksums with `--checksum`

By default rsync uses file modification time and size to decide whether to transfer a file or not. A file that exists in destination directory with same name, date and size is considered same. This is accurate in almost all cases. Comparison is very fast since only file attributes are read. In rare cases this is incorrect. There are utilities that can modify file and preserve original modification time. If modification doesn't change file size (very rarely), rsync will incorrectly consider source and destination files identical and won't transfer file. Specify `--checksum` to force calculating checksum of files and compare this checksum to determine if files are same or not. Be **warned** that this will add significant CPU time and I/O activity on both source and destination systems since all contents of both files must be read for generating checksum.

---

## Rsync Return Status

Return status should be checked to determine if transfer was successful. When rsync returns 0, it means that transfer was successful. Any other value indicates an error.

Some errors that may indicate resumable problem include:

* 22: Core buffer allocation error
* 23: Partial transfer due to error

---

## Installing and Configuring Rsync Daemon

Rsync client will only transfer data to rsync server.

Default configuration location is either `/etc/rsyncd.conf` or `/etc/rsync/rsyncd.conf`. After changing configuration send HUP signal using `kill -HUP <pid-of-rsync>`.

Several options are available for limiting connections:

* require password for connecting
* allow only several specific IP addresses for connecting
* provide read-only access to client

Here's an example basic secure configuration:

```bash
# ======================/etc/rsyncd.conf======================
pid file = /var/run/rsyncd.pid
read only = yes
uid = root
gid = root

[share1]
    path = /mnt/share1
    read only = yes
    hosts allow = myhost, 10.88.45.0/24

[backups]
    path = /var/tmp/catalyst/tmp
    read only = no
    hosts allow = mybiggie, 10.88.45.0/24

[rootfs]
    path = /
    read only = yes
    hosts allow = myhost, 10.88.45.0/24

[upload]
    path = /upload
    read only = no
    hosts allow = 172.16.0.0/16
```

---

## MS Windows and cygwin

Rsync daemon can run on MS Windows system using [cygwin](http://www.cygwin.com) which provides Linux environment.

Minimal information provided here to help you get started.

These are not emulated programs: this is simply software that we use on linux, was compiled to work on windows, so it's a windows executable file that talks directly with windows kernel.

For installing cygwin, run setup.exe program found at [cygwin website](http://www.cygwin.com)

Hard disks are visible as `/cygdrive/c/`, `/cygdrive/d/`, ...

Click on cygwin icon to launch bash shell.

You can also install rsync as a daemon on cygwin. For installing services on cygwin, you can use special program called `cygrunsrv.exe`. It installs cygwin service as normal windows service so it can automatically start when windows starts. So you don't need to start daemon manually.

Here's command to use:

```bash
cygrunsrv.exe -I "cygrsyncd" -p /usr/bin/rsync.exe -a "--config=/etc/rsyncd.conf --daemon --no-detach"
```

Then, start new service, run `services.msc` and start the service named "cygrsyncd"

---

## Using Rsync Regularly to Minimize Disaster Impact

Previous sections of this documentation focused on rsync features and how it works to help recover a system.

Here we present additional information that was thought would be useful.

Rsync is a utility that supports filesystem mirror. By updating regularly, mirror can become major asset for recovering a broken system.

Here are some additional notes on using rsync:

* rsync can make remote backups of your data files on windows, or it can replicate windows system backups created with other programs
* In combination with [LVM snapshots](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/disk-partitioning_Introduction-to-disk-partitioning.md) for creating online backups that are consistent.
* **Different backup strategies**: full, differential backups, incremental. Backups may only preserve files that have been changed since previous backup.
* run rsync on server every night to create backups of its root filesystem. Rsync will probably complain about files that were deleted during transfer (e.g., temporary files). Consider return status 23 and 24 as warnings on live system

Rsync daemon can be installed in dedicated listening mode (tcp port 873). Having [xinetd](http://en.wikipedia.org/wiki/Xinetd) configured to listen instead of rsync uses less memory and is more secure. This also means that rsync starts and configuration file is read at each connection, so changes will take effect on next connection.

Rsync catches intr, so pressing ^C stops when current file finishes. Use less common exit ^\, to abort immediately.

---

## Common Backup Scenarios

### Scenario 1: Full System Backup

```bash
# Backup entire home directory with all attributes
rsync -aAXH --progress \
    /home/user/ \
    /backup/home-user-$(date +%Y%m%d)/

# Exclude unnecessary files
rsync -aAXH --progress --exclude='*.tmp' --exclude='*.log' --exclude='.cache/' \
    /home/user/ \
    /backup/home-user-$(date +%Y%m%d)/
```

### Scenario 2: Incremental Backups

```bash
# Daily incremental backup
rsync -aAXH --progress --delete \
    /data/ \
    /backup/data-daily-$(date +%Y%m%d)/

# Weekly full backup (Sunday only)
if [ "$(date +%u)" == "7" ]; then
    rsync -aAXH --progress --remove-source-files \
        /data/ \
        /backup/data-weekly-$(date +%Y%m%d)/
fi
```

### Scenario 3: Network Backup with Resume Capability

```bash
# Large data backup with resume capability
rsync -aAXHz --partial --inplace --progress \
    /mnt/bigdata/ \
    root@backup-server:/remote/backups/bigdata/

# Monitor progress
watch -n 1 "rsync -aAXHz --progress /mnt/bigdata/ root@backup-server:/remote/backups/bigdata/"
```

### Scenario 4: Backup with Verification

```bash
# Create backup and verify checksums
rsync -aAXH --progress \
    /important-data/ \
    /backup/data-$(date +%Y%m%d)/

# Verify integrity after transfer
cd /backup/data-$(date +%Y%m%d)
find . -exec md5sum {} \; > checksums.md5
```

---

## Quick Reference Table

| Option | Description | Example |
|--------|-------------|---------|
| `-a` or `--archive` | Preserve all attributes | `rsync -av /source/ /dest/` |
| `-z` or `--compress` | Compress during transfer | `rsync -az /source/ /dest/` |
| `--progress` | Show transfer progress | Always recommended |
| `--partial` | Keep partial files | Resume interrupted transfers |
| `--inplace` | Write to destination directly | Faster for large files |
| `--delete` | Delete extra destination files | Keep mirror in sync |
| `--exclude=PATTERN` | Skip matching files | Exclude temporary files |
| `-e SSH` | Use SSH for encryption | `rsync -e ssh /source/ user@host:/dest/` |
| `--checksum` | Compare by content, not size | Most accurate but slowest |

---

## Best Practices

### ✅ Recommended

1. **Use --archive option** — Preserves permissions, ownership, and special files
2. **Enable progress display** — Monitor transfer status in real-time
3. **Test restore process** — Verify backups can be restored successfully
4. **Schedule regular backups** — Automate backup process
5. **Verify backup integrity** — Check checksums after transfer

### ❌ Not Recommended

1. **Backing up / or /home without --one-file-system** — Can cause issues with mounted filesystems
2. **Using --delete on critical data** — Accidental deletion is dangerous
3. **No verification of backups** — Always test restore capability
4. **Single backup location** — Use multiple backup destinations

---

## Security Considerations

### ⚠️ Authentication

When using rsync with remote servers:

1. **Use SSH keys instead of passwords** — More secure authentication
2. **Restrict access by IP** — Limit who can connect
3. **Encrypt connections** — Always use SSH for remote transfers
4. **Set proper permissions** — Restrict backup directory access

### rsyncd.conf Security

```bash
# Secure configuration example
[secure-backup]
    path = /backup/secure
    read only = no
    hosts allow = 192.168.1.0/24
    secrets file = /etc/rsyncd.secrets
    auth users = backupuser
    strict modes = yes
```

---

## Troubleshooting

### Transfer Aborted with Error 23

**Problem**: Partial transfer due to error (network interruption).

```bash
# Resume interrupted transfer
rsync -aAXHz --partial --inplace \
    /source/ /destination/

# Check for disk errors
dmesg | grep -i error
smartctl -a /dev/sdX
```

### Permission Denied Errors

**Problem**: Cannot write to destination.

```bash
# Check permissions
ls -la /destination/

# Fix ownership
chown -R backupuser:backupgroup /destination/

# Or use rsync with specific user
rsync -av --owner=backupuser --group=backupgroup \
    /source/ backupuser@host:/destination/
```

### "remote: invalid argument" Error

**Problem**: rsync protocol vs SSH syntax confusion.

```bash
# Correct SSH syntax (single colon)
rsync -avz -e ssh /local/path/ user@host:/remote/path/

# Incorrect (double colon is for rsync daemon only)
# rsync -avz /local/path/ user@host::/remote/path/  # WRONG!
```

### Slow Transfer Speed

**Problem**: Inefficient transfer or network issues.

```bash
# Check network speed
ping -c 4 backup-server.com
netstat -i

# Use compression for uncompressible files
rsync -avz --progress /source/ host:/dest/

# For large files, use inplace mode
rsync -aAXHz --partial --inplace --progress \
    /large-data/ host:/backup/large-data/
```

---

## Advanced Usage

### Creating Automated Backup Scripts

```bash
#!/bin/bash
# /root/backup-script.sh

BACKUP_DIR="/backup"
SOURCE_DIR="/data"
DATE=$(date +%Y%m%d_%H%M%S)
HOST="backup-server.example.com"
REMOTE_DIR="/remote/backups"

echo "Starting backup at $DATE..."

rsync -aAXHz --partial --inplace --progress \
    --exclude='*.tmp' \
    --exclude='*.log' \
    --exclude='.cache/' \
    --delete \
    "$SOURCE_DIR/" \
    root@"$HOST":"$REMOTE_DIR"/backup-$DATE/

if [ $? -eq 0 ]; then
    echo "Backup completed successfully"
else
    echo "Backup failed! Check logs."
fi
```

### Monitoring Backup Jobs

```bash
# Watch backup progress in real-time
watch -n 1 'rsync -avz --progress /source/ user@host:/dest/'

# Log all transfers
rsync -avz --progress /source/ user@host:/dest/ >> /var/log/rsync-backup.log 2>&1
```

### Using rsync with LVM Snapshots

```bash
# Create consistent snapshot before backup
lvcreate -L 10G -s -n snap-backup vg0/root

# Backup from snapshot (consistent state)
rsync -aAXH --progress \
    /dev/mapper/vg0-snap-backup/ \
    /backup/snapshot-$(date +%Y%m%d)/

# Remove snapshot after backup
lvremove vg0/snap-backup
```

---

## Next Steps

1.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Setting up remote backups
2.  **[LVM snapshots](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/disk-partitioning_Introduction-to-disk-partitioning.md)** — Consistent online backups
3.  **[Secure deletion](./manual_Secure_Deletion_of_Data_en.md)** — Cleanup after restoration

---

## Additional Resources

*   [rsync Documentation](https://rsync.samba.org/documentation.html)
*   [rsync Cheat Sheet](https://rsync.samba.org/quick-guide.html)
*   [SSH Security Best Practices](https://www.openssh.com/portable_6/prologue.html)
*   [Linux Backup Strategies](https://wiki.archlinux.org/title/Backup)

---

*Last updated: 2026-11-04*
