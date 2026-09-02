# BCF Booking / Deals — SLA & Tagging Work Completed

## Module

- Zoho CRM module: **Deals**
- Business meaning: **Bookings**

This document records the Booking / Deals work completed during the current implementation session.

## Completed Booking SLAs

| Stage | Trigger / Anchor | Delayed | Critical | Status |
|---|---|---:|---:|---|
| Unallocated | Booking enters Unallocated / SLA clock from creation | 4 days | 8 days | Completed |
| Open | Status Entry into Open | 7 days | 14 days | Completed |
| Shipped | Booking enters Shipped | 9 days | 15 days | Completed |

### Standard SLA behaviour

The Workflow Rule controls **when** the scheduled action runs.

The Deluge function controls **whether the SLA is still applicable** when the scheduled action runs.

The function retrieves the live Booking and checks the current Stage before applying the requested severity.

## Arrival Date Tag

### Requirement

Arrival Date becomes relevant once the Booking has shipped and remains relevant through the post-shipped stages.

When **Arrival Date is modified**, the workflow calls the function.

The function retrieves the live Booking, checks the current Stage, and adds `Check Arrival Date` only when the Booking is Shipped or later.

### Post-Shipped stages

- Shipped
- Paid
- Verified Copy Approved
- OBL Collected
- OBL Issued to Client
- File Closed

Cancelled is excluded.

### Workflow arguments

- `booking_id` → Booking / Deals record ID
- `severity` → `Check Arrival Date`

## Booking Tag Cleanup

Before implementing the new Arrival Date workflow, the existing CRM records were cleared of:

- `Check Arrival Date`
- `Check Departure Date`
- `Check Vessel Name`

This provided a clean starting point for the new tagging logic.

## Tag Rules

### Before Shipped

Arrival Date is not relevant.

Pending Shipment is expected to use `Check Departure Date` and `Check Vessel Name`.

### Shipped and later

Arrival Date is relevant and `Check Arrival Date` should apply.

Departure Date and Vessel Name tags should no longer remain after shipment.

### Cancelled

All three managed tags should be removed:

- `Check Arrival Date`
- `Check Departure Date`
- `Check Vessel Name`

Unrelated tags must be preserved.

## In Progress

### Pending Shipment

- 9 days → Delayed
- 18 days → Critical
- Clock: last container Gate-In / Gate-In completion date
- Booking `Gate_In` was confirmed as a number field.
- Exact source for the last container Gate-In milestone was still being investigated.

### Departure Date

- Managed tag: `Check Departure Date`
- Relevant during Pending Shipment and should not remain after Shipped.
- Status: In Progress

### Vessel Name

- Managed tag: `Check Vessel Name`
- Relevant during Pending Shipment and should not remain after Shipped.
- Status: In Progress

## Important Implementation Rules

- Booking = Deals module.
- Always retrieve the **live Booking** before acting.
- Do not rely only on the state that existed when the workflow was originally triggered.
- Do not use `input.id` for BCF functions.
- Record IDs are supplied explicitly through workflow arguments.
- Use the established CRM tag update pattern with `updateMap.put("Tag",finalTags)` and `zoho.crm.updateRecord(...)`.
- Do not use a separate tag API for this implementation.
- Existing non-managed tags must be preserved.

## Working Arrival Date Deluge

The exact working function is stored separately in `Check_Arrival_Date.deluge` in this folder.