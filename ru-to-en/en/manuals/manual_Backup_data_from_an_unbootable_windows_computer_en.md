+++
title = "Creating a Backup from an Unbootable Windows Computer"
draft = false
aliases = ["/Sysresccd-manual_ru_Backup_data_from_an_unbootable_windows_computer"]
+++

## What This Section Covers

This tutorial explains how you can create a backup of data from a Windows computer that is no longer able to boot. There are many reasons why a Windows computer may stop working: installation of a bad driver, system crash with disk damage, system files deleted, ... In this case you can simply reinstall the operating system, but most importantly — first create a backup of your data. If Windows is not bootable, you can simply boot from SystemRescue to do this. Even running Linux, it's capable of reading a Windows disk, and then you will be able to create a backup of your data over the network. All steps are described in detail in this tutorial so that Windows users without Linux knowledge can follow it to create a backup of their data.

If this is the first time you're using SystemRescue, it's recommended to read [Quick start Guide](/Quick_start_guide/) to avoid problems.

---

## Overview

SystemRescue is a livecd. It contains a complete Linux-based operating system. This means that you will be able to manipulate files from its environment, even if the Windows OS installed on this computer is broken. Of course, you will need a working computer for downloading and burning SystemRescue.

* **Step 1: Preparing bootable cdrom/usb stick**
  First you need to download and burn SystemRescue. This requires a working computer. If you cannot boot from SystemRescue with cdrom, you can also [Install SystemRescue on USB stick](./manual_Installing_SystemRescue_on_the_disk_en.md) and boot from it.

* **Step 2: Accessing your Windows disk**
  In this section we will see how to read a Windows disk(s) from Linux to see what files can be backed up.

* **Step 3: Creating a backup of your data over network**
  If you have network access (network card and another computer where you can copy the data), this section explains how to configure the network and copy data to another computer.

* **Step 4: Proper reboot**

---

## Step 1: Preparing Bootable CDROM or USB Stick

All you need to do is download the latest SystemRescue ISO for x86, burn it to a cdrom disk using any burning program. If the computer where Windows is broken cannot boot from cdrom disk, you can also [Install SystemRescue on USB stick](./manual_Installing_SystemRescue_on_the_disk_en.md) and boot from it.

Then enter your computer BIOS (you often need to press F2 or DEL during initial hardware initialization), go to the section where boot order is defined, and make sure that cdrom or USB device is first device in the list (or before hard disk).

Now reboot with inserted cdrom / usb stick. If this works, you should see ASCII-art SystemRescue logo and boot instructions. Then you can simply press enter to boot into SystemRescue with default settings. If you plan to use network, it's recommended to boot with your network settings so you don't need to configure it later. For this simply enter the following line in command line and press enter. Of course, you will need to replace this address with appropriate IP address for this computer:

```bash
rescue64 ethx=192.168.1.158 gateway=192.168.1.254
```

Gateway address is required only if you want to connect to a computer that is on another network. To use dynamic IP instead of static IP, you can enter the following:

```bash
rescue64 dodhcp
```

Don't worry about network settings now, you can change them later if this doesn't work. You can also read chapter on [boot options](./manual_Booting_SystemRescue_en.md) to get more details about this.

---

## Step 2: Accessing Your Windows Disk

When the boot process is finished, you should get a black screen where you can type commands. All following steps will be based on commands, so you can stay in this environment. You can also run commands from terminal in graphical environment if you prefer. Simply enter `startx` in this case.

### Detecting Disks

The first thing to do is to detect the partition where your data is located. Linux uses names different from Windows for disks, so you will see names such as `/dev/sda1`, `/dev/sda2` instead of `C:` or `D:`.

To detect disks and partitions simply run the following command:
```bash
fsarchiver probe simple
```

You should see output like:

```
root@sysresccd /root % fsarchiver probe -v
[=====DEVICE=====] [==FILESYS==] [=====LABEL=====] [====SIZE====] [MAJ] [MIN]
[/dev/sda1       ] [ntfs       ] [Windows-XP     ] [    25.00 GB] [  8] [  1]
[/dev/sda5       ] [ntfs       ] [Data           ] [   120.00 GB] [  8] [  5]
[/dev/sda6       ] [ntfs       ] [Backup         ] [    70.00 GB] [  8] [  6]
```

