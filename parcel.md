# Parcel lines (UPS and FedEx)

UPS and FedEx. Order grain, not line. Most of it ships direct from the factory to the end customer and never touches the warehouse, so the carrier's delivery date is the customer's delivery date. Key columns:

- `carrier`, `tracking_no`, `status_norm` (LABEL, IN_TRANSIT, OUT_FOR_DELIVERY, EXCEPTION)
- `is_drop_ship` - 1 means direct to customer; 0 means to the DRI warehouse, and two days are added for cross-dock and outbound
- `scheduled_delivery` and `scheduled_source` - the arrival date, and whether it came from the carrier or is an estimate (ship date plus five) because the carrier has not published one yet
- `customer_eta`, `promise` - what gets compared
- `exception_class` and `exception_text` - DELAY, CUSTOMS, OTHER, or NONE, with the carrier's own words
- `disposition` - EXCEPTION, LATE_DELIVERY, NOT_SHIPPED, IN_TRANSIT_LATE, IN_TRANSIT_TIGHT, ON_TIME

How to write about parcel:
- When `scheduled_source = estimate`, say "estimated" and never present the date as the carrier's commitment. "Estimated 10/5, FedEx has not published a date."
- `IN_TRANSIT_TIGHT` is a watch, not an action. Say when to look again - the day the carrier should publish.
- `LATE_DELIVERY` means the carrier is late against its own date. That is the carrier's problem to explain; say so.
- `NOT_SHIPPED` means a label exists and the package has not moved. The factory has it, not the carrier.
- `EXCEPTION` - lead with the carrier text. Customs goes to whoever handles brokerage; delay goes to the merchandiser to decide whether the customer needs a heads-up.
- There is no divert for parcel. Do not suggest one.
- Parcel moves in days. A one-day-old package with no carrier date yet is normal. Do not escalate it.
