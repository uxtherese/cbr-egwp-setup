# v2-egwp-manage-fees.html — Build History

**File:** `v2-egwp-manage-fees.html`
**Title:** "v2 CBR – EGWP Manage Fees – Multi-Table Concept"

---

## What This File Is

This is the second version of the EGWP Manage Fees tab prototype. The subtitle "Multi-Table Concept" names the defining change: four additional accordion sections — Billing Ancillary, Billing Other Fees, Medicare Part D, and Clinical Solutions — each got a full, independently functional fee table with its own data model, schema, and toolbar. The file runs to approximately 2,679 lines.

The central design question this version explored: **how should the relationship between 301 AR and WRAP AR rows be modeled when an analyst edits a WRAP row independently?** The answer introduced the three-state `wrapMapping` system.

---

## Shell and Navigation

Identical to v1:

- **Header** — CVS Health logo (base64 PNG), "CBR" label, welcome message (Sally Smith, Manager), sign-out link
- **Carrier bar** — back arrow, "Carrier #91AF", "Setup in Progress" pink badge
- **Nav tabs** — New Client Overview, AR Setup, Manage Fees (active), FAF Details, QC Packet

---

## AR Tab Structure

The three-AR tab switcher remained, with the same left-to-right order as v1:

| Tab | Chip Label | Chip Color in v2 |
|-----|------------|------------------|
| 301 AR | A91AF | Green (`#4DB21B`) |
| WRAP AR | A92AC | Purple (`#A56DBD`) |
| 231 AR | 10610 | Orange (`#FF7300`) |

> **Note:** These are the same chip colors as v1. Colors were changed again in v3.

---

## Fee Data Model

The `AR_DATA` object carried the same structure as v1 but with a key addition to WRAP rows: the `wrapMapping` property.

### 301 AR rows (6 rows, FAF 59248):
- Retail Per Claim — F1, F2, FE
- Paper Per Claim — FP
- Mail Per Claim — FM
- Per Member Per Month — PM
- Flat Annually — BAF (broker amount $10,666.00)
- Per Member Per Month — PP

### WRAP AR rows (4 rows, FAF 63522):
- Retail Per Claim — F1, F2, FE (`wrapMapping: 'mapped'`, linked to 301 id 1)
- Paper Per Claim — FP (`wrapMapping: 'mapped'`, linked to 301 id 2)
- Mail Per Claim — FM (`wrapMapping: 'mapped'`, linked to 301 id 3)
- Flat Annually — BAF (broker $8,400.00, `wrapMapping: 'mapped'`, linked to 301 id 5)
- PM (PMPM) excluded — double-billing risk

### 231 AR rows (6 rows, FAF 59248):
- Retail Per Claim — RTFE (linked to 301 id 1)
- Paper Per Claim — PAFE (linked to 301 id 2)
- Mail Per Claim — MLFE (linked to 301 id 3)
- Per Member Per Month — PMMD (linked to 301 id 4)
- Flat Annually — BAF (linked to 301 id 5)
- Per Member Per Month — PMRB (linked to 301 id 6)

---

## The `wrapMapping` System

This was the major architectural addition of v2. Each WRAP row carried a `wrapMapping` field with three possible states:

| Value | Meaning |
|-------|---------|
| `'mapped'` | Row is linked to a 301 AR counterpart; `baseAmt`, `validFrom`, `validTo` are synced from 301 whenever `syncGeneratedARs()` runs |
| `'detached'` | Row was originally linked but has been edited independently (e.g. base amount or date changed inline, or Batch Update was applied) — sync no longer overwrites this row |
| `'wrapOnly'` | Row has no 301 counterpart; added via "Add Blank Row" in the WRAP view |

**Detach-on-edit mechanics:**
- Inline base amount edit (`onWrapFieldChange`) sets `wrapMapping = 'detached'` immediately
- Inline date edit (`onDateChange` for WRAP rows) also sets `wrapMapping = 'detached'`
- Batch Update applied to a Mapped WRAP row sets it to `'detached'` on apply

This replaced v1's simpler `sharedWith301: true` boolean flag, which could only express linked vs. not linked.

---

## `syncGeneratedARs()` — Enhanced

In v2, `syncGeneratedARs()` synced **both** the 231 AR and **Mapped** WRAP rows:

- **231 AR** — always synced: propagates `baseAmt`, `validFrom`, `validTo` from the corresponding 301 row
- **WRAP Billing Admin** — only syncs rows where `wrapMapping === 'mapped'`; detached and wrapOnly rows are never overwritten
- **WRAP secondary tables** — same selective sync: mapped rows only

