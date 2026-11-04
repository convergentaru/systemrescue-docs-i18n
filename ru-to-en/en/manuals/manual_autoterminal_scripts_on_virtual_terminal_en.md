+++
title = "Passing Virtual Terminal to Your Program for User Interaction: autoterminal"
draft = false
aliases = ["/Sysresccd-manual-en_autoterminal_scripts_on_virtual_terminal"]
+++

## Autoterminal Overview

Linux provides several virtual terminals on the console that can be switched using ALT+F<N>. By default, 6 virtual terminals (ALT+F1 to ALT+F6) are created and used for getty with automatic login under user root. See boot option `noautologin` [boot option](./manual_Booting_SystemRescue_en.md) and option `NAutoVTs` in `/etc/systemd/logind.conf` if you want to change these default settings.

Autoterminal allows replacing getty and shell on a virtual terminal with a program of your choice.

---

## Configuration

Configuration is done exclusively through entries in [YAML configuration](./manual_Configuring_SystemRescue_en.md) using the `autoterminal` scope. Terminal device name (`tty<N>`) is the key, and the value is the command to execute:

```yaml
---
autoterminal:
    tty2: "/usr/bin/setkmap"
    tty3: "/root/myscript \"parameter one\""
    tty4: "/usr/bin/tmux"
    tty8: "/usr/bin/journalctl -f"
```

If you want to use command parameters that include spaces or quotes, you must escape them with backslash ("\\"). The command is written unchanged in systemd unit file, so see systemd documentation for details.

---

## Execution Order

Autoterminals start only after [autorun](./manual_Run_your_own_scripts_with_autorun_en.md) finishes. This allows using autorun to prepare data or load programs used by autoterminal from a remote server.

Systemd delays getty startup by default for 5 seconds (`Type=idle`). This prevents cluttering the terminal with output from other programs. Autoterminal always starts programmed programs immediately instead of this (`Type=simple`).

This allows direct interaction with user without waiting time.

---

## Possible Conflicts

By default, systemd creates 6 virtual terminals. But Linux allows up to 63 virtual terminals, although regular keyboards don't have more than 12 F-keys for access to them. You can use all virtual terminals accessible from your keyboard with autoterminal. For each configured autoterminal, default terminals configured in systemd will be masked to prevent conflicts.

When you use boot option `dostartx` [boot option](./manual_Booting_SystemRescue_en.md), you should leave `tty1` in its default configuration, otherwise `dostartx` won't work. You can still manually run `startx` on other terminals when `tty1` is used by autoterminal.

---

## Server Terminals

You can also use autoterminal on [server consoles](./manual_Booting_on_a_serial_console_en.md). Since server devices require different configuration, autoterminal must know that it's used with a server console. This is done by adding prefix `serial:` to terminal name:

```yaml
---
autoterminal:
    "serial:ttyS0": "/usr/bin/tmux"
```

Autoterminal doesn't configure the serial device in any way, e.g., setting baud rate, stop bits, etc.

This is usually done on kernel boot command line when using a serial device as a server console, e.g., like this: `console=ttyS0,115200n8`. But this can also be done separately, e.g., with [autorun](./manual_Run_your_own_scripts_with_autorun_en.md).

---

## Usage Examples

