# Book Cheap Freights (BCF)

Documentation and implementation repository for the Book Cheap Freights Zoho CRM automation project.

## Purpose

This repository keeps the agreed business logic, SLA rules, workflow design, Deluge implementation notes, and testing guidance for the BCF Zoho CRM setup in one place.

## Current Scope

- Haulages — Awaiting Drop-off SLA
- Haulages — Awaiting Stuffing SLA
- Clearances — TPD Pending SLA
- Clearances / Booking — Stuffing Paperwork Pending SLA

## Business Modules

### Bookings
Bookings are represented by the Deals module in Zoho CRM. Booking stage changes can act as the business event that starts a cross-module SLA.

### Haulages
Haulages represent individual transport activities associated with a Booking. SLA logic can depend on the Haulage status and, where required, whether the Haulage is the last Haulage associated with the Booking.

### Clearances
Clearances hold clearance-related work and SLA tags. Some SLA timers are driven by Clearance status; others are anchored to an upstream Booking or Haulage event.

## SLA Severity Model

- **Delayed** — the first SLA threshold has been exceeded.
- **Critical** — the escalation threshold has been exceeded.

Where the business rule requires it, Critical does not automatically remove Delayed. Status-change workflows may handle tag removal separately.

## Implementation Principle

The workflow controls **when** an SLA check runs. The Deluge function controls **whether** the tag is allowed to be applied by validating the current record state and relevant relationships before making changes.

## Repository Structure

```text
Book-Cheap-Freights/
├── README.md
├── docs/
│   ├── SLA-Overview.md
│   ├── TPD-Pending.md
│   ├── Stuffing-Paperwork-Pending.md
│   └── Awaiting-Drop-off.md
├── deluge/
│   └── README.md
└── testing/
    └── README.md
```

## Status

Initial project documentation is being established. Deluge functions and additional implementation artifacts should be added as they are finalized and tested in Zoho CRM.
