# BCF SLA Testing Guide

## Testing Approach

Each SLA should be tested first with a short timer, such as two minutes, instead of waiting the full production period.

The short timer is only a testing substitute. Production workflows must use the agreed business thresholds.

## Test Categories

### 1. Happy Path

Confirm that a record remains in the qualifying state until the scheduled action runs and receives the expected tag.

### 2. Early Status Change

Change the record to a non-qualifying status before the scheduled action. The function must stop without applying the old SLA tag.

### 3. Delayed to Critical

Confirm that the Critical threshold produces the agreed escalation behavior for the specific SLA.

### 4. Relationship Validation

For cross-module SLAs, confirm that the correct Booking, Haulage, and Clearance relationship is used.

### 5. Excluded Clearance Statuses

For TPD Pending, confirm that Pending Documents Delivery and Cleared prevent SLA tagging.

### 6. Multiple Haulages

For Stuffing Paperwork Pending, test a Booking with multiple Haulages and confirm that the last-Haulage condition is respected.

## Test Evidence

Record the following for each test:

- Record ID
- Initial status/stage
- Trigger event
- Scheduled threshold
- Expected result
- Actual result
- Tags before execution
- Tags after execution
- Function logs
- Pass/fail result

## Regression Rule

After correcting a Deluge error, repeat both the failing scenario and the relevant happy-path scenario. A technical fix should not change the agreed business logic unintentionally.
