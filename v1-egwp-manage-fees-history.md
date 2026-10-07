# v1-egwp-manage-fees.html — Build History

**File:** `v1-egwp-manage-fees.html`
**Title:** "v1 CBR – EGWP Manage Fees – Source of Truth Concept"

---

## What This File Is

This is the first version of the EGWP Manage Fees tab prototype for CBR (Client Billing Registration). It was built as a standalone single-file HTML proof-of-concept focused entirely on the **Manage Fees** tab — the AR Setup tab did not exist yet at this stage. The subtitle "Source of Truth Concept" indicates this was used to establish the foundational data model and interaction patterns before later versions were built.

---

## Shell and Navigation

The file opens with the standard CBR application shell:

- **Header** — CVS Health logo (embedded as base64 PNG), "CBR" label, welcome message with user name and sign-out link
- **Carrier bar** — back arrow, carrier title "Carrier #91AF", "Setup in Progress" pink status badge
- **Nav tabs** — New Client Overview, AR Setup, Manage Fees (active), FAF Details, QC Packet

The Manage Fees tab was the only functional page. The other tabs were present for visual context but non-functional.

---

## AR Tab Structure

The three-AR model was established from the start with a tab switcher at the top of the page:

| Tab | Chip Label | Chip Color in v1 |
|-----|------------|------------------|
| 301 AR | A91AF | Green (`#4DB21B`) |
| WRAP AR | A92AC | Purple (`#A56DBD`) |
| 231 AR | 10610 | Orange (`#FF7300`) |

> **Note:** These colors were later changed. In the current prototype, 301 AR = blue, WRAP AR = amber/orange, 231 AR = green.

Switching tabs called `switchAR()`, which re-rendered the fee table and updated toolbar visibility (Add Row and Batch Update buttons hidden for 231 AR, which is read-only/generated).

---

## Fee Data Model

A JavaScript `AR_DATA` object held all fee rows keyed by AR type. This was the "source of truth" for all three ARs.

### 301 AR rows (6 rows, FAF 59248):
- Retail Per Claim — F1, F2, FE
- Paper Per Claim — FP
- Mail Per Claim — FM
- Per Member Per Month — PM
- Flat Annually — BAF (with broker amount $10,666.00)
- Per Member Per Month — PP

### WRAP AR rows (4 rows, FAF 63522):
- Shared with 301: Retail, Paper, Mail, Flat Annually
- PMPM excluded per business rules (double-billing risk)
- `sharedWith301: true` flag disabled checkboxes; base amounts mirrored from 301 AR
- Broker amounts were WRAP FAF-specific, not synced from 301

### 231 AR rows (6 rows, FAF 59248 — same as 301):
- Each row linked to a 301 AR row via `linkedTo301` field
- Read-only, no checkboxes, selection mirrored from 301 AR
- Material codes used SSI codes: RTFE, PAFE, MLFE, PMMD, BAF, PMRB

---

## Generated AR Sync

`syncGeneratedARs()` was a key piece of the data model. Called after every 301 AR edit, it propagated `baseAmt`, `validFrom`, and `validTo` to corresponding WRAP and 231 AR rows. Broker amounts were explicitly excluded from sync.

This function ran on every `renderTable()` call so the generated views always reflected the latest 301 AR state.

---

## Fee Table

The main table was built with:

- **Frozen columns** — Selection checkbox, Fee Type, Material Code(s), Status — sticky via CSS `position: sticky` with dynamically computed `left` offsets
- **Column resize** — drag handles on all column headers using `mousedown`/`mousemove`/`mouseup` event listeners; CSS custom properties (`--frozen-col2-left`, etc.) updated on drag to keep frozen columns aligned
- **Filter row** — a second `<thead>` row with filter inputs under each column header
- **Row selection** — checkbox per row; indeterminate state on header checkbox; selection scoped per AR type
- **Row status** — clock icon (pending) or checkmark icon (sent), both embedded as base64 PNGs
- **Inline date editing** — Valid From / Valid To cells in 301 AR and WRAP-only rows used `<input type="date">` directly in the table cell

Columns: Fee Type, Material Code(s), Status, FAF(s), Year, LOB, Base Amt, Broker Amt, Bill & Remit, Lvl 1, Lvl 2, Lvl 3, Valid From, Valid To.