Broker amounts were explicitly excluded from sync in all cases.

---

## `301 Mapping` Column

A new column appeared in the main fee table, visible **only** when the WRAP AR tab was active. The `wrap-active` CSS class was toggled on `#mainTable` via `switchAR()`, and the column cells used class `col-wrap-mapping` with `display: none` / `display: table-cell` controlled by that class.

Column values:
- **Mapped** — row is still synced from 301
- **Detached** — row has been independently edited
- **WRAP Only** — no 301 counterpart

---

## Secondary Tables (Multi-Table Concept)

Four new accordion sections were added, each with a full fee table:

| Accordion | Table ID | Key difference from Billing Admin |
|-----------|----------|----------------------------------|
| Billing Ancillary | `tbl-ancillary` | Ancillary fee types; own schema |
| Billing Other Fees | `tbl-other` | Other/misc fee types |
| Medicare Part D | `tbl-medd` | Medicare Part D-specific codes |
| Clinical Solutions | `tbl-clinical` | Clinical program codes |

All secondary tables:
- Had their own Add Blank Row and Batch Update toolbar buttons (IDs like `btnAdd-ancillary`, `btnBatch-ancillary`)
- Had their own schema defined in `SEC_SCHEMAS` (see below)
- Had their own data stored in `SEC_DATA`
- Were rendered by `renderSecTable(tableKey)` and `secRenderTable(tableKey)`
- Opened a secondary Batch Update drawer via `secOpenDrawer(tableKey)`
- Added rows via `secAddRow(tableKey)`

### Schema-Driven Tables (`SEC_SCHEMAS`)

Each secondary table schema was defined as an array of column objects using column type constants:

```javascript
const CT_CHECK    = 'check';
const CT_TEXT     = 'text';
const CT_CHIPS    = 'chips';
const CT_STATUS   = 'status';
const CT_DATE     = 'date';
const CT_WRAP_MAP = 'wrapMap';  // 301 Mapping column — WRAP tables only
```

Schemas drove header rendering, filter row cells, and body cell types. This allowed secondary tables to have different column sets from the Billing Administrative main table.

### Frozen Columns in Secondary Tables

The main table (`#mainTable`) used `nth-child` CSS selectors for frozen column behavior. Secondary tables could not use the same selectors without conflicts. Solution: secondary table frozen columns were handled entirely in JavaScript using the `.frozen-col` and `.frozen-divider` CSS classes, with `style.left` values computed and set by the renderer. `nth-child` rules were scoped to `#mainTable` only in CSS.

---

## Batch Update Drawer — Contextual Info Message

The drawer info message became contextual based on which AR was active and the state of selected rows:

**When WRAP AR is active:**
- If selected WRAP rows include Mapped rows: *"N selected WRAP AR fee(s) are currently linked to the 301 AR. Applying this update will make them independent (detached)."*
- If no Mapped rows selected: *"Updates will be applied to the selected WRAP AR fees only."*

**When 301 AR is active:**
- If selected 301 rows have Detached WRAP counterparts: *"N selected 301 AR fee(s) have a detached WRAP AR counterpart. Updates will not apply to those WRAP rows."*
- Otherwise: *"Updates will be applied to the selected 301 AR fees."*

The count was computed in `openDrawer()` before showing the drawer.

The static initial message (hardcoded in HTML) read: *"Updates will be applied to the selected 301 AR material codes and any corresponding WRAP AR and 231 AR material codes."* — this was overwritten dynamically by `setDrawerInfoMsg()` each time the drawer opened.

---

## Batch Update Threshold

Lowered from v1's requirement of 2+ rows to **1+ row**. The `openDrawer()` function gates on `sel().size < 1`.

---

## Material Code Chips

In v2, chips in 301 and WRAP AR rows were still rendered with chip-x remove buttons (same as v1), but chips in 231 AR rows were `chip-readonly` (no remove button). Codes could be removed via the chip-x button, which called an inline `event.stopPropagation()` handler in v2.

No code picker dropdown existed yet in v2. Code add/remove was done inline via the chip-x for removal only; adding codes to blank rows was not yet implemented with a picker.

---

## Generated AR Sync and Selection

### 231 AR selection
- Still mirrored 301 AR selection (read-only, disabled checkboxes)
- `syncSelectAll231()` computed the count of 231 rows whose `linkedTo301` was in `selectedByAR['301']`

