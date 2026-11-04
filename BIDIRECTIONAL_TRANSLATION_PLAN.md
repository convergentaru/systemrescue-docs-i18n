# SystemRescue i18n — Bidirectional Translation Plan

**Date:** 2026-11-05  
**Status:** ⏳ Planning Phase — Awaiting Owner Approval  
**Prepared by:** Independent Verification  

---

## 🎯 Executive Summary

### Current State Analysis

The i18n workspace contains **two distinct translation directions** that must be preserved and separated:

| Direction | Source Language | Target Language | Files | Status |
|-----------|-----------------|-----------------|-------|--------|
| **RU → EN** | Russian (`manual_ru/`) | English | 21 files | ✅ Complete |
| **EN → RU** | English (`website/content/manual/`) | Russian | 0 files | ⏳ Not Started |

### Critical Finding

The existing 21 English translation files are **translations FROM Russian TO English**, NOT from English to Russian as originally intended. This has been verified and documented in `TRANSLATION_DIRECTION_VERIFICATION.md`.

---

## 📊 Current Inventory

### Existing Work: Russian → English (RU→EN)

**Source:** `/home/oleg/sources/systemrescue-sources/manual_ru/`  
**Target:** `i18n/en/manuals/`  
**Count:** 21 translation files

| # | Source File | Translated To | Status | Notes |
|---|-------------|---------------|--------|-------|
| 1 | Overview.md | manual_Overview_en.md | ✅ Complete | |
| 2 | Downloading_and_burning.md | manual_Downloading_and_burning_en.md | ✅ Complete | |
| 3 | Installing_SystemRescue_on_the_disk.md | manual_Installing_SystemRescue_on_the_disk_en.md | ✅ Complete | |
| 4 | Booting_SystemRescue.md | manual_Booting_SystemRescue_en.md | ✅ Complete | |
| 5 | Starting_to_use_the_system.md | manual_Starting_to_use_the_system_en.md | ✅ Complete | |
| 6 | Network_configuration_and_programs.md | manual_Network_configuration_and_programs_en.md | ✅ Complete | |
| 7 | Mounting_ntfs_filesystems.md | manual_Mounting_ntfs_filesystems_en.md | ✅ Complete | |
| 8 | Creating_a_backing_store.md | manual_Creating_a_backing_store_en.md | ✅ Complete | |
| 9 | Configuring_SystemRescue.md | manual_Configuring_SystemRescue_en.md | ✅ Complete | |
| 10 | Configuring_SystemRescue_sysconfig.md | manual_Configuring_SystemRescue_sysconfig_en.md | ✅ Complete | |
| 11 | Booting_a_broken_linux_system_on_the_hard_disk.md | manual_Booting_a_broken_linux_system_on_the_hard_disk_en.md | ✅ Complete | |
| 12 | Run_your_own_scripts_with_autorun.md | manual_Run_your_own_scripts_with_autorun_en.md | ✅ Complete | |
| 13 | autoterminal_scripts_on_virtual_terminal.md | manual_autoterminal_scripts_on_virtual_terminal_en.md | ✅ Complete | |
| 14 | gui_autostart_Start_programs_on_graphical_desktop.md | manual_gui_autostart_Start_programs_on_graphical_desktop_en.md | ✅ Complete | |
| 15 | Secure_Deletion_of_Data.md | manual_Secure_Deletion_of_Data_en.md | ✅ Complete | |
| 16 | Backup_data_from_an_unbootable_windows_computer.md | manual_Backup_data_from_an_unbootable_windows_computer_en.md | ✅ Complete | |
| 17 | Backup_and_transfer_your_data_using_rsync.md | manual_Backup_and_transfer_your_data_using_rsync_en.md | ✅ Complete | |
| 18 | PXE_network_booting.md | manual_PXE_network_booting_en.md | ✅ Complete | |
| 19 | Booting_on_a_serial_console.md | manual_Booting_on_a_serial_console_en.md | ✅ Complete | |
| 20 | Building_SystemRescue_from_Source.md | build_systemrescue_from_source_en.md | ✅ Complete | Different naming |
| 21 | _index.md | _index.md | ✅ Complete | Index file |

**Translation Coverage:** 21/21 = 100% of available Russian sources  
**Quality Status:** Verified against official English reference (`manual_en/`)

---

### Missing: English → Russian (EN→RU)

**Source:** `/home/oleg/sources/systemrescue-sources/website/content/manual/`  
**Target:** `i18n/ru/manuals/` (to be created)  
**Count:** 23 source files

