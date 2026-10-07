# EGWP Setup Prototype — Build Log

**File:** `egwp-setup/index.html`
**GitHub:** `cvs-health-source-code/egwp-setup-prototype`
**GitHub Pages:** `https://didactic-adventure-eenye6z.pages.github.io/`

---

## Overview

Single-file HTML prototype for the CBR (Client Billing Registration) EGWP Setup workflow. Covers two tabs:

- **AR Setup** — three-AR structure (301 AR, WRAP AR, 231 AR) with Billing Structure, Payment, and Pricing Fees sections
- **Manage Fees** — fee table with Batch Update drawer, accordion sections, pagination, and Review & Send modal

Styling uses Internal Pulse / ag-grid design tokens.

The independent WRAP AR model used in Manage Fees — where WRAP rows carry their own codes and amounts with no link tracking to 301 — was first explored in `v3-egwp-manage-fees.html` before being carried into this file.

---

## Three-AR Structure

Every EGWP setup involves three SAP ARs under one FRD:

| AR | Tag Color | Source |
|----|-----------|--------|
| 301 AR | Blue | Derived from Primary Carrier (e.g. A91AF from 91AF) |
| WRAP AR | Amber/Orange | Derived from Wrap/Enhanced Carrier (e.g. A92AC from 92AC) |
| 231 AR | Green | SSI/SilverScript AR (e.g. 10610) |

Reference example (Bristol-Myers Squibb):
- Primary carrier: 91AF → 301 AR: A91AF, FAF: 49551
- Wrap carrier: 92AC → WRAP AR: A92AC, FAF: 49608
- 231 AR: 10610, FAF: 49551 (same as 301)

---

## Change Log

### Initial Build
- Created single-file HTML prototype from screenshots and design specs
- Implemented three-AR structure as first-class sections in Billing, Payment, and Pricing Fees
- Pre-populated fields from carrier selection
- AR tag colors: 301 = blue, WRAP = amber/orange, 231 = green
- Account Names cascade: WRAP = client name + `- ENHANCED`, 231 = client name + `- EGWP`
- Contacts section: 301 shows no-contacts note; WRAP and 231 reference Online FRD tab
- Accordion sections for Billing Administrative, Billing Ancillary, Billing Other
- Bottom bar: "Review & Send Fee Updates" and "Save" with state-driven enable/disable
- Snackbar confirmations after key actions

---

### Pagination Icons
**Problem:** Multiple failed approaches for pagination icon color.
- CSS `mask-image` approach made icons invisible — Chrome blocks `mask-image` from `file://` URLs (spec behavior: failed mask renders element invisible)
- `filter: brightness(0) invert(1) brightness(x)` overrode the PNG's natural color, rendering icons in gray
- `opacity: 0.5` on disabled buttons washed out icon color further

**Resolution:** Removed all filters and opacity from pagination buttons. PNGs display at their natural `#767471` color. Hover state was later removed entirely (see below).

---

### Batch Update Drawer — Snackbar Message
- Changed Apply handler (`applyUpdate()`) snackbar to: **"Fees have been updated but not sent to Pricing Table"**
- The Review & Send confirm handler (`sendFeeUpdates()`) retains its original message: "The Pricing Table has been updated"
- These are two distinct functions; only `applyUpdate()` was changed

---

### Chip Corner Radius
- Set `border-radius: 8px` on all chip components:
  - `.ar-status-chip` (was 12px)
  - `.field-chip` (was 12px)
  - `.pf-chip` (was 10px)
  - `.chip`, `.as-bl-chip` (already 8px)

---

### Info Icon Standardization
- Replaced all base64-encoded inline info icons with `<img src="images/info-circle-f--xs.png" width="16" height="16">`
- Affects Batch Update drawer instances (231 AR tab was already correct)
- All instances standardized to 16×16 px

---

### GitHub Repository Setup
- Authenticated CVS Health GitHub Enterprise account (`Therese-Nielsen_cvsh`)
- Located repo: `cvs-health-source-code/egwp-setup-prototype`
- Cloned to `/tmp/egwp-setup-push/`
- Push workflow: copy `index.html` + any new images from local prototype to clone, commit, push
- New images added at this point: `info-circle-f--xs.png`, `info-circle-f--s.svg`, `check-circle-f--s.svg`
- **Replaces** prior repo: `theresemarienielsen/cbr-egwp-setup`

