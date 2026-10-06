# v3-egwp-manage-fees.html — Build History

**File:** `v3-egwp-manage-fees.html`
**Title:** "v3 CBR – EGWP Manage Fees – Independent WRAP Concept"

---

## What This File Is

This is the third version of the EGWP Manage Fees tab prototype. The subtitle "Independent WRAP Concept" names the defining change: the WRAP AR became fully decoupled from the 301 AR. The `wrapMapping` three-state system introduced in v2 was removed entirely. WRAP rows have no link tracking, no auto-sync from 301, and no "301 Mapping" column. The file runs to approximately 2,858 lines.

The central design question this version explored: **is the linked/detached WRAP model the right interaction pattern, or should the WRAP AR simply be treated as a parallel independent AR?** v3 answers that question by removing all coupling mechanics.

The other major addition was the **Material Code Picker** — a searchable dropdown with 130+ codes that replaced the chip-only editing pattern across all tables.

---

## Shell and Navigation

Identical to v1 and v2:

- **Header** — CVS Health logo (base64 PNG), "CBR" label, welcome message (Sally Smith, Manager), sign-out link
- **Carrier bar** — back arrow, "Carrier #91AF", "Setup in Progress" pink badge
- **Nav tabs** — New Client Overview, AR Setup, Manage Fees (active), FAF Details, QC Packet

---

## AR Tab Structure

Tab order was **reordered** from v2, and chip colors were changed:

| Tab | Chip Label | Chip Color in v3 | v2 Color |
|-----|------------|------------------|----------|
| 301 AR | A91AF | Green (`#4DB21B`) | Green (`#4DB21B`) |
| 231 AR | 10610 | Purple (`#A56DBD`) | Orange (`#FF7300`) |
| WRAP AR | A92AC | Orange (`#FF7300`) | Purple (`#A56DBD`) |

**Tab order change:** v1 and v2 were 301 → WRAP → 231. v3 changed to 301 → 231 → WRAP. The HTML markup and `data-ar` attributes reflect this; `switchAR()` logic was unchanged.

The chip border-radius remained `12px` throughout v1–v3 (the 8px standardization was applied to the current `egwp-setup/index.html` prototype, not the versioned files).

---

## Fee Data Model

The `AR_DATA` object was restructured for WRAP. The most significant changes:

### 301 AR rows (6 rows, FAF 59248) — identical to v2:
- Retail Per Claim — F1, F2, FE
- Paper Per Claim — FP
- Mail Per Claim — FM
- Per Member Per Month — PM
- Flat Annually — BAF (broker amount $10,666.00)
- Per Member Per Month — PP

### WRAP AR rows (4 rows, FAF 63522) — fully independent:
- Retail Per Claim — F1, F2, FE
- Paper Per Claim — FP
- Mail Per Claim — FM
- Flat Annually — BAF (broker $8,400.00)

**Key change:** No `linkedTo301` field. No `wrapMapping` field. WRAP rows are plain data objects with no relationship tracking. The comment in the data section reads: *"WRAP Billing Administrative rows — pre-populated from 301 (PMPM excluded); fully independent."*

### 231 AR rows (6 rows, FAF 59248) — same as v2:
- Retail Per Claim — RTFE (linked to 301 id 1)
- Paper Per Claim — PAFE (linked to 301 id 2)
- Mail Per Claim — MLFE (linked to 301 id 3)
- Per Member Per Month — PMMD (linked to 301 id 4)
- Flat Annually — BAF (linked to 301 id 5)
- Per Member Per Month — PMRB (linked to 301 id 6)

231 AR retained `linkedTo301` for display mirroring and sync purposes.

---

## `syncGeneratedARs()` — Simplified

With WRAP no longer linked to 301, `syncGeneratedARs()` syncs **231 only**:

```javascript
function syncGeneratedARs() {
  AR_DATA['231'].rows.forEach(row => {
    if (row.linkedTo301 === null) return;
    const src = AR_DATA['301'].rows.find(r => r.id === row.linkedTo301);
    if (!src) return;
    row.baseAmt   = src.baseAmt;
    row.validFrom = src.validFrom;
    row.validTo   = src.validTo;
  });
}
```

No WRAP secondary table sync. No mapped/detached checks. No broker amount sync.

---

## Removed: `301 Mapping` Column

The `col-wrap-mapping` CSS class, `CT_WRAP_MAP` column type constant, `wrap-active` class toggling on `#mainTable`, and all `wrapMapping` logic were removed. The fee table no longer shows a mapping status column under any AR view.

---

