+++
title = "Booting on a Serial Console"
draft = false
aliases = ["/Sysresccd-manual_ru_Booting_on_a_serial_console"]
+++

## Serial Console

Serial consoles instead of regular VGA or HDMI/DP output video card are common on embedded systems, but also used for remote control on servers.

To use a serial console you need to know baud rate and parameters, as well as internal port number that is used. SystemRescue uses by default the most common set, namely first serial port (**ttyS0** or COM1:) with **speed 115200, 8 bits, no parity, 1 stop bit**.

---

## Bootloader

Both bootloaders (syslinux and grub) allow using serial console fully parallel to regular graphical console without prioritizing one over the other. Thus, serial console is always activated with parameters set in configuration files (see below).

---

## Kernel Booting

Linux kernel also allows using multiple consoles in parallel, they all configured with boot command line option `console=`. But unlike bootloaders, they have priority, and only the one with highest priority gets boot messages and can be used in initramfs phase.

Thus, serial console cannot always activate in parallel, but instead has special boot menu entry.

---

## Changing Parameters

When standard SystemRescue parameters don't fit your system, you should adapt them in configuration files before booting. You should adapt these files on SystemRescue ISO image:

**`/boot/grub/grubrescd.cfg`**:

```bash
# enable serial console with common settings (ttyS0, 115200 Baud, 8n1)
# this works parallel to regular console
serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1
terminal_input --append serial
terminal_output --append serial

[...]

menuentry "Boot SystemRescue with serial console (ttyS0,115200n8)" {
        set gfxpayload=keep
        linux /sysresccd/boot/x86_64/vmlinuz archisobasedir=sysresccd archisolabel=RESCUE905 iomem=relaxed console=tty0 console=ttyS0,115200n8
        initrd /sysresccd/boot/intel_ucode.img /sysresccd/boot/amd_ucode.img /sysresccd/boot/x86_64/sysresccd.img
}
```

**`/sysresccd/boot/syslinux/sysresccd_head.cfg`**:

```bash
SERIAL 0 115200
```

**`/sysresccd/boot/syslinux/sysresccd_sys.cfg`**:

```bash
LABEL sysresccd-serial
TEXT HELP
Use a serial console.
ENDTEXT
MENU LABEL Boot SystemRescue with serial console (ttyS0,115200n8)
LINUX boot/x86_64/vmlinuz
INITRD boot/intel_ucode.img,boot/amd_ucode.img,boot/x86_64/sysresccd.img
APPEND archisobasedir=sysresccd archisolabel=RESCUE905 iomem=relaxed console=tty0 console=ttyS0,115200n8
```

To change these files you can either use [sysrescue-customize](/scripts/sysrescue-customize/) for modifying ISO image, or automate these changes, or [write SystemRescue to USB stick](./manual_Installing_SystemRescue_on_the_disk_en.md) in such a way that creates writable image, e.g. Rufus or sysrescue-usbwriter, and then make changes using text editor.

---

## Serial Console Parameters Explained

### Standard Parameters (Most Common)

```
ttyS0,115200n8
```

| Parameter | Value | Description |
|-----------|-------|-------------|
| Port | `ttyS0` | First serial port (COM1) |
| Speed | `115200` | Baud rate (bits per second) |
| Data bits | `8` | 8 data bits |
| Parity | `n` | No parity |
| Stop bits | `1` | 1 stop bit |

### Alternative Serial Port Configurations

```bash
# Different baud rates
console=ttyS0,9600n8       # Legacy equipment (slow)
console=ttyS0,57600n8      # Medium speed
console=ttyS0,115200n8     # Standard (recommended)
console=ttyS0,230400n8     # Fast
console=ttyS0,460800n8     # Very fast

# Different ports
console=ttyS1,115200n8     # Second serial port (COM2)
console=ttyS2,115200n8     # Third serial port

# Multiple consoles
console=tty0,ttyS0,115200n8  # Both video and serial
```

---

