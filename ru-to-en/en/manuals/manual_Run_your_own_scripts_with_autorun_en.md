+++
title = "Running Your Own Scripts with Autorun"
draft = false
aliases = ["/Sysresccd-manual-en_Run_your_own_scripts_with_autorun"]
+++

## Overview of Autorun

**The autorun feature** allows automatically running scripts or binary programs when the system starts. Each autorun script can handle a task. For example, you can create a backup script that backs up a database, another for cleaning the system, and so on.

By default, autorun scripts can be copied to the `autorun` directory of the ISO image or boot device, outside the compressed squashfs image, but other sources are also available: local filesystem, network partition, HTTP/HTTPS server.

---

## Configuration Methods

SystemRescue offers two methods for configuring autorun programs:

- **By name**. Programs with filenames starting with `autorun` in specified locations are loaded and executed. This is the old-style autorun configuration.
- **By YAML configuration**. A configuration file contains paths, URLs, and parameters for as many autorun programs as needed. This is the new-style autorun configuration, available since SystemRescue 9.05.

Both methods are fully supported and planned to remain available in future releases.

---

## Autorun Configuration with YAML

Syntax:

```yaml
---
autorun:
    exec:
        200:
            url: "http://web-server/adminscripts/some-script"
            parameters: 
                - "foo"
                - "bar"
            shell: false
            wait: always
            waitmode: 15
            on_error: break
        300:
            path: "rclone"
            parameters: 
                - "copy"
                - "myserver:/scripts/"
                - "/root"
            wait: on_error
            on_error: break
```

Configuration is done through entries in [YAML configuration](./manual_Configuring_SystemRescue_en.md), using `exec` entries in the `autorun` scope. Keys in the `exec` Mapping/dictionary (`200` and `300` in the example above) are names for each autorun program. Programs are executed in lexicographical order of these names.

### Options for Each exec Entry

- **url** or **path** — Each autorun program must have one `url` or `path`, but not both
  - **url** — URL for downloading the program. Supports http, https, nfs, and smb protocols, paths to devices below /dev as well.
    - When using https, server certificate is checked for validity. So ensure system clock is correct. You can [add your own certificates to trusted CA store](./manual_Configuring_SystemRescue_sysconfig_en.md) if needed.
    - nfs uses style `nfs://hostname:/path/script`
    - smb uses style `smb://hostname/share/path`. Note that only anonymous guest partitions without password are supported.
    - You can also specify path to unmounted block device, e.g., `/dev/sda1/usr/bin/script` or `/dev/disk/by-label/mylabel/usr/bin/script`.
  - **path** is used for programs on already mounted devices.
    - Absolute paths or search in $PATH are supported
    - Use `/run/archiso/bootmnt/autorun/` to access scripts in `autorun` directory on boot media.
    - This directory is also copied when using `copytoram` option, while other parts of boot media are not accessible when using `copytoram`.
  - **parameters** — List of parameters passed to the program.
  - **shell** (default: `false`) controls whether the program runs directly or through `/bin/sh` shell.
    - When using shell, all data parameters are combined and passed to shell as one string. Splitting is then done by shell.
    - Proper parameter escaping should be done by user
    - It's recommended not to use shell option unless necessary, e.g., for passing output to another program.
    - Shell script can (and usually should) run directly without this shell option. This option is about how the script is called and parameter passing, not about the language the script is written in. Shebang is recognized when running directly without shell option.
  - **wait** — Allowed values `always`, `on_error` or `never`, default `on_error`. Controls whether to wait/when to wait immediately after executing this one script.
  - **waitmode** — Allows either `key` for requiring key press, or number indicating seconds to wait. You can press key to continue before the specified number of seconds expires. Default is `30`.
  - **on_error** — Allows either `break` or `continue`, with `ar_ignorefail` controlling default. Determines what to do with following scripts after this script failed (returned non-zero status).

---

## Named Autorun

Scripts for execution are found using a list of rules (see below).
If scripts are found, entries in the `exec` Mapping/dictionary are created for them.
This allows using yaml-autorun and named autorun programs together.

### Options for Named Autorun

- **ar_source=xxx**: location where autoruns are located. This can be
  root directory of device (`/dev/sda1`), nfs partition
  (`nfs://192.168.1.1:/path/to/scripts`), samba partition (`smb://192.168.1.1/path/to/scripts`),
  or http/https directory (`http://192.168.1.1/path/to/scripts`).
  Note that this parameter contains address of folder that contains some autorun scripts. This is not full address of specific script, since script name will be added at end of source.
  
- **ar_suffixes=[0-F]**: comma-separated list of suffixes corresponding to
  autorun scripts that should be executed. For example, if you use
  `ar_suffixes=0,2,7`, then following autorun scripts will be executed:
  `autorun0`, `autorun2`, `autorun7`, and scripts with other suffixes will be
  ignored. Use `ar_suffixes=no` to disable all autorun scripts with suffix.

