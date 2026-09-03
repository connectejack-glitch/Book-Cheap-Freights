# BCF — Booking Stage SLA

## Progress

| Booking Stage | Delayed | Critical | Trigger / Anchor | Status |
|---|---:|---:|---|---|
| Unallocated | 4 days | 8 days | Record creation | Complete |
| Open | 7 days | 14 days | Stage entry | Complete |
| Pending Shipment | 9 days | 18 days | Last Container Gating In / Gate-In Completion Date | Pending |
| Shipped | 9 days | 15 days | Stage entry | Complete |
| Paid | 3 days | 7 days | Booking enters Paid | Complete |
| Verified Copy Approved | 2 days | 7 days | Booking enters stage | Complete |
| OBL Collected | 4 days | 7 days | Booking enters OBL Collected — TPD Pending | Complete |
| OBL Issued to Client | 3 days | 7 days | Booking enters stage | Complete |

## Standard Logic

- Booking is the **Deals** module.
- Stage API name is `Stage`.
- Workflow controls the SLA timing.
- Deluge checks the live Booking before applying a tag.
- Existing tags are preserved.
- `Delayed` and `Critical` are independent unless a specific SLA explicitly requires replacement.
- A scheduled action stops when the Booking no longer meets the applicable stage condition.

## Completed Booking Stage SLAs

### Unallocated
4 days → Delayed  
8 days → Critical

### Open
7 days → Delayed  
14 days → Critical

### Shipped
9 days → Delayed  
15 days → Critical

### Paid
3 days → Delayed  
7 days → Critical

### Verified Copy Approved
2 days → Delayed  
7 days → Critical

### OBL Collected — TPD Pending
The Booking entering `OBL Collected` is the anchor for the linked Clearance's TPD Pending SLA.

4 days → Delayed  
7 days → Critical

The linked Clearance is checked before tagging.

### OBL Issued to Client
3 days → Delayed  
7 days → Critical

The live Booking Stage must still be `OBL Issued to Client` when the scheduled function runs.

## Pending

### Pending Shipment
9 days → Delayed  
18 days → Critical

Anchor: **Last Container Gating In / Gate-In Completion Date**

This is the remaining Booking Stage SLA to implement.
