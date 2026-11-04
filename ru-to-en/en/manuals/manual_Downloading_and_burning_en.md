+++
title = "Downloading and Burning the Image"
draft = false
aliases = ["/Sysresccd-manual_en_Downloading_and_burning"]
+++

## Downloading

SystemRescue is provided as an ISO image that needs to be written to CD/DVD to create a bootable medium. SystemRescue cannot run as part of your existing operating system.

You can download the ISO image using your favorite browser. If you encounter issues, download the file using [wget](https://www.gnu.org/software/wget/), which is often installed by default on Linux, or use the [Windows version of wget](http://gnuwin32.sourceforge.net/packages/wget.htm).

Downloading with wget is simple. You just need to run the following command in the terminal (cmd.exe on Windows):

```bash
wget -c <iso-file-url>
```

After downloading the file, verify that there were no errors by comparing the checksum with the one listed on the download page. You can run `sha256sum` on the ISO file to get the checksum of your local copy and ensure the file wasn't corrupted.

### Verify Download Integrity

**Linux:**
```bash
sha256sum systemrescue-x.y.z.iso
# Compare with official checksum from https://www.system-rescue.org/
```

**Windows (PowerShell):**
```powershell
Get-FileHash systemrescue-x.y.z.iso -Algorithm SHA256
```

---

## Burning the Image

You can burn the ISO image using most disc burning programs. On Windows, you can right-click on the ISO image in Explorer to find a context menu offering to burn the ISO image to a DVD drive.

On Linux, you can use graphical programs such as k3b, xfburn, brasero, or command-line tools like wodim or xorriso.

### Burning with wodim (Simple Method)

First, enter `wodim --devices` to get your device identifier. Then run the following to actually burn the ISO image:

```bash
wodim dev=/dev/scXX -v systemrescue-x.y.z.iso
```

For example:

```bash
wodim dev=/dev/sr0 speed=8 -v systemrescue-x.y.z.iso
```

### Burning with xorriso (Recommended)

`xorriso` is more modern and reliable than `wodim`:

```bash
xorriso -as mkisofs \
    -b isolinux/isolinux.bin \
    -c isolinux/boot.cat \
    -no-emul-boot \
    -boot-load-size 4 \
    -boot_offset 32768 \
    -o systemrescue-x.y.z.iso \
    systemrescue-x.y.z.iso
```

### Using GUI Tools

**Linux:**
- **K3B**: Open K3B → File → Burn disc image
- **Brasero**: Import Disc Image → Select ISO → Burn
- **Xfburn**: Drag and drop ISO file onto the window

**Windows:**
- Right-click ISO file → "Burn disc image"
- Or use software like ImgBurn, Nero, or CDBurnerXP

---

## Installing to USB Flash Drive

Installing to a USB flash drive is today often a good alternative to burning to optical media. See [here](./manual_Installing_SystemRescue_on_the_disk_en.md) for more details.

### Quick Comparison: CD vs USB

| Feature | CD/DVD | USB Flash |
|---------|--------|-----------|
| Speed | Slower (4-8x) | Fast (10-20x) |
| Reusability | Write-once or rewritable | Rewritable many times |
| Portability | Requires DVD drive | Works on most modern PCs |
| Durability | Fragile optical media | Durable, no moving parts |
| Boot priority | Often lower than USB | Usually first boot device |

---

## Troubleshooting

### Download Issues
- **Incomplete download**: Use `-c` flag with wget to continue interrupted downloads
- **Checksum mismatch**: Re-download the file; it may be corrupted
- **Slow speed**: Check your internet connection or try mirror sites

### Burning Issues
- **"Device not found"**: Check if burner is detected: `dmesg | grep -i cdrom`
- **Write failed**: Try lowering write speed (add `speed=4` to wodim command)
- **ISO too large for DVD**: Use a USB drive instead (supports larger sizes)

---

## Recommended Tools

| Tool | Platform | Type | Recommendation |
|------|----------|------|----------------|
| wget | Linux/Windows | CLI | ⭐⭐⭐ Standard download |
| curl | Linux/Windows | CLI | ⭐⭐ Alternative downloader |
| K3B | Linux | GUI | ⭐⭐⭐ Recommended burner |
| Brasero | Linux | GUI | ⭐⭐ Good alternative |
| ImgBurn | Windows | GUI | ⭐⭐⭐ Reliable burner |
| xorriso | Linux/Unix | CLI | ⭐⭐⭐ Modern standard |

---

## Next Steps

1.  **[Installing on USB](./manual_Installing_SystemRescue_on_the_disk_en.md)** — Install to USB flash drive for faster booting
2.  **[Booting options](./manual_Booting_SystemRescue_en.md)** — Learn all available boot parameters
3.  **[Getting started](./manual_Starting_to_use_the_system_en.md)** — First steps after booting

---

*Last updated: 2026-11-04*
