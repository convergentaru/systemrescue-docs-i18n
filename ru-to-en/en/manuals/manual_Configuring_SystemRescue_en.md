+++
title = "Configuring SystemRescue"
+++

## Overview

SystemRescue comes with options that allow users to change how it works. For example, there are options for managing keyboard layout, whether to load the system from memory or from boot device, whether to automatically start a graphical environment and so on. You can follow the link to see a list of [boot options](./manual_Booting_SystemRescue_en.md) supported in the command line.

These options are set in the boot command line. To change these options, you can either manually edit the boot command line from the bootloader in working mode, or make the changes permanent by editing the bootloader configuration files, which are either isolinux if you're running in BIOS mode, or Grub if you're running in UEFI mode.

---

## Configuration Features

Since SystemRescue 9.00, there's another way to configure SystemRescue. Now you can edit YAML files on the boot device for configuration. This configuration method was introduced for easily creating persistent configuration changes, and these options are effective for both BIOS and UEFI modes.

SystemRescue configuration YAML files are located in the `sysrescue.d` folder on the boot device.

SystemRescue comes with a default YAML configuration file, so it provides a good example of such a file that you can use as a starting point. You can edit it to replace options with your preferences.

---

## Scopes (Areas of Application)

These YAML configuration files support multiple scopes. The main scope is called `global` and contains general configuration entries. Almost all entries in the `global` scope have boot command line options of the same name and are documented in [Boot Options](./manual_Booting_SystemRescue_en.md).

The `autorun` scope is used to define configuration entries that are used by [autostart](./manual_Run_your_own_scripts_with_autorun_en.md) and documented there.

The `gui_autostart` scope is used to define configuration entries that are used by [gui_autostart](./manual_gui_autostart_Start_programs_on_graphical_desktop_en.md) and documented there.

The `autoterminal` scope is used to define configuration entries that are used by [autoterminal](./manual_autoterminal_scripts_on_virtual_terminal_en.md) and documented there.

The `sysconfig` scope is used to define certain aspects of system configuration. You can find more information about entries in [sysconfig](./manual_Configuring_SystemRescue_sysconfig_en.md).

---

## Configuration Entries Example

Below is an example of a valid YAML configuration file. In this example, there are entries in both `global` and `autorun` and `sysconfig` scopes. The `copytoram` option is enabled, so the system is fully copied into memory upon boot, the `checksum` option is also enabled, so the system checks its integrity upon boot, as well as the `dostartx` option for automatic graphical environment start. And the `nofirewall` and `loadsrm` options are left disabled, so the firewall won't be turned off, and SRM modules won't be loaded. The `setkmap` option is used to set a French-style keyboard layout. All these general options belong to the `global` scope.

```yaml
---
global:
    copytoram: true
    checksum: true
    nofirewall: false
    loadsrm: false
    late_load_srm: "https://example.com/myconfig.srm"
    setkmap: "fr-latin1"
    dostartx: true
    dovnc: false
    rootshell: "/bin/bash"
    rootcryptpass: "$6$Y.AolXkpG/Js2Zqx$z7J893qtB7jKn3z39ucbgvpkJ6wTrJ8N0CBVr5cJ.uXugGTMTSjMI7qsSTu4UTFGGKpGyEG/BnYNRE6oZFO4b0"
    rootpass: "MyRootPassword123"
    vncpass: "MyVncPassword456"

autorun:
    ar_disable: false
    ar_nowait: false
    ar_nodel: false
    ar_attempts: 1
    ar_ignorefail: false
    ar_suffixes: "0,1,2,3,4,5"

autoterminal:
    tty2: "/usr/bin/tmux"
    
sysconfig:
    ca-trust:
        example-ca: |
            -----BEGIN CERTIFICATE-----
            MIIDlTCCAn2gAwIBAgIUbB4K7H53E3spHfMtSb0To+Fyb3wwDQYJKoZIhvcNAQEL
            BQAwWjELMAkGA1UEBhMCWFgxFTATBgNVBAcMDERlZmF1bHQgQ2l0eTEcMBoGA1UE
            [...]
            VmbLuXNBNjfcAk1xqTb1j9dMeHDZKV4Imr0W3qfsHnWFqihxGyKJ79Qb2bL1Kquc
            vgI/6+yHyDlw
            -----END CERTIFICATE-----
```

---

## Feature Status

At this stage, only some options are supported in the configuration file. Plans to add support for more options or services in the future. You can also use this mechanism and use these YAML configuration files for configuring your own scripts that run from SystemRescue. You should create new scopes in your configuration files if you plan to do this.

