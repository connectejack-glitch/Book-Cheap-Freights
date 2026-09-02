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
- The workflow controls when the function runs.
- The function checks the LIVE Clearance status before applying the requested tag.
- The requested SLA tag is not added if it already exists.
- Existing SLA and non-SLA tags are preserved.
- `Delayed` and `Critical` are independent and may coexist.

### Final Deluge Function

**Function:** `Unallocated_Apply_SLA_Universal_Tag_Clearance_Critical`

**Arguments:**

- `orecid`
- `otag`
- `expected_status`
- `module_name`

**File:** `Unallocated_Apply_SLA_Universal_Tag_Clearance_Critical.deluge`

The function does not calculate SLA days. The Workflow owns the timer.

## CCI Pending

- 4 days → `Delayed`.
- 8 days → `Critical`.
- The SLA clock is anchored to **NESS Payment Date**, not simply the status-change time.

## TPD Pending

TPD Pending is a **Clearance SLA**, but its business anchor is the Booking.

### Anchor

The SLA clock starts **when the Booking enters `OBL Collected`**.

It does **not** start when the Booking leaves `OBL Collected`, and it does not use the Clearance's own status-change time as the SLA anchor.

### Flow

Booking enters `OBL Collected`

→ TPD Pending clock starts

→ Workflow schedules the SLA checks

→ Find the Clearance linked to that Booking

→ Retrieve the LIVE Clearance

→ Check the current Clearance Status

→ If Clearance is still `TPD Pending`, apply the requested severity

### Timing

- 4 days → `Delayed`.
- 7 days → `Critical`.

### Booking movement

**Once the SLA has been triggered, subsequent Booking Stage changes do NOT stop the TPD Pending SLA.**

The Booking may leave `OBL Collected` and continue through later stages.

### Clearance status guardrail

- Clearance = `TPD Pending` → continue and apply the requested severity.
- Clearance has moved to **any other status** → stop.

> **Booking movement does NOT stop the SLA. Clearance movement DOES stop the SLA.**

### Relationship

The Clearance is found using the **Booking lookup**.

Do not use Job as the relationship anchor because one Job can contain multiple Bookings.

### Tagging

The SLA severity is applied to the **Clearance record**, not the Booking.

Only the requested severity is added. Existing tags are preserved.

### Final Deluge Function

**Function:** `TPD_Pending_SLA_Critical`

**File:** `TPD_Pending_SLA_Critical.deluge`

**Arguments:**

- `booking_id`
- `severity`

The function accepts only `Delayed` or `Critical`.

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

The SLA clock starts from the **last Haulage exiting Awaiting Stuffing**.

## Architecture

Workflow controls **when** the check runs.

Deluge controls **whether the SLA is still valid and what action is taken**.

The function retrieves the live record, validates the current state, evaluates the relevant relationship/anchor, and applies the requested tag.

## Important Rules

- Clearances = `Clearances` module.
- Do not confuse Clearance `Status` with Booking `Stage` or Haulage `Status`.
- Preserve unrelated CRM tags.
- Do not hardcode SLA timing inside Deluge when the Workflow controls the schedule.
- TPD Pending uses the Booking milestone `OBL Collected` as its business anchor.
- **Leaving `OBL Collected` does not stop TPD Pending.**
- **Only the linked Clearance moving out of `TPD Pending` stops the SLA from being applied at the scheduled check.**
- Booking IDs are passed explicitly through workflow arguments; these functions do not use `input.id`.
