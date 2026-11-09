# Fields in the Orders table

Produced by `orders_table.sql` against SAP B1. Open sales orders plus orders closed in the last 30 days, lines with a factory PO only. One row per sales order line. When a delivery has been posted against the line, one row per delivery line instead. A line delivered in two shipments is two rows.

## Identity
| Column | Source | Meaning |
|---|---|---|
| entity | constant | DRI. DDT and BCS end-customer orders are not in the table yet. |
| ORDR_DocNum, RDR1_LineNum | ORDR, RDR1 | Sales order and line. |
| customer_po | ORDR.NumAtCard | The customer's PO. Primary lookup key. |
| DRI_po | U_POPrefix + DocNum | DRI's reference as printed. Second lookup key. |
| CardCode, CardName | ORDR | Customer. Affiliate customers are named "BCS - Apparel", "DDT - Stock Apparel" and so on. |
| item, item_desc | RDR1 | Style and description. |
| qty_ordered, qty_delivered, qty_open | RDR1 | Units. |
| order_status | ORDR.DocStatus | O or C. Closed rows stay 30 days after UpdateDate. |
| segment | CardName prefix | CDN, DDT, or BCS. BCS and DDT rows are DRI selling to the affiliate, not the affiliate's end customer. |

## Dates
| Column | Source | Meaning |
|---|---|---|
| Line_Quoted_Delivery | RDR1.ShipDate | Original promise. |
| cust_commit_dt | RDR1.U_CustomerCommitDt | Date last told to the customer. Null until set. |
| effective_promise | derived | cust_commit_dt if set, else Line_Quoted_Delivery. |
| orig_ex_factory_dt | ORDR.U_OrigShipDt | Original ex-factory date. Never overwritten. |
| act_ex_factory_dt | ORDR.U_ActShipDt | Actual ex-factory date. Null until the factory ships. |
| orig_delivery_dt | ORDR.U_OrigDelivery | Original delivery date. Never overwritten. |
| rev_delivery_dt | ORDR.U_RevDelivery | Revised delivery date, internal plan. |
| order_ship_type | OSHP via ORDR.TrnspCode | Shipping type on the order. "Air -" prefixes mean the shipment may go factory-direct to the customer. |

## Factory PO and GRPO
| Column | Source | Meaning |
|---|---|---|
| po_docnum, po_due_dt | OPOR, POR1 | Factory PO raised from this line and its line due date. Always present - lines with no PO are excluded. Latest PO when more than one. |
| po_rev_delivery | OPOR.U_RevDelivery | Best estimate of arrival at DRI while the goods are still at the factory. |
| factory_code, factory | OPOR.CardCode, CardName | The vendor on the PO. J&F, Ratone, Mexico (VM00797) and so on. Name it in every answer. |
| ships_direct | derived | 1 when factory_code = VM00797. Mexico ships straight to the customer and never comes to DRI. Report only original ex-factory, sales order revised delivery, PO revised delivery, and line tracking. |
| date_conflict | derived | 1 when the sales order's revised delivery is earlier than its ex-factory date. The revised delivery is wrong; say so. |
| grpo_docnum, grpo_date | OPDN | Latest GRPO against the PO line. A GRPO is created when the goods leave the factory, so it means in transit, not received. |
| grpo_rev_delivery | OPDN.U_RevDelivery | Arrival at DRI per the GRPO. Governs once a GRPO exists. |
| clm_comments | OPDN.U_CLMComments | Gnosis status text for the shipment. Quote verbatim. |
| container_no | OPDN.U_ContRefNo | Container number. Lookup key and the reference Ops uses in email. |

## Line tracking
| Column | Source | Meaning |
|---|---|---|
| line_tracking | RDR1.U_LineTrackNo and RDR1.U_tracking, joined with a comma | Tracking numbers recorded on the sales order line itself. Mexico direct-ship UPS numbers live here. Comma-separated when several. Quote as written; link each one that fits a UPS or FedEx format. |

## Governing date
| Column | Meaning |
|---|---|
| governing_eta | dln_due_dt, else grpo_rev_delivery, else po_rev_delivery, else po_due_dt. The one date to quote. |
| days_past_eta | Days today is past governing_eta, set only while no delivery is posted. Null means not overdue. A value means the date is stale, not a forecast. |

