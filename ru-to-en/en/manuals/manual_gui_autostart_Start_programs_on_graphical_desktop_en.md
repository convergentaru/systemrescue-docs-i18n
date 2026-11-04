+++
title = "Start Programs When the Graphical Desktop Comes Up: gui_autostart"
draft = false
aliases = ["/Sysresccd-manual-en_gui_autostart_Start_programs_on_graphical_desktop"]
+++

## Overview

SystemRescue contains a X.Org graphical desktop that can be started either by using the `dostartx` or `dovnc` [boot options](./manual_Booting_SystemRescue_en.md) or running `startx` on the command line.
You can automatically start programs as soon as the graphical desktop is run.

---

## Configuration

Configuration for this is done solely through entries in the [YAML configuration](./manual_Configuring_SystemRescue_en.md)
using the `gui_autostart` scope. The keys in the map / dictionary (100 and 200 in the example) are names for each program. 
The names are used for the `.desktop` files that are created to implement this:

```yaml
---
gui_autostart:
    100:
        desktop: "/usr/share/applications/thunar.desktop"
    200:
        exec: "bash -c \"read -p 'press any key'\""
        terminal: true
```

### desktop

If there already is a `.desktop` file (see below) for the program you want to start, you can use the
`desktop` keyword to point to this file.

Using the `desktop` keyword is mutually exclusive with the `exec` keyword.

**Example:**

```yaml
gui_autostart:
    001-file-manager:
        desktop: "/usr/share/applications/thunar.desktop"
    
    002-terminal:
        desktop: "/usr/share/applications/xfce4-terminal.desktop"
    
    003-browser:
        desktop: "/usr/share/applications/firefox.desktop"
```

### exec

