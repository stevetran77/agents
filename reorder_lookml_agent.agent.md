# LookML Field Group Reordering Ticket Template

## Overview
This template guides agents through reordering LookML field groups by adjusting `group_label` spacing and prefixes to control sort order in Looker's field picker.

---

## 1. DISCOVERY PHASE

### Objective
Identify all occurrences of target field groups across the codebase and document their current state.

### Steps

1. **Search for target group labels**
   ```bash
   grep -r "group_label.*{TARGET_GROUP_NAME}" --include="*.view.lkml" {WORKSPACE_PATH}
   ```

2. **Document all files and locations**
   - Record file path
   - Record line number
   - Capture full `group_label` value (including any leading spaces)
   - Identify parent `view_label`

3. **Create discovery summary**
   ```
   File: path/to/file.lkml
   - Line X: group_label: "{CURRENT_VALUE}"
     view_label: "Route (Location)"
     
   File: path/to/another_file.lkml
   - Line Y: group_label: "{CURRENT_VALUE}"
     view_label: "Route (Location)"
   ```

---

## 2. ANALYSIS PHASE

### Objective
Map the existing hierarchy and determine the correct spacing level for desired sort order.

### Steps

1. **Extract all group_label entries in the same view_label**
   - Identify unique group labels
   - Count leading spaces for each
   - Document exact spacing pattern

2. **Create spacing hierarchy map**
   ```
   Example from Route (Location):
   
   4 spaces: "    Coverage Adjustment Location"    ← NEW TARGET (top)
   4 spaces: "    Coverage Location"                ← NEW TARGET (top)
   3 spaces: "   From Movement Location"            ← Existing
   2 spaces: "  To Movement Location"               ← Existing
   0 spaces: "Port Location"                        ← Existing
   0 spaces: "Post Relay Port Location"             ← Existing
   ```

3. **Determine target spacing**
   - What position should target groups occupy?
   - What spacing is needed? (typically 1 space above highest existing, or match existing high-level spacing)
   - Example: If max spacing is 3 spaces, use 4 spaces to sort above

4. **Document strategy**
   ```
   REORDER STRATEGY:
   - Current Coverage Location spacing: 1 space
   - Current max spacing in view: 3 spaces (From Movement Location)
   - Target spacing: 4 spaces (to sort above all others)
   - Result order:
     1. Coverage Adjustment Location (4 spaces)
     2. Coverage Location (4 spaces)
     3. From Movement Location (3 spaces)
     4. To Movement Location (2 spaces)
     5. Others (0 spaces)
   ```

---

## 3. PLANNING PHASE

### Objective
Create a detailed change plan with before/after examples.

### Steps

1. **List all affected fields**
   ```
   FILE: dmc_location_r.view.lkml
   VIEW: dmc_location_coverage
   - 18 dimensions with group_label: " Coverage Location"
   
   VIEW: dmc_location_cvrg_adj
   - 17 dimensions with group_label: " Coverage Adjustment Location"
   
   FILE: dms_dmt_pck.view.lkml
   - cvr_loc_cd: group_label: " Coverage Location"
   - cvrg_loc_cd: group_label: " Coverage Adjustment Location"
   
   TOTAL CHANGES: 38 occurrences across 2 files
   ```

2. **Create before/after examples**
   ```
   BEFORE:
   dimension: loc_nm {
     view_label: "Route (Location)"
     group_label: " Coverage Location"
     group_item_label: "02. Name"
     label: "Coverage Location Name"
   }
   
   AFTER:
   dimension: loc_nm {
     view_label: "Route (Location)"
     group_label: "    Coverage Location"  ← 4 spaces instead of 1
     group_item_label: "02. Name"
     label: "Coverage Location Name"
   }
   ```

3. **Document replacement pairs**
   ```
   REPLACEMENT MAP:
   
   FILE: dmc_location_r.view.lkml
   OLD: group_label: " Coverage Location"
   NEW: group_label: "    Coverage Location"
   COUNT: 18
   
   OLD: group_label: " Coverage Adjustment Location"
   NEW: group_label: "    Coverage Adjustment Location"
   COUNT: 17
   
   FILE: dms_dmt_pck.view.lkml
   OLD: group_label: " Coverage Location"
   NEW: group_label: "    Coverage Location"
   COUNT: 1
   
   OLD: group_label: " Coverage Adjustment Location"
   NEW: group_label: "    Coverage Adjustment Location"
   COUNT: 1
   ```

---

## 4. IMPLEMENTATION PHASE

