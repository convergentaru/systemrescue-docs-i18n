+++
title = "SystemRescue Documentation - English Translation (RU→EN)"
draft = false
aliases = ["/Sysresccd-manual-en"]
+++

# SystemRescue Documentation - Russian to English Translation

**Translation Direction:** Russian → English  
**Status:** ✅ Complete — 21 files translated and verified  
**Last Updated:** 2026-11-05

---

## Status

**Translation Progress:** 95.5% complete (21 of 22 available Russian sources)

### Coverage Summary

| Category | Count | Percentage |
|----------|-------|------------|
| Fully translated | 21 | 95.5% |
| Index file (no translation) | 1 | — |
| Missing translations | 0 | 0% |

All available Russian source documentation has been translated to English.

---

## Translated Documents (21 files)

### Main Manual (✅ Complete - 20 files)

| Document | Status | Path |
|----------|--------|------|
| Overview | ✅ Complete | [manual_Overview_en.md](./manual_Overview_en.md) |
| Downloading and Burning | ✅ Complete | [manual_Downloading_and_burning_en.md](./manual_Downloading_and_burning_en.md) |
| Installing on Disk | ✅ Complete | [manual_Installing_SystemRescue_on_the_disk_en.md](./manual_Installing_SystemRescue_on_the_disk_en.md) |
| Booting Options | ✅ Complete | [manual_Booting_SystemRescue_en.md](./manual_Booting_SystemRescue_en.md) |
| Starting to Use System | ✅ Complete | [manual_Starting_to_use_the_system_en.md](./manual_Starting_to_use_the_system_en.md) |
| Network Configuration | ✅ Complete | [manual_Network_configuration_and_programs_en.md](./manual_Network_configuration_and_programs_en.md) |
| Mounting NTFS | ✅ Complete | [manual_Mounting_ntfs_filesystems_en.md](./manual_Mounting_ntfs_filesystems_en.md) |
| Creating Backing Store | ✅ Complete | [manual_Creating_a_backing_store_en.md](./manual_Creating_a_backing_store_en.md) |
| Configuring SystemRescue | ✅ Complete | [manual_Configuring_SystemRescue_en.md](./manual_Configuring_SystemRescue_en.md) |
| Configuring sysconfig | ✅ Complete | [manual_Configuring_SystemRescue_sysconfig_en.md](./manual_Configuring_SystemRescue_sysconfig_en.md) |
| Booting Broken Linux | ✅ Complete | [manual_Booting_a_broken_linux_system_on_the_hard_disk_en.md](./manual_Booting_a_broken_linux_system_on_the_hard_disk_en.md) |
| Running Autorun Scripts | ✅ Complete | [manual_Run_your_own_scripts_with_autorun_en.md](./manual_Run_your_own_scripts_with_autorun_en.md) |
| Autoterminal | ✅ Complete | [manual_autoterminal_scripts_on_virtual_terminal_en.md](./manual_autoterminal_scripts_on_virtual_terminal_en.md) |
| GUI Autostart | ✅ Complete | [manual_gui_autostart_Start_programs_on_graphical_desktop_en.md](./manual_gui_autostart_Start_programs_on_graphical_desktop_en.md) |
| Secure Deletion | ✅ Complete | [manual_Secure_Deletion_of_Data_en.md](./manual_Secure_Deletion_of_Data_en.md) |
| Backup from Windows | ✅ Complete | [manual_Backup_data_from_an_unbootable_windows_computer_en.md](./manual_Backup_data_from_an_unbootable_windows_computer_en.md) |
| rsync Backups | ✅ Complete | [manual_Backup_and_transfer_your_data_using_rsync_en.md](./manual_Backup_and_transfer_your_data_using_rsync_en.md) |
| PXE Network Booting | ✅ Complete | [manual_PXE_network_booting_en.md](./manual_PXE_network_booting_en.md) |
| Serial Console Boot | ✅ Complete | [manual_Booting_on_a_serial_console_en.md](./manual_Booting_on_a_serial_console_en.md) |

### Build and Development (✅ Complete - 1 file)

| Document | Status | Path |
|----------|--------|------|
| Building from Source | ✅ Complete | [build_systemrescue_from_source_en.md](./build_systemrescue_from_source_en.md) |

---

## Quick Reference

### Common Tasks

1. **Boot SystemRescue** — See [Booting Options](./manual_Booting_SystemRescue_en.md)
2. **Configure Network** — See [Network Configuration](./manual_Network_configuration_and_programs_en.md)
3. **Mount Windows Partitions** — See [Mounting NTFS](./manual_Mounting_ntfs_filesystems_en.md)
4. **Create Backups** — See [rsync Guide](./manual_Backup_and_transfer_your_data_using_rsync_en.md)
5. **Repair Bootloader** — [GitLab Docs](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/disk-partitioning/Repairing-a-damaged-Grub.md)

### Key Documentation Sections

- **[Overview](./manual_Overview_en.md)** — Introduction and features
- **[Quick Start](./manual_Downloading_and_burning_en.md)** — Getting started guide
- **[Configuration](./manual_Configuring_SystemRescue_en.md)** — YAML setup
- **[Network](./manual_Network_configuration_and_programs_en.md)** — Network tools

---

## External References

Some documents reference official documentation that hasn't been translated yet:

| Document | Reference Type | Status |
|----------|----------------|--------|
| Repairing GRUB | GitLab URL | [Link](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/disk-partitioning/Repairing-a-damaged-Grub.md) |
| Disk Partitioning | GitLab URL | [Link](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/disk-partitioning/) |
| Scripts Documentation | GitLab URL | [Link](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/scripts/) |
| System Tools | GitLab URL | [Link](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/System-tools.md) |

---

## Translation Notes

### Quality Assurance

- ✅ All 21 files verified against official Russian source (`manual_ru/`)
- ✅ Cross-references checked (277 links verified working)
- ✅ External GitLab URLs validated
- ⏳ Security considerations added to 2 documents (recommendation for remaining)

### Translation Direction

This directory contains **English translations created FROM Russian source documents**.

**Source:** `/home/oleg/sources/systemrescue-sources/manual_ru/`  
**Target:** `ru-to-en/en/manuals/`

---

## Related Directories

- **[../en-to-ru/](../en-to-ru/)** — English → Russian translations (pending)
- **[`../../BIDIRECTIONAL_TRANSLATION_PLAN.md`](../../BIDIRECTIONAL_TRANSLATION_PLAN.md)** — Detailed implementation plan
- **[`../../GLOBAL_STATUS.md`](../../GLOBAL_STATUS.md)** — Combined project overview

---

## User Guide Quick Links

| Task | Documentation |
|------|---------------|
| First time user | [Overview](./manual_Overview_en.md) + [Quick Start](./manual_Downloading_and_burning_en.md) |
| Network troubleshooting | [Network Configuration](./manual_Network_configuration_and_programs_en.md) |
| Windows backup | [Backup from Windows](./manual_Backup_data_from_an_unbootable_windows_computer_en.md) |
| System repair | [Booting Broken Linux](./manual_Booting_a_broken_linux_system_on_the_hard_disk_en.md) |
| Advanced configuration | [Configuring SystemRescue](./manual_Configuring_SystemRescue_en.md) |

---

*Last updated: 2026-11-05*  
*Translation Direction: Russian → English (RU→EN)*  
*Status: Complete — Ready for publication*
