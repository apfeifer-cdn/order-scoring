# Scoring rules

Every open order line on an active inbound container gets a score 0-100. Eleven rules each produce a score; the line takes the highest, plus 10 if two unrelated rules both exceed 40. Rules about a container apply to every line on it.

Critical 70+ · High 40-69 · Watch 20-39 · OK under 20. Two more bands override the number: EVENT MISS and CANCEL DATE MISS mean the order can no longer be fulfilled on time. Report them first.

| Rule | What it means | Score |
|---|---|---|
| Dwell | Off the ship, still at the port. Rail-bound: 45 at 6 days, +9/day. Truck: 40 at 5 days, +10/day. | to 100 |
| Rail ETA drift | Rail arrival estimate has moved later than first published. 40 at 5 days late, +6/day. | to 90 |
| Vessel ETA drift | Vessel arrival estimate has moved later. 35 at 5 days, +7/day. | to 85 |
| Silence | Gnosis has stopped reporting on the container. 25 at 4 days, +15/day. | to 100 |
| Missed transship | Off ship A at Colombo or Singapore, waiting for ship B. Ship B's departure has passed and the box is still on the quay. 70, +15/day. Typically a 7-14 day hit. | to 100 |
| Ex-factory move | Factory pushed the ship date later, before the date, container not yet shipped. 90 inside 3 days, 75 inside 7, 40 inside 14. Only 12% of moves land inside 7 days. | to 90 |
| Missed quoted date | Warehouse ETA is past the effective promise - the date told to the customer, or for BCS lines the date BCS needs it at the warehouse, or the original quote. 40 when late, +4/day. Under 7 days of buffer scores 15. Affiliate stock lines: 40 flat if they have cover through the delay, cap 70 if some stock, full if out of stock. | to 100 |
| Missed planned date | Warehouse ETA is past our internal SO delivery date. 25, +3/day. | to 60 |
| No warehouse ETA | We cannot project arrival. Not the same as no risk. 35, or 50 if no discharge date either. 40 if Gnosis has no row for the container (untracked). | 35-50 |
| Customs hold | Discharged 3+ days, still at port, forwarder import has a row but no clearance date. | 70 |
| Event deadline | Firm event date on the order. Within 7 days multiplies by 1.5. Past it is a hard flag. | x1.5 / 100 |
| Affiliate cancel date | BCS and DDT lines. Last date the goods can arrive. Within 7 days multiplies by 1.5. Past it is a hard flag - the order is dead. | x1.5 / 100 |

## Parcel rules

Separate set. The clock is days, not weeks, so thresholds are tighter. Delivered and void packages are excluded before scoring.

| Rule | What it means | Score |
|---|---|---|
| Late to promise | Arrival date is past the customer promise. 40 when late, +10/day. Within a day of the promise scores 15, or 25 if the date is only an estimate. | to 100 |
| Past scheduled | The carrier's own delivery date has passed and the package is not delivered. The carrier is late against itself. | 85 |
| Exception | Carrier has flagged a problem. Customs 75, delay 70, other 50. | 50-75 |
| Scheduled drift | Carrier moved its published date later. 35 at 1 day, +15/day. Only measured between carrier dates, never against an estimate. | to 85 |
| Silence | In transit with no status change across 3 daily pulls. | 50 |
| Not moving | Label created, nothing scanned after 2 days. The factory has it, not the carrier. | 60 |
| Event deadline | Same multiplier as ocean. | x1.5 / 100 |

Same combination: max of rules, +10 when two exceed 40, multiplier last.

## Why the caps differ

Capped at 100: the rule is the outcome or one step from it - the customer is late, the box is sitting, the connection is gone, the container is lost.

Capped at 85-90: strong leading indicators that are still forecasts about forecasts. Rail drift (median 2 days, p90 11). Ex-factory move.

Capped at 60-85: weaker. Vessel drift mostly recovers (median 0). Internal plan is a plan, not a promise.

Flat: no-ETA keeps lines visible rather than ranking them. Customs hold is an exception with its own owner.

Multipliers: event date and BCS cancel date do not score on their own. They amplify whatever the other rules say, and past the date they force 100.

A line at 85 from one rule reads as "watch." The same line with a second rule over 40 gets +10 and reads as "act."

## Affiliate stock adjustments

Applied after everything else, to BCS and DDT lines where `aff_po_type = stock` and the line is late.

- **Cover through the delay** - `aff_days_cover` is at least the days late. Final score capped at 50. The container may be stuck, but this line does not hurt anyone.
- **Out of stock** - `aff_available` is zero or less. Final score floored at 70. The affiliate has orders against empty shelves.

Customer-backed affiliate lines and all other customers are untouched by these.

## Provisional

Thresholds come from six months of DRI container data and are starting values. The shadow quarter compares flags against what actually shipped late before anyone acts on a score.