## Hardware Requirements

### Serial Port Types

| Type | Connection | Common Use |
|------|------------|------------|
| DB9 (Male) | RS-232 | Terminal adapters |
| DB9 (Female) | RS-232 | Server ports |
| RJ45 | USB-to-serial | Modern laptops |
| GPIO header | UART | Embedded systems |

### Terminal Adapter Setup

For connecting laptop to server serial console:

```bash
# Install necessary tools
apt-get install screen minicom

# Configure terminal adapter (example for USB-to-serial)
lsusb  # Identify adapter

# Load appropriate driver
modprobe ftdi_sio
# or
modprobe cp210x
# or
modprobe pl2303
```

---

## Usage Examples

### Example 1: Basic Serial Console Connection

Using `screen`:

```bash
# Connect to serial console (replace with actual port and speed)
screen /dev/ttyUSB0 115200

# Or with specific terminal settings
screen -S sysrescue /dev/ttyUSB0 115200 8n1
```

Using `minicom`:

```bash
# Start minicom with configuration file
minicom -f sysrescue.conf

# Or start without config (first time prompts for setup)
minicom -s /dev/ttyUSB0
```

### Example 2: Serial Console with Autologin

```bash
# Boot SystemRescue with serial console and auto-login
linux ... console=ttyS0,115200n8 noautologin

# Then login via serial
screen /dev/ttyUSB0 115200
root@sysresccd login: root
Password: [enter password]
```

### Example 3: Serial Console for Kernel Debugging

```bash
# Boot with debug options
linux ... console=ttyS0,115200n8 debug earlylog=serial

# View kernel messages in real-time
journalctl -f --boot=0
```

---

## Common Issues and Solutions

### "Serial port not found"

**Problem:** Serial device doesn't exist or isn't accessible.

**Solutions:**
1. Check if serial port is detected:
   ```bash
   dmesg | grep -i serial
   ls /dev/ttyS*
   ```
2. Verify kernel module loaded:
   ```bash
   lsmod | grep 8250
   modprobe 8250_serial
   ```
3. Check if port is in use:
   ```bash
   fuser -v /dev/ttyS0
   lsof /dev/ttyS0
   ```

### Wrong Baud Rate

**Problem:** Communication fails due to incorrect speed.

```bash
# Test different baud rates
screen /dev/ttyUSB0 9600
# or
screen /dev/ttyUSB0 115200
# or
screen /dev/ttyUSB0 57600
```

### No Output on Serial Console

**Problem:** Serial console shows nothing.

```bash
# Check kernel is sending to serial
dmesg | tail -50

# Verify console parameter in kernel
cat /proc/cmdline

# Test with direct serial output
echo "Test message" > /dev/ttyS0
```

### Minicom/Screen Won't Connect

**Problem:** Terminal program cannot establish connection.

```bash
# Check port permissions
ls -la /dev/ttyUSB*

# Add user to dialout group
sudo usermod -aG dialout $USER

# Verify driver is loaded
dmesg | grep ttyUSB

# Test with simple echo
echo "test" > /dev/ttyUSB0
cat /dev/ttyUSB0
```

---

## Quick Reference Table

| Task | Command | Notes |
|------|---------|-------|
| Connect via screen | `screen /dev/ttyUSB0 115200` | Most common tool |
| Connect via minicom | `minicom -s /dev/ttyUSB0` | Alternative terminal |
| Check serial ports | `ls /dev/ttyS*` | List available ports |
| Load kernel module | `modprobe 8250_serial` | Enable serial support |
| View dmesg | `dmesg | grep -i serial` | Check kernel messages |

---

## Serial Console Best Practices

### ✅ Recommended

1. **Use standard baud rate** — 115200 is widely supported
2. **Document your setup** — Record port and speed settings
3. **Test connection before production** — Verify serial works
4. **Use proper terminal emulation** — 8n1 is standard
5. **Check hardware compatibility** — Ensure UART support

### ❌ Not Recommended