---

### AR Setup — Page Size Options
- Updated page size select options from `[10, 25, 50]` to **`[10, 20, 50, 100]`**
- Pagination was already functional; selecting a value resets to page 1 and re-renders

---

### Pagination — Removed Hover State
- Removed `.pg-btn:not(:disabled):hover` (blue background) and `.pg-btn:not(:disabled):hover img` (white icon filter)
- Pagination buttons now have no hover styling

---

### AR Setup — Billing FAF Chip
- Changed the Billing FAF chip value from `CARRIER` to **`59248`**

---

### AR Setup — Account Name Auto-Population
- 301 Account Name input now drives 231 and WRAP Account Name fields in real time
- All Account Name text is automatically uppercased as the user types
- Logic:
  - 231 Account Name = `{301 value} - EGWP`
  - WRAP Account Name = `{301 value} - ENHANCED`
- State persists in `_as301AcctName` variable so values survive tab navigation (AR Setup re-renders on each visit)
- Cursor position preserved during uppercase transform

---

## Key Fixed Rules (AR Setup)

### 301 AR
- Billing Level: CARRIER — Detail Level: RPT-GRP
- AEF List Name: FUND4XMTH (fixed chip)
- Invoice Type: EXCL_CR (fixed chip)
- Terms of Pmt: XAH2 (disabled)
- B/E Charges: XAH2 (disabled)
- Invoicing Dates: 99 (fixed)
- PMPM fee: always yes
- No contacts

### WRAP AR
- Separate FAF from 301
- PMPM Indicator: fixed to NONE (double-billing risk with 301)
- Contacts via Online FRD tab
- Material codes: same as 301 minus PM and PP fees

### 231 AR
- Company Code: SLVR (fixed chip)
- Payment Code Family: SA-prefix (fixed chip)
- Uses 301's FAF
- Linked to 301 via "previous account number" field (auto-populated, disabled)
- Contacts via Online FRD tab
- Pricing upload: manual via VK11

---

## File Locations

| Location | Path |
|----------|------|
| Local prototype | `/Users/c723749/Desktop/Claude prototypes/egwp-setup/index.html` |
| GitHub clone | `/tmp/egwp-setup-push/` |
| GitHub repo | `cvs-health-source-code/egwp-setup-prototype` |
| GitHub Pages | `https://didactic-adventure-eenye6z.pages.github.io/` |

---

### Second GitHub Repo — uxtherese/cbr-egwp-setup
- Added `uxtherese/cbr-egwp-setup` as a second remote (`uxtherese`)
- GitHub Pages live at `https://uxtherese.github.io/cbr-egwp-setup/`
- All pushes now go to both `origin` (CVS) and `uxtherese`
- Push workflow requires switching gh auth accounts: `Therese-Nielsen_cvsh` for origin, `uxtherese` for uxtherese remote

---

### Figma MCP — Component Fidelity Workflow
**How it works:** Paste any Internal Pulse Figma component URL into a prompt. Claude calls `get_design_context` with `disableCodeConnect: true` and reads exact design tokens (colors, sizes, states, border radii, focus colors). Use this to update specific components to match the design system.

**Limitation discovered (checkbox):** `get_design_context` returns component *specs* and *sub-asset URLs*, not the assembled component. For composite shapes (a box with an embedded icon), you get the pieces separately — e.g. the checkmark SVG and the box color. Reassembling them in CSS introduces positioning error. The exported sub-asset `check--xs.svg` also carries `preserveAspectRatio="none"`, a Figma export artifact that allows the browser to distort the shape when rendered at a non-native size.

**Resolution for composites:** Export the component frame directly from Figma as an SVG and use that file. It captures the final rendered shape exactly as Figma drew it, with no reassembly required.

---

### Checkbox — Correct Checked State
**Problem:** The CSS border trick (`border-left` / `border-bottom` + `rotate(-45deg)`) rendered the checkmark askew. Replaced with `check--xs.svg` (Internal Pulse asset via Figma MCP) as `background-image` — also askew due to `preserveAspectRatio="none"` on the exported SVG distorting at non-native render sizes.

