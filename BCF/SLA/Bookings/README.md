# BCF Booking / Deals — SLA & Tagging

## Module

- Zoho CRM module: **Deals**
- Business meaning: **Bookings**
- Stage field: `Stage`

## Booking Stage & Automation Status

| Stage | Trigger / Anchor | SLA / Tag | Status |
|---|---|---|---|
| Open | Booking enters `Open` | 7 days → Delayed; 14 days → Critical | Completed |
| Pending Shipment | Booking enters `Pending Shipment` | Pending Shipment SLA under investigation; Gate-In is the relevant business milestone | In Progress |
| Shipped | Booking enters `Shipped` | 9 days → Delayed; 15 days → Critical | Completed |
| Paid | Booking enters `Paid` | 3 days → Delayed; 7 days → Critical | Completed |
| Verified Copy Approved | Stage reached after Shipped | `Check Arrival Date` remains relevant | Tag logic covered by post-Shipped Arrival Date rule |
| OBL Collected | Booking enters `OBL Collected` | TPD Pending starts from this Booking milestone for linked Clearance | Completed for TPD Pending trigger |
| OBL Issued to Client | Stage reached after OBL Collected | `Check Arrival Date` remains relevant | Covered by post-Shipped Arrival Date rule |
| File Closed | Final Booking stage | `Check Arrival Date` remains relevant | Covered by post-Shipped Arrival Date rule |
| Cancelled | Booking enters `Cancelled` | Remove `Check Arrival Date`, `Check Departure Date`, `Check Vessel Name` | Requirement defined |

> The Booking stages above are the stages confirmed in the current BCF project context and CRM screenshots/logs. Zoho CRM allows Deal stages to be customized, so this list should be updated if the live pipeline is changed. citeturn0search0turn0search2

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

`Check Arrival Date` is added when the Booking is:

- Shipped
- Paid
- Verified Copy Approved
- OBL Collected
- OBL Issued to Client
- File Closed

It does nothing before Shipped and does not add the tag to Cancelled records.

### Departure Date

`Check Departure Date` is relevant while the Booking is in Pending Shipment.

After the Booking has shipped, the Departure Date tag should no longer remain.

**Status: In Progress.**

### Vessel Name

`Check Vessel Name` is relevant while the Booking is in Pending Shipment.

After the Booking has shipped, the Vessel Name tag should no longer remain.

**Status: In Progress.**

### Cancelled

When Stage is `Cancelled`, all three managed tags should be removed:

- `Check Arrival Date`
- `Check Departure Date`
- `Check Vessel Name`

Unrelated CRM tags must be preserved.

## TPD Pending — Booking Trigger

TPD Pending is a **Clearance SLA**, but the process starts from the Booking.

**Trigger:** Booking enters `OBL Collected`.

The workflow/function then finds the Clearance linked to that Booking and validates the Clearance status before applying the SLA tag.

- 4 days → `Delayed`
- 7 days → `Critical`

If the linked Clearance is already `Pending Documents Delivery` or `Cleared`, no TPD SLA action is taken.

## In Progress

### Pending Shipment

- The SLA timing currently being worked on is **9 days / 15 days** where applicable to the Booking workflow.
- The relevant business milestone being investigated is Gate-In / completion of the Booking containers.
- `Gate_In` on the Booking was confirmed as a number field.

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
- Workflow controls timing/trigger execution.
- Deluge validates the live Booking before acting.
- Booking IDs are supplied explicitly through workflow arguments.
- Do not use `input.id` in these BCF functions.
- Preserve unrelated existing tags.
- Use the established `updateMap.put("Tag",finalTags)` and `zoho.crm.updateRecord(...)` pattern.
- Do not use a separate tag API for this implementation.