These features are relatively new, so it's recommended to use the very latest version of SystemRescue to get the latest features and bug fixes related to configuration support implementation.

The way YAML configuration files are loaded changed from SystemRescue 9.03. Versions 9.00 to 9.02 implemented only partial merging, and when the `sysrescuecfg` option was used, only specified files were loaded.

---

## Merging Configuration Files and Order

To be able to configure SystemRescue using YAML configuration files, you either edit an existing YAML file located in `sysrescue.d` on the boot device, or create additional YAML files in the same location. Files must have `.yaml` extension (not `.yml`), otherwise the file will be ignored. Also make sure you follow YAML syntax correctly. Keys and values are separated by colons, not equals signs, and indentation is very important.

The system reads all `.yaml` files located in the `sysrescue.d` folder on the boot device in lexicographical order. **Files loaded later merge with previous ones, overwriting already configured individual options.** When a later loaded file contains an empty option, it's removed from the final configuration and not just overwritten with an empty value.

Files provided by SystemRescue have names starting with a number to make the order more explicit. It's recommended to follow this practice for additional files as well.

For example, if you have a file `100-defaults.yaml`:

```yaml
---
global:
    nofirewall: false
    dovnc: false
    setkmap: "fr-latin1"
```

and file `200-my-options.yaml`:

```yaml
---
global:
    nofirewall: true
    setkmap:
```

then the `nofirewall` option will be `true`, `dovnc` will be `false`, and there won't be a configuration entry for `setkmap`.

After all files in the `sysrescue.d` directory are read and merged, the system looks for the `sysrescuecfg` option in the boot command line. You can add more YAML files to load and merge with this option.

The `sysrescuecfg` option allows using HTTP and HTTPS URLs to load remote files. When using HTTPS, **certificate verification is NOT performed**, since the trusted CA database isn't set up yet during the initramfs boot stage when YAML files are evaluated.

Alternatively, you can specify absolute and relative paths. Relative paths are relative to the `sysrescue.d` directory on the boot device. If this path points to a directory, all `.yaml` files in that directory are loaded and merged in lexicographical order. This can be used, for example, for loading YAML files from different subdirectories of `sysrescue.d`. You can specify the `sysrescuecfg` option multiple times in the boot command line.

After loading and merging YAML files, boot command line options are evaluated. See [boot options](./manual_Booting_SystemRescue_en.md). Boot command line options take precedence over what's configured in YAML files.

---

## Storing YAML Configuration Files

The easiest way to place your configuration files on SystemRescue media is to create a writable bootable USB drive for SystemRescue. This can be done with Rufus or SystemRescue USB writer [as described here](./manual_Installing_SystemRescue_on_the_disk_en.md). You can mount the USB drive afterwards and copy configuration files into the `sysrescue.d` directory using a regular file manager.

If you install SystemRescue on a USB flash drive using `dd` or any other tool that performs a physical ISO image copy, it won't produce a writable filesystem, and you won't be able to edit the configuration file on the device.

If creating a writable USB drive is not possible, you can modify the ISO image using custom build options or manual filesystem editing. See [Building SystemRescue from Source](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/scripts/Building_SystemRescue_from_Source.md) for details on adding custom packages.

---

## Implementation Details

Support for options located in YAML configuration file is implemented in several places:

