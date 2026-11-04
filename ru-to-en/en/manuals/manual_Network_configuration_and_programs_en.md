+++
title = "Network Configuration and Programs"
+++

## Network Configuration Tools

If your system has supported hardware, the Ethernet or Wifi network adapter should be automatically detected, driver loaded, and DHCP auto-configuration attempted.

SystemRescue uses NetworkManager as the default network configuration tool. It provides a very user-friendly graphical interface for network configuration. It simplifies configuration, especially for wireless networks. For example, wireless networks will be automatically discovered, and connecting to them is very easy. Inside X11, NetworkManager is available as a small icon in the taskbar right next to the clock. It also provides `nmcli` and `nmtui` if you want to configure the network using command line or semi-graphical interface from terminal.

You can also configure the network with other tools such as standard `ifconfig` or `ip` commands. In this case, you should first stop the NetworkManager service, otherwise a conflict will occur and you'll lose your settings. You can stop it by running `systemctl stop NetworkManager` in the shell.

---

## Manual Network Configuration

By default, all network interfaces are activated and auto-configured. The `nmcli` command will show an overview of what's currently configured.

To disable and re-enable interfaces, use `nmcli conn down "Wired connection 1"` and `nmcli conn up "Wired connection 1"`. Note that "Wired connection 1", used as an example here, is not the interface name as assigned by the kernel, but is an internal name for NetworkManager. Use `nmcli` to get an overview of all connections and their names.

To set a static IP, netmask, gateway manually, you can also use nmcli:

```bash
nmcli conn mod "Wired connection 1" ipv4.method manual
nmcli conn mod "Wired connection 1" ipv4.addr 192.168.1.20/24
nmcli conn mod "Wired connection 1" ipv4.gateway 192.168.1.1
nmcli conn mod "Wired connection 1" ipv4.dns 192.168.1.1
nmcli conn up "Wired connection 1"
```

The last "conn up" is required to reconfigure the interface with new parameters.

To return to automatic DHCP configuration, use:

```bash
nmcli conn mod "Wired connection 1" ipv4.method auto
nmcli conn up "Wired connection 1"
```

### Alternative: Using ip and ifconfig Commands

If NetworkManager is stopped, you can use standard commands:

```bash
# View interfaces
ip addr show
ifconfig -a

# Bring interface up
ip link set eth0 up

# Assign static IP
ip addr add 192.168.1.20/24 dev eth0

# Set default gateway
ip route add default via 192.168.1.1

# Set DNS
echo "nameserver 192.168.1.1" > /etc/resolv.conf
```

---

## Firewall

Starting from version 6.0.4, SystemRescue comes with iptables firewall enabled by default to block incoming connection requests. You need to update iptables configuration or stop iptables and ip6tables services if you need to receive incoming connections from outside. You can boot into SystemRescue with the `nofirewall` option in the boot command line if you don't want a firewall.

### Firewall Commands

```bash
# Check firewall status
systemctl status ufw        # or iptables service

# Stop firewall temporarily
systemctl stop ufw
systemctl stop ip6tables

# Allow SSH (port 22)
ufw allow ssh
ufw allow 22/tcp

# Enable firewall again
ufw enable

# View rules
ufw status verbose

# Reset to default
ufw --force reset
```

### iptables Rules Example

```bash
# Flush existing rules
iptables -F
iptables -X

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Drop all other incoming
iptables -A INPUT -j DROP
```

---

## Starting SSH Server

SSH allows you to use the shell on another computer, and you can copy files (with scp or rsync through ssh). Sshd is already running in background. But to allow incoming connections, you need to allow connection through firewall (see above) and change root password. Just enter `passwd` to set a password. You can also use boot options `rootpass=xxx` or `rootcryptpass=xxxx` before starting SystemRescue to set the root password.

A more secure alternative is to use SSH key files. You can set authorized keys on your SystemRescue media with [YAML file](./manual_Configuring_SystemRescue_sysconfig_en.md).

### Enable and Configure SSH

```bash
# Start SSH service
systemctl start sshd

# Enable auto-start on boot
systemctl enable sshd

# Check status
systemctl status sshd

# Test connection locally
ssh root@localhost
```

### SSH Configuration

Edit `/etc/ssh/sshd_config`:

```bash
nano /etc/ssh/sshd_config

# Important settings:
PermitRootLogin yes           # Allow root login
PasswordAuthentication yes    # Allow password auth
PubkeyAuthentication yes     # Enable key-based auth
Port 22                      # SSH port
ListenAddress 0.0.0.0        # Listen on all interfaces

# Security recommendations:
# PermitRootLogin prohibit-password  (more secure)
# PasswordAuthentication no         (use keys only)
# AllowUsers username              (restrict users)
```

After editing, restart SSH:
```bash
systemctl restart sshd
```

---

## Accessing Windows Shares via CIFS/SMB

SystemRescue comes with smbfs/cifs client package that allows connecting to a Windows machine with shared drives. In recent kernels, smbfs support has been replaced by cifs, so you should try to avoid using smbfs.

The mount-cifs package allows you to access a Windows computer on the network. Here's an example explaining how to access Windows shared folders. Let's assume that the Windows box is at 192.168.10.3 and has a share directory called mydata, accessible by a user named robert:

