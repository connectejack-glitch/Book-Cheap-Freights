# BCF — Booking Stage SLA Progress

| Booking Stage | Delayed | Critical | Trigger / Anchor | Status |
|---|---:|---:|---|---|
| Unallocated | 4 days | 8 days | Record creation | Complete |
| Open | 7 days | 14 days | Stage entry | Complete |
| Pending Shipment | 9 days | 18 days | Last Container Gating In / Gate-In Completion Date | Pending |
| Shipped | 9 days | 15 days | Stage entry | Complete |
| Paid | 3 days | 7 days | Booking enters Paid | Complete |
| Verified Copy Approved | 2 days | 7 days | Booking enters stage | Complete |
| OBL Collected / TPD Pending | 4 days | 7 days | Booking enters OBL Collected | Complete |
| OBL Issued to Client | 3 days | 7 days | Booking enters stage | Complete |

## Completed

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
Booking enters OBL Collected → SLA anchor for the linked Clearance.

4 days → Delayed  
7 days → Critical

The linked Clearance is checked before tagging.

### OBL Issued to Client
3 days → Delayed  
7 days → Critical

The live Booking Stage must still be OBL Issued to Client when the scheduled function runs.

## Remaining

### Pending Shipment
9 days → Delayed  
18 days → Critical

Anchor: Last Container Gating In / Gate-In Completion Date.