If there is no `.desktop` file for your program or it is missing things like the correct arguments, you can
use the `exec` keyword to execute an arbitrary command. The content is taken as-is and written to the
`Exec` key of the generated `.desktop` file. See the [documentation of the Exec key](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html#exec-variables) for details.

When using `exec` you can also set `terminal` to let the program run in a terminal window
(`Terminal` option in the `.desktop` file).

Using the `exec` keyword is mutually exclusive with the `desktop` keyword.

**Example:**

```yaml
gui_autostart:
    001-custom-tool:
        exec: "/root/my-script.sh --verbose"
        terminal: true
    
    002-simple-command:
        exec: "ls -la /mnt/windows"
        terminal: false
```

---

## Implementation and Relevant Standards

The feature is implemented by creating .desktop files in the `~/.config/autostart` directory,
as described in:

*   [Overview in the Arch Wiki](https://wiki.archlinux.org/title/Desktop_entries)
*   [Desktop Application Autostart Specification](https://specifications.freedesktop.org/autostart-spec/autostart-spec-latest.html)
*   [Desktop Entry Specification](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html)

---

## Creating Custom .desktop Files

### Method 1: Using YAML Configuration (Recommended)

```yaml
---
gui_autostart:
    001-my-tool:
        desktop: "/root/.local/share/applications/my-tool.desktop"
```

The system will create the `.desktop` file automatically.

### Method 2: Manual .desktop File Creation

Create a custom `.desktop` file manually:

```bash
# Create directory for custom applications
mkdir -p ~/.local/share/applications

# Create desktop file
cat > ~/.local/share/applications/my-custom-tool.desktop << 'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=My Custom Tool
Exec=/root/my-script.sh --verbose
Terminal=true
Categories=Utility;System;
Icon=utilities-terminal
StartupNotify=true
EOF

# Make it executable
chmod +x ~/.local/share/applications/my-custom-tool.desktop
```

---

## Common Use Cases

### Use Case 1: File Manager

```yaml
---
gui_autostart:
    001-file-manager:
        desktop: "/usr/share/applications/thunar.desktop"
    
    002-terminal:
        desktop: "/usr/share/applications/xfce4-terminal.desktop"
```

### Use Case 2: Network Tools

```yaml
---
gui_autostart:
    001-network-monitor:
        exec: "nethogs"
        terminal: true
    
    002-connection-checker:
        exec: "ping -c 3 8.8.8.8 && nmcli device status"
        terminal: true
```

### Use Case 3: Monitoring Dashboard

```yaml
---
gui_autostart:
    001-process-monitor:
        desktop: "/usr/share/applications/htop.desktop"
    
    002-memory-monitor:
        exec: "bmon"
        terminal: true
    
    003-disk-monitor:
        exec: "ncdu /backup"
        terminal: true
```

### Use Case 4: Backup Tools

```yaml
---
gui_autostart:
    001-backup-manager:
        exec: "/root/backup-manager.sh"
        terminal: true
    
    002-rsync-progress:
        exec: "rsync --progress -av /data /backup/"
        terminal: true
```

### Use Case 5: Remote Access Tools

```yaml
---
gui_autostart:
    001-vnc-viewer:
        desktop: "/usr/share/applications/realvnc-vncviewer.desktop"
    
    002-xfreerdp:
        exec: "xfreerdp /u:user /v:remote-pc /security:rdp"
        terminal: false
```

### Use Case 6: Custom Diagnostic Tool

```yaml
---
gui_autostart:
    001-diagnostics:
        exec: "/root/system-diag.sh"
        terminal: true
```

---

## Advanced Configuration

### Complex Command with Parameters

```yaml
---
gui_autostart:
    001-advanced-tool:
        exec: "bash -c \"
            echo 'Starting diagnostic tool...';
            /root/diag-check.sh --full --verbose;
            echo 'Diagnostic complete';
        \""
        terminal: true
```

### Multiple Commands Sequentially

```yaml
---
gui_autostart:
    001-step1-command:
        exec: "command1 --option1"
        terminal: false
    
    002-step2-command:
        exec: "command2 --option2"
        terminal: false
```

### Terminal with Custom Properties

```yaml
---
gui_autostart:
    001-terminal-session:
        exec: "bash -c \"exec xterm -title 'SystemRescue Session'\""
        terminal: true
```

---

## Best Practices

### ✅ Recommended

1. **Use numbered prefixes** — `001-filemanager`, `002-terminal`
2. **Prefer `.desktop` files** — Use existing desktop files when available
3. **Set appropriate categories** — Helps with application discovery
4. **Test before deploying** — Verify programs start correctly
5. **Document dependencies** — Add comments explaining requirements

### ❌ Not Recommended

1. **Blocking GUI startup** — Don't use `exec` that waits indefinitely
2. **Hardcoded credentials** — Don't store passwords in scripts
3. **No error handling** — Always check program exit status
4. **Overly complex commands** — Keep commands simple and maintainable

---

## Security Considerations

### ⚠️ Executable Permissions

When using `exec`, ensure proper permissions:

```yaml
# ❌ Insecure - no permission check
gui_autostart:
    001-tool:
        exec: "/root/untrusted-script.sh"

# ✅ Secure - verify script first
gui_autostart:
    001-verified-tool:
        exec: "/root/verified-script.sh"
```

### Password Security

Never store passwords in `.desktop` files or scripts:

```yaml
# ❌ Don't do this - password in config
gui_autostart:
    001-tool:
        exec: "myapp --password=secret123"

# ✅ Better - use environment variables or secure storage
gui_autostart:
    001-secure-app:
        exec: "myapp --config /root/.config/myapp/config.yaml"
```

---

## Troubleshooting

### Program Not Starting

**Problem:** GUI program doesn't start automatically.

**Solutions:**
1. Check `.desktop` file exists and is valid:
   ```bash
   ls -la ~/.config/autostart/*.desktop
   cat ~/.config/autostart/myapp.desktop
   ```
2. Verify desktop file syntax:
   ```bash
   desktop-file-validate ~/.config/autostart/myapp.desktop
   ```
3. Check if program is installed and executable:
   ```bash
   which thunar
   ls -la /usr/share/applications/thunar.desktop
   ```

### Wrong Desktop File Created

**Problem:** System creates wrong `.desktop` file.

```bash
# List all autostart entries
ls -la ~/.config/autostart/

# Check YAML configuration
cat /run/archiso/config/sysrescue-effective-config.json | jq '.gui_autostart'

# Remove incorrect entries
rm ~/.config/autostart/wrong-file.desktop
```

### Terminal Not Working

**Problem:** `terminal: true` doesn't work as expected.

```bash
# Check if terminal emulator is installed
which xfce4-terminal
which xterm

# Test terminal manually
xfce4-terminal &
```

---

## Quick Reference Table

| Option | Description | Example |
|--------|-------------|---------|
| `desktop` | Use existing .desktop file | `/usr/share/applications/thunar.desktop` |
| `exec` | Run arbitrary command | `/root/my-script.sh --verbose` |
| `terminal` | Run in terminal window | `true` or `false` |

---

## Common Desktop Files

| Application | Desktop File Path | Category |
|-------------|-------------------|----------|
| Thunar (File Manager) | `/usr/share/applications/thunar.desktop` | Utility;FileBrowser |
| xfce4-terminal | `/usr/share/applications/xfce4-terminal.desktop` | TerminalEmulator |
| Firefox | `/usr/share/applications/firefox.desktop` | Network;WebBrowser |
| htop | `/usr/share/applications/htop.desktop` | MonitorSystemResources |
| xdg-open | `/usr/share/applications/xdg-open.desktop` | Application |

---

## Creating a Complete Custom .desktop File

```bash
#!/bin/bash
# Script to create custom desktop file

APP_NAME="My Backup Tool"
EXEC_PATH="/root/backup-tool.sh"
ICON_PATH="/usr/share/icons/hicolor/scalable/apps/system-rescue.svg"
COMMENT="Automatically backup important data on graphical desktop start"

cat > ~/.local/share/applications/${APP_NAME}.desktop << EOF
[Desktop Entry]
Version=1.0
Type=Application
Name=${APP_NAME}
Comment=${COMMENT}
Exec=${EXEC_PATH} --gui-mode
Terminal=true
Categories=Utility;Backup;System;
Icon=${ICON_PATH}
StartupNotify=true
EOF

chmod +x ~/.local/share/applications/${APP_NAME}.desktop

echo "Desktop file created successfully!"
```

---

## Debugging Autostart Issues

### View Effective Configuration

```bash
# Check YAML config
cat /run/archiso/config/sysrescue-effective-config.json | jq '.gui_autostart'

# List created .desktop files
ls -la ~/.config/autostart/

# Show desktop file contents
for f in ~/.config/autostart/*.desktop; do
    echo "=== $f ==="
    cat "$f"
    echo
done
```

### Test Autostart Manually

```bash
# Create test .desktop file
cat > /tmp/test-autostart.desktop << 'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=Test Autostart
Exec=/bin/echo "Autostart works!"
Terminal=true
EOF

# Run desktop file directly
xdg-open /tmp/test-autostart.desktop
```

### Monitor Startup Process

```bash
# Watch for X session start
watch -n 1 "ps aux | grep -E 'Xorg|startx'"

# Check autostart entries after boot
systemctl list-units --type=service | grep autostart
```

---

## Next Steps

1.  **[Autoterminal](./manual_autoterminal_scripts_on_virtual_terminal_en.md)** — Terminal-based programs
2.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Remote desktop access
3.  **[Running scripts](./manual_Run_your_own_scripts_with_autorun_en.md)** — Automation integration

---

## Additional Resources

*   [Desktop Entry Specification](https://specifications.freedesktop.org/desktop-entry-spec/)
*   [Autostart Specification](https://specifications.freedesktop.org/autostart-spec/)
*   [XFCE Autostart Guide](https://wiki.archlinux.org/title/XFCE#Autostart)
*   [GNOME Autostart](https://help.gnome.org/users/gnome-help/stable/starting-applications.html.en)

---

*Last updated: 2026-11-04*
