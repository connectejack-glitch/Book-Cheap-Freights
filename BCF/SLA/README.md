# Book Cheap Freights — Haulages SLA

## Current agreed timing

| Status | Delayed | Critical | Anchor |
|---|---:|---:|---|
| Unallocated | 3 days | — | Created Time |
| Awaiting Empty Pick-up | 3 days | 7 days | Status Entry |
| En-route Customer | 2 days | 4 days | Status Entry |

## Architecture

The Workflow controls the SLA timing. The Deluge function receives haulage_id and severity, checks the live Haulage status, preserves existing tags, applies the requested SLA tag, removes Delayed when Critical is applied, and verifies the update.

For testing, scheduled actions can temporarily use +2 minutes instead of the production duration.