## `switchAR()` — Calls `closeOverlay()` First

A fix from v2: `switchAR()` now calls `closeOverlay()` before switching the active AR. In v2, switching tabs while the Batch Update drawer was open left the drawer visible with stale context. In v3, any open overlay is closed first.

```javascript
function switchAR(ar) {
  closeOverlay();
  activeAR = ar;
  perClaimsMode = false;
  // ...
}
```

---

## WRAP AR Selection — Simplified

Because there are no mapped/detached states, `onRowCheck()` and `onSelectAll()` no longer auto-select or auto-deselect WRAP counterparts when 301 rows are checked:

```javascript
function onRowCheck(id, cb) {
  if (cb.checked) sel().add(id);
  else sel().delete(id);
  renderTable();
}
```

The auto-selection cascade (301 check → WRAP check) from v2 is entirely removed.

---

## Review Button — Gated on Unsent Status

v3 changed `syncBottomButtons()` to enable "Review & Send Fee Updates" only when there are **pending (not yet sent)** rows selected:

```javascript
function syncBottomButtons() {
  const hasPending301  = AR_DATA['301'].rows.some(r => selectedByAR['301'].has(r.id) && !r.sent);
  const hasPendingWrap = AR_DATA['wrap'].rows.some(r => selectedByAR['wrap'].has(r.id) && !r.sent);
  document.getElementById('btnReview').classList.toggle('btn-disabled', !hasPending301 && !hasPendingWrap);
  document.getElementById('btnSave').classList.toggle('btn-disabled', !hasChanges);
}
```

In v2, the Review button was enabled for any selection, including already-sent rows. In v3, selecting rows that have already been sent does not re-enable the button.

---

## Review & Send Modal — Opens to Active AR Tab

v3 changed `openModal()` to open to the currently active AR tab rather than always defaulting to 301 AR.

The modal tab order in v3 matches the page tab order: 301 → 231 → WRAP.

---

## Batch Update Drawer Info Message — Simplified

With no mapping states, the contextual message logic was replaced with two static messages:

```javascript
function setDrawerInfoMsg(arContext) {
  var span = document.querySelector('#batchDrawer .drawer-info-msg span');
  if (!span) return;
  if (arContext === 'wrap') {
    span.textContent = 'Updates will be applied to the selected WRAP AR fees only.';
  } else {
    span.textContent = 'Updates will be applied to the selected 301 AR material codes and any corresponding 231 AR material codes.';
  }
}
```

No more count-based messaging about detached counterparts. The complexity of the v2 contextual message was only needed because of the mapped/detached system.

The static message hardcoded in the HTML reads: *"Updates will be applied to the selected 301 AR material codes and any corresponding 231 AR material codes."* — consistent with the 301 AR path.

---

## Material Code Picker

The most visible new feature in v3. A searchable dropdown picker replaced typing codes manually.

### How it works

- Material code cells (`<td class="codes-cell">`) in 301 and WRAP AR views are clickable
- Clicking a codes cell calls `openCodePicker(triggerEl, ar, rowId, tableKey)`
- A dropdown div (`#codePickerDropdown`) is created once and reused, positioned via `getBoundingClientRect()` of the trigger element
- A search input filters the list in real time via `_renderPickerList()`
- Clicking a code option calls `selectPickerCode(code)`, which dispatches to `addCode()` or `addSecCode()` depending on whether it's the main table or a secondary table

### Code list
130+ codes with descriptions, covering all fee categories. Codes already present on the row are filtered out of the picker list. The list is capped at 80 displayed results per search.

### Functions introduced
- `openCodePicker(triggerEl, ar, rowId, tableKey)` — positions and opens the dropdown
- `_renderPickerList()` — filters `MATERIAL_CODES` and renders options
- `selectPickerCode(code)` — routes to `addCode()` or `addSecCode()`
- `closeCodePicker()` — hides the dropdown and clears `_cpCtx`
- `addCode(ar, rowId, code)` — adds code to main table row; calls `syncGeneratedARs()` if 301 AR
- `removeCode(ar, rowId, code)` — removes code from main table row
- `addSecCode(tableKey, ar, rowId, code)` — adds code to secondary table row
- `removeSecCode(tableKey, ar, rowId, code)` — removes code from secondary table row

### `_cpCtx`
A module-level variable `{ ar, rowId, tableKey }` holds the context of the currently open picker so that `selectPickerCode()` knows which row and table to update.

### Close behavior
The picker closes on: selecting a code, clicking outside the dropdown and outside any `.codes-cell`, or calling `closeCodePicker()` directly (e.g. from `switchAR()`).