---

## Toolbar

Above the table:

- **Generated notice** — a small inline message shown when mirrored rows (WRAP or 231) were actively affected by a 301 selection; text varied by AR
- **Add Blank Row** — prepended a blank row to 301 or WRAP AR; disabled for 231 AR
- **Batch Update** — enabled/disabled based on selection count; two icon states (enabled/disabled) embedded as base64 PNGs; opened the Batch Update drawer

---

## Batch Update Drawer

Opened when 2 or more rows were selected (the threshold was later lowered to 1). Slid in from the right; page content and bottom bar shifted left to make room (CSS transition).

Drawer contents:
- Info message explaining updates would apply to selected 301 AR codes and corresponding WRAP/231 codes
- Checkboxes to select which fields to update: Base Amount, Broker Amount, Valid From, Valid To, Add Per All Claims fee
- Broker Amount option hidden when no selected row had a broker fee
- Per All Claims disabled when a broker-fee row was selected
- Fields rendered dynamically as checkboxes were checked
- **Apply to selected fees** button — disabled until at least one field was checked and filled; applied values to all selected rows and synced generated ARs

Info icon in the drawer was embedded as a base64 PNG inline in the HTML (later replaced with a file reference to `images/info-circle-f--xs.png`).

---

## FAF Notes Drawer

Triggered by the FAF Notes button in the page header. Contained:
- A dropdown to select which FAF to view (59248 or 63522)
- Billing Notes section
- Clinical Notes section (empty/null for both FAFs in v1)

FAF notes were stored in a `FAF_NOTES` object in JavaScript.

---

## Review & Send Modal

Opened from the "Review & Send Fee Updates" bottom bar button (enabled only when 301 AR rows were selected).

The modal contained:
- AR tabs (301, WRAP, 231) to switch between review views
- A table showing pending rows (highlighted) and previously sent rows
- Rows sorted: pending first, already-sent second
- "Send Updates to Pricing Table" button — marked selected rows as `sent`, cleared selection, re-rendered table
- Snackbar confirmation: "Fee updates sent to Pricing Fees table"

---

## Accordion Sections

- **Billing Administrative** — open by default, contained the full fee table
- **Billing Ancillary** — collapsed, empty body
- **Billing Other** — collapsed, empty body

---

## Bottom Bar

Fixed at the bottom of the viewport:
- **Review & Send Fee Updates** — enabled only when 301 AR rows were selected; opened the modal
- **Save** — enabled only when `hasChanges` was true; cleared the flag on click

---

## Pagination

Basic static pagination row at the bottom of the Billing Administrative accordion. Page size select had only `<option>10</option>`. Navigation buttons used HTML entity characters (`|<`, `<`, `>`, `>|`) rather than PNG icons. No functional pagination logic — the row count display was hardcoded.

> In later versions, pagination buttons were replaced with PNG icons from the `images/` folder, and the AR Setup tab gained fully functional pagination with per-AR state.

---

## Open Questions Section

A `<details>` element at the bottom of the page (outside the production interface) listed 6 unresolved design questions:

1. Can generated 231 AR rows ever be excluded independently from a fee send?
2. Can WRAP-only fees be edited somewhere other than the generated WRAP view?
3. Does Review & Send submit all related AR outcomes together, or can the analyst choose?
4. Does every selected 301 AR fee have a 231 AR equivalent?
5. Which 301 AR fees apply to the WRAP AR?
6. How should the experience handle a correction needed only in a generated view?

---

## Key Differences from Current Prototype

| Feature | v1 | Current (`egwp-setup/index.html`) |
|---------|----|-----------------------------------|
| Tabs | Manage Fees only | AR Setup + Manage Fees |
| AR chip colors | Green / Purple / Orange | Blue / Amber / Green |
| Pagination buttons | HTML entities (`|<` `<` `>` `>|`) | PNG icons from `images/` folder |
| Info icon | Base64 inline | `images/info-circle-f--xs.png` |
| Batch Update threshold | 2+ rows | 1+ rows |
| Pagination options | 10 only (static) | 10, 20, 50, 100 (functional) |
| AR Setup tab | Not present | Full billing/payment/pricing sections |
| Chip border-radius | 12px | 8px |
| Snackbar (Apply) | "Fees have been updated but not sent to Pricing Table" | Same |