### WRAP AR selection
- WRAP rows became **fully independently selectable** in v2 — a major change from v1 where mapped rows were disabled
- `syncSelectAllCheckboxWrap()` counted from `selectedByAR['wrap']` directly
- When a 301 row was checked, its corresponding `mapped` WRAP row was auto-selected (via `onRowCheck()` and `onSelectAll()`)
- When a 301 row was unchecked, its corresponding `mapped` WRAP row was auto-deselected
- Detached and wrapOnly WRAP rows were never auto-toggled by 301 selection

---

## Review & Send Modal

The modal aggregated rows from all tables (Billing Administrative + all secondary tables) across all three ARs. AR tabs in the modal were in the same order as the page tabs: 301 → WRAP → 231.

`openModal()` in v2 always opened to the 301 AR tab. The active AR tab at the time of opening did not influence the modal's initial state (this changed in v3).

`sendFeeUpdates()` marked sent rows across both 301 and WRAP ARs (since WRAP became independently selectable), plus 231 rows linked to selected 301 rows.

Snackbar on send: **"Fee updates sent to Pricing Fees table"**

---

## FAF Notes Drawer

Identical structure to v1: dropdown to select FAF (59248 or 63522), Billing Notes section, Clinical Notes section. Both FAFs had billing notes; clinical notes were null for both.

The drawer pre-selected the FAF matching the active AR on open (`AR_DATA[activeAR].fafNumber`).

---

## Bottom Bar

- **Review & Send Fee Updates** — enabled when `selectedByAR['301'].size > 0` OR `selectedByAR['wrap'].size > 0` (expanded from v1 which only watched 301)
- **Save** — enabled when `hasChanges` was true

---

## Pagination

Static in the main Billing Administrative section (single `<option>10</option>`); pagination bar was present but not functional. Secondary tables did not have pagination.

---

## `closeOverlay()` Behavior

`closeOverlay()` closed both the Batch Update drawer and FAF Notes drawer, reset `perClaimsMode`, and unchecked the `chkPerClaims` checkbox. **Switching AR tabs did not close the overlay** in v2 (this was fixed in v3 where `switchAR()` calls `closeOverlay()` first).

---

## Concept Notes Section

A `<details>` element at the bottom of the page (outside the production interface) listed concept-specific design notes. Unlike v1's "open questions" framing, v2's concept notes described what the Multi-Table Concept was testing, without a numbered question list.

---

## Workbook-Derived Mapping Tables

Two constants were introduced to capture business rules derived from the Material Code Mappings workbook:

### `WRAP_PMPM_CODES`
A `Set` of material codes excluded from WRAP AR due to double-billing risk with the 301 AR's PMPM fees:
```javascript
const WRAP_PMPM_CODES = new Set([
  'PM','PP','BPM','D9','IBM','D9S','BPF','BPQ','VACS','PMS','PDG','PDM'
]);
```

### `CODE_TO_231`
A mapping from 301 material codes to their corresponding 231 SSI codes, organized by fee category:
- AdminFee: AJ→RLAJ, F1→SPFE, FE→RTFE, FM→MLFE, FP→PAFE, etc.
- AncillaryFee: M3PA→M3P1, TEM→TEME, etc.
- OtherFee: ELF→ELFE, GR→GREV, TE→LIPF, etc.
- MedDFee: PM→PMMD, PP→PMRB
- ClinicalFee: D4→D4WG, D5→D5EG, D6→D6EG

---

## Key Differences from v1

| Feature | v1 | v2 |
|---------|----|----|
| WRAP AR row state | `sharedWith301: true/false` | Three-state `wrapMapping: 'mapped'/'detached'/'wrapOnly'` |
| WRAP row selectability | Mapped rows disabled (couldn't be independently checked) | All WRAP rows independently selectable |
| 301 Mapping column | Not present | Shown in WRAP AR view only |
| Secondary tables | Not present (Billing Ancillary/Other collapsed with empty bodies) | 4 secondary tables with full schemas, data, toolbars |
| Secondary table frozen columns | N/A | JS-driven via `.frozen-col` + `style.left` |
| Batch Update info message | Static | Contextual based on AR and mapping state |
| Batch Update threshold | 2+ rows | 1+ row |
| `syncGeneratedARs()` scope | 231 only | 231 + Mapped WRAP Billing Admin + Mapped WRAP secondary tables |
| `switchAR()` | Does not close open overlays | Does not close open overlays (v3 fixed this) |
| `CODE_TO_231` / `WRAP_PMPM_CODES` | Not present | Present |
| Material code picker | Not present | Not present (added in v3) |
