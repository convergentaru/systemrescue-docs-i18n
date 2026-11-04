# SystemRescue Documentation i18n Project — Bidirectional Architecture

**Translation Direction:** Russian ↔ English (Two independent directions)  
**Status:** ✅ RU→EN Complete | ⏳ EN→RU Pending  
**Last Updated:** 2026-11-05  

---

## 📋 Project Overview

This repository maintains **two separate translation directions** for SystemRescue documentation:

| Direction | Source → Target | Files | Coverage | Status |
|-----------|-----------------|-------|----------|--------|
| **RU → EN** | Russian → English | 21 | 95.5% | ✅ Complete |
| **EN → RU** | English → Russian | 31 | 0% | ⏳ Not Started |

### Critical Finding (Verified)

The existing 21 English translations are **translations FROM Russian TO English**, NOT from English to Russian as originally intended. This has been verified in [`TRANSLATION_DIRECTION_VERIFICATION.md`](./TRANSLATION_DIRECTION_VERIFICATION.md).

---

## 🏗️ Directory Structure

```
i18n/
├── ru-to-en/                     # Russian → English translations (COMPLETE)
│   ├── en/
│   │   └── manuals/              # 21 English translation files
│   │       ├── _index.md         # Updated index with correct paths
│   │       ├── manual_Overview_en.md
│   │       └── ... (20 more files)
│   ├── README.md                 # Direction-specific documentation
│   └── _index.md                 # Coverage statistics for RU→EN only
│
├── en-to-ru/                     # English → Russian translations (PENDING)
│   ├── ru/
│   │   └── manuals/              # To be created
│   ├── README.md                 # Direction-specific documentation
│   └── _index.md                 # Coverage statistics for EN→RU only
│
├── BIDIRECTIONAL_TRANSLATION_PLAN.md  # Detailed implementation plan
├── GLOBAL_STATUS.md                    # Combined project overview
├── VERIFICATION_RESULTS.md             # Pre-migration verification
├── MIGRATION_MANIFEST.md               # Migration documentation
├── AGENTS.md                           # Both workflows defined
├── README.md                           # This file
└── [Audit Reports]                     # Historical context (preserved)
    ├── AUDIT_REPORT.md
    ├── AUDIT_FINAL_VERIFICATION.md
    └── ...
```

---

## 📊 Translation Statistics

### RU → EN Direction (Complete)

| Metric | Value | Status |
|--------|-------|--------|
| Total Russian Source Files | 22 (including index) | — |
| Translated to English | 21 files | ✅ 95.5% |
| Quality Verified | Yes | ✅ Against official sources |
| Cross-References Working | 277 links | ✅ Verified |

### EN → RU Direction (Not Started)

| Metric | Value | Status |
|--------|-------|--------|
| Total English Source Files | 31 (23 manual + 5 scripts + 3 disk-partitioning) | — |
| Translated to Russian | 0 files | ❌ 0% |
| Quality Verified | N/A | ⏳ Pending |

### Combined Statistics

| Direction | Files | Coverage | Status |
|-----------|-------|----------|--------|
| RU → EN | 21 | **95.5%** | ✅ Complete |
| EN → RU | 31 | **0%** | ⏳ Not Started |
| **Total** | **52** | **40.4%** | **Mixed** |

---

## 📚 Translated Documents (RU→EN)

### Main Manual (20 files)