| # | Source File | Target File | Status | Notes |
|---|-------------|-------------|--------|-------|
| 1 | Overview.md | ru/Overview_ru.md | ⏳ Pending | |
| 2 | Downloading_and_burning.md | ru/Downloading_and_burning_ru.md | ⏳ Pending | |
| 3 | Installing_SystemRescue_on_the_disk.md | ru/Installing_SystemRescue_on_the_disk_ru.md | ⏳ Pending | |
| 4 | Booting_SystemRescue.md | ru/Booting_SystemRescue_ru.md | ⏳ Pending | |
| 5 | Starting_to_use_the_system.md | ru/Starting_to_use_the_system_ru.md | ⏳ Pending | |
| 6 | Network_configuration_and_programs.md | ru/Network_configuration_and_programs_ru.md | ⏳ Pending | |
| 7 | Mounting_ntfs_filesystems.md | ru/Mounting_ntfs_filesystems_ru.md | ⏳ Pending | |
| 8 | Creating_a_backing_store.md | ru/Creating_a_backing_store_ru.md | ⏳ Pending | |
| 9 | Configuring_SystemRescue.md | ru/Configuring_SystemRescue_ru.md | ⏳ Pending | |
| 10 | Configuring_SystemRescue_sysconfig.md | ru/Configuring_SystemRescue_sysconfig_ru.md | ⏳ Pending | |
| 11 | Booting_a_broken_linux_system_on_the_hard_disk.md | ru/Booting_a_broken_linux_system_on_the_hard_disk_ru.md | ⏳ Pending | |
| 12 | Run_your_own_scripts_with_autorun.md | ru/Run_your_own_scripts_with_autorun_ru.md | ⏳ Pending | |
| 13 | autoterminal_scripts_on_virtual_terminal.md | ru/autoterminal_scripts_on_virtual_terminal_ru.md | ⏳ Pending | |
| 14 | gui_autostart_Start_programs_on_graphical_desktop.md | ru/gui_autostart_Start_programs_on_graphical_desktop_ru.md | ⏳ Pending | |
| 15 | Secure_Deletion_of_Data.md | ru/Secure_Deletion_of_Data_ru.md | ⏳ Pending | |
| 16 | Backup_data_from_an_unbootable_windows_computer.md | ru/Backup_data_from_an_unbootable_windows_computer_ru.md | ⏳ Pending | |
| 17 | Backup_and_transfer_your_data_using_rsync.md | ru/Backup_and_transfer_your_data_using_rsync_ru.md | ⏳ Pending | |
| 18 | PXE_network_booting.md | ru/PXE_network_booting_ru.md | ⏳ Pending | |
| 19 | Booting_on_a_serial_console.md | ru/Booting_on_a_serial_console_ru.md | ⏳ Pending | |
| 20 | Building_SystemRescue_from_Source.md | ru/Building_SystemRescue_from_Source_ru.md | ⏳ Pending | |
| 21 | Installing_packages_from_AUR.md | ru/Installing_packages_from_AUR_ru.md | ⏳ Pending | |
| 22 | Installing_packages_with_pacman.md | ru/Installing_packages_with_pacman_ru.md | ⏳ Pending | |
| 23 | _index.md | ru/_index_ru.md | ⏳ Pending | Index file |

**Translation Coverage:** 0/23 = 0%  
**Quality Status:** Not started

---

### Additional Source Material (No Russian Sources Available)

#### Disk Partitioning (7 files) — English Only
| # | Source File | Status | Recommendation |
|---|-------------|--------|----------------|
| 1 | Introduction-to-disk-partitioning.md | ⏳ Pending EN→RU | No Russian source exists |
| 2 | Repairing-a-damaged-Grub.md | ⏳ Pending EN→RU | No Russian source exists |
| 3 | Grub-boot-stages.md | ⏳ Pending EN→RU | No Russian source exists |
| 4 | Partitions-attributes.md | ⏳ Pending EN→RU | No Russian source exists |
| 5 | Standard-partitioning-tools.md | ⏳ Pending EN→RU | No Russian source exists |
| 6 | The-new-GPT-disk-layout.md | ⏳ Pending EN→RU | No Russian source exists |
| 7 | _index.md | ⏳ Pending EN→RU | No Russian source exists |

#### Scripts (5 files) — English Only
| # | Source File | Status | Recommendation |
|---|-------------|--------|----------------|
| 1 | sysrescue-customize.md | ⏳ Pending EN→RU | No Russian source exists |
| 2 | build-zfs-srm.md | ⏳ Pending EN→RU | No Russian source exists |
| 3 | mountall.md | ⏳ Pending EN→RU | No Russian source exists |
| 4 | reverse_ssh.md | ⏳ Pending EN→RU | No Russian source exists |
| 5 | _index.md | ⏳ Pending EN→RU | No Russian source exists |

