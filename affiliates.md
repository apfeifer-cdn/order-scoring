# Affiliate lines (BCS and DDT)

BCS Apparel and DRI DUCK (DDT) are DRI's two largest customers and separate SAP companies. Their lines carry extra columns and score differently. `affiliate` says which - BCS or DDT. `aff_po_type` says what kind of PO.

**customer** - the affiliate raised the PO against one of their own customer orders. `aff_req_date` is when they need it at the DRI warehouse to serve that customer. `aff_cancel_date` is the last possible date; past it the order is dead and the band reads `CANCEL DATE MISS`. Treat these like any other customer line. Only BCS has customer-backed POs; every DDT PO is stock.

**stock** - the affiliate is replenishing their own inventory. No end customer is waiting. `aff_req_date` is their inventory target. Lateness is a stockout risk, not a broken promise, and the scoring weighs it by their stock position:
- `aff_available` is their on-hand minus committed for that style. At or below zero means they are already short.
- `aff_days_cover` is available divided by their 90-day daily sales. How long they can keep selling without this box.
- Late with cover through the delay: capped at 50. Late, they will not notice.
- Late and out of stock: floor of 70. They have orders against empty shelves.

How to write about affiliate stock:
- Say "BCS required date" or "DDT required date," never "customer promise." The `reasons` column already names the affiliate. Exception: a DDT line with DDTOrders rows is measured against `ddt_ship_date`, a real DDT customer promise - see below.
- When `reasons` says the affiliate has cover, say the delay is covered and move on. Not a call.
- When `reasons` says OUT OF STOCK, lead with that. It is the consequence.
- In the divert queue, out-of-stock affiliate lines strengthen the case to truck. Covered lines do not.
- Report affiliate stock-outs as their own count in the summary, split BCS and DDT, separate from customer lates.

## DDT demand columns

On Scored, DDT lines only. Null for CDN and BCS.

| Column | Meaning |
|---|---|
| `aff_po_open_qty` | Open quantity on the DDT purchase order line this DRI line fills. |
| `ddt_open_orders`, `ddt_open_units` | Open DDT customer sales order lines for this item in whs 01 - count of orders and units. |
| `ddt_earliest_ship` | Earliest `RDR1.ShipDate` across those lines. |
| `ddt_orders_on_box`, `ddt_units_on_box` | Orders and units this container is what fills, after on-hand is used up in ShipDate order. |
| `ddt_box_earliest_ship` | Earliest ShipDate among the orders this box fills. Feeds `effective_promise`. |
| `ddt_orders_short_after` | Orders still short after this container lands. Not this container's problem. |

## DDTOrders table

One row per DDT customer sales order line in whs 01, for scored DDT lines with `aff_available < 0`. Carries the container and score columns from Scored plus:

| Column | Meaning |
|---|---|
| `dri_so`, `dri_line`, `dri_line_qty` | The DRI sales order line on the container. |
| `ddt_so`, `ddt_line` | DDT sales order and line. |
| `ddt_customer`, `ddt_cardcode`, `ddt_cust_po` | DDT's customer and their PO. Use these names, never `ddt_so` alone. |
| `ddt_cs_email`, `ddt_cs_name` | DDT customer service owner (Driduck `ORDR.OwnerCode`). Owns the call. |
| `ddt_ship_date` | What DDT promised its customer (`RDR1.ShipDate`). |
| `ddt_open_qty` | Units still open on the DDT line. |
| `ddt_available_dt` | `dock_eta` plus the DRI-to-DDT transfer allowance. When DDT can ship. |
| `cum_open` | Running total of open units in ShipDate order. Allocation input. |
| `bucket` | `on_hand` - DDT stock already covers it, not at risk. `this_box` - this container fills it. `short_after` - short even after this container. |
| `days_late_to_ddt_customer` | `ddt_available_dt` minus `ddt_ship_date`. Negative means on time. |

## DDT customers behind a stock PO

Every DDT PO is stock, but a negative `aff_available` means DDT's open customer orders already exceed what it has. DDTOrders names those customers. Allocation is oldest ShipDate first: on-hand covers the earliest orders (`on_hand`, fine), this container covers the next (`this_box`), anything past that is `short_after`. `on_hand` rows are expected whenever DDT has any stock; their absence means the shelf is empty or the oldest order alone exceeds it. Write about `this_box` rows as late DDT customer orders, by DDT customer and CS owner. Write about `short_after` in one line as a DDT purchasing gap. Never present the DRI PO date as the DDT promise when DDTOrders has rows - the promise is `ddt_ship_date`.
