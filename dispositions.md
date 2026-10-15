# Dispositions

Every scored line carries one. It says which decision is still open, who owns it, and what to do in SAP.

## DIVERT_WINDOW
**Owner:** department leaders.
**Means:** on the water, more than 72 hours from vessel ETA, and at least one line on the box is late by rail but on time by truck.
**Action:** decide truck vs rail for the container. The divert queue shows orders saved, units saved, customer POs. Containers sharing a vessel and ETA are one decision.
**Set in SAP:** on the GRPO, Divert Decision = DIVERT or HOLD, and Divert Decision ETA = vessel ETA at the time.
**Clears when:** either decision is recorded. DIVERT also moves every line to the truck arrival date.
**Returns when:** HOLD only - vessel ETA slips 2+ days past Divert Decision ETA. Marked RE-FIRED.

## NOTIFY_EARLY
**Owner:** merchandiser (route_to).
**Means:** on the water, divert window open, but late by rail AND by truck. These were late at booking. Nothing in logistics fixes them.
**Action:** call the customer now. Do not wait for discharge. Give the projected arrival plus outbound transit.
**Set in SAP:** on the order line, Customer Commit Date = the date you gave the customer.
**Clears when:** Customer Commit Date is set and warehouse ETA is on or before it.
**Returns when:** warehouse ETA slips 2+ days past Customer Commit Date. Marked STALE. Second call.

## PRE_DISCHARGE
**Owner:** nobody - visibility only.
**Means:** on the water, window closed, not projected late. Committed to rail, on schedule.
**Action:** none. If ETA drifts late it becomes NOTIFY at discharge.

## NOTIFY
**Owner:** merchandiser (route_to).
**Means:** discharged, committed to rail, warehouse ETA past what the customer was told.
**Action:** call the customer with the revised arrival. If the box must turn same-day on landing, set Priority so receiving knows.
**Set in SAP:** on the order line, Customer Commit Date. On the order, Priority if expedited outbound is needed.
**Clears when:** Customer Commit Date is set and warehouse ETA is on or before it.
**Returns when:** warehouse ETA slips 2+ days past Customer Commit Date. Marked STALE.

## ARRIVING
**Owner:** merchandiser and receiving.
**Means:** at destination rail terminal or out for delivery. At the warehouse within a day or two.
**Action:** confirm Priority is set if late. Nothing else upstream.
**Clears when:** rail discharged 5+ days ago or status reaches Empty Returned. Drops from scoring.

## Affiliate stock lines in the queues

A BCS or DDT stock line follows the same dispositions, but the action changes. There is no end customer to call. For NOTIFY and NOTIFY_EARLY on stock lines, the action is to tell the affiliate's buyer the revised arrival and let them decide whether to reorder from elsewhere or wait. If `reasons` says they have cover, there is nothing to tell them. If it says out of stock, that call is urgent.

In the divert queue, out-of-stock affiliate lines strengthen the case to truck. Covered lines do not.

## Parcel dispositions

Parcel has its own set. No divert - a package does not get re-routed mid-flight.

**EXCEPTION** - the carrier has flagged a problem. Customs: brokerage. Delay: merchandiser decides whether the customer needs a heads-up. Lead with the carrier's text.

**LATE_DELIVERY** - the carrier's published date has passed and the package is not delivered. The carrier is late against itself. Merchandiser calls the customer; the carrier owes an explanation.

**NOT_SHIPPED** - label created, no movement after two days. The package is still at the factory. Whoever manages that factory relationship asks why.

**IN_TRANSIT_LATE** - moving, but the arrival date is past the promise. Merchandiser calls the customer with the new date. Say whether the date is the carrier's or an estimate.

**IN_TRANSIT_TIGHT** - moving, arrival within a day of the promise. Watch, do not act. Look again when the carrier publishes a real date.

**ON_TIME** - moving with room to spare. Not shown.

## What a merchandiser sees
The merch queue shows OPEN and STALE lines only - no Customer Commit Date, or the ETA has moved past it. Everything acknowledged and still accurate is hidden. Setting Customer Commit Date clears a line and is also what brings it back if the date later proves wrong.

## What a leader sees
The divert queue sorted by hours to deadline, only containers where truck saves something. A rollup by merchandiser (open items, re-fired) and by container. Re-fired is the count to watch - handled and got worse anyway.
