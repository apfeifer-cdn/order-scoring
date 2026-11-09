# Fields in the scored table

## Identity and routing
| Column | Meaning |
|---|---|
| Container_Number | ISO container number. Air and parcel references are excluded. |
| ref_type | CONTAINER. (AIR_PARCEL rows are filtered out before scoring.) |
| VesselName, FirstUSPort, pod_terminal_name | Vessel and discharge port/terminal. |
| ORDR_DocNum, RDR1_LineNum, RDR1_StyleNumber | Sales order, line, item. |
| customer_po | The customer's own PO number (ORDR.NumAtCard). The lookup key when a user asks about an order. |
| DRI_po | U_POPrefix + DRI sales order DocNum. DRI's reference as printed on documents. |
| CardCode, CardName | Customer. |
| SO_Line_Qty | Units on this line. |
| owner_email, owner_name | Sales order owner from OHEM. Often the generic Manager account. |
| item_merchandiser | OITM.U_Merchandiser. A name. |
| route_to | Owner email unless missing or generic, else item merchandiser. The routing key. |
| factory | Production report source (Unimas, RTN, AYM, UNIP). |

## Dates
| Column | Meaning |
|---|---|
| Line_Quoted_Delivery | Original customer promise. Never overwritten. |
| cust_commit_dt | Date most recently told to the customer (RDR1.U_CustomerCommitDt). Null until set. |
| effective_promise | cust_commit_dt if set, else BCS required date, else ddt_box_earliest_ship, else DDT PO date, else Line_Quoted_Delivery. What lateness is measured against. |
| aff_req_date | Affiliate lines. Customer POs: their sales order ReqDate. Stock POs: their PO ReqDate, else DocDueDate. |
| aff_cancel_date | Affiliate lines. Last possible date; past it the order is dead. |
| segment | CDN, DDT, or BCS. From CardCode. Every line has one. CDN = DRI's own customers. |
| affiliate | BCS or DDT when the affiliate join succeeded. Null otherwise. segment is the reliable one for filtering; affiliate tells you the affiliate dates and inventory are present. |
| aff_orig_ship | BCS customer lines only. Original ship date on the BCS sales order. |
| aff_po_type | customer = PO raised against an affiliate sales order; stock = inventory replenishment. All DDT POs are stock. |
| aff_item | The affiliate's item code. BCS codes differ from DRI's in colour and size; DDT codes match exactly. |
| aff_on_hand | Stock lines. Units on hand at the affiliate across warehouses. |
| aff_committed | Stock lines. Units committed to the affiliate's open sales orders. |
| aff_available | on_hand minus committed. At or below zero = affiliate is out of stock on this style. |
| aff_daily_velocity | Stock lines. Affiliate's 90-day invoiced units / 90. |
| aff_days_cover | available / daily velocity. Days the affiliate can sell before needing this box. Null when no recent sales. |
| SO_OriginalDelivery, SO_RevisedDelivery | Internal plan dates (U_OrigDelivery, U_RevDelivery). |
| current_vessel_eta, baseline_vessel_eta, vessel_eta_drift | Vessel ETA now, first observed, and the difference in days. |
| current_rail_eta, baseline_rail_eta, rail_eta_drift | Same for rail. |
| PODUnloaded | Discharged from vessel. Starts the dwell clock. |
| RailATA, RailDischarged | Rail arrived at destination ramp; discharged from rail. |
| dock_eta | Projected warehouse arrival. See dock_eta_source. |
| dock_eta_source | gnosis if from gnosis_estimated_dc_eta_dt, else derived. |
| U_EventDt, U_Event, U_EventDateType | Event date, flag (Y/N), firm or flexible. |

## State
| Column | Meaning |
|---|---|
| Status | Gnosis status: On Water, Awaiting Discharge, At Ocean Terminal, Awaiting Rail Load, On Rail, At Rail Terminal, Out for Delivery. |
| Location | Port or city from TrackLocation. Null if not present. |
| IsRailing, IsRailing_Source | Whether a rail leg exists. inferred until is_railing is exported. |
| Mode | SEA or AIR. |
| days_since_discharge, days_in_status, days_since_seen | Clocks. |
| is_active | 0 when terminal status or rail discharged 5+ days ago. Those rows are excluded. |
| U_DivertDecision, U_DivertDecisionETA | Leader decision on the GRPO: DIVERT or HOLD, and vessel ETA at the time. |
| CustomsClearDate, ff_present | From the forwarder import. ff_present = 1 means the import has a row for this container. |

## Scores
| Column | Meaning |
|---|---|
| r1_dwell … r12_customs | Each rule's score, 0-100. |
| r13_event_mult | 1.0, 1.5, or 99.0 (hard flag sentinel). |
| r14_cancel_mult | Same shape for affiliate cancel date. The larger of r13 and r14 applies. |
| max_rule | Highest rule score. |
| strong_signals | Count of rules over 40. |
| base_score | max_rule plus corroboration bump times the larger multiplier, capped at 100. Before BCS adjustments. |
| risk_score | Final 0-100. base_score with the affiliate stock ceiling (50 if covered) or floor (70 if out of stock) applied. |
| risk_band | CRITICAL, HIGH, WATCH, OK, EVENT MISS, or CANCEL DATE MISS. |
| primary_signal | Which rule produced max_rule. untracked when Gnosis has no row. |
| reasons | Plain-language explanation of every rule that fired. Use verbatim. |

## Disposition and divert
| Column | Meaning |
|---|---|
| disposition | DIVERT_WINDOW, NOTIFY_EARLY, PRE_DISCHARGE, NOTIFY, ARRIVING. See dispositions.md. |
| hours_to_divert_deadline | Hours until 72h before vessel ETA. Pre-discharge only. |
| truck_saves_line | 1 if late by rail projection and on time by truck (vessel ETA + 4). |
| container_orders, container_units | Totals for the whole container this line rides on. |
| cdn_orders, ddt_orders, bcs_orders | Divert queue only. Orders on the container by segment. |

## Parcel output (separate table)
| Column | Meaning |
|---|---|
| carrier | UPS or FedEx. |
| tracking_no | Carrier tracking number. One row per package per order. |
| status_norm | LABEL, IN_TRANSIT, OUT_FOR_DELIVERY, EXCEPTION. Delivered and void are excluded. |
| ship_date | Manifest (UPS) or ship (FedEx) date. |
| scheduled_delivery | Arrival date. Carrier-published when available, otherwise ship date + 5. |
| scheduled_source | carrier or estimate. Say which when you quote the date. |
| delivered_date | Null while in transit. |
| ship_to, is_drop_ship | Recipient. 1 = direct to customer, 0 = DRI warehouse. |
| exception_class, exception_text | DELAY, CUSTOMS, OTHER, NONE. Carrier text. "Import COD charges paid" is a note, not an exception. |
| so_docnum, ORDR_DocEntry, CardCode, CardName, customer_po, DRI_po, route_to | Order and routing, same meaning as the ocean table. |
| promise | Earliest line quoted delivery on the order. |
| customer_eta | scheduled_delivery for drop-ship; + 2 days for warehouse. |
| first_scheduled, scheduled_drift | First carrier-published date and days moved since. Null while only estimates exist. |
| status_since, days_in_status | When the current status began and how long it has held. |
| p1_late … p6_not_moving | Rule scores. |
| risk_score, risk_band, primary_signal, reasons, disposition | Same meaning as the ocean table. |
