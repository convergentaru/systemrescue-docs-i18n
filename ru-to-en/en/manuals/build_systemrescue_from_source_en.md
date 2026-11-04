+++
title = "Building SystemRescue from Source"
draft = false
aliases = ["/Sysresccd-manual_ru_Building_SystemRescue_from_Source"]
+++

## Building SystemRescue Itself

SystemRescue is based on Arch Linux and uses a modified version of archiso for creating released .iso images.

You can find all source code, build tools, and build documentation at https://gitlab.com/systemrescue/systemrescue-sources

---

## Arch Linux Packages

The build process described above imports all programs that are not developed directly as part of SystemRescue from Arch Linux package repositories. Source code and build tools for all these packages are available from Arch Linux.

It's recommended to read [Arch Build System Documentation](https://wiki.archlinux.org/title/Arch_Build_System) first.

Build instructions for all packages are stored in PKGBUILD files. They, and possibly patches, are available for download from https://gitlab.archlinux.org/archlinux/packaging/packages

Ensure that you're using the correct branch and tagged version. If you want to rebuild a package used in a specific SystemRescue version, use the package version that was current as of the SystemRescue release date. See e.g. `BUILD_ID` in `/etc/os-release` inside SystemRescue for this.

When you have the right PKGBUILD file, run `makepkg` to build. By default this will automatically download the necessary source files from upstream project. Alternatively Arch Linux publishes these sources at `https://archive.archlinux.org/repos/<date>/sources/`. Use the release date as indicated above.

---

## systemrescue-custompkg

Several packages that are included in SystemRescue are not available in binary form from official Arch Linux repositories. For example, they're packaged [Arch Linux AUR](https://aur.archlinux.org/) or packaged by SystemRescue.

PKGBUILD files, patches and documentation for these are available at https://gitlab.com/systemrescue/systemrescue-custompkg

---

## Build Requirements

### System Requirements

To build SystemRescue from source, you need:

* **Hardware:**
  * CPU: x86_64 or ARM64 (for respective architectures)
  * RAM: Minimum 2GB, recommended 4GB+
  * Disk space: Minimum 10GB free space
  * Graphics: X11 support for building ISO

* **Software:**
  * Arch Linux or compatible distribution
  * Git for cloning repositories
  * Python 3.8+ for build scripts
  * Required build tools (autoconf, automake, etc.)

### Install Build Dependencies

```bash
# Basic build dependencies
pacman -S git python pacman-base-archiso archlinux-keyring \
    dosfstools mkisofs grub2 efibootmgr squashfs-tools \
    xorg-server xorg-xinit xfce4 thunar \
    ntfs-3g cifs-utils e2fsprogs xfsprogs btrfs-progs

# Additional tools for ISO building
pacman -S xorriso genisoimage libisoburn
```

---

## Building Process Overview

### Step 1: Clone the Repository

```bash
# Clone SystemRescue sources
git clone https://gitlab.com/systemrescue/systemrescue-sources.git
cd systemrescue-sources

# Checkout desired version
git checkout tags/v9.05  # Replace with desired version
```

### Step 2: Configure Build Options

Create or edit `config.conf`:

```bash
# Basic configuration
ARCH=x86_64
ISO_NAME=sysrescue
VERSION=9.05

# Enable/disable features
FEATURES="copytoram checksum nofirewall loadsrm"

# Customization options
SETKMAP=us
DOSTARTX=true
DOVNC=false
ROOTPASS=""
```

### Step 3: Build the ISO

```bash
# Build for x86_64
make ARCH=x86_64 iso

# Build with custom configuration
make ARCH=x86_64 iso -c config.conf

# Build multiple architectures
make ARCH=x86_64 ARCH=aarch64 iso
```

### Step 4: Verify the Build

```bash
# Check ISO integrity
sha256sum sysrescue-*.iso

# Mount and verify contents
mount -o loop sysrescue-*.iso /mnt/sysrescue
ls /mnt/sysrescue/
umount /mnt/sysrescue
```

---

## Customizing SystemRescue

### Adding Custom Packages

