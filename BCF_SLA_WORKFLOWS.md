# Book Cheap Freights - Recommended SLA Workflows

## Core rule

The workflow owns the timer. The Deluge function owns the validation and tag update. Every scheduled function re-fetches the live record and stops if the record is no longer in the expected stage/status.

Zoho CRM supports time-based workflow actions and custom functions, so the SLA timers should remain in the workflow layer rather than being calculated inside Deluge. citeturn3search3turn3search1

## 1. Clearances - Unallocated

**SLA:** 3 days = Delayed; 5 days = Critical.

**Workflow module:** Clearances

**Trigger:** Record is created or edited and Status becomes `Unallocated`.

**Scheduled actions:**
- +3 days: `Unallocated_Apply_SLA_Universal_Tag_Clearance_Delayed1` with `orecid = Clearance ID`, `otag = Delayed`, `expected_status = Unallocated`, `module_name = Clearances`.
- +5 days: same function with `otag = Critical`.

**Result:** Delayed remains when Critical is added. Both tags may coexist.

## 2. Clearances - Pending Documents Delivery (PPD)

**SLA:** 2 days = Delayed; 5 days = Critical.

**Workflow module:** Clearances

**Trigger:** Status becomes `Pending Documents Delivery`.

**Scheduled actions:**
- +2 days: `PPD_Pending_Apply_SLA_Universal_Tag_Clearance_Critical` with `otag = Delayed`, `expected_status = Pending Documents Delivery`, `module_name = Clearances`.
- +5 days: same function with `otag = Critical`.

**Result:** Delayed remains when Critical is added. Both tags may coexist.

## 3. Clearances - TPD Pending

**SLA anchor:** Booking reaches `OBL Collected`.

**SLA:** +4 days = Delayed; +7 days = Critical.

**Workflow module:** Deals / Bookings.

**Trigger:** Booking Stage becomes `OBL Collected`.

**Scheduled actions:**
- +4 days: `TPD_Pending_SLA` with `booking_id = Booking ID`, `severity = Delayed`.
- +7 days: `TPD_Pending_SLA` with `booking_id = Booking ID`, `severity = Critical`.

The function re-fetches the Booking. If it has left `OBL Collected`, it stops. It then finds Clearances through the Clearance `Booking` lookup. It does not require a Clearance to exist at the moment the Booking enters OBL Collected; if none exists when the scheduled action runs, it skips safely.

`Pending Documents Delivery` and `Cleared` are protected statuses: no TPD tag is applied to those Clearances.

**Result:** At Day 7, Critical is added without removing Delayed.

## 4. Clearances - Remove temporary Delayed tag

**Workflow module:** Clearances

**Trigger:** Status is modified.

**Instant action:** `Remove_SLA_Tag_Clearances` with the Clearance ID.

**Result:** Only `Delayed` is removed. `Critical` and all other tags remain untouched.

This is intentionally separate from the SLA application functions.

## 5. Haulages - Awaiting Stuffing

**SLA:** 2 days = Delayed; 4 days = Critical.

**Workflow module:** Haulages

**Trigger:** Status becomes `Awaiting Stuffing`.

**Scheduled actions:**
- +2 days: `Awaiting_Stuffing_Apply_SLA_Universal_Tag_Haulage_Critical` with `orecid = Haulage ID`, `otag = Delayed`, `expected_status = Awaiting Stuffing`, `module_name = Haulages`.
- +4 days: same function with `otag = Critical`.

**Result:** Delayed and Critical may coexist.

## 6. Haulages - Awaiting Drop-off

**SLA:** 4 days = Delayed; 7 days = Critical.

**Workflow module:** Haulages

**Trigger:** Status becomes `Awaiting Drop-off`.

**Scheduled actions:**
- +4 days: `Awaiting_Dropoff_Apply_SLA_Universal_Tag_Haulage_Critical` with `orecid = Haulage ID`, `otag = Delayed`, `expected_status = Awaiting Drop-off`, `module_name = Haulages`.
- +7 days: same function with `otag = Critical`.

**Result:** Delayed and Critical may coexist.

## 7. Haulages - Stuffing Paperwork Pending

**SLA:** +2 days = Delayed; +4 days = Critical.

**Workflow module:** Haulages.

**Trigger:** Haulage Status becomes `Awaiting Drop-off`.

**Scheduled actions:**
- +2 days: `Stuffing_Paperwork_Pending_SLA_Delayed_Haulages` with `haulage_id = Haulage ID`, `severity = Delayed`.
- +4 days: same function with `severity = Critical`.

The function checks the Booking, determines whether another active Haulage remains, finds the Clearance through the Booking lookup, and only proceeds when the Clearance is `Stuffing Paperwork Pending`.

**Important exception:** For this specific SLA, the agreed rule is that Critical replaces Delayed. Therefore the Critical execution removes Delayed while preserving all other tags.

## 8. Haulages - Remove temporary Delayed tag

**Workflow module:** Haulages

**Trigger:** Status is modified.

**Instant action:** `Remove_SLA_Tag_Haulages` with the Haulage ID.

**Result:** Only `Delayed` is removed. `Critical` and all other tags remain untouched.

## Function responsibility

| Function | Timer | Target | Critical removes Delayed? |
|---|---:|---|---|
| Unallocated Clearances | 3 / 5 days | Clearance | No |
| PPD Pending | 2 / 5 days | Clearance | No |
| TPD Pending | 4 / 7 days | Linked Clearance | No |
| Awaiting Stuffing | 2 / 4 days | Haulage | No |
| Awaiting Drop-off | 4 / 7 days | Haulage | No |
| Stuffing Paperwork Pending | 2 / 4 days | Linked Clearance | **Yes** |
| Remove Clearances Delayed | On status change | Clearance | Removes Delayed only |
| Remove Haulages Delayed | On status change | Haulage | Removes Delayed only |

## Testing recommendation

For testing, temporarily use a short scheduled delay such as 2 minutes in the workflow UI. Do not change the Deluge SLA logic to calculate elapsed time. Once testing passes, restore the production day values.

After each scheduled action, verify:
1. The record is still in the expected status/stage.
2. The requested tag exists.
3. Existing business tags remain.
4. Critical is preserved where required.
5. Delayed is removed only by the dedicated removal workflow, except for Stuffing Paperwork Pending where Critical intentionally removes it.

Zoho documents that workflow time-based actions can invoke custom functions and that scheduled actions are evaluated from the workflow trigger/date configuration. citeturn3search3
