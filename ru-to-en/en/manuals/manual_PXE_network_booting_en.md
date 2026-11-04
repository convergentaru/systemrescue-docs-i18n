+++
title = "PXE Network Booting"
draft = false
aliases = ["/Sysresccd-manual_ru_PXE_network_booting"]
+++

## Overview

[PXE Environment](http://en.wikipedia.org/wiki/Preboot_Execution_Environment) allows booting a computer with SystemRescue even if this computer has no CD-ROM drive or USB port. This allows booting SystemRescue from the network, and then faster troubleshooting on computers in your network after installing PXE boot server. Also this is useful if you need to boot a computer without optical drive (CD or DVD) or troubleshoot a remote computer where you don't have physical access for inserting a disk.

Instructions below correspond to 64-bit version (amd64/x86_64) SystemRescue files. If you're using 32-bit version, you need to replace all instances of `x86_64` with `i686`.

---

## Requirements

* **Servers providing PXE services**, including DHCP with SystemRescue files available for servicing client requests
* **Client computer** with network card supporting PXE, in same network and sufficient memory for storing SystemRescue files completely

---

## How PXE Booting Process Works

### PXE Boot Server

PXE boot server consists of three stages:

* **stage 0:** client booted via PXE sends extended DHCPDISCOVER
* **stage 1:** DHCP server sends IP address to client and additional information such as TFTP server address
* **stage 2:** client, booted via PXE, configures network and requests first boot files from TFTP server (bootloader, kernel image, and sysresccd.img)
* **stage 3:** client, booted via PXE, requests squashfs root filesystem image from HTTP or NFS or NBD server

These three parts can be installed either on one computer or on several.

### PXE Boot Process

You may need to understand what's happening when SystemRescue is booted from network. You'll need this information for troubleshooting in case of problems. Here are the most important steps of PXE boot process:

* When client computer attempts to boot via PXE, it first sends DHCP request over network to get IP address.
* Then DHCP server responds with DHCP offer containing new IP address that hasn't been assigned yet, and some specific options (DNS, default routing), and TFTP server IP address.
* Client receives this DHCP offer and accepts it. It connects to TFTP server (it got its IP address on previous stage) to get boot files.
* TFTP server sends bootloader files (`pxelinux`) and text files displayed on screen `pxelinux`.
* Client displays pxelinux prompt, and user can select boot options. Then it requests from TFTP server kernel and initramfs files needed for starting system.
* TFTP server sends kernel and initramfs files (e.g.: `vmlinuz` + `sysresccd.img`) to client.
* Client loads this kernel and executes early boot programs provided by initramfs.
* During its initialization, kernel again sends DHCP request because of kernel boot parameters `ip=dhcp`. Indeed, kernel doesn't know IP address used by computer at pxelinux stage.
* Client needs file `airootfs.sfs`. If you're using HTTP or TFTP for third stage, `airootfs.sfs` will be loaded into RAM, so client must have sufficient memory (estimated requirement: 1 GB). If you're using either NFS or NBD, you don't have this memory requirement, and client will make persistent requests to server each time it needs a file from root filesystem.
* Client mounts `airootfs.sfs`, and now it can complete boot process.
* At this stage client stores all files in memory (if you used TFTP/HTTP for third stage), so it no longer requires boot server. If you're using NFS or NBD, connection is still required.

---

## Customizing Boot Command Line

PXE server consists of several services. On second stage server uses TFTP to send client various things: bootloader (pxelinux.0), kernel image (vmlinuz), initramfs (sysresccd.img). Bootloader is `pxelinux.0`, and it comes with configuration file that is sent to client. This configuration file contains boot command line that will be used by client for starting Linux kernel. This command line is important since it contains SystemRescue boot options needed for starting third stage. Boot command line can be used for specifying network settings and method that PXE client will use on third stage of PXE boot process. Here are two examples of valid command lines for PXE booting:

---

## PXE Boot Options Supported by SystemRescue

SystemRescue is based on Arch Linux since version 6.0, so it supports PXE boot options implemented upstream. These boot options provide support for various protocols (HTTP, NFS, NBD) and are documented on the following page: https://gitlab.archlinux.org/archlinux/archiso/-/raw/v43/docs/README.bootparams

Most important of them are:

### HTTP Option

* **archiso_http_srv=** Set HTTP URL (must end with /) from where to load *.sfs file when using HTTP. `${archisobasedir}` and architecture (`x86_64`) are added to specified URL.
  http and https are supported. When using https certificate verification is not performed, since initramfs environment doesn't have trusted CA database.

### NFS Option

* **archiso_nfs_srv=** Set address of NFS server IP:/path from where to load *.sfs file when using NFS. `${archisobasedir}` and architecture (`x86_64`) are added to specified path. Note that due to limitations of tools only IPv4 addresses are supported, no host DNS names or IPv6.

---

## Example PXE Configuration for Booting SystemRescue with TFTP and HTTP

Here's an example PXE configuration that allows booting SystemRescue using only TFTP and HTTP. This is one of the simplest PXE configurations you can have, so it's recommended to follow this example if you need a simple PXE configuration for SystemRescue.

In this example bootloader is `pxelinux.0`, and TFTP is used for getting kernel image and initramfs images. Then it uses HTTP server for loading large squashfs filesystem `airootfs.sfs`.

When you configure TFTP server, you need to copy bootloader (pxelinux binary files), pxelinux configuration file in `pxelinux.cfg/default`, and you need to copy `sysresccd` directory as is (although not all files are required by TFTP server) from ISO image to provide files that will be requested via TFTP.

Here's list of all required files and their location on TFTP server:

```
./sysresccd/boot/x86_64/vmlinuz
./sysresccd/boot/x86_64/sysresccd.img
./sysresccd/boot/memtest.COPYING
./sysresccd/boot/memtest
./sysresccd/boot/intel_ucode.LICENSE
./sysresccd/boot/intel_ucode.img
./sysresccd/boot/amd_ucode.LICENSE
./sysresccd/boot/amd_ucode.img
./sysresccd/VERSION
./sysresccd/pkglist.x86_64.txt
./pxelinux.cfg/default
./ldlinux.c32
./pxelinux.0
```

Here's an example configuration located in `pxelinux.cfg/default`:

```
DEFAULT sysresccd
LABEL sysresccd
  LINUX sysresccd/boot/x86_64/vmlinuz
  INITRD sysresccd/boot/intel_ucode.img,sysresccd/boot/amd_ucode.img,sysresccd/boot/x86_64/sysresccd.img
  APPEND archisobasedir=sysrescue ip=dhcp archiso_http_srv=http://10.0.2.4/ checksum
  SYSAPPEND 3
```

Option SYSAPPEND is important, and you need to ensure that you're using latest version of pxelinux, since versions older than 5.10 don't support it. Also PXE booting users are recommended to use latest versions of SystemRescue.

You need to update HTTP server IP address from where squashfs image will be loaded. In this example it will load this file and its checksum from following URLs:

```
http://10.0.2.4/sysresccd/x86_64/airootfs.sfs
http://10.0.2.4/sysresccd/x86_64/airootfs.sha512
```

Ensure that you can successfully download these files using regular web client before attempting to boot via PXE. You can customize boot command line by adding options such as `setkmap=us` after `checksum` option.

---

## Complete PXE Setup Guide

### Step 1: Prepare TFTP Server Directory Structure

```bash
# Create directory structure on TFTP server
cd /var/lib/tftpboot
mkdir -p sysresccd/boot/x86_64
mkdir -p pxelinux.cfg

# Extract files from SystemRescue ISO
mount -o loop,ro SysRescue.iso /mnt/sysresccd
cp -r /mnt/sysresccd/sysresccd/* .
cp -r /mnt/sysresccd/pxelinux.cfg/default .
umount /mnt/sysresccd

# Copy ucode files (required for CPU microcode updates)
cp sysresccd/boot/intel_ucode.img .
cp sysresccd/boot/amd_ucode.img .
```

### Step 2: Configure pxelinux.cfg/default

```
# PXE Boot Configuration for SystemRescue
DEFAULT sysresccd

LABEL sysresccd
    MENU LABEL SystemRescue (HTTP rootfs)
    LINUX sysresccd/boot/x86_64/vmlinuz
    INITRD sysresccd/boot/x86_64/sysresccd.img,sysresccd/boot/intel_ucode.img,sysresccd/boot/amd_ucode.img
    APPEND archisobasedir=sysrescue \
           ip=dhcp \
           archiso_http_srv=http://192.168.1.100/ \
           checksum \
           setkmap=us \
           nofirewall
    SYSAPPEND 3

LABEL sysrescue-nfs
    MENU LABEL SystemRescue (NFS rootfs)
    LINUX sysresccd/boot/x86_64/vmlinuz
    INITRD sysresccd/boot/x86_64/sysresccd.img,sysresccd/boot/intel_ucode.img,sysresccd/boot/amd_ucode.img
    APPEND archisobasedir=sysrescue \
           ip=dhcp \
           archiso_nfs_srv=192.168.1.100::/opt/sysrescue \
           checksum
    SYSAPPEND 3

LABEL memtest86+
    MENU LABEL Memory Test
    KERNEL sysresccd/boot/memtest
    APPEND ddr512=0x90000 vga=791
```

### Step 3: Configure DHCP Server

For ISC DHCP server (`/etc/dhcpd.conf`):

```bash
# PXE Boot Configuration for DHCP
subnet 192.168.1.0 netmask 255.258.0.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    
    # PXE Boot options
    next-server 192.168.1.100;  # TFTP server IP
    filename "pxelinux.0";
    
    # Class for SystemRescue clients
    class "SystemRescue" {
        match if substring(option vendor-class-identifier, 0, 13) = "PXEClient Arch";
        
        option host-name "SysRescue-Client";
        filename "pxelinux.0";
        next-server 192.168.1.100;
    }
}
```

### Step 4: Configure HTTP Server for Root Filesystem

For Apache/Nginx serving SystemRescue root filesystem:

```bash
# Enable directory indexing and serve files from /var/www/sysrescue
# Or configure to serve from specific location
Location /sysresccd/ {
    Alias /var/www/sysrescue/
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
}
```

---

## Quick Reference Table

| Component | Purpose | Configuration |
|-----------|---------|---------------|
| TFTP Server | Bootloader and kernel delivery | `vsftpd`, `atftpd`, `tftpd-hpa` |
| DHCP Server | IP address assignment | ISC DHCP, dnsmasq |
| HTTP Server | Root filesystem delivery | Apache, Nginx |
| NFS Server | Alternative root delivery | NFS server daemon |
| PXE Client | Boot from network | Network card with PXE support |

---

## Common PXE Boot Options

| Option | Description | Example |
|--------|-------------|---------|
| `ip=dhcp` | Get IP address via DHCP | Always include |
| `archiso_http_srv=` | HTTP server for rootfs | `http://192.168.1.100/` |
| `archiso_nfs_srv=` | NFS server for rootfs | `192.168.1.100:/path` |
| `checksum` | Verify ISO integrity | Recommended |
| `setkmap=` | Set keyboard layout | `setkmap=us` |
| `nofirewall` | Disable firewall | For network access |
| `copytoram` | Copy system to RAM | Faster operation |

---

## Troubleshooting PXE Boot

### "No bootable device" Error

**Problem:** Client cannot find PXE boot files.

**Solutions:**
1. Verify TFTP server is running:
   ```bash
   netstat -tlnp | grep 69
   ps aux | grep tftp
   ```
2. Check file permissions on TFTP root:
   ```bash
   ls -la /var/lib/tftpboot/pxelinux.0
   chmod 644 /var/lib/tftpboot/pxelinux.0
   ```
3. Verify DHCP is assigning correct next-server:
   ```bash
   tcpdump -i any port 67 -n | grep -E 'next|filename'
   ```

### Bootloader Not Found

**Problem:** pxelinux.0 cannot be loaded.

```bash
# Check TFTP server configuration
/tftpboot/pxelinux.0

# Test TFTP manually
tftp 192.168.1.100
get pxelinux.0
```

### HTTP Download Fails

**Problem:** Cannot download airootfs.sfs from HTTP server.

```bash
# Test HTTP connection
curl -I http://192.168.1.100/sysresccd/x86_64/airootfs.sfs

# Check if files exist
ls -la /var/www/sysrescue/x86_64/airootfs.sfs

# Verify file integrity
sha512sum /var/www/sysrescue/x86_64/airootfs.sfs
```

### Memory Insufficient

**Problem:** Client doesn't have enough RAM for full system in memory.

**Solutions:**
1. Use NFS or NBD instead of HTTP/TFTP for root filesystem
2. Increase client RAM (add more memory to machine)
3. Use smaller SystemRescue version if available

### Network Timeout

**Problem:** PXE boot times out waiting for server response.

```bash
# Check network connectivity between clients and servers
ping 192.168.1.100

# Verify firewall allows necessary ports
iptables -L -n | grep -E '67|68|69|80'

# Check for VLAN issues
ip link show
```

---

## Best Practices

### ✅ Recommended

1. **Use dedicated PXE server** — Separate from production servers
2. **Enable checksum verification** — Ensure ISO integrity
3. **Document configuration** — Keep records of all settings
4. **Test with multiple clients** — Verify compatibility
5. **Regular updates** — Keep SystemRescue and servers current

### ❌ Not Recommended

1. **Mixing old and new configurations** — Use consistent versions
2. **No network segmentation** — Isolate PXE traffic
3. **Single point of failure** — Have backup servers
4. **Overloading single server** — Distribute load across multiple servers

---

## Security Considerations

### ⚠️ Network Access Control

When setting up PXE boot:

1. **Restrict to trusted networks only** — Limit which clients can PXE boot
2. **Use MAC address filtering** — Allow only known client MAC addresses
3. **Disable PXE on production machines** — Prevent unauthorized booting
4. **Monitor PXE traffic** — Detect suspicious activity

### DHCP Spoofing Prevention

```bash
# Bind DHCP server to specific interface
dhcpd -cf /etc/dhcpd.conf -pf /var/run/dhcpd.pid eth0

# Enable logging
log-facility local7;
```

---

## Alternative Boot Methods

### Using dnsmasq for Lightweight PXE

```bash
# Install dnsmasq
apt-get install dnsmasq

# Configure for PXE boot
cat > /etc/dnsmasq.conf << EOF
bind-interfaces
dhcp-range=192.168.1.100,192.168.1.200,12h
dhcp-boot=pxelinux.0
enable-tftp
tftp-root=/var/lib/tftpboot
EOF

# Enable TFTP
apt-get install tftpd-hpa
```

### Using iPXE for Advanced PXE Features

```bash
# iPXE offers more flexibility than pxelinux
# Can redirect to other boot options dynamically
kernel http://192.168.1.100/sysresccd/x86_64/vmlinuz \
    initrd=http://192.168.1.100/sysresccd/x86_64/sysresccd.img \
    archisobasedir=sysrescue ip=dhcp checksum setkmap=us
```

---

## Advanced: Multiple Boot Options

### Configuration with Multiple Choices

```
DEFAULT sysrescue-default
LABEL sysrescue-default
    MENU LABEL SystemRescue (Default)
    LINUX vmlinuz
    INITRD sysresccd.img
    APPEND archisobasedir=sysrescue ip=dhcp checksum setkmap=us

LABEL sysrescue-quick
    MENU LABEL SystemRescue (Quick - No Firewall)
    LINUX vmlinuz
    INITRD sysresccd.img
    APPEND archisobasedir=sysrescue ip=dhcp checksum nofirewall

LABEL sysrescue-debug
    MENU LABEL SystemRescue (Debug Mode)
    LINUX vmlinuz debug
    INITRD sysresccd.img
    APPEND archisobasedir=sysrescue ip=dhcp debug

LABEL sysrescue-noprompt
    MENU LABEL SystemRescue (No Prompt - Auto Boot)
    LINUX vmlinuz noprompt
    INITRD sysresccd.img
    APPEND archisobasedir=sysrescue ip=dhcp checksum noprompt
```

---

## Next Steps

1.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Advanced network setup for PXE
2.  **[Booting options](./manual_Booting_SystemRescue_en.md)** — Command line vs network boot comparison
3.  **[Serial console](./manual_Booting_on_a_serial_console_en.md)** — Alternative remote access method

---

## Additional Resources

*   [PXE Boot Specification](https://www.x.org/docs/PXESPEC.pdf)
*   [ISC DHCP Documentation](https://www.isc.org/downloads/software-support/download/isc-dhcp/)
*   [dnsmasq PXE Guide](http://thekelleys.org.uk/dnsmasq/doc.html#pxe)
*   [iPXE Documentation](http://www.ipxe.org/documentation)

---

*Last updated: 2026-11-04*
