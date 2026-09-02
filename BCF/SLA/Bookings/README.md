# BCF Booking / Deals — SLA & Tagging

## Module

- Zoho CRM module: **Deals**
- Business meaning: **Bookings**
- Stage field: `Stage`

## Booking Stage & Automation Status

| Stage | Trigger / Anchor | SLA / Tag | Status |
|---|---|---|---|
| Open | Booking enters `Open` | 7 days → Delayed; 14 days → Critical | Completed |
| Pending Shipment | Booking enters `Pending Shipment` | 9 days → Delayed; 15 days → Critical; anchor is the last Container Gate-In / Gate-In completion date | In Progress |
| Shipped | Booking enters `Shipped` | 9 days → Delayed; 15 days → Critical | Completed |
| Paid | Booking enters `Paid` | 3 days → Delayed; 7 days → Critical | Completed |
| Verified Copy Approved | Stage reached after Shipped | `Check Arrival Date` remains relevant | Covered by post-Shipped Arrival Date rule |
| OBL Collected | Booking enters `OBL Collected` | TPD Pending starts from this Booking milestone for linked Clearance | Completed for TPD Pending trigger |
| OBL Issued to Client | Stage reached after OBL Collected | `Check Arrival Date` remains relevant | Covered by post-Shipped Arrival Date rule |
| File Closed | Final Booking stage | `Check Arrival Date` remains relevant | Covered by post-Shipped Arrival Date rule |
| Cancelled | Booking enters `Cancelled` | Remove `Check Arrival Date`, `Check Departure Date`, `Check Vessel Name` | Requirement defined |

> The Booking stages above are the stages confirmed in the current BCF project context and CRM screenshots/logs.

## Completed Booking SLA Work

### Open

- Trigger: Booking Stage changes to `Open`.
- 7 days → `Delayed`.
- 14 days → `Critical`.
- The function validates the live Booking Stage before tagging.

### Paid

- Trigger: Booking Stage changes to `Paid`.
- 3 days → `Delayed`.
- 7 days → `Critical`.
- The function validates the live Booking Stage before tagging.

### Shipped

- Trigger: Booking Stage changes to `Shipped`.
- 9 days → `Delayed`.
- 15 days → `Critical`.
- The function validates the live Booking Stage before tagging.

### Delayed Tag Removal

- Trigger: Booking Stage is modified.
- Workflow passes the Booking ID through `orecid`.
- Function retrieves the LIVE Booking from `Deals`.
- If `Delayed` exists, it is removed using the CRM `remove_tags` action.
- `Critical` and all other existing tags are preserved.
- The Booking Stage is not changed.
- This function has been tested successfully from the workflow.

## Booking Date / Vessel Tags

### Managed tags

- `Check Arrival Date`
- `Check Departure Date`
- `Check Vessel Name`

Before implementing the new Arrival Date workflow, the CRM was cleared of all three managed tags so the new logic could start from a clean state.

### Arrival Date

**Trigger:** Arrival Date is modified.

The workflow calls `BOOKING_Check_Arrival_Date`.

The function retrieves the LIVE Booking and checks the current Stage.

`Check Arrival Date` is relevant from **Shipped onward** and remains relevant through the later Booking stages, including:

- Shipped
- Verified Copy Approved
- OBL Collected
- OBL Issued to Client
- Paid
- File Closed

It should not be present before Shipped and should not be added to Cancelled records.

### Departure Date

`Check Departure Date` is relevant for **Pending Shipment**.

It should not remain after the Booking has shipped.

**Status: In Progress.**

### Vessel Name

`Check Vessel Name` is relevant for **Pending Shipment**.

It should not remain after the Booking has shipped.

**Status: In Progress.**

### Cancelled

When Stage is `Cancelled`, all three managed tags should be removed:

- `Check Arrival Date`
- `Check Departure Date`
- `Check Vessel Name`

Unrelated CRM tags must be preserved.

**Status: Requirement defined / implementation pending.**

## TPD Pending — Booking Trigger

TPD Pending is a **Clearance SLA**, but the process starts from the Booking.

**Trigger:** Booking enters `OBL Collected`.

The workflow/function then finds the Clearance linked to that Booking and validates the Clearance status before applying the SLA tag.

- 4 days → `Delayed`
- 7 days → `Critical`

If the linked Clearance is already `Pending Documents Delivery` or `Cleared`, no TPD SLA action is taken.

## In Progress

### Pending Shipment

- SLA: **9 days → Delayed / 15 days → Critical**.
- Anchor: **last Container Gate-In / Gate-In completion date**.
- `Gate_In` on the Booking was confirmed as a number field.
- The Booking Stage must still be `Pending Shipment` when the SLA function runs.

### Departure Date

- Managed tag: `Check Departure Date`.
- Relevant for Pending Shipment.
- Should not remain after Shipped.

### Vessel Name

- Managed tag: `Check Vessel Name`.
- Relevant for Pending Shipment.
- Should not remain after Shipped.

## Implementation Rules

- Booking = `Deals`.
- Stage API name = `Stage`.
- Workflow controls timing and trigger execution.
- Deluge validates the live Booking before acting.
- Booking IDs are supplied explicitly through workflow arguments.
- Do not use `input.id` in these BCF functions.
- Preserve unrelated existing tags.
- Booking SLA tag removal uses the established CRM `remove_tags` action with the `zoho_crm` connection.
- Do not change the Booking Stage through SLA functions.
- For tag removal, use the proven CRM tag action rather than relying on `zoho.crm.updateRecord()` to remove CRM tags.