## Which date governs
Delivery `dln_due_dt` if a delivery exists. Else `grpo_rev_delivery` if a GRPO exists. Else `po_rev_delivery`, else `po_due_dt`. Always alongside `orig_ex_factory_dt` and `orig_delivery_dt` so the slip is visible.

## Delivery leg
| Column | Source | Meaning |
|---|---|---|
| dln_docnum, dln_line | ODLN, DLN1 | Delivery document and line. Null until posted. |
| dln_date | ODLN.DocDate | Ship date from DRI, or factory ship date for factory-direct. |
| dln_due_dt | ODLN.DocDueDate | Delivery due date to the customer. Governs when present. |
| dln_qty | DLN1.Quantity | Units on this delivery line. |
| ship_type | OSHP.TrnspName | SAP shipping type as entered - a service term like "UPS Ground - No Charge", "LTL - Collect", "Best Way", "Air - Collect", "Ocean", "TRUCK". Not reliable for carrier. |
| carrier | derived | Parsed from text in the tracking cell first ("3180079125 FEDEX FRT" → FedEx Freight), then the number format (1Z → UPS, 12/15/20/22 digits → FedEx), then ship_type. "LTL - carrier not recorded" when an LTL ship type has no carrier text. |
| carrier_mode | derived | PARCEL, LTL, COURIER, AFFILIATE_TRANSFER, FORWARDER, CUSTOMER_PICKUP, UNKNOWN. |
| tracking_no | ODLN.TrackNo | The cell as typed, trimmed. Often "number carrier-name". |
| tracking_clean | derived | First token of the cell when it looks like a number. Use for links. Null when the cell is text or empty. |
| tracking_valid | derived | 1 when tracking_clean fits the carrier, or the mode needs no number (AFFILIATE_TRANSFER, CUSTOMER_PICKUP). Drives SHIPPED vs SHIPPED_NO_TRACKING. |
| ship_to | ODLN.Address2 | Ship-to address block. "c/o DRI Edwardsville" means the affiliate's stock at DRI's own dock. |

## Routing
| Column | Meaning |
|---|---|
| route_to | Owner email unless generic, else OITM.U_Merchandiser. DRI side. |
| cs_owner | Null for now. Reserved for DDT and BCS CS owners. |

## Stage
Set in SQL, first rule that matches wins.

| Stage | Rule |
|---|---|
| SHIPPED | dln_docnum present and tracking_valid = 1 |
| SHIPPED_NO_TRACKING | dln_docnum present and tracking_valid = 0 |
| IN_TRANSIT_TO_DRI | grpo_docnum present, no delivery. Left the factory. |
| NOT_SHIPPED | PO only. Still at the factory. |

ships_direct does not change the stage. It changes what the dates mean - for a Mexico order NOT_SHIPPED is "at the factory, shipping direct," and IN_TRANSIT_TO_DRI is "left Mexico for the customer."


## Carrier modes and what they mean
- **PARCEL** - UPS or FedEx. Link and live check.
- **LTL** - PRO number plus carrier. Link to the carrier's trace page.
- **COURIER** - Pulse, local. No public tracking page. Give the reference and say to call Pulse.
- **AFFILIATE_TRANSFER** - ship_type Ocean, TRUCK, or AIR/Ocean. BCS or DDT stock delivered on DRI's inbound container or truck, often to DRI's own dock. Not missing tracking. Say delivered to the affiliate on dln_date.
- **FORWARDER** - factory-direct air. The forwarder holds the AWB. The tracking cell usually says "Freight Forwarder." Say so.
- **CUSTOMER_PICKUP** - released at the dock.
- **UNKNOWN** - ship_type gave nothing usable ("Best Way", "Courier") and the cell is empty. Treat as no tracking.

## What the table cannot say
- Whether the carrier delivered. SAP has no carrier feed. DELIVERED is not a stage.
- Whether an IN_TRANSIT_TO_DRI shipment has physically arrived at Edwardsville. Only a posted delivery proves it moved on. `clm_comments` is the nearest thing to a live status.
- Container, vessel, dwell, or risk. That is the risk digest's table.
- Anything closed more than 30 days ago.
- Lines with no factory PO yet. An order that returns no match may simply not be ordered from the factory; say so.
- DDT and BCS deliveries to their own customers.
