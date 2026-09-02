# BCF SLA Automation

Book Cheap Freights — Zoho CRM SLA, workflow and tagging implementation.

## Architecture

| Responsibility | Owned By |
|---|---|
| Trigger / schedule | Workflow Rule |
| Live record validation | Deluge |
| Business-rule validation | Deluge |
| SLA tag action | Deluge |

The core design is **Workflow = WHEN** and **Deluge = VALIDATION + BUSINESS LOGIC + ACTION**. This prevents stale scheduled actions from applying incorrect tags. citeturn37file0turn38file12

## Master SLA Matrix

| Module | Stage / Status | Trigger / Anchor | Delayed | Critical | Status |
|---|---|---|---:|---:|---|
| Deals / Bookings | Open | Stage enters Open | 7 days | 14 days | Completed |
| Deals / Bookings | Shipped | Stage enters Shipped | 9 days | 15 days | Completed |
| Deals / Bookings | Paid | Stage enters Paid | 3 days | 7 days | Completed |
| Deals / Bookings | Pending Shipment | Booking / container Gate-In milestone | 9 days | 15 days | In Progress |
| Deals / Bookings | Arrival Date tag | Arrival Date modified; Shipped or later | — | — | Completed |
| Clearances | Unallocated | Record creation / status | 3 days | 5 days | Completed |
| Clearances | CCI Pending | NESS Payment Date | 4 days | 8 days | Completed |
| Clearances | TPD Pending | Booking enters OBL Collected | 4 days | 7 days | Completed |
| Clearances | Pending Documents Delivery | Status entry | 2 days | 5 days | Completed |
| Clearances | Stuffing Paperwork Pending | Last Haulage exits Awaiting Stuffing | 2 days | 4 days | Completed / documented |
| Haulages | Unallocated | Status / creation | 3 days | — | Existing implementation |
| Haulages | Awaiting Stuffing | Arrival Date / status entry | 2 days | 4 days | Completed |
| Haulages | Awaiting G2G | Blueprint transition | — | — | No SLA configured |
| Haulages | Awaiting Drop-off | Stuffing Date / status entry | 4 days | 7 days | Completed |
| Haulages | Awaiting Empty Pick-up | Allocated Date | 3 days | TBD | In Progress |
| Haulages | En-route Customer | Status entry | 2 days | 4 days | Existing implementation |

## Booking Tags

### Managed tags

- `Check Arrival Date`
- `Check Departure Date`
- `Check Vessel Name`

Before the new Arrival Date workflow was implemented, all three managed tags were cleared from the CRM to establish a clean baseline.

### Tag behaviour

| Booking lifecycle | Tags |
|---|---|
| Before Shipped / Pending Shipment | `Check Departure Date`, `Check Vessel Name` |
| Shipped and later | `Check Arrival Date` |
| Cancelled | Remove all three managed tags |

`Check Arrival Date` remains relevant after Shipped through Verified Copy Approved, OBL Collected, OBL Issued to Client and File Closed.

## Booking Stages

The current documented Booking / Deals stages are:

- Open
- Pending Shipment
- Shipped
- Paid
- Verified Copy Approved
- OBL Collected
- OBL Issued to Client
- File Closed
- Cancelled

Zoho CRM allows Deal stages to be customized, so the live pipeline remains the source of truth for any future stage changes. citeturn0search0turn0search2

## Module Folders

- `BCF/SLA/Bookings/` — Booking / Deals SLA and tag logic
- `BCF/SLA/Clearances/` — Clearance SLA logic
- `BCF/SLA/Haulages/` — Haulage SLA logic

## Sources & Decisions

- The BCF implementation report confirms the Clearance SLA timings and the Haulage Awaiting Drop-off / Awaiting Stuffing timings. citeturn38file6turn38file15
- TPD Pending is anchored to the Booking entering OBL Collected and evaluates the linked Clearance. citeturn38file7turn38file10
- Awaiting G2G is a Haulage process stage between Awaiting Stuffing and Awaiting Drop-off; no separate SLA threshold is confirmed in the source. citeturn37file19