In this example there were three partitions on the computer, you may have only one or two partitions. The first column is very important as it's the device name of the partition that we will need to access your data. You should write this name down on a piece of paper.

### Accessing the Right Partition

Now we need to make these data accessible from the system. Let's assume that you want to create a backup of data located on disk named `/dev/sda1`. The first thing to do is to mount the partition to a directory. This means only that this directory will contain a representation of your disk, nothing will be written to it:

```bash
ntfs-3g -o ro /dev/sda1 /mnt/windows
```

The only option used here is `ro`, it means that data on disk is mounted with read-only access. We use only this option to ensure that we cannot make mistakes that could destroy this data, but you can mount the disk with full read/write access if you prefer (just remove this option from the command). Now let's look at the partition contents:

```bash
cd /mnt/windows
ls -l
```

You should see something like:

```
root@sysresccd /mnt/windows % ls -l
total 132856
-r-------- 1 root root      245 2008-08-05 19:36 boot.ini
dr-x------ 1 root root     4096 2008-07-02 07:34 cygwin
dr-x------ 1 root root     4096 20007-10-23 21:18 Documents and Settings
-r-------- 1 root root    47772 2005-03-25 12:00 NTDETECT.COM
-r-------- 1 root root   297072 2007-10-21 18:50 ntldr
-r-------- 1 root root 16777216 2008-09-22 16:41 pagefile.sys
dr-x------ 1 root root     4096 2008-08-10 19:07 Program Files
dr-x------ 1 root root        0 2008-07-02 07:45 RECYCLER
dr-x------ 1 root root     4096 2007-10-21 17:59 System Volume Information
dr-x------ 1 root root    32768 2008-08-29 22:48 WINDOWS
```

If you want to browse your data, it's recommended to use Midnight Commander. Simply enter `mc` in the console to get it. It's a very intuitive tool similar to Norton Commander that can be used to browse your disk. It can also be used for copying, moving, renaming files and directories using function keys. Press F10 to exit Midnight Commander.

---

## Step 3: Creating a Backup of Your Data Over Network

There are several ways to create a backup of your data. Using network is recommended as it's fast and limited only by the amount of space available on another computer. Basic network configuration is not difficult under Linux if you have a network card.

### IP Network Configuration

First let's check what cards are detected. No driver installation needed. Linux comes with all network drivers by default. Network cards are named `eth0`, `eth1`, `eth2`, ... under Linux. If you have only one card, it will be `eth0`. If you have more than one card, remember that the first may not be the one you expect. In this case you can simply configure all cards with IP address and this should work. Enter the following command:

```bash
ifconfig -a
```

You should see a screen like:

```
root@sysresccd /mnt/windows % ifconfig -a
eth0      Link encap:Ethernet  HWaddr 00:18:f3:ce:0e:36
          inet addr:192.168.1.158  Bcast:192.168.1.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
          Interrupt:16 Base address:0xe000
```

If IP address is not configured yet, you can set a static IP address (and gateway if required) as follows:

```bash
ifconfig eth0 192.168.1.158 netmask 255.255.255.0
route add default gw 192.168.1.254
```

For dynamic address via DHCP simply give the command:

```bash
dhclient eth0
```

### Configuring SFTP Server

In this section we will assume that the broken computer uses 192.168.1.158, and the computer where you want to copy the data is 192.168.1.20.

Several protocols can be used for transferring your data. In this tutorial we will use SCP/SSH protocol for several reasons:

* it uses only one TCP connection on port 22, which is easier to configure than FTP which requires two connections to work
* it's reliable, and it's encrypted, so there's no risk for your data if you don't trust people connected to your local network.
* it's very easy to configure on both client and server side (samba can be used for sharing your files, but configuration can be tricky)

The broken computer runs OpenSSH server, which means that another computer can connect to it using tcp/22 port as long as it knows the password. The computer where data will be copied should run an SFTP client. We recommend Filezilla-client, which is a very good free cross-platform FTP/SFTP client. You can use it exactly the same as any other FTP client except the protocol will be SFTP. You can also use WinSCP if you prefer.