```bash
# Add custom package to build
cp your-package.pkgbuild systemrescue-sources/packaging/custom-packages/

# Modify PKGBUILD as needed
nano systemrescue-sources/packaging/custom-packages/your-package/PKGBUILD

# Rebuild affected packages
cd systemrescue-sources
makepkg -si

# Continue with ISO build
make iso
```

### Modifying Configuration Files

```bash
# Edit default configuration
nano sysrescue-sources/sysrescue.d/100-defaults.yaml

# Add custom settings
global:
    copytoram: true
    checksum: true
    setkmap: "us"
    
autorun:
    ar_disable: false
```

### Creating Custom Scripts

```bash
# Create script in airootfs
mkdir -p sysrescue-sources/airootfs/root/custom-scripts
nano sysrescue-sources/airootfs/root/custom-scripts/my-script.sh

chmod +x sysrescue-sources/airootfs/root/custom-scripts/my-script.sh
```

---

## Build Options Reference

### Common Make Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `ARCH` | Target architecture | `x86_64`, `aarch64` |
| `ISO_NAME` | ISO filename prefix | `sysrescue` |
| `VERSION` | Version number | `9.05` |
| `SETKMAP` | Default keyboard layout | `us`, `de`, `fr` |
| `DOSTARTX` | Auto-start GUI | `true`, `false` |
| `NOFIREWALL` | Disable firewall | `true`, `false` |

### Build Targets

```bash
# Build ISO image
make iso

# Build individual packages
makepkg -si

# Clean build artifacts
make clean

# Rebuild everything
make ARCH=x86_64 iso clean

# Debug build with verbose output
make ARCH=x86_64 iso VERBOSE=1
```

---

## Troubleshooting Build Issues

### "Package not found" Errors

**Problem:** Required package missing from repository.

```bash
# Update package database
pacman -Syu

# Search for package
pacman -Ss <package-name>

# Install from AUR if not in official repos
yay -S <aur-package-name>
```

### "Makefile not found" Error

**Problem:** Build directory not initialized.

```bash
# Initialize build environment
makepkg --verifysource

# Or configure manually
./configure --prefix=/build/output
make
make install
```

### ISO Build Fails

**Problem:** ISO creation fails during build.

```bash
# Check available disk space
df -h

# Verify xorriso is installed
which xorriso

# Try with verbose output
make iso VERBOSE=1 2>&1 | tee build.log
```

---

## Best Practices

### ✅ Recommended

1. **Use tagged releases** — Build from stable versions
2. **Document changes** — Keep changelog for modifications
3. **Test on target hardware** — Verify builds work as expected
4. **Keep dependencies updated** — Regular package updates
5. **Backup build artifacts** — Preserve working configurations

### ❌ Not Recommended

1. **Building from master branch** — Unstable, may break
2. **Ignoring warnings** — Address all build warnings
3. **No version tracking** — Lose track of changes
4. **Mixing package versions** — Use consistent versions

---

## Security Considerations

### ⚠️ Build Environment Security

When building SystemRescue:

1. **Verify source signatures** — Check GPG signatures
2. **Use trusted mirrors** — Download from official sources only
3. **Audit custom packages** — Review all added software
4. **Keep build system updated** — Patch security vulnerabilities

### Signing Builds

```bash
# Generate GPG key for signing
gpg --gen-key

# Sign ISO image
gpg --detach-sign sysrescue-*.iso

# Create checksums
sha256sum sysrescue-*.iso > sysrescue-*.iso.sha256
```

---

## Quick Reference Table

| Task | Command | Notes |
|------|---------|-------|
| Clone repo | `git clone ...` | Get source code |
| Checkout version | `git checkout tags/vX.Y` | Use stable release |
| Build ISO | `make iso` | Create image |
| Add package | Copy PKGBUILD to build tree | Include custom software |
| Verify build | `sha256sum` | Check integrity |
| Mount ISO | `mount -o loop ...` | Test contents |

---

## Additional Resources

*   [SystemRescue Git Repository](https://gitlab.com/systemrescue/systemrescue-sources)
*   [Arch Build System Wiki](https://wiki.archlinux.org/title/Arch_Build_System)
*   [archiso Documentation](https://wiki.archlinux.org/title/archiso)
*   [AUR Package Search](https://aur.archlinux.org/)

---

*Last updated: 2026-11-04*
