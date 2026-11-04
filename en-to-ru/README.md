# English → Russian Translation Directory

**Translation Direction:** English → Russian  
**Status:** ⏳ Pending (0 files)  
**Last Updated:** 2026-11-05  

---

## Overview

This directory will contain **Russian translations** of official SystemRescue documentation, translated **FROM English TO Russian**.

### Source Language
- **English** (`/home/oleg/sources/systemrescue-sources/manual_ru/`)

### Target Language
- **Russian** (this directory)

---

## File Inventory (Pending - 22 files to be created)

### Main Manual (21 files)

| # | Russian Source File | Target English File | Status |
|---|---------------------|---------------------|--------|
| 1 | Overview.md | en/Overview_en.md | ⏳ Pending |
| 2 | Downloading_and_burning.md | en/Downloading_and_burning_en.md | ⏳ Pending |
| 3 | Installing_SystemRescue_on_the_disk.md | en/Installing_SystemRescue_on_the_disk_en.md | ⏳ Pending |
| 4 | Booting_SystemRescue.md | en/Booting_SystemRescue_en.md | ⏳ Pending |
| 5 | Starting_to_use_the_system.md | en/Starting_to_use_the_system_en.md | ⏳ Pending |
| 6 | Network_configuration_and_programs.md | en/Network_configuration_and_programs_en.md | ⏳ Pending |
| 7 | Mounting_ntfs_filesystems.md | en/Mounting_ntfs_filesystems_en.md | ⏳ Pending |
| 8 | Creating_a_backing_store.md | en/Creating_a_backing_store_en.md | ⏳ Pending |
| 9 | Configuring_SystemRescue.md | en/Configuring_SystemRescue_en.md | ⏳ Pending |
| 10 | Configuring_SystemRescue_sysconfig.md | en/Configuring_SystemRescue_sysconfig_en.md | ⏳ Pending |
| 11 | Booting_a_broken_linux_system_on_the_hard_disk.md | en/Booting_a_broken_linux_system_on_the_hard_disk_en.md | ⏳ Pending |
| 12 | Run_your_own_scripts_with_autorun.md | en/Run_your_own_scripts_with_autorun_en.md | ⏳ Pending |
| 13 | autoterminal_scripts_on_virtual_terminal.md | en/autoterminal_scripts_on_virtual_terminal_en.md | ⏳ Pending |
| 14 | gui_autostart_Start_programs_on_graphical_desktop.md | en/gui_autostart_Start_programs_on_graphical_desktop_en.md | ⏳ Pending |
| 15 | Secure_Deletion_of_Data.md | en/Secure_Deletion_of_Data_en.md | ⏳ Pending |
| 16 | Backup_data_from_an_unbootable_windows_computer.md | en/Backup_data_from_an_unbootable_windows_computer_en.md | ⏳ Pending |
| 17 | Backup_and_transfer_your_data_using_rsync.md | en/Backup_and_transfer_your_data_using_rsync_en.md | ⏳ Pending |
| 18 | PXE_network_booting.md | en/PXE_network_booting_en.md | ⏳ Pending |
| 19 | Booting_on_a_serial_console.md | en/Booting_on_a_serial_console_en.md | ⏳ Pending |
| 20 | Building_SystemRescue_from_Source.md | en/build_systemrescue_from_source_en.md | ⏳ Pending |
| 21 | _index.md | en/_index_en.md | ⏳ Index |

### Scripts (5 files) — No Russian Source Exists

**Note:** These documents have no corresponding Russian source in `manual_ru/`. They will need to be translated from English sources without direct comparison.

| # | English Source File | Target Russian File | Status |
|---|---------------------|---------------------|--------|
| 1 | Installing_packages_from_AUR.md | ru/Installing_packages_from_AUR_ru.md | ⏳ Pending (no source) |
| 2 | Installing_packages_with_pacman.md | ru/Installing_packages_with_pacman_ru.md | ⏳ Pending |
| 3 | sysrescue-customize.md | ru/sysrescue-customize_ru.md | ⏳ Pending (no source) |
| 4 | build-zfs-srm.md | ru/build-zfs-srm_ru.md | ⏳ Pending (no source) |
| 5 | mountall.md | ru/mountall_ru.md | ⏳ Pending (no source) |
| 6 | reverse_ssh.md | ru/reverse_ssh_ru.md | ⏳ Pending (no source) |
| 7 | _index_scripts_en.md | ru/_index_scripts_ru.md | ⏳ Index |

---

## Translation Coverage

| Metric | Value |
|--------|-------|
| Total Russian Source Files Available | 22 files |
| Translated to English | 21 files (in `ru-to-en/`) |
| Translation Coverage | **95.5%** |
| Quality Verified | Yes |

### Priority Classification

| Priority | Count | Notes |
|----------|-------|-------|
| High (Has Russian Source Reference) | 20 | Can verify against `manual_ru/` |
| Medium (No Russian Source - Scripts) | 2 | Create from English sources |
| Low (Disk Partitioning) | 7 | No source available, create from scratch |

---

## Quality Verification

When translations are created:

1. **Verify against official English source** (`website/content/manual/`)
2. **Cross-reference with existing RU→EN translations** where available
3. **Add security considerations** to technical documentation
4. **Check internal Markdown links** for correctness
5. **Validate GitLab URLs** point to correct documentation

---

## Working Rules for This Direction

### For Translators (Claude Code)

1. Read Russian source file from `manual_ru/` directory
2. Create English translation maintaining structure
3. Verify against official English reference (`manual_en/`)
4. Add security considerations to technical docs
5. Update `_index.md` statistics after each translation

### For Auditors (Hermes)

1. Verify translations against official sources
2. Cross-check with existing RU→EN translations (where applicable)
3. Check cross-reference integrity
4. Validate GitLab URLs are working
5. Document any discrepancies found

---

## Source Repository

**Official SystemRescue Sources:**  
https://gitlab.com/systemrescue/systemrescue-sources

**Russian Documentation Location:**  
`/home/oleg/sources/systemrescue-sources/manual_ru/`

**English Reference (for verification):**  
`/home/oleg/sources/systemrescue-sources/manual_en/`

---

## Related Directories

- **[`../ru-to-en/`](../ru-to-en/)** — Russian → English translations (Complete)
- **[`../../BIDIRECTIONAL_TRANSLATION_PLAN.md`](../../BIDIRECTIONAL_TRANSLATION_PLAN.md)** — Detailed implementation plan
- **[`../../GLOBAL_STATUS.md`](../../GLOBAL_STATUS.md)** — Combined project overview

---

*Last updated: 2026-11-05*  
*Translation Direction: English → Russian (EN→RU)*  
*Status: Pending — Ready for translations to begin*
