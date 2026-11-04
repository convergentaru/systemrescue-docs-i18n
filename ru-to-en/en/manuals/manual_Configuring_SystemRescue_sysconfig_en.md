+++
title = "SystemRescue Configuration: sysconfig Scope"
draft = false
aliases = ["/Sysresccd-manual-en_Configuring_SystemRescue_sysconfig"]
+++

## Overview

This chapter explains options available in the `sysconfig` scope of [YAML configuration files](./manual_Configuring_SystemRescue_en.md) for SystemRescue. See there for details on where to place YAML files and how they're interpreted.

---

## Syntax Example

```yaml
---
sysconfig:
    authorized_keys:
        "user@example.com": "ssh-rsa AAAAB3NzaC1...kQoVPcn3jpgywi/twXcOw=="
        "other-user@example.com": "no-port-forwarding ssh-rsa ZDWo0UmISKEn...dAq33PUQh"
        "ca@example.org": "cert-authority,principals=\"bob,joe\" ssh-rsa AAAAB3NzaC1y...Zhk0="

    bash_history:
        100: "setkmap"
        200: "reverse_ssh support.example.com 1234"

    bookmarks:
        0100:
            title: "SystemRescue"
            url: "https://www.system-rescue.org/"
        0200:
            title: "Arch Linux Package Search"
            url: "https://archlinux.org/packages/"

    ca-trust:
        example-ca: |
            -----BEGIN CERTIFICATE-----
            MIIDlTCCAn2gAwIBAgIUbB4K7H53E3spHfMtSb0To+Fyb3wwDQYJKoZIhvcNAQEL
            BQAwWjELMAkGA1UEBhMCWFgxFTATBgNVBAcMDERlZmF1bHQgQ2l0eTEcMBoGA1UE
            [...]
            VmbLuXNBNjfcAk1xqTb1j9dMeHDZKV4Imr0W3qfsHnWFqihxGyKJ79Qb2bL1Kquc
            vgI/6+yHyDlw
            -----END CERTIFICATE-----

    hosts:
        "192.168.1.1": "example.net.lan"
        "192.168.1.10": "foo.net.lan foo"

    rclone:
        config:
            arch-linux-archive:
                type: "http"
                url: "https://archive.archlinux.org/"

            my-nas:
                type: "webdav"
                url: "https://my-nas.local"
                vendor: "other"
                user: "my-login"
                pass: "9JbSJzgcQXsnV2dkzzhBR3za1e_rqixvKKp6"

    ssh_known_hosts:
        myhost.example.org: "ssh-ed25519 AAAAC3NzaC1l...JJTO48B"
        "@cert-authority *.mydomain.org": "ssh-rsa AAAAB3NzaC1y...Zhk0="

    sysctl:
        net.ipv4.ip_forward: "1"
        net.ipv4.conf.all.arp_filter: "1"

    timezone: "Europe/Berlin"
    
    display_scale: "disabled"
```

---

## SSH authorized_keys

Allows configuring trusted public keys that are allowed for login as user `root` via SSH. These keys are added to `/root/.ssh/authorized_keys`.

This option is structured as a Mapping/dictionary with the SSH authorized_keys line's "comment" field being the key and the beginning of the entry up to the comment being the value. Since the "comment" field is often used to denote the username or email address of the owner, entries are sorted by these owner names. If the file already contains an entry with that "comment", it's not added again.

You can use additional key options as documented by sshd, adding them before the key type. See for example the `no-port-forwarding` option in the syntax above.

You can trust signatures from SSH Certificate Authorities (CA) by adding a CA public key with the `cert-authority` option. The `principals=` option allows controlling which certificate owner names ("principals", set with `-n` option when creating certificates) are allowed for login. The principals option has nothing to do with local account names (e.g., "root").

---

## Bookmarks

This option allows configuring bookmarks for the installed Firefox browser. This is implemented through Firefox's `policies.json` file.

This option is structured as two levels of Mapping/dictionaries. The first-level key is used for lexicographical sorting of entries. Each entry must have `title` and `url` keys.

When the `policies.json` file already contains an entry with that `title`, the bookmark won't be added again. Firefox has been observed removing entries with duplicate URLs.

---

## Certificate Authority (CA) Trust

If you run one or more local Certificate Authorities (CA), you can add them as trust anchors in SystemRescue by listing them in the `ca-trust` section of the YAML file. You must give each CA an individual name within the `ca-trust` section. This name will be used as a filename inside the `/etc/ca-certificates/trust-source/anchors/` directory.

Configured CAs will be added to SystemRescue's default trust anchors, which are derived from Mozilla's CA list.

