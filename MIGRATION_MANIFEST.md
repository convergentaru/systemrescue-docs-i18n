# Migration Manifest

**Date:** 2026-11-05  
**Status:** Ready for Execution  

---

## Overview

This manifest documents the controlled migration of existing translation files from the monolithic `i18n/en/manuals/` structure into the bidirectional architecture with separate `ru-to-en/` and `en-to-ru/` directories.

---

## Migration Rules

1. **Preserve Git History** — No rewrites, no force pushes
2. **Preserve File Contents** — Exact byte-for-byte copies
3. **Preserve Internal Links** — Update relative paths after move
4. **Verify After Move** — Checksums, links, and references

---

## Migration Manifest (Old Path → New Path)

| # | Old Path | New Path | Status |
|---|----------|----------|--------|
| 1 | `i18n/en/manuals/build_systemrescue_from_source_en.md` | `i18n/ru-to-en/en/manuals/build_systemrescue_from_source_en.md` | Pending |
| 2 | `i18n/en/manuals/_index.md` | `i18n/ru-to-en/en/manuals/_index.md` | Pending |
| 3 | `i18n/en/manuals/manual_autoterminal_scripts_on_virtual_terminal_en.md` | `i18n/ru-to-en/en/manuals/manual_autoterminal_scripts_on_virtual_terminal_en.md` | Pending |
| 4 | `i18n/en/manuals/manual_Backup_and_transfer_your_data_using_rsync_en.md` | `i18n/ru-to-en/en/manuals/manual_Backup_and_transfer_your_data_using_rsync_en.md` | Pending |
| 5 | `i18n/en/manuals/manual_Backup_data_from_an_unbootable_windows_computer_en.md` | `i18n/ru-to-en/en/manuals/manual_Backup_data_from_an_unbootable_windows_computer_en.md` | Pending |
| 6 | `i18n/en/manuals/manual_Booting_a_broken_linux_system_on_the_hard_disk_en.md` | `i18n/ru-to-en/en/manuals/manual_Booting_a_broken_linux_system_on_the_hard_disk_en.md` | Pending |
| 7 | `i18n/en/manuals/manual_Booting_on_a_serial_console_en.md` | `i18n/ru-to-en/en/manuals/manual_Booting_on_a_serial_console_en.md` | Pending |
| 8 | `i18n/en/manuals/manual_Booting_SystemRescue_en.md` | `i18n/ru-to-en/en/manuals/manual_Booting_SystemRescue_en.md` | Pending |
| 9 | `i18n/en/manuals/manual_Configuring_SystemRescue_en.md` | `i18n/ru-to-en/en/manuals/manual_Configuring_SystemRescue_en.md` | Pending |
| 10 | `i18n/en/manuals/manual_Configuring_SystemRescue_sysconfig_en.md` | `i18n/ru-to-en/en/manuals/manual_Configuring_SystemRescue_sysconfig_en.md` | Pending |
| 11 | `i18n/en/manuals/manual_Creating_a_backing_store_en.md` | `i18n/ru-to-en/en/manuals/manual_Creating_a_backing_store_en.md` | Pending |
| 12 | `i18n/en/manuals/manual_Downloading_and_burning_en.md` | `i18n/ru-to-en/en/manuals/manual_Downloading_and_burning_en.md` | Pending |
| 13 | `i18n/en/manuals/manual_gui_autostart_Start_programs_on_graphical_desktop_en.md` | `i18n/ru-to-en/en/manuals/manual_gui_autostart_Start_programs_on_graphical_desktop_en.md` | Pending |
| 14 | `i18n/en/manuals/manual_Installing_SystemRescue_on_the_disk_en.md` | `i18n/ru-to-en/en/manuals/manual_Installing_SystemRescue_on_the_disk_en.md` | Pending |
| 15 | `i18n/en/manuals/manual_Mounting_ntfs_filesystems_en.md` | `i18n/ru-to-en/en/manuals/manual_Mounting_ntfs_filesystems_en.md` | Pending |
| 16 | `i18n/en/manuals/manual_Network_configuration_and_programs_en.md` | `i18n/ru-to-en/en/manuals/manual_Network_configuration_and_programs_en.md` | Pending |
| 17 | `i18n/en/manuals/manual_Overview_en.md` | `i18n/ru-to-en/en/manuals/manual_Overview_en.md` | Pending |
| 18 | `i18n/en/manuals/manual_PXE_network_booting_en.md` | `i18n/ru-to-en/en/manuals/manual_PXE_network_booting_en.md` | Pending |
| 19 | `i18n/en/manuals/manual_Run_your_own_scripts_with_autorun_en.md` | `i18n/ru-to-en/en/manuals/manual_Run_your_own_scripts_with_autorun_en.md` | Pending |
| 20 | `i18n/en/manuals/manual_Secure_Deletion_of_Data_en.md` | `i18n/ru-to-en/en/manuals/manual_Secure_Deletion_of_Data_en.md` | Pending |
| 21 | `i18n/en/manuals/manual_Starting_to_use_the_system_en.md` | `i18n/ru-to-en/en/manuals/manual_Starting_to_use_the_system_en.md` | Pending |

**Total Files to Migrate:** 21 (20 translations + 1 index)

---

## Post-Migration Verification Checklist

### File Integrity

- [ ] All 21 files present in new location
- [ ] File sizes unchanged (checksums verified)
- [ ] No files corrupted during move

### Link Verification

- [ ] Internal Markdown links within moved files work
- [ ] No broken references to old paths
- [ ] Cross-references between files intact

### Documentation Updates

- [ ] `i18n/ru-to-en/README.md` created
- [ ] `i18n/ru-to-en/_index.md` updated with new paths
- [ ] Root `README.md` updated to reflect bidirectional structure
- [ ] Root `AGENTS.md` updated with both workflows
- [ ] Root `GLOBAL_STATUS.md` updated with combined view

### Git Status

- [ ] All moved files staged for commit
- [ ] No unintended changes in working tree
- [ ] Audit reports preserved at root

---

## Checksums (Before Migration)

To be generated after verification:

```bash
# Generate checksums before migration
cd /home/oleg/client-rescue-usb/i18n/en/manuals
find . -name "*.md" -type f -exec sha256sum {} \; > /tmp/checksums_before.txt
cat /tmp/checksums_before.txt
```

---

*Generated: 2026-11-05*  
*Status: Ready for controlled migration*
