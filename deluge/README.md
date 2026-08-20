# Deluge Implementation

This folder is reserved for the production Deluge functions used by the BCF SLA automations.

## Current Functions Discussed

### Awaiting Drop-off

Responsible for applying the Awaiting Drop-off SLA tags after validating the current Haulage status.

### TPD Pending

Responsible for finding the Clearance associated with a Booking and applying the correct SLA tag after validating the Clearance status.

### Stuffing Paperwork Pending

Responsible for validating the Haulage exit from Awaiting Stuffing, confirming the last-Haulage condition, finding the related Clearance, and applying the SLA severity.

## Function Design Rule

Functions should validate current CRM data before modifying tags. They should not assume that the record is still in the state that existed when the workflow scheduled the function.

## Production Code Policy

Only tested and approved Deluge should be stored as production implementation files. Working snippets, experiments, and debugging versions should be clearly labelled as such rather than mixed with the production functions.
