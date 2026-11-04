+++
title = "Booting SystemRescue"
+++

## Booting from Removable Media

To boot from a CD-ROM or USB device, ensure that your computer's firmware (BIOS or UEFI settings) is configured to boot from the correct device and priority is set correctly.

To run SystemRescue, insert the CD or USB drive into the drive and power on or restart the computer, or press the key for selecting an alternative boot device at startup.

---

## Boot Command Line Options

In the bootloader you can press TAB (syslinux, used for classic BIOS booting) or "e" (grub, used for UEFI booting) to edit the boot command line of the currently selected boot entry. Then you can add options listed below.

### Available in Both Command Line and YAML Configuration

Most options are available in the boot command line as well as in [YAML configuration](./manual_Configuring_SystemRescue_en.md) in the `global` section. Boot command line options take precedence over YAML configuration options.

The following options are supported in both the boot command line and YAML configuration:

* **setkmap=xx** specifies the keyboard layout, where `xx` corresponds to a keyboard map code. For example `setkmap=de` will configure German keyboard layout during the boot process. You can also set the keyboard layout by running a command such as `setkmap de` from the shell after booting, and you can run `setkmap` without parameters in the terminal of running SystemRescue to get a list of all supported keyboard maps.

* **copytoram** forces SystemRescue to fully load into memory. This corresponds to the `docache` option in previous versions. Slow start, but after completion the system will be more responsive and it won't require the original device to work. This means you can actually work with the device where SystemRescue is installed. This requires 2 GB of memory for caching the system.

* **checksum** runs a squashfs checksum verification during boot so you know if the file was corrupted. You should use this boot option if you're getting unexpected errors when booting SystemRescue.

* **rootpass=password123**: Sets the livecd root system password to `password123`. This allows you to connect over the network and ssh to the livecd and authenticate using this password. For security reasons, it's recommended to use the alternative option `rootcryptpass` instead of `rootpass` so the password is not visible in plaintext.

* **rootcryptpass=xxxx**: Sets the livecd root system password so you can connect to the system remotely via ssh and use a password for authentication on the livecd. The password must be encrypted using the `mkpasswd` command. The mkpasswd program is available on most Linux distributions, it's often provided as part of the `whois` package.
  
  You need to provide the full output of mkpasswd for `rootcryptpass=`, including the `$` prefix. The encrypted password contains dollar signs, so you should check if your bootloader requires their escaping.
  
  If in doubt, check `/proc/cmdline` of running SystemRescue.

* **nofirewall** stops iptables and ip6table services that are enabled by default for blocking incoming connection requests. You need to use this option if you need to establish connections with a SystemRescue running system from outside (e.g., connections to sshd). This option was introduced in SystemRescue-6.0.4.

* **rootshell=/bin/myshell** uses an alternative shell such as /bin/zsh instead of /bin/bash. This option was introduced in SystemRescue-6.1.1.

* **cow_label=xxxx** Sets the label of the filesystem where upperdir/workdir files for overlayfs should be stored. By default, this option is not set, and changes made to SystemRescue files will be lost after reboot. In other words, you can use a Linux filesystem to save all changes made to SystemRescue while it's running, such as new Firefox bookmarks, application configuration files, etc. You should provide the label of the filesystem to use for saving these changes. All these changes will be isolated in a directory with prefix `persistent` unless you override this with `cow_directory`.

* **cow_directory=xxxx** The name of the directory where to store changes made to the system. You need to specify a cow-device for this change to take effect.

* **loadsrm=y** Loads all SRM modules located on the boot device

* **noautologin** Don't perform automatic console login. Use this option if you plan to use SystemRescue for running automated tasks for remote connection and want to prevent local users from using the system from the console. This option was introduced in SystemRescue-8.06.

* **dostartx** Automatically start graphical environment. This option was introduced in SystemRescue-8.06.

* **dovnc** Automatically start VNC server for remote graphical session connection possibility. This option was introduced in SystemRescue-8.06.

* **vncpass=password123** Set VNC password for remote connection. Without this option, connections to the VNC server will be unrestricted. This option was introduced in SystemRescue-8.06.

### Command Line Only Options

The following options are supported only in the boot command line:

* **cow_spacesize=xx** sets the size of the Copy-on-Write area, which is stored in a tmpfs filesystem in memory. It accepts values such as 512M, 4G, and defaults to 25% of system memory.