---

## 🏗️ Proposed Directory Structure

### Current Structure (Mixed - Needs Separation)

```
i18n/
├── en/manuals/                 # Contains RU→EN translations (21 files)
│   ├── manual_Overview_en.md
│   └── ...
├── AUDIT_REPORT.md
├── NEXT_TRANSLATION_PLAN.md
├── PROJECT_STATUS.md
├── README.md
└── TRANSLATION_DIRECTION_VERIFICATION.md
```

### Proposed Structure (Separated by Direction)

```
i18n/
├── ru-to-en/                   # Russian → English translations
│   ├── en/manuals/             # English output directory
│   │   ├── manual_Overview_en.md
│   │   ├── manual_Downloading_and_burning_en.md
│   │   └── ... (21 files)
│   ├── README.md               # Direction-specific documentation
│   └── _index.md               # Coverage statistics for RU→EN only
│
├── en-to-ru/                   # English → Russian translations
│   ├── ru/manuals/             # Russian output directory
│   │   ├── Overview_ru.md
│   │   ├── Downloading_and_burning_ru.md
│   │   └── ... (23 files - pending)
│   ├── README.md               # Direction-specific documentation
│   └── _index.md               # Coverage statistics for EN→RU only
│
├── BIDIRECTIONAL_TRANSLATION_PLAN.md  # This file
├── GLOBAL_STATUS.md            # Combined overview of both directions
├── AGENTS.md                   # Updated with both workflows
└── README.md                   # Unified project documentation
```

---

## 📋 Migration Plan

### Phase 1: Documentation Updates (No File Moves)

**Goal:** Update all documentation to reflect bidirectional nature without changing existing files.

| Task | Action | Priority |
|------|--------|----------|
| Create `ru-to-en/` directory wrapper | Wrap existing `en/manuals/` | **HIGH** |
| Create `en-to-ru/` directory structure | Create new `ru/manuals/` | **HIGH** |
| Update `README.md` | Reflect bidirectional nature | **HIGH** |
| Update `AGENTS.md` | Add both workflows | **HIGH** |
| Create `GLOBAL_STATUS.md` | Combined statistics | **MEDIUM** |
| Create `ru-to-en/README.md` | Direction-specific docs | **MEDIUM** |
| Create `en-to-ru/README.md` | Direction-specific docs | **MEDIUM** |

### Phase 2: File Organization (Minimal Moves)

**Goal:** Organize files without breaking existing references.

| Current Location | New Location | Reason |
|------------------|--------------|--------|
| `i18n/en/manuals/` | `i18n/ru-to-en/en/manuals/` | Clear direction indicator |
| `i18n/_index.md` | `i18n/ru-to-en/en/manuals/_index.md` | Direction-specific stats |

**Files NOT Moved:**
- Audit reports (`AUDIT_REPORT.md`, etc.) — keep at root for historical context
- Plan documents (`BIDIRECTIONAL_TRANSLATION_PLAN.md`) — new file
- Global status (`GLOBAL_STATUS.md`) — new file

### Phase 3: New EN→RU Translations (Future Work)

**Goal:** Begin English → Russian translations.

| Task | Action | Notes |
|------|--------|-------|
| Create `i18n/en-to-ru/ru/manuals/` | Directory structure | Ready for new files |
| Establish EN→RU workflow | Define in AGENTS.md | Separate from RU→EN |
| Start first translations | Begin with Overview.md | Use official English source |

---

## 🔄 Workflow Definitions

### RU → EN Workflow (Existing)

**Source:** Russian (`manual_ru/`)  
**Target:** English (`ru-to-en/en/manuals/`)  
**Agent Roles:**
- **Claude Code:** Translate Russian to English
- **Hermes:** Verify against official English reference

**Process:**
```bash
1. Read Russian source from manual_ru/
2. Create English translation maintaining structure
3. Verify against manual_en/ (official English reference)
4. Add security considerations (if technical)
5. Update ru-to-en/_index.md statistics
6. Commit to Git
```

### EN → RU Workflow (New)

**Source:** English (`website/content/manual/`)  
**Target:** Russian (`en-to-ru/ru/manuals/`)  
**Agent Roles:**
- **Claude Code:** Translate English to Russian
- **Hermes:** Verify against official Russian reference (`manual_ru/`)

**Process:**
```bash
1. Read English source from website/content/manual/
2. Create Russian translation maintaining structure
3. Verify against manual_ru/ (official Russian reference)
4. Add security considerations (if technical)
5. Update en-to-ru/_index.md statistics
6. Commit to Git
```