Now we need to set a password on the server side (on linux) so we can connect from the other end. Simply enter the following command, and enter the password twice (it won't appear on screen for security reasons):

```bash
passwd
```

### Configuring SFTP Client

Now the server is ready, you can go to another computer (which is working), install Filezilla-client on it (Filezilla works on Windows and Linux), and do the following:

* Click on File/Site-manager in menu bar
* Click on new-site to create a new connection
* Fill the dialog box with the following information:
  * Host: 192.168.1.158 (IP address of computer running SystemRescue)
  * Port: 22
  * Server type: SFTP (SSH File Transfer Protocol)
  * Login type: Normal
  * Login: root
  * Password: enter the password you entered in previous step.
  * Click on "Advanced"
  * Default Directory: /mnt/windows
* Now click on connect

Now you should be connected to OpenSSH server running in SystemRescue, and you should see local disk on one side of window, and remote disk on other side. If you don't see your remote files, you need to go to directory `/mnt`, then into `windows`.

---

## Step 4: Reboot Your Computer

When you're done using SystemRescue, it's recommended to unmount your Windows partition, and then reboot the computer properly:

```bash
umount /mnt/windows
reboot
```

---

## Alternative Backup Methods

### Using rsync for Large Backups

```bash
# Create backup directory on remote server
mkdir -p /backup/windows

# Use rsync for incremental backups
rsync -avz --progress /mnt/windows/Documents/ root@192.168.1.20:/backup/windows/

# Backup entire partition (except system files)
rsync -avz --progress \
  /mnt/windows/Documents/ \
  /mnt/windows/Pictures/ \
  /mnt/windows/Music/ \
  /mnt/windows/Downloads/ \
  root@192.168.1.20:/backup/windows/userdata/

# Exclude system files
rsync -avz --progress --exclude='*.sys' --exclude='*.dll' \
  /mnt/windows/ root@192.168.1.20:/backup/windows/full/
```

### Using tar for Compressed Archives

```bash
# Create compressed archive and transfer in one command
tar -czf - /mnt/windows/Documents/ | ssh root@192.168.1.20 "cat > /backup/docs.tar.gz"

# Or use remote archive creation
ssh root@192.168.1.20 "tar -xzf - -C /backup/" < docs.tar.gz

# Create multiple archives by date
DATE=$(date +%Y%m%d)
tar -czf - /mnt/windows/Documents/ | ssh root@192.168.1.20 "cat > /backup/docs-${DATE}.tar.gz"
```

### Using USB Drive as Backup Destination

If network is not available, use USB drive:

```bash
# Identify USB device
lsblk

# Mount USB drive (replace with actual device)
mount /dev/sdb1 /mnt/usb

# Copy data to USB
cp -rv /mnt/windows/Documents/ /mnt/usb/
cp -rv /mnt/windows/Pictures/ /mnt/usb/

# Or use rsync for incremental copies
rsync -av --progress /mnt/windows/Documents/ /mnt/usb/documents/
```

---

## Quick Reference Table

| Task | Command | Notes |
|------|---------|-------|
| Mount Windows disk | `ntfs-3g -o ro /dev/sdXN /mnt/windows` | Read-only for safety |
| List partitions | `fsarchiver probe simple` | Find partition names |
| Configure IP static | `ifconfig eth0 192.168.1.158 netmask 255.255.255.0` | Manual IP setup |
| Configure IP DHCP | `dhclient eth0` | Automatic IP assignment |
| Set password | `passwd` | Required for SSH access |
| Transfer files SCP | `scp /mnt/windows/* root@host:/backup/` | Secure file transfer |
| Transfer files rsync | `rsync -avz /source/ root@host:/dest/` | Incremental backup |
| Unmount partition | `umount /mnt/windows` | Before reboot |

---

## Common Issues and Solutions

### "ntfs-3g: mount failed"

**Problem:** Cannot mount NTFS partition.

**Solutions:**
1. Check if ntfs-3g is installed:
   ```bash
   which ntfs-3g
   ```
2. Try with read-only mode:
   ```bash
   mount -t ntfs3 -o ro /dev/sdXN /mnt/windows
   ```
3. Check disk for errors:
   ```bash
   ntfsfix /dev/sdXN
   ```

### "Network not accessible"

**Problem:** Cannot connect to remote computer.

**Solutions:**
1. Verify IP addresses are correct:
   ```bash
   ping 192.168.1.20
   ```
2. Check firewall settings:
   ```bash
   systemctl status ufw
   ufw allow ssh
   ```
3. Enable SSH service:
   ```bash
   systemctl start sshd
   ```

### "Permission denied" on remote server

**Problem:** Cannot write to backup location.

```bash
# Check permissions on remote server
ssh root@192.168.1.20 "ls -la /backup/"

# Create backup directory with proper permissions
ssh root@192.168.1.20 "mkdir -p /backup && chmod 755 /backup"
```

---

## Best Practices

### ✅ Recommended

1. **Use read-only mount** — Prevents accidental data modification
2. **Verify backup integrity** — Check file sizes and counts after transfer
3. **Exclude system files** — Don't backup unnecessary Windows system files
4. **Test restore process** — Verify backup can be restored successfully
5. **Document partition layout** — Record device names for future reference

### ❌ Not Recommended

1. **Writing to mounted partitions** — Risk of data corruption
2. **Backing up entire Windows partition** — Too much unnecessary data
3. **No verification of backups** — Always check backup completeness
4. **Single backup location** — Use multiple backup destinations

---

## Security Considerations

### ⚠️ Data Privacy

When backing up sensitive data:

1. **Encrypt backups** — Use encrypted transfer methods
2. **Remove passwords from Windows** — Don't leave login credentials
3. **Secure the backup destination** — Protect backup files with access controls
4. **Use strong authentication** — Enable SSH key-based authentication

### Password Management

```bash
# Remove stored passwords from Windows
rm /mnt/windows/Documents/.passwords
rm /mnt/windows/AppData/.credentials

# Clear browser password stores
rm /mnt/windows/Local Settings/Application Data/Mozilla/Firefox/Profiles/*.defaults
```

---

## Troubleshooting Guide

### Partition Not Detected

**Problem:** `fsarchiver probe` shows no partitions.

**Steps:**
1. Check if disk is detected:
   ```bash
   lsblk
   fdisk -l /dev/sdX
   ```
2. Verify disk is not mounted elsewhere:
   ```bash
   mount | grep /dev/sdX
   ```
3. Check for hardware issues:
   ```bash
   dmesg | grep -i error
   smartctl -a /dev/sdX
   ```

### Cannot Read NTFS Files

**Problem:** Files appear but cannot be accessed.

```bash
# Check filesystem health
ntfsfix /dev/sdXN

# Try different mount options
mount -t ntfs3 -o uid=1000,gid=1000,umask=022 /dev/sdXN /mnt/windows

# Or use ntfs-3g explicitly
ntfs-3g /dev/sdXN /mnt/windows
```

### SSH Connection Failed

**Problem:** Cannot connect via SFTP/SSH.

```bash
# Verify SSH is running
systemctl status sshd

# Check firewall allows SSH
ufw status | grep 22

# Test connection locally
ssh root@localhost

# Enable password authentication
nano /etc/ssh/sshd_config
# Ensure: PasswordAuthentication yes
```

---

## Next Steps

1.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Advanced network setup
2.  **[Secure deletion](./manual_Secure_Deletion_of_Data_en.md)** — Clean up after backup
3.  **[Backup tools](./manual_Backup_and_transfer_your_data_using_rsync_en.md)** — Automated backup solutions

---

## Additional Resources

*   [rsync Documentation](https://rsync.samba.org/README.html)
*   [OpenSSH Manual](https://man7.org/linux/man-pages/man1/ssh.1.html)
*   [FileZilla Client](http://filezilla-project.org/)
*   [WinSCP](https://winscp.net/)
*   [NTFS-3G](https://tuxera.com/kb/ntfs-3g-faq/)

---

*Last updated: 2026-11-04*