* **nomodeset** forces the system to work with a basic display driver at lower resolution instead of using optimal display settings. Use this option if information is not displayed correctly on the screen.

* **findroot**: Load a Linux OS installed on disk using SystemRescue kernel. This is very useful if you can't run a Linux OS directly via normal process. This may be caused by broken bootloader, for example. This option allows you to run your system through SystemRescue so you can fix the bootloader. This option works by scanning block devices during the boot process to find filesystems where a Linux operating system is installed (i.e., filesystems containing a file named `/sbin/init`). This includes block devices that are encrypted luks, so the user must provide a passphrase for access to these devices. Then it will show a list of all block devices that appear to be Linux root filesystems. The user then needs to select the block device to boot from. This option was introduced in SystemRescue version 6.1.4.

* **break**: Stop the boot process before the root filesystem is mounted. The shell will be executed from initramfs. This option allows manual command execution for troubleshooting if SystemRescue can't start normally.

* **break=postmount**: Stop the boot process after the root filesystem has been mounted. The shell will be executed from initramfs. This option allows manual command execution for troubleshooting if SystemRescue can't start normally.

* **sysrescuecfg=filename.yaml** adds additional yaml configuration files for booting after standard ones in the `sysrescue.d` directory. This option can be specified once or multiple times in the boot command line. See [YAML configuration](./manual_Configuring_SystemRescue_en.md) for details.
  This option was introduced in SystemRescue-9.01.

* **archisolabel=xxxx** Set the label of the filesystem where SystemRescue files are located. In other words, the system will try to find SystemRescue files on a filesystem with the specified label, so it's important to know which device contains SystemRescue files. By default this is set to `RESCUEXYZ` from version 7.00 (previously it was `SYSRCDXYZ` from versions 6.x), where XYZ corresponds to the SystemRescue version (e.g., label `RESCUE803` for version `8.03`)

* **archisobasedir=xxxx** Set the base directory where all SystemRescue files are located, and it defaults to `sysresccd`. In other words, the system will try to find SystemRescue files in a directory named `sysresccd`, which is located at the root of the filesystem.

* **img_label=xxxx** Set the label of the filesystem where the SystemRescue ISO image is located. This is used only when booting SystemRescue using loopback option from Grub2.

* **img_loop=xxxx** Set the path to the SystemRescue ISO image inside the filesystem so the boot process can mount it and find the squashfs filesystem. This is used only when booting SystemRescue using loopback option from Grub2.

* **nomdlvm** SystemRescue by default automatically activates all md RAID, device mapper, and LVM devices that are found in the system. Their activation may result in writes to disk, and this is sometimes undesirable, for example during forensic examination or working with defective disks. This option disables this behavior, and all devices must be activated manually.

* **archiso_http_srv=** Set HTTP URL (must end with /) from where to download *.sfs file when using HTTP. `${archisobasedir}` and architecture (`x86_64`) are added to the provided URL.
  http and https are supported. When using https, certificate verification is not performed since initramfs environment doesn't have a trusted CA database.

* **archiso_nfs_srv=** Set NFS-IP:/path server from where to download *.sfs file when using NFS. `${archisobasedir}` and architecture (`x86_64`) are added to the provided path. Note that due to tool limitations, only IPv4 addresses are supported, no DNS hostname or IPv6.

