# Stuffing Paperwork Pending SLA

## Business Logic

This SLA connects the Haulage process to the related Clearance.

The important business event is the Haulage moving **out of Awaiting Stuffing**. The SLA should not be anchored simply to the time a Clearance status changes because the operational event originates from the Haulage process.

### SLA Flow

**Haulage exits Awaiting Stuffing → determine whether it is the last Haulage for the Booking → use the exit event as the SLA anchor → find the related Clearance → apply SLA tags**

The agreed thresholds are:

- After **2 days**, apply **Delayed**.
- After **4 days**, escalate to **Critical**.

## Last Haulage Rule

A Booking can have multiple Haulages. The SLA is only relevant when the Haulage that exits Awaiting Stuffing is the **last Haulage for the Booking**.

Therefore, the function must validate the Haulage/Booking relationship before applying the SLA.

## Workflow Design

The workflow should be tied to the operational event that starts the timer.

### Trigger

**Module:** Haulages

**Event:** Haulage moves out of `Awaiting Stuffing`.

### Scheduled Actions

- **2 days after the trigger:** run the function with severity `Delayed`.
- **4 days after the trigger:** run the function with severity `Critical`.

## Function Responsibility

The Deluge function should:

1. Validate the requested severity.
2. Retrieve the triggering Haulage.
3. Confirm the relevant stage transition occurred.
4. Determine whether this is the last Haulage for the Booking.
5. Establish the Haulage exit event as the SLA anchor.
6. Find the Clearance linked through the Booking.
7. Validate the current Clearance conditions required by the business rule.
8. Apply the requested SLA severity.
9. Stop safely when any required condition is not satisfied.

## Current Implementation Note

The function under development is `Stuffing_Paperwork_Pending_SLA(haulage_id, severity)`.

A previous test exposed a Deluge data-type issue when reading related records: the `get` operation received a value that did not match the expected BIGINT type. This should be resolved by ensuring IDs used with list/map access and CRM relationship calls are handled with the correct Deluge data type.

The function should retain detailed execution logs during testing so that relationship lookup and stage-history failures can be identified without changing the business logic.