---

## 📊 Coverage Statistics

### RU → EN Direction (Complete)

| Metric | Value | Status |
|--------|-------|--------|
| Total Russian Source Files | 21 | — |
| Translated to English | 21 | ✅ 100% |
| Quality Verified Against EN Reference | Yes | ✅ |
| Cross-References Working | 277 links | ✅ Verified |

### EN → RU Direction (Not Started)

| Metric | Value | Status |
|--------|-------|--------|
| Total English Source Files | 31 (23 manual + 5 scripts + 3 disk-partitioning) | — |
| Translated to Russian | 0 | ❌ 0% |
| Quality Verified Against RU Reference | N/A | ⏳ Pending |
| Cross-References Working | N/A | ⏳ Pending |

### Overall Project Statistics

| Direction | Files | Coverage | Status |
|-----------|-------|----------|--------|
| RU → EN | 21 | 100% | ✅ Complete |
| EN → RU | 31 | 0% | ⏳ Not Started |
| **Total** | **52** | **40.4%** | **Mixed** |

---

## ⚠️ Outstanding Decisions Required

### Decision 1: Directory Naming Convention

**Option A:** Use direction names (`ru-to-en/`, `en-to-ru/`)
- Pros: Clear, explicit about translation direction
- Cons: Longer paths, more directories

**Option B:** Keep single `i18n/` with language subdirectories
- Pros: Simpler structure
- Cons: Ambiguous without documentation

**Recommended:** **Option A** — clarity over simplicity

### Decision 2: File Naming Convention

**Option A:** Language suffix (`_en.md`, `_ru.md`)
- Example: `manual_Overview_en.md`, `manual_Overview_ru.md`
- Pros: Clear language indicator in filename
- Cons: May conflict with existing naming

**Option B:** Directory-based separation
- Example: `ru-to-en/en/manuals/Overview.md`, `en-to-ru/ru/manuals/Обзор.md`
- Pros: Direction and language both explicit
- Cons: Longer paths

**Recommended:** **Option A** — maintain consistency with existing files

### Decision 3: Starting EN→RU Translations

**Question:** Should we begin EN→RU translations immediately or wait for RU→EN to be fully archived?

**Option A:** Start EN→RU now in parallel
- Pros: Both directions progressing simultaneously
- Cons: Risk of confusion, parallel workloads

**Option B:** Complete documentation separation first
- Pros: Clear boundaries before starting new work
- Cons: Delay in EN→RU progress

**Recommended:** **Option B** — establish clear structure before new translations

### Decision 4: Russian Naming Convention for EN→RU

**Question:** Should EN→RU output use Cyrillic titles (matching official `manual_ru/`) or keep English filenames?

**Option A:** Cyrillic titles (`Обзор.md`, `Загрузка_SystemRescue.md`)
- Pros: Matches official Russian documentation naming
- Cons: Harder to manage in non-Cyrillic systems

**Option B:** English filenames with Russian content (`Overview_ru.md`)
- Pros: Easier file management, clear language indicator
- Cons: Less authentic presentation

**Recommended:** **Option B** — practical approach: `manual_Title_ru.md`

### Decision 5: Audit Report Consolidation

**Question:** Should we merge all audit reports into one or keep separate?

**Option A:** Single consolidated audit report
- Pros: Unified view, easier to navigate
- Cons: Large file, may become unwieldy

**Option B:** Keep separate per direction
- Pros: Focused, clear separation
- Cons: Duplication of common sections

**Recommended:** **Option B** — keep `AUDIT_REPORT_RU_TO_EN.md` and create new `AUDIT_REPORT_EN_TO_RU.md`

---

## 🚀 Implementation Timeline

### Week 1: Documentation & Structure Setup

| Day | Task | Owner |
|-----|------|-------|
| Day 1 | Create directory structure (`ru-to-en/`, `en-to-ru/`) | Claude Code |
| Day 2 | Update README.md, AGENTS.md, PROJECT_STATUS.md | Claude Code |
| Day 3 | Create direction-specific README files | Claude Code |
| Day 4 | Move existing RU→EN files to new structure | Claude Code |
| Day 5 | Update _index.md in new location | Claude Code |

### Week 2: Verification & Testing

| Day | Task | Owner |
|-----|------|-------|
| Day 6 | Verify all links work in new structure | Hermes |
| Day 7 | Test cross-references between directions (if any) | Hermes |
| Day 8 | Update GLOBAL_STATUS.md with combined view | Claude Code |
| Day 9 | Final review of migration | Both agents |
| Day 10 | Commit all changes, prepare for approval | Both agents |