1. **Using wrong baud rate** — Most common cause of failures
2. **No terminal settings** — Always specify 8N1
3. **Ignoring port permissions** — May prevent access
4. **Mixing multiple console types** — Can cause conflicts

---

## Security Considerations

### ⚠️ Serial Console Access

Serial console provides root access:

1. **Enable password authentication** — Don't use unauthenticated serial
2. **Restrict physical access** — Limit who can connect serial cable
3. **Monitor serial traffic** — Detect unauthorized access
4. **Use encryption for remote serial** — Secure shell over serial

### Serial Password Setup

```bash
# Set root password (if not already set)
passwd

# Configure SSH for serial sessions
nano /etc/ssh/sshd_config
PermitRootLogin yes
PasswordAuthentication yes
```

---

## Troubleshooting Guide

### No Boot Messages on Serial

**Problem:** Cannot see kernel boot messages.

**Steps:**
1. Verify serial console enabled in bootloader:
   ```bash
   # GRUB configuration
   grep -i serial /boot/grub2/grub.cfg
   
   # Syslinux configuration
   cat /sysresccd/boot/syslinux/sysresccd_head.cfg
   ```
2. Check kernel parameters include console=ttyS0:
   ```bash
   cat /proc/cmdline
   ```
3. Verify serial driver loaded:
   ```bash
   dmesg | grep -i ttyS
   ```

### Slow Serial Communication

**Problem:** Data transmission is very slow.

```bash
# Check for errors
dmesg | grep -i error

# Verify cable quality
# Replace with known-good serial cable

# Test at different speeds
screen /dev/ttyUSB0 57600
```

### Multiple Serial Devices

**Problem:** Confusion between multiple serial ports.

```bash
# List all serial devices
ls -la /dev/ttyS* /dev/ttyUSB* /dev/ttyACM*

# Identify each device
for port in /dev/ttyS* /dev/ttyUSB*; do
    echo "=== $port ==="
    cat "$port" 2>&1 | head -5
done
```

---

## Advanced: Custom Serial Configuration

### GRUB Serial Configuration

```bash
# Enable serial console in GRUB
cat >> /etc/default/grub << 'EOF'
GRUB_TERMINAL=serial
GRUB_SERIAL_COMMAND="serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1"
GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8"
EOF

# Update GRUB configuration
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### Syslinux Serial Configuration

```bash
# Edit sysresccd_head.cfg
cat > /sysresccd/boot/syslinux/sysresccd_head.cfg << 'EOF'
SERIAL 0 115200
TIMEOUT 50
DEFAULT menu.c32
EOF

# Edit sysresccd_sys.cfg to add serial menu entry
```

### Kernel Command Line Options

```bash
# Full serial console configuration
linux /vmlinuz \
    root=/dev/sda2 \
    console=tty0 \
    console=ttyS0,115200n8 \
    console=ttyS1,115200n8 \
    earlylog=serial \
    debug \
    ip=dhcp \
    archisobasedir=sysrescue

# Explanation of options:
# console=tty0     - Primary VGA console
# console=ttyS0    - First serial port (primary for messages)
# earlylog=serial  - Output early kernel messages to serial
# debug            - Enable verbose kernel debugging
```

---

## Next Steps

1.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Remote access via network
2.  **[Booting options](./manual_Booting_SystemRescue_en.md)** — Command line parameters
3.  **[Secure deletion](./manual_Secure_Deletion_of_Data_en.md)** — Data protection

---

## Additional Resources

*   [GRUB Serial Console](https://www.gnu.org/software/grub/manual/grub/html_node/Serial-Console.html)
*   [Syslinux Serial Support](http://www.hackers-delight.de/syslinux/)
*   [Linux Console Parameters](https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html)
*   [screen Manual](https://man7.org/linux/man-pages/man1/screen.1.html)
*   [minicom Documentation](http://www.minicom.org/z/minicom-manual-en.pdf)

---

*Last updated: 2026-11-04*
