# BCF Haulages — SLA & Tagging

## Module

- Zoho CRM module: **Haulages**
- Status field: Haulage `Status`

## Haulage Status & Automation

| Status / Process | Trigger / Anchor | Delayed | Critical | Status |
|---|---|---:|---:|---|
| Unallocated | Haulage creation / status | 3 days | — | Existing implementation |
| Awaiting Stuffing | Entry into status / Arrival Date anchor | 2 days | 4 days | Completed |
| Awaiting G2G | Blueprint transition from Awaiting Stuffing | — | — | Process stage documented; no SLA configured |
| Awaiting Drop-off | Entry into status / Stuffing Date anchor | 4 days | 7 days | Completed |
| Awaiting Empty Pick-up | Allocated Date | 3 days | TBD | In Progress |
| En-route Customer | Entry into status | 2 days | 4 days | Existing implementation |

## Awaiting Drop-off

- 4 days → `Delayed`.
- 7 days → `Critical`.
- The function checks the LIVE Haulage Status before applying the requested severity.
- If the Haulage has already moved out of `Awaiting Drop-off`, the scheduled action stops.

## Awaiting Stuffing

- 2 days → `Delayed`.
- 4 days → `Critical`.
- Validate that the live Haulage remains in `Awaiting Stuffing`.

## Awaiting G2G

`Awaiting G2G` is a process stage between `Awaiting Stuffing` and `Awaiting Drop-off`.

It is entered through the **Stuffing Completed** Blueprint action and leaves through the **G2G** action to `Awaiting Drop-off`.

The stage includes the G2G confirmations and G2G Cut-off Date documented in the project requirements.

No separate SLA timing has been confirmed for Awaiting G2G. citeturn37file19

## Awaiting Empty Pick-up

- Anchor: `Allocated Date`.
- Target status: `Awaiting Empty Pick-up`.
- 3 days → `Delayed`.
- Critical threshold: not yet confirmed.

Core validation:

Allocated Date

→ Wait for the workflow SLA interval

→ Get LIVE Haulage

→ Check Status

→ `Awaiting Empty Pick-up` = apply tag

→ Any other status = stop

## Implementation Rules

- Haulage = `Haulages` module.
- Workflow controls timing.
- Deluge validates the live Haulage status.
- Preserve unrelated tags.
- Scheduled actions must not tag a Haulage that has already moved to another status.
- Production timings should be used after testing; short delays may be used temporarily for testing.