The main configuration processing part is implemented in the following script:
[sysrescue-configuration.lua](https://gitlab.com/systemrescue/systemrescue-sources/-/blob/main/airootfs/usr/bin/sysrescue-configuration.lua).

This script runs during the initramfs boot stage and processes YAML configuration files available on the local boot device, as well as options specified in the boot command line, and determines the "effective" configuration. It's a single JSON file saved to `/run/archiso/config/sysrescue-effective-config.json`, which contains a single definition of each supported option and allows various programs to determine what value applies for a specific option without needing to process all possible configuration sources. Several scripts such as `sysrescue-initialize` and `sysrescue-autorun` use the effective configuration file to determine what to do. Python scripts have built-in support for reading JSON files. Shell scripts can use the standard `jq` command to read values from this JSON file.

The Lua library used for reading YAML files and processing them into a JSON file imposes some limitations on supported YAML syntax and types. For example, all numeric values are converted to Lua floating-point numbers and written as such in JSON.

When adding new values, it's recommended to use list structures very carefully because there's no obvious semantics for their merging. New list values currently overwrite values at the same position when merging multiple files. Using dictionary structures that are then evaluated with their keys in lexicographical order is a preferred way instead. This allows full merging and targeted removal of previous entries.

Some [archiso hooks](https://gitlab.com/systemrescue/systemrescue-sources/-/blob/main/patches/archiso-v43-07-yaml-config.patch) read the effective configuration to determine option values such as `copytoram`, `checksum` and `loadsrm` that need to be used early in the boot process, as part of initramfs.

The sysrescue-initialize script also uses the effective configuration to determine how the system should be initialized, at a later stage of the boot process. It's split into two parts:
[sysrescue-initialize-prenet](https://gitlab.com/systemrescue/systemrescue-sources/-/blob/main/airootfs/etc/systemd/scripts/sysrescue-initialize-prenet) and
[sysrescue-initialize-whilenet](https://gitlab.com/systemrescue/systemrescue-sources/-/blob/main/airootfs/etc/systemd/scripts/sysrescue-initialize-whilenet). The first runs before network setup starts, and configures things like firewall, latter runs in parallel with network setup.

---

## Configuration File Best Practices

### ✅ Recommended

1. **Use numbered prefixes** — `100-base.yaml`, `200-custom.yaml`
2. **Keep files small** — One scope per file when possible
3. **Document your changes** — Add comments explaining why
4. **Test incrementally** — Make one change at a time

### ❌ Not Recommended

1. **Single large file** — Hard to maintain and debug
2. **Non-numeric prefixes** — Unpredictable merge order
3. **Overwriting with empty values unintentionally** — Can disable features
4. **Using .yml extension** — Files will be ignored

---

## Quick Reference: Common Configuration

### Basic Customization

```yaml
---
global:
    # Load system into RAM for better performance
    copytoram: true
    
    # Enable checksum verification
    checksum: true
    
    # Set keyboard layout
    setkmap: "us"  # or "de", "fr", "ru", etc.
    
    # Auto-start graphical environment
    dostartx: true
    
    # Disable firewall for remote access
    nofirewall: true
    
    # Load SRM modules
    loadsrm: true
```

### SSH Access Configuration

```yaml
---
global:
    # Set root password (encrypted)
    rootcryptpass: "$6$encoded_password_here"
    
    # Or set plain password (less secure)
    rootpass: "YourPassword123"
    
    # Enable VNC for remote desktop
    dovnc: true
    vncpass: "VncPassword123"
```

### Auto-Run Scripts Configuration

```yaml
---
autorun:
    # Run autorun scripts
    ar_disable: false
    
    # Don't wait for key press after script
    ar_nowait: false
    
    # Keep temporary files after execution
    ar_nodel: true
    
    # Retry failed loads 3 times
    ar_attempts: 3
    
    # Ignore script failures and continue
    ar_ignorefail: true
    
    # Run specific scripts by suffix
    ar_suffixes: "0,1,backup"
```

---

## Troubleshooting Configuration Issues

### YAML Syntax Errors

Check for common mistakes:

```bash
# Validate YAML syntax
python3 -c "import yaml; yaml.safe_load(open('sysrescue.d/test.yaml'))"

# Common errors:
# - Wrong indentation (must be 2 spaces)
# - Using tabs instead of spaces
# - Missing colons after keys
# - Using equals signs instead of colons
```

### Configuration Not Applied

1. Check file extension is `.yaml` (not `.yml`)
2. Verify file is in `sysrescue.d/` directory
3. Check merge order with `ls sysrescue.d/*.yaml`
4. View effective config: `cat /run/archiso/config/sysrescue-effective-config.json`

### Certificate Verification Warnings

HTTPS URLs don't verify certificates during boot:

```yaml
# This is expected and safe during boot
sysrescuecfg: "https://example.com/config.yaml"

# Warning in logs is normal
# Ignore certificate verification warnings at boot time
```

---

## Next Steps

1.  **[sysconfig scope](./manual_Configuring_SystemRescue_sysconfig_en.md)** — Advanced system configuration
2.  **[Running scripts](./manual_Run_your_own_scripts_with_autorun_en.md)** — Automation with YAML config
3.  **[Booting options](./manual_Booting_SystemRescue_en.md)** — Command line vs YAML comparison

---

## Additional Resources

*   [YAML Specification](https://yaml.org/spec/1.2/)
*   [jq Manual](https://stedolan.github.io/jq/manual/)
*   [SystemRescue GitHub](https://gitlab.com/systemrescue/systemrescue-sources/)
*   [Configuration Lua Script](https://gitlab.com/systemrescue/systemrescue-sources/-/blob/main/airootfs/usr/bin/sysrescue-configuration.lua)

---

*Last updated: 2026-11-04*