* You can run **tmux** on a virtual terminal. This will allow you (among other things) to scroll back in the terminal again, a feature that was removed from virtual terminals with kernel 5.9 due to bugs.
* You can create small **interactive text menus**. SystemRescue comes with `dialog` program and [Python library for it](https://pythondialog.sourceforge.io/), which allows easily creating such menus.
* Show latest information from **journal** on a virtual terminal to see errors and warnings. Use `journalctl -f` to enable follow mode.

---

## Common Use Cases

### Use Case 1: Interactive Keyboard Setup

```yaml
---
autoterminal:
    tty2: "/usr/bin/setkmap"
    tty3: "dialog --title=\"Keyboard Layout\" --menu \"Select keyboard layout:\" 60 15 9 \\<br/>us <br/>de <br/>fr <br/>ru <br/>es <br/>it <br/>pt <br/>fi <br/>sv"
```

### Use Case 2: Full Terminal Multiplexer

```yaml
---
autoterminal:
    tty2: "/usr/bin/tmux"
    tty3: "/usr/bin/screen -S session1"
    tty4: "/usr/bin/screen -S session2"
```

### Use Case 3: System Monitoring Dashboard

```yaml
---
autoterminal:
    tty2: "/usr/bin/nethogs"      # Network monitoring
    tty3: "/usr/bin/iostat -x 1"   # Disk I/O monitoring
    tty4: "/usr/bin/htop"          # Process monitoring
```

### Use Case 4: Log Viewer with Search

```yaml
---
autoterminal:
    tty2: "journalctl -f -p err"   # Error logs only
    tty3: "less /var/log/syslog"   # Full syslog viewer
```

### Use Case 5: Interactive Backup Tool

```yaml
---
autoterminal:
    tty2: "/root/backup-tool.sh"   # Custom backup script with menu
    tty3: "rsync --progress -av /data /backup/"  # Visual backup progress
```

---

## Advanced Configuration

### Using Parameters

```yaml
---
autoterminal:
    tty2: "/usr/bin/dialog --title=\"Network Config\" --form 0 0 1"
    tty3: "/usr/bin/screen -L -S debug -Logfile /var/log/screen.log"
    tty4: "journalctl -f --no-pager -u sshd"
```

### Combining with Autorun

```yaml
---
# autorun scope - prepare environment
autorun:
    exec:
        001-load-tools:
            url: "http://tools.example.com/nethogs.sh"
            parameters: ["--install"]
            wait: always
        
        002-setup-tty2:
            path: "/usr/bin/setkmap"
            wait: never

# autoterminal scope - start programs
autoterminal:
    tty2: "/usr/bin/nethogs"
    tty3: "dialog --title=\"Network Setup\" --menu \"Select interface:\" 10 40 5 \\<br/>eth0 <br/>eth1 <br/>wlan0"
```

### Serial Console Configuration

```yaml
---
# Configure serial terminal for server console
autoterminal:
    "serial:ttyS0": "/usr/bin/tmux -c root"
    
# Or with custom baud rate setup via autorun
autorun:
    exec:
        001-setup-serial:
            path: "stty"
            parameters:
                - "-F"
                - "/dev/ttyS0"
                - "115200"
                - "cs8"
                - "parenb"
                - "oddpar"
                - "-crtscts"
```

---

## Quick Reference Table

| Terminal | Default Program | Common Use Case |
|----------|-----------------|-----------------|
| `tty1` | getty (login) | Keep for graphical start |
| `tty2` | Custom | Interactive tools, setup menus |
| `tty3` | Custom | Monitoring (htop, iostat) |
| `tty4` | Custom | Log viewing (journalctl) |
| `tty5` | getty | Secondary login |
| `tty6` | getty | Additional terminal |
| `tty7-12` | Masked | Available for autoterminal |

---

## Common Programs for Autoterminal

### Terminal Multiplexers

| Program | Description | Example |
|---------|-------------|---------|
| `tmux` | Terminal multiplexer | `/usr/bin/tmux` |
| `screen` | Legacy multiplexer | `/usr/bin/screen -S session` |
| `zsh` | Shell with plugins | `/usr/bin/zsh` |

### Monitoring Tools

| Program | Description | Example |
|---------|-------------|---------|
| `htop` | Process viewer | `/usr/bin/htop` |
| `nethogs` | Network monitor | `/usr/bin/nethogs` |
| `iostat` | Disk I/O | `/usr/bin/iostat -x 1` |
| `iftop` | Bandwidth | `/usr/bin/iftop` |

### Interactive Tools

| Program | Description | Example |
|---------|-------------|---------|
| `dialog` | Text menus | `/usr/bin/dialog --menu ...` |
| `whiptail` | Alternative menus | `/usr/bin/whiptail ...` |
| `ncurses` | Terminal UI | `/usr/bin/ncursesw6` |

### Log Viewers

| Program | Description | Example |
|---------|-------------|---------|
| `journalctl` | systemd logs | `/usr/bin/journalctl -f -p err` |
| `less` | Text pager | `/usr/bin/less /var/log/syslog` |
| `grep` | Search tool | `/usr/bin/grep -i error /var/log/*` |

---

## Configuration Examples

### Complete Interactive Setup Menu

```yaml
---
autoterminal:
    tty2: "/root/setup-menu.sh"  # Main setup with dialog menu
    tty3: "dialog --title=\"Disk Status\" \\<br/>df -h \\<br/>blkid"
    tty4: "dialog --title=\"Network Status\" \\<br/>ip addr \\<br/>nmcli device status"
```

### Monitoring Dashboard

```yaml
---
autoterminal:
    tty2: "/usr/bin/htop"          # Processes
    tty3: "/usr/bin/nethogs"       # Network
    tty4: "/usr/bin/iostat -x 1"   # Disk I/O
    tty5: "journalctl -f --no-pager"  # Logs
```

### Backup Progress Viewer

```yaml
---
autoterminal:
    tty2: "/usr/bin/rsync --progress -av /source /dest/"
    tty3: "/usr/bin/ncdu /backup/"  # Disk usage analysis
```

---

## Best Practices

### ✅ Recommended

1. **Keep tty1 for login** — Don't override default getty on tty1
2. **Use tmux/screen** — Provides better UX with scrollback
3. **Test before deploying** — Verify terminal works as expected
4. **Document parameters** — Add comments explaining command options
5. **Handle errors gracefully** — Use try/except in scripts

### ❌ Not Recommended

1. **Blocking all terminals** — Leave some for emergency access
2. **Complex nested menus** — Keep menus simple and intuitive
3. **No error handling** — Always check script exit codes
4. **Hardcoded paths** — Use environment variables

---

## Security Considerations

### ⚠️ Terminal Access

Autoterminal programs run with root privileges:

1. **Verify program sources** — Don't execute untrusted scripts
2. **Limit terminal access** — Only enable needed terminals
3. **Monitor usage** — Check logs for suspicious activity
4. **Use least privilege** — Run as non-root when possible

### Input Validation

When accepting user input:

```yaml
# ❌ Insecure - no validation
tty2: "/usr/bin/dialog --inputbox \"Enter password:\" 10 50"

# ✅ Secure - validate input first
tty2: "/root/validate-input.sh < /dev/tty2"
```

---

## Troubleshooting

### Terminal Not Starting

**Problem:** Autoterminal program doesn't start.

**Solutions:**
1. Check systemd unit file was created:
   ```bash
   ls -la /etc/systemd/system/autoterminal@*.service
   ```
2. Verify command exists and is executable:
   ```bash
   which /usr/bin/tmux
   ls -la /root/myscript.sh
   ```
3. Check for syntax errors in command:
   ```bash
   bash -n "/path/to/command"
   ```

### Program Crashes Immediately

**Problem:** Program exits immediately after starting.

```bash
# Check if program needs specific parameters
tmux -h        # Help output
screen --help  # Help output

# Test command directly
/usr/bin/tmux
```

### Terminal Conflicts

**Problem:** Multiple programs trying to use same terminal.

```bash
# Check which processes are using terminals
ls -la /dev/tty*

# View systemd unit status
systemctl status autoterminal@tty2.service
```

### Serial Console Issues

**Problem:** Serial terminal not responding.

```bash
# Check serial device exists
ls -la /dev/ttyS*

# Verify baud rate settings
stty -F /dev/ttyS0

# Test serial communication
minicom /dev/ttyS0,115200
```

---

## Debugging Autoterminal

### View Configuration

```bash
# Check effective YAML configuration
cat /run/archiso/config/sysrescue-effective-config.json | jq '.autoterminal'

# List systemd units for autoterminal
systemctl list-units | grep autoterminal
```

### Monitor in Real-time

```bash
# Watch terminal output
watch -n 1 "ls -la /dev/tty*"

# Follow journal logs
journalctl -f -u autoterminal@tty2.service

# Check for errors
dmesg | grep -i tty
```

### Test Commands Manually

```bash
# Test if command works
/usr/bin/tmux -V        # Version check
which /usr/bin/tmux     # Path verification

# Try running with verbose output
/usr/bin/tmux -f /tmp/tmux-test.log
cat /tmp/tmux-test.log
```

---

## Advanced: Custom Terminal Programs

### Creating a Custom Setup Script

```bash
#!/bin/bash
# /root/setup-menu.sh

dialog --title "SystemRescue Setup" \
  --menu "Select operation:" 20 60 15 \
  "Check Disk Space" "df -h" \
  "View System Logs" "journalctl -f" \
  "Network Diagnostics" "ip addr && ping -c 4 8.8.8.8" \
  "Exit" ""

case $? in
    1) df -h ;;
    2) journalctl -f ;;
    3) ip addr && ping -c 4 8.8.8.8 ;;
    0) exit 0 ;;
esac
```

### Python-Based Interactive Tool

```python
#!/usr/bin/env python3
# /root/system-diagnostics.py

import subprocess
import dialog

def run_command(cmd):
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

title = "System Diagnostics"
menu_items = [
    ("Disk Usage", "df -h"),
    ("Memory Info", "free -m"),
    ("Network Status", "ip addr"),
    ("Exit", ""),
]

while True:
    choice, cmd = dialog.menu(title, menu_items)
    if not choice:
        break
    print(f"\n{cmd}\n")
    print(run_command(cmd))
```

---

## Next Steps

1.  **[GUI Autostart](./manual_gui_autostart_Start_programs_on_graphical_desktop_en.md)** — Graphical environment setup
2.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Remote terminal access
3.  **[Running scripts](./manual_Run_your_own_scripts_with_autorun_en.md)** — Complex automation

---

## Additional Resources

*   [systemd Autologin](https://www.freedesktop.org/software/systemd/man/systemd-logind.service.html)
*   [tmux Manual](https://man7.org/linux/man-pages/man1/tmux.1.html)
*   [dialog Library](https://invisible-island.net/dialog/)
*   [ncurses Documentation](https://invisible-island.net/ncurses/faq.html)

---

*Last updated: 2026-11-04*