### Objective
Execute all changes safely and efficiently.

### Steps

1. **Prepare replacement array**
   - Include 3-5 lines of context BEFORE and AFTER the target text
   - Ensure each oldString is unique (no duplicates across files)
   - Use multi_replace_string_in_file for batch operations

2. **Example extraction** (from file context)
   ```
   OLD STRING:
   "  dimension: loc_nm {
      view_label: \"Route (Location)\"
      group_label: \" Coverage Location\"
      group_item_label: \"02. Name\"
      label: \"Coverage Location Name\""
   
   NEW STRING:
   "  dimension: loc_nm {
      view_label: \"Route (Location)\"
      group_label: \"    Coverage Location\"
      group_item_label: \"02. Name\"
      label: \"Coverage Location Name\""
   ```

3. **Execute batch replacements**
   - Use multi_replace_string_in_file tool
   - Verify succeeds without errors
   - Expected result: "The following files were successfully edited"

4. **Verify results**
   - Spot-check 2-3 changed fields in each file
   - Confirm spacing is correct (count spaces visually or via grep)

---

## 5. VALIDATION PHASE

### Objective
Confirm all changes are consistent and correct.

### Steps

1. **Verify spacing consistency**
   ```bash
   grep "group_label.*Coverage" {FILE_PATH} | wc -l
   ```
   - Should return count of all Coverage fields
   - All should show "    " (4 spaces)

2. **Check no partial changes**
   ```bash
   grep "group_label.*[^0-9 ]Coverage" --include="*.view.lkml" {WORKSPACE_PATH}
   ```
   - Should return NO results (all are consistent)

3. **Confirm sort order visually**
   - In Looker, open Explore with affected views
   - Navigate to field picker
   - Verify Coverage groups appear at top of "Route (Location)"
   - Verify relative order is correct

4. **Cross-file consistency**
   - Confirm same group_label has same spacing across all files
   - Example: " Coverage Location" should be "    Coverage Location" everywhere

5. **Create validation checklist**
   ```
   ✓ All 38 replacements completed
   ✓ No partial matches remain
   ✓ Spacing is consistent (4 spaces)
   ✓ File parseable (no syntax errors)
   ✓ Fields appear in correct order in Looker
   ✓ Related dimensions grouped together
   ```

---

## Common Issues & Solutions

### Issue: Some fields still have wrong spacing
**Solution:** Check for typos in oldString (extra/missing spaces, quotes). Re-run grep to find any remaining instances.

### Issue: Changes not reflected in Looker
**Solution:** Looker may cache metadata. Perform a "Save all" in IDE, then refresh/reload Looker.

### Issue: Different files have different spacing levels
**Solution:** Audit all related files. Standardize spacing at the same hierarchy level.

### Issue: Can't find all occurrences
**Solution:** Use regex grep: `grep -E 'group_label.*[[:space:]]+Coverage'` to find any spacing variation.

---

## Template Checklist

- [ ] Discovery: All target occurrences documented
- [ ] Analysis: Spacing hierarchy mapped
- [ ] Planning: Before/after examples created
- [ ] Planning: Replacement list verified (no duplicates)
- [ ] Implementation: Batch replacements executed
- [ ] Validation: Spacing consistency confirmed
- [ ] Validation: Sort order verified in Looker
- [ ] Documentation: Changes committed with clear message

---

## Example Commit Message

```
docs: Reorder LookML field groups - push Coverage groups to top

- dmc_location_r.view.lkml: Updated 36 fields (dmc_location_coverage, dmc_location_cvrg_adj)
- dms_dmt_pck.view.lkml: Updated 2 fields (cvr_loc_cd, cvrg_loc_cd)

Changed group_label spacing from " " (1 space) to "    " (4 spaces)
to sort Coverage groups above From/To Movement Location groups.

Sort order in "Route (Location)" field picker:
1. Coverage Adjustment Location (4 spaces)
2. Coverage Location (4 spaces)
3. From Movement Location (3 spaces)
4. To Movement Location (2 spaces)
5. Other groups (0 spaces)

Fixes #TICKET-ID
```

---

## Quick Reference

| Phase | Key Action | Output |
|-------|-----------|--------|
| Discovery | Search all files for target group_label | File list, locations, current values |
| Analysis | Map spacing hierarchy | Hierarchy diagram, target spacing level |
| Planning | Extract before/after replacements | Replacement pairs with context |
| Implementation | Execute multi_replace_string_in_file | Success confirmations |
| Validation | Confirm spacing & sort order | Checklist completion, Looker verification |