```bash
# Create mount point
mkdir /mnt/windows

# Mount CIFS share
mount -t cifs //192.168.10.3/mydata /mnt/windows \
    -o username=robert,password=passwd,iocharset=utf8

# Change to the mounted directory
cd /mnt/windows

# Now you should be able to see files in /mnt/windows
ls -la
```

Remember to unmount the directory when you're done with what you're doing in the shared folder:

```bash
umount /mnt/windows
```

### CIFS Mount Options

| Option | Description | Example |
|--------|-------------|---------|
| `username=` | Username for authentication | `username=robert` |
| `password=` | Password (⚠️ visible in process list) | `password=secret` |
| `iocharset=` | Character encoding | `iocharset=utf8` |
| `sec=` | Security level | `sec=ntlm` or `sec=ntlmv2` |
| `vers=` | SMB protocol version | `vers=2.0` or `vers=3.0` |

---

## Mounting Remote SSH Partitions as Local Filesystems

If you want to access files located on a remote system that you can connect to via SSH, you can mount the remote directory with ssfs and FuSE library. "Userland FileSystem" allows you to mount a partition and work with remote files just like you would work with any local files. With these filesystems, you can unmount the partition with the standard `umount` command. Here's an example of how to mount an SSH filesystem in /mnt/ssh as anonymous (read-only):

```bash
# Create mount point
mkdir /mnt/ssh

# Set root password for remote access
passwd root

# Mount SSH filesystem
sshfs root@ssh.server.org:/path/to/dir /mnt/ssh -o ro

# Change to the directory
cd /mnt/ssh

# List files
ls -la

# Unmount when done
umount /mnt/ssh
```

### sshfs Options

| Option | Description |
|--------|-------------|
| `-o ro` | Mount read-only |
| `-o allow_other` | Allow other users to access |
| `-o uid=1000,gid=1000` | Set ownership |
| `-o server_opts="CacheDir=/tmp"` | Set cache directory |

---

## Accessing Cloud Storage (WebDAV, S3, and Others)

SystemRescue contains [rclone](https://rclone.org) program that allows accessing, synchronizing, and mounting many different cloud storage services. To use it, you first need to configure the storage service with `rclone config` in an interactive menu. Each service gets a remote name this way.

After that, use `rclone ls <remote name>:` to list the contents of the remote.

Use `rclone copy /local/path <remote name>:destpath` to upload a file, and `rclone copy <remote name>:sourcefile /local/targetpath` to download.

### Configure Rclone

```bash
# Start interactive configuration
rclone config

# Select new remote
n
# Enter remote name (e.g., mygoogle)
# Choose storage type (e.g., google drive)
# Fill in credentials
# Save configuration
y
```

### Common Cloud Storage Remotes

| Service | Remote Name Pattern | Example Command |
|---------|---------------------|-----------------|
| Google Drive | `gdrive:` | `rclone ls gdrive:/photos` |
| Dropbox | `dropbox:` | `rclone copy /local dropbox:/backup` |
| Amazon S3 | `s3:` | `rclone ls s3:bucket-name/photos` |
| WebDAV | `webdav:` | `rclone copy /local webdav:/home/user/file.txt` |

### Upload/Download Examples

```bash
# Upload directory to cloud
rclone copy /path/to/local/dir gdrive:/backup/

# Download from cloud
rclone copy gdrive:/photos/vacation.zip /home/user/

# Sync (two-way, deletes missing files)
rclone sync /local gdrive:/backup --progress

# Mount as filesystem
rclone mount gdrive:/ /mnt/gdrive \
    -o allow_other,uid=1000,gid=1000
```

---

## Network Diagnostics

### Check Connectivity

```bash
# Test network connectivity
ping -c 4 8.8.8.8

# Test DNS resolution
nslookup google.com
dig google.com

# Check route table
ip route show
netstat -rn

# Check connected networks
nmcli device status
```

### Troubleshooting Network Issues

**Can't get IP address:**
```bash
# Restart network service
systemctl restart dhcpcd

# Or use nmcli
nmcli connection down eth0
nmcli connection up eth0
```

**Can't connect to remote server:**
```bash
# Check if port is open
nc -zv remote.server.com 22

# Trace route
traceroute remote.server.com

# Check firewall rules
iptables -L -n -v
```

---

## Quick Reference: Network Commands

| Task | Command | Notes |
|------|---------|-------|
| View interfaces | `ip addr show` | Shows all network interfaces |
| Start DHCP | `dhcpcd eth0` | Auto-configure with DHCP |
| Static IP | `nmcli conn mod ... ipv4.method manual` | Manual configuration |
| Enable SSH | `systemctl start sshd` | Remote access enabled |
| Allow CIFS | `mount -t cifs //server/share /mnt` | Windows shares |
| SSHFS mount | `sshfs user@host:/path /mnt` | Remote filesystem |
| Rclone config | `rclone config` | Cloud storage setup |
| Firewall status | `ufw status` | Check firewall rules |

---

## Next Steps

1.  **[System tools](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/System-tools.md)** — Overview of available utilities
2.  **[Partitioning](./manual_Mounting_ntfs_filesystems_en.md)** — Working with disk partitions
3.  **[Backup tools](./manual_Backup_and_transfer_your_data_using_rsync_en.md)** — Data backup and transfer

---

*Last updated: 2026-11-04*