**Resolution:** Therese exported the checkbox component frame directly from Figma as `Checkbox Frame.svg`. This SVG contains the rounded box and checkmark as a single combined path at the correct proportions. Saved to `images/checkbox-checked.svg` and used as the full visual for `:checked` state — background transparent, border transparent, `background-size: 16px 16px`.

---

### Date Picker — Internal Pulse Styling
- Border removed from `.date-input` (dates live inside table cells; a border creates visual noise)
- Calendar icon updated to Internal Pulse `calendar--s.svg` at 16px via `background-image` on `::-webkit-calendar-picker-indicator`
- Calendar popup disabled via `pointer-events: none` on the indicator (popup is OS-rendered and cannot be CSS-styled)
- Tooltip added to all date inputs: `title="Calendar not available in prototype"`
- All empty Valid To dates defaulted to `12/31/9999`

---

### Row Selection — Date Input Background
- Date inputs inside selected rows (`tr.row-selected .date-input`) now show `background: #cce6ff !important` to match the row highlight color

---

### Add Blank Row — Material Code Entry
- Blank rows (no codes) previously had `pointer-events: none` on all cells including the codes-cell, blocking code entry
- Fixed: changed CSS to `tr.row-no-codes td:not(.codes-cell) { pointer-events: none }` so the codes-cell remains clickable
- Code picker opens normally; user can search and select from the `MATERIAL_CODES` list
- If a typed search term has no match, an **"Add 'XYZ'"** option appears (teal) to add a custom code not in the list

---

### Figma Make Chapter

#### The Fidelity Gap

The Claude Code prototype diverged visually from the Internal Pulse design system — spacing, typography, and component styling did not match. The AR Setup tab in Figma Make had noticeably higher fidelity because it was built directly against design system components. The gap was the motivation to attempt a Figma Make round-trip.

#### Attempt 1: Ingesting the Full Prototype File (Failed)

The first attempt fed `egwp-figma-make-foundation.html` directly to Figma Make. The file was over 3,000 lines and included the complete application UI, all CSS, material code inventory (130+ entries), full AR data model, and multiple Base64-encoded images. Every attempt produced a backend failure:

```
Something went wrong. Try again.
ID: srid_73EADDNBVCNW1ZHDZGKJXDFW6
```

The error appeared after "1 edited file" — the agent received and processed the file, then crashed during execution. The diagnosis: input complexity overload, not a problem with the prototype itself. Contributing factors:
- File size far exceeded a normal screen-recreation artifact
- Large Base64 image payloads inflated the payload
- The `MATERIAL_CODES` array alone added hundreds of records the agent did not need
- The JavaScript was a full rendering engine, not a layout description

#### Attempt 2: Screenshots + Behavioral Summary (Eventually Worked, But Moot)

The recommended approach was to detach the HTML from the request — attach design PNGs, screenshots, and the material code workbook instead, and submit the HTML as a "behavioral reference only" with a note not to parse or recreate the code. This eventually produced a working result in Figma Make. However, by that point the workflow in Claude Code was already complete and correct.

The decision was: **bypass Figma Make entirely** and fix the visual layer directly in Claude Code.

#### Resolution: Visual Fidelity Pass in Claude Code

The AR Setup Final Design PNG was attached directly to Claude Code with the following prompt (condensed):

> AR Setup visual fidelity pass only. The workflow is now correct. Do not modify: Manage Fees, Review and Send, Pricing Fees population, 301-to-231 mappings, WRAP behavior, pass-through logic, status lifecycle, navigation, material code behavior, buttons, or field enabled/disabled states. The only goal is to make AR Setup visually match the approved design. Use the attached AR Setup Final Design as the visual source of truth. Focus on: layout hierarchy, section spacing, field grouping, Billing Structure arrangement, Payment Structure arrangement, Pricing Fees spacing and typography, heading and label sizes, container sizing. Treat the current implementation as functionally complete. This is a pure visual recreation.

No code was transferred from Figma Make. The final `index.html` is a Claude Code artifact throughout.