1. **Overview** — [manual_Overview_en.md](./ru-to-en/en/manuals/manual_Overview_en.md)
2. **Downloading and Burning** — [manual_Downloading_and_burning_en.md](./ru-to-en/en/manuals/manual_Downloading_and_burning_en.md)
3. **Installing on Disk** — [manual_Installing_SystemRescue_on_the_disk_en.md](./ru-to-en/en/manuals/manual_Installing_SystemRescue_on_the_disk_en.md)
4. **Booting Options** — [manual_Booting_SystemRescue_en.md](./ru-to-en/en/manuals/manual_Booting_SystemRescue_en.md)
5. **Starting to Use System** — [manual_Starting_to_use_the_system_en.md](./ru-to-en/en/manuals/manual_Starting_to_use_the_system_en.md)
6. **Network Configuration** — [manual_Network_configuration_and_programs_en.md](./ru-to-en/en/manuals/manual_Network_configuration_and_programs_en.md)
7. **Mounting NTFS Filesystems** — [manual_Mounting_ntfs_filesystems_en.md](./ru-to-en/en/manuals/manual_Mounting_ntfs_filesystems_en.md)
8. **Creating Backing Store** — [manual_Creating_a_backing_store_en.md](./ru-to-en/en/manuals/manual_Creating_a_backing_store_en.md)
9. **Configuring SystemRescue** — [manual_Configuring_SystemRescue_en.md](./ru-to-en/en/manuals/manual_Configuring_SystemRescue_en.md)
10. **Configuring sysconfig** — [manual_Configuring_SystemRescue_sysconfig_en.md](./ru-to-en/en/manuals/manual_Configuring_SystemRescue_sysconfig_en.md)
11. **Booting Broken Linux** — [manual_Booting_a_broken_linux_system_on_the_hard_disk_en.md](./ru-to-en/en/manuals/manual_Booting_a_broken_linux_system_on_the_hard_disk_en.md)
12. **Running Autorun Scripts** — [manual_Run_your_own_scripts_with_autorun_en.md](./ru-to-en/en/manuals/manual_Run_your_own_scripts_with_autorun_en.md)
13. **Autoterminal Scripts** — [manual_autoterminal_scripts_on_virtual_terminal_en.md](./ru-to-en/en/manuals/manual_autoterminal_scripts_on_virtual_terminal_en.md)
14. **GUI Autostart** — [manual_gui_autostart_Start_programs_on_graphical_desktop_en.md](./ru-to-en/en/manuals/manual_gui_autostart_Start_programs_on_graphical_desktop_en.md)
15. **Secure Deletion of Data** — [manual_Secure_Deletion_of_Data_en.md](./ru-to-en/en/manuals/manual_Secure_Deletion_of_Data_en.md)
16. **Backup from Windows** — [manual_Backup_data_from_an_unbootable_windows_computer_en.md](./ru-to-en/en/manuals/manual_Backup_data_from_an_unbootable_windows_computer_en.md)
17. **rsync Backups** — [manual_Backup_and_transfer_your_data_using_rsync_en.md](./ru-to-en/en/manuals/manual_Backup_and_transfer_your_data_using_rsync_en.md)
18. **PXE Network Booting** — [manual_PXE_network_booting_en.md](./ru-to-en/en/manuals/manual_PXE_network_booting_en.md)
19. **Serial Console Boot** — [manual_Booting_on_a_serial_console_en.md](./ru-to-en/en/manuals/manual_Booting_on_a_serial_console_en.md)
20. **Building from Source** — [build_systemrescue_from_source_en.md](./ru-to-en/en/manuals/build_systemrescue_from_source_en.md)

---

## 🔗 External References

Some documents reference official SystemRescue documentation that hasn't been translated yet:

- [Repairing GRUB](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/disk-partitioning/Repairing-a-damaged-Grub.md)
- [Disk Partitioning](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/disk-partitioning/)
- [Scripts Documentation](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/scripts/)
- [System Tools](https://gitlab.com/systemrescue/systemrescue-sources/tree/main/manual/System-tools.md)

---

## 🛠️ Working Rules

### For Translators (Claude Code)

#### RU → EN Workflow (Active)

1. Read Russian source from `manual_ru/` directory
2. Create English translation maintaining structure
3. Verify against official English reference (`manual_en/`)
4. Add security considerations to technical docs
5. Update `_index.md` statistics after each translation

#### EN → RU Workflow (Pending)

1. Read English source from `website/content/manual/`
2. Create Russian translation maintaining structure
3. Verify against official Russian reference (`manual_ru/`)
4. Add security considerations to technical docs
5. Update `_index.md` statistics after each translation

### For Auditors (Hermes)

1. Verify all claims against source repositories
2. Check cross-reference integrity before publishing
3. Validate GitLab URLs point to correct documentation
4. Document any discrepancies in audit reports
5. Update translation statistics with accurate numbers

---

## 📝 Migration History

### Phase 1: Verification (Completed)

- ✅ Verified all 21 existing files against Russian sources
- ✅ Confirmed translation direction: Russian → English
- ✅ Generated checksums for file integrity verification

### Phase 2: Migration (Completed)

- ✅ Created `ru-to-en/` directory structure
- ✅ Moved 21 files from old location to new location
- ✅ Updated `_index.md` with correct paths
- ✅ Created direction-specific README files

### Phase 3: Documentation Updates (In Progress)

- ⏳ Update root `README.md` — Complete
- ⏳ Update `AGENTS.md` for both directions
- ⏳ Create `en-to-ru/README.md`
- ⏳ Update `GLOBAL_STATUS.md` with combined view

---

## 📞 Contact & Support

- **Official Repository:** [SystemRescue on GitLab](https://gitlab.com/systemrescue/systemrescue-sources)
- **Translation Issues:** Create GitHub issue in this repository
- **Feature Requests:** Submit to official SystemRescue project

---

*Last updated: 2026-11-05*  
*Migration Status: RU→EN Complete, EN→RU Pending*  
*Next Action: Update AGENTS.md and create en-to-ru documentation*