Please read [archiso documentation](https://gitlab.archlinux.org/archlinux/archiso/-/raw/v43/docs/README.bootparams) for more advanced boot options and PXE booting in particular.

### Auto-run Options

These options are available in the boot command line as well as in [YAML configuration](./manual_Configuring_SystemRescue_en.md) in the `autorun` section.

* **ar_source=xxx**: location where autostarts are stored. This can be a partition root directory (`/dev/sda1`),
  nfs partition (`nfs://192.168.1.1:/path/to/scripts`),
  samba partition (`smb://192.168.1.1/path/to/scripts`), or
  http(s) directory (`http://192.168.1.1/path/to/scripts`).

* **ar_ignorefail**: Continue executing the script chain even if a script
  fails (returns non-zero status)

* **ar_nodel**: Don't delete temporary copy of autostart scripts located
  in `/var/autorun/tmp` after execution

* **ar_disable**: Completely disable autostart, simple `autorun` script won't be executed

* **ar_nowait**: Don't wait for key press after autostart script execution

* **ar_attempts**: Use this option if you want to try loading a file
  multiple times

* **ar_suffixes=[0-9]**: Comma-separated list of suffixes corresponding to
  autostart scripts to be executed. For example with `ar_suffixes=0,2,7` the autostart scripts `autorun0`, `autorun2`, `autorun7` will be executed, and scripts
  with other suffixes will be ignored. Use `ar_suffixes=no` to disable all autostart scripts with suffix.

For more detailed information, please read
[autostart page](./manual_Run_your_own_scripts_with_autorun_en.md)

---

## Booting via Serial Console

See [Booting on a serial console](./manual_Booting_on_a_serial_console_en.md).

---

## PXE Network Booting

It's also possible to boot SystemRescue from the network. Since setup is not simple, there's a special page about
[PXE network booting](./manual_PXE_network_booting_en.md)

---

## Quick Reference: Common Boot Options

| Option | Description | Use Case |
|--------|-------------|----------|
| `copytoram` | Load entire system into RAM | Better performance, no media needed |
| `setkmap=de` | Set keyboard layout (de=German) | Fix keyboard input issues |
| `rootcryptpass=` | SSH password (encrypted) | Remote access security |
| `nofirewall` | Disable firewall | Allow external connections |
| `break` | Halt before root mount | Manual troubleshooting |
| `break=postmount` | Halt after root mount | Post-mount troubleshooting |
| `dostartx` | Auto-start graphical desktop | GUI workflow |
| `dovnc` | Auto-start VNC server | Remote desktop access |

---

## Boot Parameter Examples

### Basic Live Boot
```bash
# Standard boot with US keyboard
linux /sysresccd/boot/x86_64/vmlinuz setkmap=us
initrd /sysresccd/boot/x86_64/sysresccd.img
```

### Remote Access Enabled
```bash
# SSH access with encrypted password
linux /sysresccd/boot/x86_64/vmlinuz rootcryptpass=$(mkpasswd -m sha-512 MySecurePassword) nofirewall setkmap=us
initrd /sysresccd/boot/x86_64/sysresccd.img
```

### Full RAM Load with Persistence
```bash
# Load into RAM with persistent storage on mounted drive
linux /sysresccd/boot/x86_64/vmlinuz copytoram cow_label=USBSTORAGE setkmap=us
initrd /sysresccd/boot/x86_64/sysresccd.img
```

### Troubleshooting Mode
```bash
# Halt for manual debugging
linux /sysresccd/boot/x86_64/vmlinuz break nomodeset
initrd /sysresccd/boot/x86_64/sysresccd.img
```

---

## Security Considerations

### ⚠️ Remote Access Security

When enabling remote access (SSH, VNC) for SystemRescue:

**1. Use encrypted passwords:**
```bash
# Always use rootcryptpass instead of rootpass
rootcryptpass=$(mkpasswd -m sha-512 YourSecurePassword)
```

**2. Restrict network access:**
```bash
# Only allow connections from specific IP ranges
nofirewall  # Disable firewall if needed, but restrict in YAML config
```

**3. VNC security:**
```bash
# Always set VNC password when starting graphical interface
vncpass=SecureVncPassword dovnc
```

### ⚠️ Boot Parameter Exposure

Boot parameters are visible in system logs and can be viewed by anyone with physical access:

- **Avoid sensitive data**: Don't include passwords directly in boot parameters
- **Use encrypted alternatives**: Prefer `rootcryptpass` over `rootpass`
- **Clear after use**: Remove temporary configurations from `/proc/cmdline`

### ⚠️ Persistence Risks

When using `cow_label` or `cow_directory` for persistence:

**1. Data isolation:**
```bash
# Changes are isolated in persistent/ directory
# Don't expose sensitive data to other users
```

**2. Backup persistent changes:**
```bash
# Before making sensitive changes, backup the persistent directory
tar -czf persistent-backup.tar.gz /persistent
```

### ⚠️ Network Boot Security

For PXE booting and network services:

- **Verify ISO authenticity**: Check checksums before network boot
- **Use encrypted connections**: Prefer HTTPS for remote configuration
- **Disable unnecessary services**: Only enable what you need in YAML config

---

## Next Steps

1.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Configure network access after booting
2.  **[Getting started](./manual_Starting_to_use_the_system_en.md)** — First steps in the SystemRescue environment
3.  **[System tools](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/System-tools.md)** — Overview of available utilities

---

*Last updated: 2026-11-04*
