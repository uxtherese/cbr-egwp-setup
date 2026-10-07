# Historical Knowledge — Prototype

**File:** `historical-knowledge.html`
**Concept:** Synthetic prototype exploring how historical FAF context could surface during Maintenance of Business work

---

## What It Is

A concept prototype for the CBR **Maintenance of Business** workbench. The core idea: when a specialist opens a work item, they can see related historical FAF records for that client — previous implementations, prior owners, and any notable setup considerations from those older records.

This prototype uses entirely synthetic data.

---

## Layout

| Region | Description |
|--------|-------------|
| Top bar | CVS Health logo, app name ("CBR"), signed-in user, sign out |
| Left nav | Collapsible; Home, New Implementation, Maintenance of Business (Workbench / QC Queue), Broker Payments, Material Codes, Reports |
| Status cards | Scrollable row of clickable cards — All Work, Triage, Scheduled, Workable, Pending, RFC — each showing a count and filtering the table when clicked |
| Work item table | Filterable columns: FAF ID, Account Name, LOB, Pricing Eff., FAF Owner, Status, Reason, Days |
| Side drawer | "Historical Context" panel — opens when the clock icon is clicked on a row with history |

---

## Work Item Table

Columns:
- **FAF ID** — clickable link (shows toast in prototype); rows with history show a clock icon button
- **Account Name**
- **LOB** — Commercial, EGWP, Med-D
- **Pricing Eff.** — effective date
- **FAF Owner** — assigned specialist
- **Status** — color-coded dot: Triage (purple), Scheduled (pink), Workable (green), Pending (cyan), RFC (orange)
- **Reason** — e.g. Fee Schedule, Billing Frequency, No Release
- **Days** — age of work item

Column-level text filters appear below the header row. Status cards filter via the Status column.

---

## Historical Context Drawer

Opens by clicking the clock icon on a row that has history (`hasHistory: true`). Sections:

1. **Related Project History** — table of matched FAF records (FAF ID, owner, pricing effective date)
2. **Why these records were matched** — bulleted list of match criteria (same client, same account name)
3. **Considerations** — per-FAF notes on what custom setup exists in each historical record (e.g. custom billing frequency, escalating pricing, additional fee schedule entries)

The drawer closes via the × button or by selecting a row with no history.

---

## Interactions

- **Status card click** → filters table to that status; active card gets a colored border
- **Column filter inputs** → live filter across all columns simultaneously
- **Row click** → selects the row (blue highlight); if drawer is open and row has history, repopulates drawer for that row
- **Clock icon click** → opens drawer for that row without requiring row selection first
- **FAF ID link click** → shows toast ("Opening FAF XXXXX — not implemented in prototype")
- **Left nav** → hamburger toggles collapse; Maintenance of Business parent expands/collapses children

---

## Design Notes

- Status dot colors match the status cards (Triage = `#A56DBD`, Scheduled = `#FF6699`, Workable = `#4DB21B`, Pending = `#00C7E8`, RFC = `#FF7300`)
- Styling uses CVS blue (`#0047BB` / `#003087`) and the CBR neutral palette, not Internal Pulse
- Footer note: "Concept prototype using synthetic project data"
