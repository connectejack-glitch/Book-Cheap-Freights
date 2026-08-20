# BCF SLA Overview

## 1. Purpose

The BCF SLA framework monitors records that remain in operational stages for longer than the agreed business timeframe.

The purpose is to make overdue work visible without allowing an old scheduled action to tag a record that has already moved on.

## 2. Severity Levels

### Delayed

Applied when the first agreed SLA threshold is reached.

### Critical

Applied when the escalation threshold is reached.

Critical is an escalation state. For TPD Pending, the agreed rule is that Critical is added while Delayed may remain present. For other SLAs, the exact tag transition should follow the individual SLA definition.

## 3. General Pattern

The standard pattern is:

1. A business event starts the SLA timer.
2. A workflow schedules the SLA checks.
3. The scheduled Deluge function retrieves the current record.
4. The function confirms that the record is still in the expected business state.
5. The function validates any required Booking, Haulage, or Clearance relationship.
6. The function applies the appropriate SLA tag.
7. If the record no longer satisfies the business condition, the function stops without changing the record.

## 4. Important Design Rule

The workflow determines **when** to run the check. The function determines **whether** the SLA condition is still valid.

This guardrail is required because a scheduled action can fire after a user has already changed the record.

## 5. Status Changes and Tag Removal

Tag removal should be handled by explicit status-change logic where required. An SLA escalation function should not assume that the record is still overdue simply because the scheduled action was created earlier.

## 6. Testing

For development testing, the production day thresholds can be temporarily represented by a short timer such as two minutes. After validation, the workflow should be returned to the actual business SLA periods.
