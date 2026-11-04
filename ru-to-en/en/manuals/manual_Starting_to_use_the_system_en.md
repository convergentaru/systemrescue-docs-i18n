+++
title = "Getting Started with the System"
+++

Upon booting, you should read the messages that give you the most important information about how to use this system. You should read the text.

You can log into another console. The root password is empty. Enter
`dhcpcd eth0` (or similar) to automatically configure the network using DHCP.

To start an ssh server on this system, enter `systemctl start sshd`.
If you need to log in remotely as root, enter `passwd root` to reset
the root user password to a known value.

If you need graphical tools (such as GParted), you need to switch to the
graphical environment. You should simply enter `startx` to launch it.

You might need to use an editor. Available editors: vim, nano in
console mode. You can also use featherpad and geany in graphical environment.

---

## First Steps After Booting

### 1. Check System Status

```bash
# Check kernel version
uname -r

# Check available memory
free -h

# Check disk space
df -h

# List mounted filesystems
mount | grep -v "^proc\|^sys\|^dev"
```

### 2. Network Configuration

**Automatic (DHCP):**
```bash
# Start network service
dhcpcd eth0
# or for other interface names
dhcpcd wlan0
dhcpcd enp3s0
```

**Manual Configuration:**
```bash
# Edit network config
nano /etc/dhcpcd.conf

# Or use nmcli if NetworkManager is available
nmcli connection add type ethernet con-name MyConnection dev eth0 ip4 192.168.1.100/24 gw4 192.168.1.1
nmcli connection up MyConnection
```

### 3. Enable Remote Access (Optional)

**Start SSH Server:**
```bash
# Start sshd service
systemctl start sshd

# Enable auto-start on boot
systemctl enable sshd

# Check status
systemctl status sshd
```

**Set Root Password for SSH:**
```bash
# Set a known password for root
passwd root

# Or create a new user
useradd -m -s /bin/bash username
passwd username
```

**Allow Remote Root Login (⚠️ Security Warning):**
```bash
# Edit sshd_config
nano /etc/ssh/sshd_config

# Add or modify these lines:
PermitRootLogin yes
PasswordAuthentication yes
```

### 4. Start Graphical Environment (Optional)

```bash
# Start X11 display manager
startx

# Or use the menu if available
xfce4-session
```

### 5. Available Editors

**Console Mode:**
- `vim` — Full-featured editor (vi enhanced)
- `nano` — Simple, user-friendly editor
- `ed` — Basic line editor

**Graphical Mode:**
- `featherpad` — Lightweight text editor
- `geany` — Full-featured IDE
- `gedit` — GNOME text editor

### 6. Useful Commands

| Command | Purpose |
|---------|---------|
| `mc` | Midnight Commander (file manager) |
| `lsblk` | List block devices |
| `fdisk -l` | Show disk partitions |
| `blkid` | Show filesystem labels/types |
| `dmesg` | Kernel ring buffer messages |
| `journalctl -xe` | View system logs |

---

## Common Tasks

### Mounting Partitions

```bash
# List available devices
lsblk

# Create mount point
mkdir /mnt/data

# Mount ext4 partition
mount /dev/sdXN /mnt/data

# Mount NTFS partition (requires ntfs-3g)
mount -t ntfs-3g /dev/sdXN /mnt/data

# Mount UUID instead of device name
mount /dev/disk/by-uuid/XXXX-XXXX /mnt/data
```

### Creating Backup

```bash
# Using tar
tar -czvf backup.tar.gz /path/to/backup

# Using rsync (faster, incremental)
rsync -av --progress /source/ /destination/

# Full disk image
dd if=/dev/sdX of=/path/to/image.img bs=4M status=progress
```

### Checking Disk Health

```bash
# SMART data (if available)
smartctl -a /dev/sdX

# Filesystem checks
fsck /dev/sdXN

# Check for bad blocks
badblocks -v /dev/sdXN
```

---

## Quick Reference

| Task | Command | Notes |
|------|---------|-------|
| Network (auto) | `dhcpcd eth0` | Replace with your interface |
| Start SSH | `systemctl start sshd` | For remote access |
| Set root password | `passwd root` | Required for SSH login |
| Start GUI | `startx` | Requires X11 support |
| File manager | `mc` | Norton Commander clone |
| Text editor | `vim` or `nano` | Console mode editors |
| Graphical editor | `featherpad` or `geany` | In X11 environment |

---

## Security Considerations

### ⚠️ Important Security Notes

1. **Root Password**: Default is empty — change it immediately!
   ```bash
   passwd root
   ```

2. **SSH Access**: Enable firewall before allowing remote connections
   ```bash
   systemctl start ufw  # or iptables
   ```

3. **Network**: Disable unnecessary network services
   ```bash
   systemctl stop httpd    # Apache if not needed
   systemctl stop nfs      # NFS server if not needed
   ```

4. **Persistence**: Changes made in live mode won't persist unless:
   - Using `copytoram` with persistence storage
   - Mounting a persistent filesystem with `cow_label=` option

---

## Troubleshooting

### SSH Not Starting
```bash
# Check service status
systemctl status sshd

# Enable it
systemctl enable sshd

# Check logs
journalctl -u sshd
```

### Network Not Working
```bash
# Check interface
ip addr show

# Try manual config
ip addr add 192.168.1.100/24 dev eth0
ip route add default via 192.168.1.1
```

### GUI Won't Start
```bash
# Check display manager
systemctl status gdm   # or lightdm, sddm

# Try starting X manually
startx

# Check logs
journalctl -xe | grep -i xorg
```

---

## Next Steps

1.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Detailed network setup guide
2.  **[System tools](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/System-tools.md)** — Overview of available utilities
3.  **[Partitioning](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/disk-partitioning_Introduction-to-disk-partitioning.md)** — Disk partitioning guide

---

## Additional Resources

*   **Midnight Commander Guide**: [mc-linux.org](http://www.mc-linux.org/)
*   **Vim Tutorial**: [vim.tutor](https://vim.sourceforge.io/docs.php#tutor)
*   **SSH Security Best Practices**: [OpenSSH FAQ](https://www.openssh.com/faq.html)

---

*Last updated: 2026-11-04*
