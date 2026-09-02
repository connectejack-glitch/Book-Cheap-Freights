# BCF Clearances — SLA & Tagging

## Module

- Zoho CRM module: **Clearances**
- Status field: `Status`

## Clearance SLA Status

| Status | Trigger / Anchor | Delayed | Critical | Status |
|---|---|---:|---:|---|
| Unallocated | Clearance record creation / entry into Unallocated | 3 days | 5 days | Completed |
| CCI Pending | NESS Payment Date | 4 days | 8 days | Completed |
| TPD Pending | Booking enters `OBL Collected` → linked Clearance | 4 days | 7 days | Completed |
| Pending Documents Delivery | Entry into status | 2 days | 5 days | Completed |
| Stuffing Paperwork Pending | Last Haulage exits `Awaiting Stuffing` | 2 days | 4 days | Completed / implementation documented |
| Cleared | Terminal status | No SLA | No SLA | No SLA configured |

## Unallocated

- 3 days → `Delayed`.
- 5 days → `Critical`.
- Validate the live Clearance status before applying the requested severity.

## CCI Pending

- 4 days → `Delayed`.
- 8 days → `Critical`.
- The SLA clock is anchored to **NESS Payment Date**, not simply the status-change time.

## TPD Pending

TPD Pending starts from the Booking rather than from the Clearance status-change time.

### Flow

Booking enters `OBL Collected`

→ Find the Clearance linked to that Booking

→ Check the live Clearance status

→ Apply the requested SLA severity

### Timing

- 4 days → `Delayed`.
- 7 days → `Critical`.

### Stop conditions

- Clearance = `Pending Documents Delivery` → stop.
- Clearance = `Cleared` → stop.
- Booking leaves `OBL Collected` before the scheduled action → stop.

The Booking-to-Clearance relationship uses the Booking lookup, not Job, because one Job can contain multiple Bookings. citeturn38file7turn38file10

## Pending Documents Delivery

- 2 days → `Delayed`.
- 5 days → `Critical`.
- Validate that the live Clearance remains in `Pending Documents Delivery` before applying the SLA action.

## Stuffing Paperwork Pending

This SLA begins only after the haulage process has completed stuffing.

### Trigger

A Haulage moves out of `Awaiting Stuffing`.

The system checks all Haulages for the same Booking to determine whether the triggering Haulage was the last one still requiring stuffing.

If another Haulage remains in `Awaiting Stuffing`, no SLA starts.

If no Haulage remains in `Awaiting Stuffing`:

- 2 days → `Delayed`.
- 4 days → `Critical`.

### Anchor

The SLA clock starts from the **last Haulage exiting Awaiting Stuffing**. citeturn38file6

## Architecture

Workflow controls **when** the check runs.

Deluge controls **whether the SLA is still valid and what action is taken**.

The function retrieves the live record, validates the current state, evaluates the relevant relationship/anchor, and applies the requested tag.

## Important Rules

- Clearances = `Clearances` module.
- Do not confuse Clearance `Status` with Booking `Stage` or Haulage `Status`.
- Preserve unrelated CRM tags.
- Do not invent or hardcode SLA timing inside Deluge when the Workflow controls the schedule.
- TPD Pending uses the Booking milestone `OBL Collected` as its business anchor.