---

## `MATERIAL_CODES` Array

The full material code list was added as a constant array of `{ code, name }` objects. Categories include:
- Retail, Paper, Mail, Specialty per-claim codes (F1, F2, FE, FP, FM, etc.)
- PMPM and PEPM codes (PM, PP, BPM, EI, EJ, etc.)
- Annual and flat fees (BAF, MM, BSF, etc.)
- Medicare Part D clinical codes (D4, D5, D6, EA, ED, etc.)
- Ancillary codes (M3PA, M3PE, M3PH, etc.)
- Subsidy / SilverScript codes (CVDG, DISB, LIPS, MLCL, RTCL, etc.)
- PSM program codes (HGOF, SLRF, TLOF, TLRF)
- Weight management codes (GLP1, GLP2, GLPE, WEG1, WEG2, etc.)

---

## Secondary Tables (Retained from v2)

All four secondary tables from v2 were retained:
- Billing Ancillary (`tbl-ancillary`)
- Billing Other Fees (`tbl-other`)
- Medicare Part D (`tbl-medd`)
- Clinical Solutions (`tbl-clinical`)

Key changes from v2:
- WRAP rows in secondary tables no longer had `wrapMapping` or `linkedTo301` — fully independent
- Secondary table `addSecCode()` / `removeSecCode()` wired up through the picker
- `syncGeneratedARs()` no longer touched secondary WRAP tables

Secondary table frozen columns continued to use the JS-driven `.frozen-col` + `style.left` approach from v2.

---

## `on301FieldChange()` / `onWrapFieldChange()` — WRAP No Longer Detaches

In v2, `onWrapFieldChange()` set `row.wrapMapping = 'detached'` when editing. In v3, `onWrapFieldChange()` simply updates the field and re-renders — no state tracking:

```javascript
function onWrapFieldChange(field, rowId, value) {
  const row = AR_DATA['wrap'].rows.find(r => r.id === rowId);
  if (!row) return;
  row[field] = (field === 'baseAmt' || field === 'brokerAmt') ? (formatCurrency(value) || value) : value;
  hasChanges = true;
  syncBottomButtons();
  renderTable();
}
```

Similarly, `onDateChange()` for WRAP no longer sets `wrapMapping = 'detached'`.

---

## `applyUpdate()` — No Detach on Apply

In v2, `applyUpdate()` for WRAP rows included:
```javascript
if (activeAR === 'wrap' && row.wrapMapping === 'mapped') row.wrapMapping = 'detached';
```

In v3, this line is absent. WRAP rows are edited in place with no state change.

---

## Concept Notes Section

A `<details>` element remained at the bottom of the page (outside the production interface), but the open questions section from v1 was removed. v3's concept notes described what the Independent WRAP Concept tested.

---

## FAF Notes Drawer

Same structure as v1 and v2. Two FAFs (59248, 63522), Billing Notes filled, Clinical Notes null.

---

## Bottom Bar

- **Review & Send Fee Updates** — enabled only when pending (unsent) rows are selected in 301 or WRAP (changed from v2)
- **Save** — enabled when `hasChanges` is true

---

## Pagination

Same as v2: static single-option `<option>10</option>` in the main table; not functional. Secondary tables had no pagination.

---

## Key Differences from v2

| Feature | v2 | v3 |
|---------|----|----|
| WRAP AR row state | `wrapMapping: 'mapped'/'detached'/'wrapOnly'` | No mapping state — fully independent rows |
| `linkedTo301` on WRAP rows | Present | Removed |
| `301 Mapping` column | Shown in WRAP view | Removed entirely |
| Auto-select WRAP on 301 check | Yes (Mapped rows) | No |
| Detach-on-edit | Yes | No |
| `syncGeneratedARs()` scope | 231 + Mapped WRAP | 231 only |
| AR tab order | 301 → WRAP → 231 | 301 → 231 → WRAP |
| WRAP chip color | Purple (`#A56DBD`) | Orange (`#FF7300`) |
| 231 chip color | Orange (`#FF7300`) | Purple (`#A56DBD`) |
| Material Code Picker | Not present | Present (130+ codes, searchable) |
| `switchAR()` | Does not close overlays | Calls `closeOverlay()` first |
| Review button enable condition | Any selection | Pending (unsent) selection only |
| Modal opens to | Always 301 AR | Active AR at time of open |
| Batch Update info message | Contextual (count-based) | Two static messages (301 vs. WRAP) |
| Open questions section | Present | Removed |