### Script Search Rules for Named Autorun

At startup, a list of locations is checked for autorun files. They are, in order:

- if parameter **ar_source=** was passed at boot, root directory of specified location.
- **autorun** directory located at root of boot device (since SystemRescue 9.00)
- superuser home directory (**/root**)
- directory **/usr/share/sys.autorun**

If autorun files are found in some location, they are added to the `exec` Mapping/dictionary, and following locations are ignored.

In each source directory there are two possible operating modes:

- simple: if a shell script with name `autorun` is found, it is executed
- flexible: if scripts `autorun#` (# — digit from 0 to 9 or letter from A to F) are found and either
  - boot parameter `ar_suffixes=` was not specified, or
  - value of boot parameter `ar_suffixes=` contains #

Example: with `ar_suffixes=0,1,4` only scripts `autorun0`, `autorun1` and `autorun4` will be executed if they are present. Other scripts such as `autorun2` and `autorun3` will be ignored.

A script with filename `autorun` will get name `1000-autorun` inside the `exec` Mapping/dictionary.
`autorun0` will get `1010-autorun0`, `autorun1` will be `1011-autorun1` and so on, up to `1025-autorunF`.
This numbering scheme allows controlling execution order when using yaml-autorun together with named autorun.

---

## Common Autorun Options

These options are used for controlling overall autorun behavior. They can be
specified either in boot command line or through
[YAML configuration files](./manual_Configuring_SystemRescue_en.md)

- **ar_nowait**: don't wait for key press after all autorun scripts finished (default: true)
- **ar_ignorefail**: continue executing chain of scripts even if script failed (returned non-zero status) (default: false)
- **ar_nodel**: don't delete temporary copy of autorun scripts located in `/var/autorun/tmp` after execution (default: false)
- **ar_disable**: completely disable all autorun execution (default: false)
- **ar_attempts** — number of attempts to load script from remote URL (default: 1)

---

## Installing Autorun Programs

The easiest way to place autorun programs on SystemRescue media is usually to create a writable bootable USB drive for SystemRescue. This can be done with Rufus or SystemRescue USB writer [as described here](./manual_Installing_SystemRescue_on_the_disk_en.md). You can mount the USB drive afterwards and copy autorun scripts into the `autorun` directory using a regular file manager.

If you install SystemRescue on a USB flash drive using `dd` or any other tool that performs a physical ISO image copy, it won't produce a writable filesystem, and you won't be able to copy autorun scripts to the device.

If creating a writable USB drive is not possible, you can modify the ISO image using custom build options or manual filesystem editing. See [Building SystemRescue from Source](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/scripts/Building_SystemRescue_from_Source.md) for details on adding custom packages.

For writing programs to directory `/root` or `/usr/share/sys.autorun`, you need to modify not the boot media but the filesystem that SystemRescue boots from (airootfs). This is best achieved using [SystemRescue Modules (SRM)](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/scripts/SRM) or using [persistent storage](./manual_Creating_a_backing_store_en.md).

---

## Executing Programs

A script can be any script using an installed interpreter, such as shell, Python, Perl, etc.
It must have a shebang (#!) at the beginning to specify which interpreter to use.
ELF binaries are also supported.

Autorun programs run on the main console of the system. All their output is written to the console, and input from console is passed to the program. Escape sequences are not passed, so using text UI based on curses (e.g., like `setkmap`) is not supported. For programs with user interaction, using [autoterminal](./manual_autoterminal_scripts_on_virtual_terminal_en.md) is preferred solution instead of autorun.

---

## Troubleshooting

Since SystemRescue 9.00, you can find a general autorun log file at `/var/log/sysrescue-autorun.log` to determine what went wrong. Use this log file if you have a general autorun issue, e.g., if your script was not executed. Autorun output is also written to systemd journal.

Each autorun script additionally creates its own log file and returncode file in `/var/autorun/log`. Use these logs for investigating problems affecting a specific autorun script.

For debugging, it may be useful to re-run autorun later when the system is already running. You can do this by calling `/etc/systemd/scripts/sysrescue-autorun`.

---

## Common Tasks Examples

### Task 1: Backup Database Before Boot

```yaml
---
autorun:
    exec:
        001-backup-db:
            path: "/usr/local/bin/backup-database.sh"
            parameters: ["--full", "--compress"]
            wait: always
            on_error: break
```

### Task 2: Clean Temporary Files

```yaml
---
autorun:
    exec:
        100-clean-temp:
            path: "find"
            parameters:
                - "/tmp"
                - "-mtime"
                - "+7"
                - "-delete"
            wait: never
            on_error: continue
```

### Task 3: Mount Network Drive

```yaml
---
autorun:
    exec:
        200-mount-nas:
            path: "mount"
            parameters:
                - "-t"
                - "cifs"
                - "//192.168.1.100/share"
                - "/mnt/nas"
                - "-o"
                - "username=user,password=pass"
            wait: always
            on_error: break
```

### Task 4: Remote SSH Backup with Wait

```yaml
---
autorun:
    exec:
        300-ssh-backup:
            url: "https://backup-server/scripts/backup.sh"
            parameters: ["--remote", "--verify"]
            shell: true
            wait: always
            waitmode: key
            on_error: continue
```

---

## Quick Reference Table

| Option | Default | Description | Example |
|--------|---------|-------------|---------|
| `ar_disable` | false | Disable all autorun | Disable scripts |
| `ar_nowait` | true | Don't wait after scripts | Auto-start |
| `ar_ignorefail` | false | Continue on error | Keep running |
| `ar_nodel` | false | Keep temp files | Debug mode |
| `ar_attempts` | 1 | Retry count | Load reliability |
| `wait` | on_error | When to wait | always/never |
| `waitmode` | 30 | Wait duration | key or seconds |
| `on_error` | break | Error handling | continue/break |

---

## Best Practices

### ✅ Recommended

1. **Use numbered prefixes** — `001-backup`, `100-clean`, `200-mount`
2. **Specify wait behavior** — Don't block with `wait: always` unless needed
3. **Handle errors gracefully** — Use `on_error: continue` for non-critical scripts
4. **Document scripts** — Add comments explaining purpose
5. **Test incrementally** — Make one change at a time

### ❌ Not Recommended

1. **Blocking all scripts** — Don't use `wait: always` for everything
2. **Ignoring errors** — Don't use `on_error: continue` for critical operations
3. **No error checking** — Always verify script execution
4. **Hardcoded credentials** — Store passwords securely

---

## Security Considerations

### ⚠️ Script Execution Risks

Autorun scripts run with root privileges:

1. **Verify script sources** — Don't execute untrusted scripts
2. **Check permissions** — Only allow trusted scripts to modify system files
3. **Test in safe mode** — Use `ar_disable` during testing
4. **Log all executions** — Monitor `/var/log/sysrescue-autorun.log`

### Password Storage

Avoid storing passwords in YAML:

```yaml
# ❌ Don't do this - password visible in config
parameters:
  - "-o username=user,password=secret123"

# ✅ Better - use SSH keys instead
authorized_keys:
  "user@server": "ssh-ed25519 AAAAC3NzaC1lZDI..."
```

---

## Advanced: Complex Script Chains

### Sequential Operations with Dependencies

```yaml
---
autorun:
    exec:
        001-mount-nfs:
            path: "mount"
            parameters: ["-t nfs", "server:/export", "/mnt/nfs"]
            wait: always
            on_error: break
        
        002-backup-files:
            path: "rsync"
            parameters:
                - "-av"
                - "/mnt/nfs/important/"
                - "/backup/important/"
            wait: always
            on_error: break
        
        003-unmount-nfs:
            path: "umount"
            parameters: ["/mnt/nfs"]
            wait: never
```

### Parallel Execution with Timeout

```yaml
---
autorun:
    exec:
        100-clean-temp:
            path: "find"
            parameters: ["-delete", "/tmp/*"]
            wait: never
        
        101-clear-cache:
            path: "journalctl"
            parameters: ["--vacuum-time=7d"]
            wait: never
        
        102-archive-old:
            path: "tar"
            parameters:
                - "-czf"
                - "/backup/old-data.tar.gz"
                - "/data/old/"
            wait: always
```

---

## Debugging Tools

### View Autorun Status

```bash
# Check if autorun is enabled
systemctl status sysrescue-autorun

# View recent logs
journalctl -u sysrescue-autorun --since "10 minutes ago"

# Check temporary files kept
ls -la /var/autorun/tmp/

# View return codes
cat /var/autorun/log/*returncode*
```

### Manual Autorun Execution

```bash
# Run autorun manually (already running system)
/etc/systemd/scripts/sysrescue-autorun

# Debug mode with verbose output
DEBUG=1 /etc/systemd/scripts/sysrescue-autorun

# Test single script
/path/to/script.sh --test
```

### Log Analysis

```bash
# General autorun log
cat /var/log/sysrescue-autorun.log

# Individual script logs (in /var/autorun/log/)
ls -la /var/autorun/log/
tail -f /var/autorun/log/*log*
```

---

## Next Steps

1.  **[Autoterminal](./manual_autoterminal_scripts_on_virtual_terminal_en.md)** — Interactive terminal scripts
2.  **[GUI Autostart](./manual_gui_autostart_Start_programs_on_graphical_desktop_en.md)** — Graphical program startup
3.  **[Network configuration](./manual_Network_configuration_and_programs_en.md)** — Remote script execution

---

## Additional Resources

*   [Systemd Documentation](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)
*   [rsync Manual](https://man7.org/linux/man-pages/man1/rsync.1.html)
*   [cifs-utils](https://www.kernel.org/pub/linux/utils/fs/cifs/cifs-utils/)
*   [SystemRescue SRM Modules](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/scripts/SRM)

---

*Last updated: 2026-11-04*
