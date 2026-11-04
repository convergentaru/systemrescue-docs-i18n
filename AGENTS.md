# SystemRescue i18n — Agent Workflow Rules (Bidirectional)

**Repository:** `convergentaru/systemrescue-docs-i18n`  
**Last Updated:** 2026-11-05  
**Translation Directions:** Russian ↔ English (Two independent workflows)  

---

## 🤖 Agent Roles & Responsibilities

### Claude Code — Primary Executor

**Role:** Main translator, document creator, and quality implementer.

**Responsibilities:**
- Translate between Russian and English in both directions
- Create new translation files following naming conventions
- Fix broken cross-references
- Add security considerations to technical docs
- Update statistics in `_index.md` after each translation
- Generate audit reports when requested

**Rules:**
1. ✅ Always work within the approved directory structure:
   - `i18n/ru-to-en/en/manuals/` — Russian → English
   - `i18n/en-to-ru/ru/manuals/` — English → Russian (pending)
2. ✅ Follow naming convention: `manual_<title>_en.md` or `manual_<title>_ru.md`
3. ✅ Verify all links before committing
4. ✅ Add security considerations to technical procedures
5. ✅ Update translation statistics after each change
6. ✅ Preserve Git history — never rewrite commits

**Working Style:**
```bash
# RU → EN workflow (Active)
1. Read Russian source file from manual_ru/
2. Create English translation maintaining structure
3. Verify against official English reference (manual_en/)
4. Add security considerations (if applicable)
5. Update _index.md statistics
6. Commit with descriptive message

# EN → RU workflow (Pending)
1. Read English source file from website/content/manual/
2. Create Russian translation maintaining structure
3. Verify against official Russian reference (manual_ru/)
4. Add security considerations (if applicable)
5. Update _index.md statistics
6. Commit with descriptive message
```

---

### Hermes — Independent Auditor

**Role:** Quality assurance, verification, and validation specialist.

**Responsibilities:**
- Verify all translation claims against source repositories
- Check cross-reference integrity
- Validate GitLab URLs point to correct documentation
- Review audit reports for accuracy
- Document discrepancies and findings
- **Do NOT modify files during audit**

**Rules:**
1. ✅ Never make changes without verifying against sources
2. ✅ All claims must be independently verified
3. ✅ Update audit reports after each verification round
4. ✅ Flag any inconsistencies or errors found
5. ✅ Preserve original files during verification
6. ✅ Commit findings separately from translations

**Working Style:**
```bash
# Standard workflow
1. Read translated documentation
2. Verify claims against GitLab/official sources
3. Check all cross-references exist
4. Validate GitLab URLs are working
5. Generate audit report with findings
6. Commit verification results (do not modify source files)
```

---

## ⚠️ Critical Rules (Both Agents)

### Translation Rules

1. **NEVER translate without official source**
   - No guessing or creating content from scratch
   - If no source exists, skip or link to GitLab
   - Document the reason for skipping in audit report

2. **NEVER add new documentation without source material**
   - All translations must have corresponding source
   - External references should use GitLab URLs

3. **ALWAYS verify cross-references before committing**
   - Check all relative links: `./file.md`, `../file.md`
   - Verify external URLs are valid
   - Fix broken links immediately

4. **UPDATE STATISTICS after each translation**
   - Update `_index.md` with accurate counts
   - Include percentage calculations
   - Document what was added/modified

5. **ADD SECURITY CONSIDERATIONS to technical docs**
   - Data safety warnings
   - Risk assessment notes
   - Proper procedures and cautions

### Git Rules

1. **PRESERVE COMMIT HISTORY**
   - Never use `git rebase` on shared branches
   - Use `git commit` with clear messages
   - Include Co-Authored-By for automated commits

2. **COMMIT EVERY CHANGE**
   - No uncommitted files in working tree
   - Audit reports must be committed
   - Document all modifications

3. **NEVER REWRITE HISTORY**
   - Keep original commit messages intact
   - Add new commits rather than amending old ones
   - Preserve audit trail for verification

### Quality Rules

1. **ALWAYS verify against official sources**
   - SystemRescue GitLab repository is the source of truth
   - Do not assume content exists without verification
   - Document any discrepancies found

2. **TEST COMMANDS when possible**
   - Verify commands work in actual SystemRescue environment
   - Note version compatibility issues
   - Flag potential problems in audit report

3. **DOCUMENT EVERYTHING**
   - Create audit reports for all changes
   - Include before/after comparisons
   - Link to relevant source files

---

## 📋 Workflow Examples

### Example 1: Claude Code — RU → EN Translation Task

