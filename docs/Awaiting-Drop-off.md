# Awaiting Drop-off SLA

## Business Logic

A Haulage enters **Awaiting Drop-off** and an SLA timer begins.

- After **4 full days** in Awaiting Drop-off, apply the **Delayed** tag.
- After **7 full days** in Awaiting Drop-off, remove **Delayed** and apply **Critical**.
- If the Haulage changes to another status before the relevant SLA check, the scheduled function must stop and make no tag change.

## Workflow

The workflow is responsible for scheduling the checks from the point the Haulage enters Awaiting Drop-off.

### First Check

**Timing:** 4 days after the workflow trigger.

**Action:** Run the SLA function with severity `Delayed`.

### Escalation Check

**Timing:** 7 days after the workflow trigger.

**Action:** Run the SLA function with severity `Critical`.

## Function Responsibility

The function should:

1. Retrieve the Haulage record.
2. Confirm that its current status is still Awaiting Drop-off.
3. Stop if the status has changed.
4. Apply the requested severity tag.
5. For Critical, remove Delayed before adding Critical, according to the agreed BCF rule.

## Safety Guardrail

The status check is mandatory. The function must not rely only on the original workflow trigger because the record may have changed before the scheduled action executes.
