# Answer templates

Line one names the order, customer, stage, factory, and ship type. Line two gives the governing date. Then the date bullets, each as `Document - field: date`. Then the link and live check if any. Then the owner. Nothing after the owner.

## In transit to DRI
> **PO 8261981 - Halo Branded Solutions.** In transit to DRI. J&F, ocean.
> Arriving at DRI Nov 9 per the GRPO.
> - Sales Order - original ex-factory: Aug 10
> - Sales Order - original delivery: Sep 25
> - Sales Order - revised delivery: Sep 25
> - GRPO 2015-10-05 - date: Aug 26
> - GRPO - revised delivery: Nov 9
> - GRPO - container: TGBU8081853
> - GRPO - Gnosis status: "Empty Returned"
> 45 days past the revised delivery.
> Owner - klyman@capsdirect.com

## Several orders, same status
> **POs 8304434 and 8304436 - Halo Branded Solutions.** In transit to DRI. Ratone, ocean. Factory POs 2016-11-07 for both.
> Arriving at DRI Nov 6 per the GRPO.
> - Sales Order - original ex-factory: Sep 7
> - Sales Order - original delivery: Oct 23
> - Sales Order - revised delivery: Oct 23
> - GRPO - date: Sep 21
> - GRPO - revised delivery: Nov 6
> - GRPO - container: TEMU8777219
> - GRPO - Gnosis status: "On Rail"
> 14 days past the revised delivery.
> Owner - ahicks@capsdirect.com

## At the factory
> **POs 8461917, 8461919, 8461925, 8461927, 8461929, 8461931 - Halo Branded Solutions.** At the factory. J&F and Ratone, air. Factory POs 2236-08-24, 2236-08-27, 2236-08-26, 2236-08-21, 2236-08-20, 2236-08-22 in that order.
> Arriving at DRI Dec 14 per the purchase orders.
> - Sales Order - original ex-factory: Nov 30
> - Sales Order - original delivery: Dec 14
> - Sales Order - revised delivery: Dec 14
> - Purchase Order - revised delivery: Dec 14
> Owner - ahicks@capsdirect.com

## At the factory, date passed
> **PO 8661193 - Staples Promotional Products.** At the factory. Ratone, air.
> Arrival at DRI was due Nov 2, 3 days ago, and no GRPO is posted. Either the factory has not shipped or the GRPO is not entered yet.
> - Sales Order - original ex-factory: Oct 29
> - Sales Order - original delivery: Nov 2
> - Sales Order - revised delivery: Nov 2
> - Purchase Order - revised delivery: Nov 2
> Ships to the customer after receipt. No date until the factory confirms.
> Owner - yforbes@capsdirect.com. Ask the factory for a new ex-factory date.

## Direct ship from Mexico
> **PO 8463483 - Halo Branded Solutions.** At the factory. Mexico (VM00797), ships direct to the customer.
> Ex-factory Nov 9 per the sales order.
> - Sales Order - original ex-factory: Nov 9
> - Sales Order - revised delivery: Oct 21 - before the ex-factory date, cannot be right
> - Purchase Order - revised delivery: Nov 12
> - Sales Order Line - tracking: 1Z0B75J70324833787, 1Z0B75J70324955619, 1Z0B75J70328213396, 1Z0B75J70329554007, 1Z0B75J70333315178
> Use the ex-factory date. The revised delivery needs correcting on the sales order.
> Owner - ahicks@capsdirect.com

## Shipped, parcel
> **PO 48812 - Lakeshore Promotions.** Shipped to the customer. UPS.
> Due Nov 5 per the delivery.
> - Sales Order - original ex-factory: Oct 10
> - Sales Order - original delivery: Oct 30
> - Sales Order - revised delivery: Nov 5
> - Sales Order - customer commit: Nov 7
> - Delivery 221880 - ship date: Nov 1
> - Delivery - due date: Nov 5
> - Delivery - tracking: 1Z999AA10123456784
> - Delivery - ship-to: Lakeshore Promotions, Grand Rapids
> Per web search, as of Nov 4 - in transit, scheduled delivery Nov 5. https://www.ups.com/track?tracknum=1Z999AA10123456784
> Owner - Dana R.

## Shipped, LTL
> **PO 48790 - Midwest Team Sports.** Shipped to the customer. Old Dominion, 6 cartons.
> Due Nov 3 per the delivery.
> - Sales Order - original delivery: Nov 3
> - Delivery 221904 - ship date: Oct 30
> - Delivery - due date: Nov 3
> - Delivery - tracking: 11234567 ODFL
> - Delivery - ship-to: Omaha
> Unverified - open the link and paste the PRO. https://www.odfl.com/us/en/tools/trace.html
> Owner - Dana R.

## Shipped, no tracking
> **PO 48801 - Prairie Outfitters.** Shipped to the customer. XPO.
> Due Nov 1 per the delivery.
> - Delivery 221904 - ship date: Oct 29
> - Delivery - due date: Nov 1
> - Delivery - tracking: "see BOL" - no PRO recorded
> Nothing to trace until the PRO is entered on delivery 221904.
> Owner - Marcus T.

## Partial shipment
> **PO 48750 - Highline Golf.** 1,200 of 2,000 units shipped. FedEx.
> - Delivery 221850 - ship date: Oct 28, 1,200 units, tracking 794644790131. Per web search, as of Nov 4 - delivered Oct 31.
> - Remaining 800 units - in transit to DRI, arriving Nov 10 per the GRPO.
> Owner - Dana R.

## Affiliate transfer
> **PO BCS-2291 - BCS - Apparel.** Delivered to BCS on DRI's inbound. 2,400 units.
> - Delivery - ship date: Oct 27
> - Delivery - ship-to: BCS warehouse, Edwardsville
> No outbound carrier. Nothing to trace.
> Owner - Marcus T.

## No match
> No order matches 48999 in customer PO, DRI PO, order number, tracking, container, or delivery number. The table holds open orders with a factory PO, plus 30 days of closed. A brand-new order with no PO yet, or one closed longer ago, is an SAP lookup.

## Customer name only
> Lakeshore Promotions has 4 open orders. Nearest promise first - PO 48812 (Nov 7, shipped), PO 48830 (Nov 12, in transit to DRI), PO 48877 (Nov 20, in transit to DRI), PO 48902 (Dec 1, at the factory). Which one?

## Words
- "Shipped," not "closed."
- "Per web search, as of <date>" on any carrier status. Never "the carrier says."
- "Unverified" when the search returned nothing. Not "probably delivered."
- Days late as a number.
- One owner per answer, last line, nothing after it.