These CAs are in X.509 format and are used primarily for TLS. Don't confuse with CA for SSH.

---

## Timezone

Allows configuring the timezone to use. Accepts a timezone name as defined in [IANA Time Zone Database](https://www.iana.org/time-zones), which is used by most Linux distributions. Default is "UTC".

**Available timezones:**
```yaml
timezone: "Europe/Moscow"
timezone: "America/New_York"
timezone: "Asia/Tokyo"
timezone: "UTC"
```

---

## Sysctl

Allows customizing kernel parameters through the `sysctl` interface. This option is structured as a Mapping/dictionary with the key being the sysctl variable and the value being the value to set. Using program `sysctl` pattern option is **NOT** supported, you must use explicit variable names.

It's recommended to use strings for values in YAML configuration because numeric values may be converted to floating-point numbers during configuration merging. The kernel will reject floating-point values for most variables.

**Examples:**
```yaml
# Enable IP forwarding
sysctl:
    net.ipv4.ip_forward: "1"
    
# Enable ARP filtering
sysctl:
    net.ipv4.conf.all.arp_filter: "1"
    
# Disable SYN cookies
sysctl:
    net.ipv4.tcp_syncookies: "0"
```

---

## rclone

Allows configuring [rclone](https://rclone.org/). Entries under key `sysconfig.rclone.config` are written as sections to file `/root/.config/rclone/rclone.conf`. For creating sections in the correct format, it's recommended to use the `rclone config` command for configuring rclone, then transfer data from `rclone.conf` file into YAML configuration.

Note that rclone requires all passwords to be encoded in a proprietary easily reversible scheme. SystemRescue will just copy passwords and won't encrypt or change them, so YAML configuration should contain them already in encoded format. If an attacker can read the YAML configuration, this encoding won't protect the passwords.

**Example:**
```yaml
rclone:
    config:
        my-dropbox:
            type: "dropboxweb"
            token: "your_dropbox_token_here"
            
        my-s3:
            type: "s3"
            provider: "AWS"
            access_key_id: "AKIA..."
            secret_access_key: "your_secret_key"
            region: "us-east-1"
```

---

## bash_history

Allows preconfiguring common commands in the bash shell so they can be accessed just by pressing the up arrow key or found with ctrl+r. The Mapping/dict key is used for lexicographical reverse sorting of entries. Reverse sorting means that the smallest key will be shown first when pressing the up arrow key.

By default, the `setkmap` command is inserted into bash_history with key `100` for ability to change keyboard layout without needing to type on a possibly foreign keyboard layout. This is mentioned in the welcome text that's shown upon system login. Remember that the welcome text doesn't adapt automatically when you replace `setkmap` with another command.

**Example:**
```yaml
bash_history:
    100: "setkmap us"        # Keyboard layout
    200: "reverse_ssh host"  # SSH reverse tunnel
    300: "systemctl restart sshd"  # Restart SSH
    400: "df -h"             # Check disk space
```

---

## Hosts

Allows manually configuring hostname/IP mappings that take precedence over DNS lookups through /etc/hosts file. The Mapping/dict key is the IP (IPv4 or IPv6), value — hostname and, optionally, alias names for it.

**Example:**
```yaml
hosts:
    "192.168.1.1": "router.local"
    "192.168.1.10": "server.local webserver"
    "::1": "localhost ipv6-localhost"
```

---

## ssh_known_hosts

Allows configuring known host keys when creating outgoing SSH connections. These are added to `/etc/ssh/ssh_known_hosts`.

This option is structured as a Mapping/dictionary with the SSH ssh_known_hosts line's "marker" and "hostnames" (space-separated) fields being the key and "keytype", "base64-encoded key" and "comment" being the value. If the file already contains an entry with that record, it's not added again.

You can trust signatures of host keys from SSH Certificate Authorities (CA) by adding a CA public key with marker `@cert-authority`. After this marker and space, you can use wildcards for specifying hostnames for which CA signatures are trusted, or use `*` for trusting everything signed by CA.

**Example:**
```yaml
ssh_known_hosts:
    # Specific host
    "server.example.com": "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA..."
    
    # Wildcard for subdomain
    "@cert-authority *.internal.example.com": "ssh-rsa AAAAB3NzaC1..."
    
    # Trust all from CA
    "@cert-authority *.mydomain.org": "ssh-ed25519 AAAAC3NzaC1..."
```

---

## display_scale

Everything is too small to use in Xorg/XFCE by default on very high-resolution (aka HiDPI) screens, such as a 3840x2160 resolution screen. A scaling factor can be applied manually from XFCE display settings to make everything look much bigger on such screens, but this requires manual intervention every time the user uses the system, since configuration isn't persistent. This is why a script was developed to apply the scaling factor automatically when starting the graphical environment. The script is located in `/usr/bin/configure-display` and runs automatically when Xorg starts thanks to `/etc/xdg/autostart/configure-display.desktop`. Behavior of this script is controlled by the `display_scale` value in the YAML configuration file. The default value of this option is `disabled`, which means that the script won't do anything. This is default behavior to prevent adverse effects of the script until it's well tested. You need to set this option either to `auto` or a number such as `1.5` or `2.0` for the script to actually apply the scaling factor when starting Xorg. If you use `auto`, the script will automatically determine what scaling factor should be based on screen resolution (in pixels) and size (in millimeters). This behavior is useful in case you want to use one and the same SystemRescue media with one and the same configuration on different computers that don't all have identical screen types. You can also use this option with numbers such as `1.5`, `2.0`, or `2.5` for applying a specific scaling factor. The number should be between `1.0` and `5.0`, otherwise it will be considered invalid. The script uses `xrandr --query` output for getting information about screens, and it writes logs to `/var/log/configure-display.log` in case you need to troubleshoot problems.

**Options:**
- `disabled` — No automatic scaling (default)
- `auto` — Automatically detect optimal scale based on screen DPI
- `1.5`, `2.0`, etc. — Apply specific scaling factor (1.0-5.0)

**Examples:**
```yaml
# Disable auto-scaling
display_scale: "disabled"

# Auto-detect for HiDPI screens
display_scale: "auto"

# Apply 2x scaling
display_scale: "2.0"

# Apply 1.5x scaling
display_scale: "1.5"
```

---

## Quick Reference Table

| Option | Purpose | Example |
|--------|---------|---------|
| `authorized_keys` | SSH key authentication | Add trusted SSH keys |
| `bookmarks` | Firefox bookmarks | Pre-configured browser links |
| `ca-trust` | TLS certificate trust | Add custom CA certificates |
| `timezone` | System timezone | Set regional time |
| `sysctl` | Kernel parameters | Tune network settings |
| `rclone` | Cloud storage config | Configure cloud sync |
| `bash_history` | Common commands | Quick access commands |
| `hosts` | Hostname mappings | /etc/hosts entries |
| `ssh_known_hosts` | SSH host verification | Pre-verify server keys |
| `display_scale` | HiDPI scaling | Fix small text on 4K screens |

---

## Security Considerations

### ⚠️ Password Storage

rclone passwords are stored in plaintext in YAML configuration:

```yaml
# This is visible to anyone who can read the config file
pass: "your_secret_password"
```

**Recommendations:**
1. Restrict file permissions: `chmod 600 sysrescue.d/*.yaml`
2. Don't store highly sensitive credentials
3. Use separate configuration for sensitive data
4. Encrypt YAML files if needed

### SSH Keys Security

When adding SSH keys:

```yaml
# Always include the key type and comment
"user@example.com": "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... user@example"
```

---

## Troubleshooting

### Configuration Not Applied

1. Check YAML syntax is valid
2. Verify file extension is `.yaml`
3. Check merge order with numeric prefixes
4. View effective config after boot: `cat /run/archiso/config/sysrescue-effective-config.json`

### rclone Connection Issues

```bash
# Test rclone configuration
rclone config save

# Check rclone.conf exists
ls -la /root/.config/rclone/rclone.conf

# View configuration
rclone config file /root/.config/rclone/rclone.conf
```

### Display Scaling Not Working

```bash
# Check log file
cat /var/log/configure-display.log

# Manual test
xrandr --query

# Verify script exists
ls -la /usr/bin/configure-display
```

---

## Next Steps

1.  **[Running scripts](./manual_Run_your_own_scripts_with_autorun_en.md)** — Automation with sysconfig
2.  **[Autoterminal](./manual_autoterminal_scripts_on_virtual_terminal_en.md)** — Terminal configuration
3.  **[GUI Autostart](./manual_gui_autostart_Start_programs_on_graphical_desktop_en.md)** — Graphical environment setup

---

## Additional Resources

*   [rclone Documentation](https://rclone.org/docs/)
*   [sysctl Manual](https://man7.org/linux/man-pages/man8/sysctl.8.html)
*   [Firefox Policies](https://support.mozilla.org/en-US/kb/policy-restrictions-firefox-60-and-later)
*   [CA Certificate Management](https://www.kernel.org/doc/html/latest/crypto/ca-certificates.html)

---

*Last updated: 2026-11-04*
