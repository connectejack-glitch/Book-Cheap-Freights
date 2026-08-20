# TPD Pending SLA

## Business Logic

TPD Pending is a Clearance SLA anchored to the related **Booking**, not to the time the Clearance itself enters a particular status.

The Booking is represented by a Deal in Zoho CRM.

### SLA Flow

**Booking → OBL Collected → timer starts → linked Clearance is checked**

- After **4 days** from OBL Collected, add **Delayed** to the linked Clearance.
- After **7 days**, add **Critical** to the linked Clearance.
- At Day 7, **Delayed remains** and Critical is added.
- If the Clearance status is **Pending Documents Delivery** or **Cleared**, do nothing.

## SLA Anchor

The agreed business event is the Booking reaching **OBL Collected**.

The implementation can use the Booking stage transition as the workflow trigger. The `OBL_Received_Date_Time` field has also been identified as a field that closely represents the OBL Collected event and should only become the authoritative anchor if the client confirms that requirement.

## Relationship Logic

The Booking does not necessarily have a Clearance at the moment it reaches OBL Collected.

When the scheduled function runs, it should:

1. Retrieve the Booking.
2. Confirm the Booking is still relevant to the SLA.
3. Find the Clearance through the Clearance's **Booking lookup** to the Deal.
4. Retrieve the current Clearance status.
5. Stop if the Clearance is Pending Documents Delivery or Cleared.
6. Apply Delayed or Critical as appropriate.

The Clearance's Job relationship should not be used as the primary route for this SLA because one Job can contain multiple Bookings.

## Workflow Design

### Trigger

**Module:** Deals / Bookings

**Event:** Booking stage becomes `OBL Collected`.

### Scheduled Actions

- **4 days after trigger:** TPD Pending — Delayed
- **7 days after trigger:** TPD Pending — Critical

The workflow schedules the checks. The Deluge function performs the live validation and tag operation.

## Important Guardrail

The scheduled function must check the current Clearance status before applying a tag. Reaching the scheduled time alone is not enough to qualify the Clearance for the SLA.