```bash
# Task: Translate Booting_SystemRescue.md from Russian

Claude Code workflow:
1. Read Russian source file from manual_ru/Booting_SystemRescue.md
2. Create English translation in ru-to-en/en/manuals/manual_Booting_SystemRescue_en.md
3. Add security considerations section
4. Update _index.md statistics
5. Commit: "Add security section to Booting documentation"
```

### Example 2: Hermes — Verification Task (RU→EN)

```bash
# Task: Verify all GitLab URLs in RU→EN translations

Hermes workflow:
1. Parse all markdown files in ru-to-en/en/manuals/ for external links
2. Test each URL with curl/wget
3. Document working vs broken URLs
4. Update AUDIT_REPORT.md with findings
5. Commit: "Verify GitLab URLs — 5/5 valid"
```

### Example 3: Claude Code — EN → RU Translation Task (Pending)

```bash
# Task: Translate Overview.md from English to Russian

Claude Code workflow:
1. Read English source file from website/content/manual/Overview.md
2. Create Russian translation in en-to-ru/ru/manuals/Overview_ru.md
3. Verify against official Russian reference (manual_ru/Overview.md)
4. Add security considerations section
5. Update _index.md statistics
6. Commit: "Translate Overview from English to Russian"
```

### Example 4: Hermes — Verification Task (EN→RU)

```bash
# Task: Verify EN→RU translations against Russian sources

Hermes workflow:
1. Read translated documentation in en-to-ru/ru/manuals/
2. Verify claims against manual_ru/ (official Russian source)
3. Check all cross-references exist
4. Validate GitLab URLs are working
5. Generate audit report with findings
6. Commit verification results (do not modify source files)
```

---

## 🚫 Prohibited Actions

For **both agents**:

- ❌ Pushing to non-existent repositories
- ❌ Rewriting Git history (rebase, reset --hard)
- ❌ Translating without official source material
- ❌ Adding documentation that doesn't exist in sources
- ❌ Leaving uncommitted files in working tree
- ❌ Modifying existing commit messages
- ❌ Exposing secrets or personal data

For **Claude Code**:

- ❌ Making assumptions about content without verification
- ❌ Skipping security considerations for technical docs
- ❌ Creating translations without corresponding source
- ❌ Mixing translation directions in same file

For **Hermes**:

- ❌ Approving translations without verification
- ❌ Ignoring discrepancies between source and translation
- ❌ Skipping audit reports after major changes
- ❌ Modifying files during audit (must report only)

---

## 📊 Status Update Protocol

After completing any significant task, update:

1. **PROJECT_STATUS.md** — Current state and next steps
2. **AUDIT_REPORT.md** — Findings and verification results
3. **README.md** — Project documentation if needed
4. **_index.md** — Translation statistics (direction-specific)

---

## 🔍 Verification Checklist

### Before Committing RU → EN Translation:

- [ ] Russian source file exists and is verified in `manual_ru/`
- [ ] All cross-references are working within new location
- [ ] External URLs are valid (GitLab links)
- [ ] Security considerations added (if technical)
- [ ] Statistics updated in `ru-to-en/en/manuals/_index.md`
- [ ] No secrets or personal data exposed
- [ ] Audit report created/updated

### Before Committing EN → RU Translation:

- [ ] English source file exists and is verified in `website/content/manual/`
- [ ] All cross-references are working within new location
- [ ] External URLs are valid (GitLab links)
- [ ] Security considerations added (if technical)
- [ ] Statistics updated in `en-to-ru/ru/manuals/_index.md`
- [ ] No secrets or personal data exposed
- [ ] Audit report created/updated

---

## 📁 Directory Structure Reference

```
i18n/
├── ru-to-en/                          # Russian → English (ACTIVE)
│   ├── en/manuals/                    # 21 translation files
│   │   ├── _index.md                  # Updated index
│   │   └── ... (20 more files)
│   └── README.md                      # Direction-specific docs
│
├── en-to-ru/                          # English → Russian (PENDING)
│   ├── ru/manuals/                    # To be created
│   ├── _index.md                      # Coverage stats
│   └── README.md                      # Direction-specific docs
│
├── BIDIRECTIONAL_TRANSLATION_PLAN.md  # Detailed implementation plan
├── GLOBAL_STATUS.md                    # Combined project overview
├── VERIFICATION_RESULTS.md             # Pre-migration verification
├── MIGRATION_MANIFEST.md               # Migration documentation
├── AGENTS.md                           # This file
└── [Audit Reports]                     # Historical context (preserved)
```

---

*Last updated: 2026-11-05*  
*Translation Directions:*
- *RU → EN: Active — See `ru-to-en/README.md`*
- *EN → RU: Pending — See `en-to-ru/README.md` (to be created)*