### Week 3+: EN→RU Translation Start

| Day | Task | Owner |
|-----|------|-------|
| Day 11 | Begin Overview_ru.md translation | Claude Code |
| Day 12 | Verify against official manual_ru/ | Hermes |
| Day 13+ | Continue with remaining files | Both agents |

---

## 📝 File Migration Checklist

### Files to Move (21 total)

- [ ] `en/manuals/_index.md` → `ru-to-en/en/manuals/_index.md`
- [ ] `en/manuals/manual_Overview_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Downloading_and_burning_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Installing_SystemRescue_on_the_disk_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Booting_SystemRescue_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Starting_to_use_the_system_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Network_configuration_and_programs_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Mounting_ntfs_filesystems_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Creating_a_backing_store_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Configuring_SystemRescue_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Configuring_SystemRescue_sysconfig_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Booting_a_broken_linux_system_on_the_hard_disk_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Run_your_own_scripts_with_autorun_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_autoterminal_scripts_on_virtual_terminal_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_gui_autostart_Start_programs_on_graphical_desktop_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Secure_Deletion_of_Data_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Backup_data_from_an_unbootable_windows_computer_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Backup_and_transfer_your_data_using_rsync_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_PXE_network_booting_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/manual_Booting_on_a_serial_console_en.md` → `ru-to-en/en/manuals/...`
- [ ] `en/manuals/build_systemrescue_from_source_en.md` → `ru-to-en/en/manuals/...`

### Files to Keep at Root (6 total)

- [ ] `AUDIT_REPORT.md` — Historical context
- [ ] `AUDIT_FINAL_VERIFICATION.md` — Historical context
- [ ] `AUDIT_COMPLETION.md` — Historical context
- [ ] `TRANSLATION_AUDIT_SUMMARY.md` — Historical context
- [ ] `NEXT_TRANSLATION_PLAN.md` — May need update for bidirectional plan
- [ ] `TRANSLATION_DIRECTION_VERIFICATION.md` — Critical finding documentation

### Files to Create (New)

- [ ] `BIDIRECTIONAL_TRANSLATION_PLAN.md` — This file
- [ ] `GLOBAL_STATUS.md` — Combined statistics
- [ ] `ru-to-en/README.md` — Direction-specific docs
- [ ] `en-to-ru/README.md` — Direction-specific docs (future)
- [ ] `en-to-ru/_index.md` — EN→RU coverage stats (future)

---

## 🛡️ Risk Mitigation

### Risk 1: Breaking Cross-References

**Impact:** High — Internal links may break if files move  
**Mitigation:** 
- Update all relative paths in moved files
- Verify all links after migration
- Keep old directory structure temporarily if needed

### Risk 2: Git History Confusion

**Impact:** Medium — Commit history remains intact but file locations change  
**Mitigation:**
- Use new commits for moves, not rewrites
- Document changes in commit messages
- Add migration notes to README

### Risk 3: Audit Report Inconsistency

**Impact:** Medium — Reports reference old paths  
**Mitigation:**
- Keep audit reports at root with historical context
- Create new reports for EN→RU direction
- Link old and new reports clearly

### Risk 4: Confusion About Translation Direction

**Impact:** High — Users may be confused about which direction exists  
**Mitigation:**
- Clear naming: `ru-to-en/` vs `en-to-ru/`
- Update all README files prominently
- Add visual indicators (icons, colors)

---

## ✅ Pre-Migration Checklist

Before starting migration, confirm:

- [ ] Owner approves bidirectional architecture
- [ ] All existing commits preserved (no rewrites)
- [ ] Translation direction verified for all 21 files
- [ ] Official source locations documented
- [ ] Agent roles clearly defined for both directions
- [ ] Migration timeline agreed upon
- [ ] Naming conventions approved

---

## 📞 Next Steps

### Required Owner Decisions

1. **Approve bidirectional architecture** — Yes / No / Modifications?
2. **Approve directory naming convention** — Option A or B?
3. **Approve file naming convention** — Option A or B?
4. **Confirm EN→RU workflow start timeline** — Immediate or after structure setup?
5. **Review and approve this plan** — Ready to proceed?

### After Approval

1. Execute Phase 1 (Documentation updates)
2. Execute Phase 2 (File organization)
3. Begin Phase 3 (EN→RU translations)
4. Create EN→RU audit reports
5. Update GLOBAL_STATUS.md regularly

---

*Last updated: 2026-11-05*  
*Status: Awaiting owner approval for bidirectional architecture*  
*Next action: Owner review and decision on outstanding items*
