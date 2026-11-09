# Carriers, links, and the live check

## Number formats
Use these to sanity-check `tracking_clean` against `carrier`, and to answer "which carrier is this."

| Pattern | Carrier | Mode |
|---|---|---|
| 1Z + 16 letters/digits | UPS | PARCEL |
| 12 digits | FedEx Express | PARCEL |
| 15 digits, or 20/22 digits starting 96 | FedEx Ground / Home | PARCEL |
| 20-22 digits starting 9 (not 96) | USPS | PARCEL |
| 7-11 digits, sometimes with a hyphen | LTL PRO number - carrier must come from `carrier` | LTL |
| Starts with letters, not 1Z | Likely a BOL or customer reference, not a tracking number | - |

A PRO number identifies nothing without the carrier. If `carrier` is empty, say the number cannot be traced until the carrier is recorded on delivery `dln_docnum`.

## Links
Build the link, show it every time, then run the search. Parcel links accept the number. LTL links open the carrier's trace page; the user pastes the PRO.

| Carrier (as in SAP) | Mode | Link |
|---|---|---|
| UPS | PARCEL | https://www.ups.com/track?tracknum=NUMBER |
| FedEx | PARCEL | https://www.fedex.com/fedextrack/?trknbr=NUMBER |
| FedEx Freight | LTL | https://www.fedex.com/fedextrack/?trknbr=NUMBER - same page, PRO works |
| R+L Carriers | LTL | https://www.rlcarriers.com/freight/shipping/shipment-tracing |
| AAA Cooper | LTL | https://www.aaacooper.com/ - Trace a Shipment |
| Pulse | COURIER | No public page. Local courier - call with the reference. |
| USPS | PARCEL | https://tools.usps.com/go/TrackConfirmAction?tLabels=NUMBER |
| Old Dominion / ODFL | LTL | https://www.odfl.com/us/en/tools/trace.html |
| Estes | LTL | https://www.estes-express.com/myestes/shipment-tracking |
| XPO | LTL | https://www.xpo.com/track |
| SAIA | LTL | https://www.saia.com/track |
| ABF / ArcBest | LTL | https://arcb.com/tools/tracking |
| Averitt | LTL | https://www.averitt.com/tracking |
| Dayton Freight | LTL | https://www.daytonfreight.com/Tracking |
| Southeastern Freight | LTL | https://www.sefl.com/webconnect/tracing |
| TForce Freight | LTL | https://www.tforcefreight.com/ltl/apps/Tracking |
| Central Transport | LTL | https://www.centraltransport.com/tools/tracking |
| Customer routed / unknown | LTL | No link. Say the customer's routing guide names the carrier. |

Carriers vary by customer routing guide, so this list will miss some. When `carrier` is not in the table, search "<carrier name> freight tracking" and give the first official result. Add it here afterward.

## Modes with no live check
AFFILIATE_TRANSFER rides DRI's inbound - say delivered to the affiliate on `dln_date`, nothing to trace. FORWARDER is factory-direct air; the forwarder has the AWB. CUSTOMER_PICKUP was released at the dock. COURIER (Pulse) has no page. UNKNOWN with an empty cell is no tracking. None of these get a search.

## Running the live check
1. Build the link from `tracking_clean`, never the raw cell.
2. Search the tracking number together with the carrier name. One query.
3. Read the result. Report a status only if the result states one with a date - "Delivered Nov 3," "In transit, scheduled Nov 6," "Exception - address correction."
4. Write it as "per web search, as of <today>." The carrier page is the record; the search is a snapshot.
5. No usable result - "Unverified. Open the link." Nothing else.

## What not to do
- Do not infer delivered from elapsed days. A five-day-old ground shipment is not delivered until a carrier page says so.
- Do not try a second carrier when the first search fails. Wrong carrier, wrong answer.
- Do not search a number that fails the format check for its carrier. Report the mismatch instead.
- Do not quote more than the status line and date from a carrier result.

## Upgrade path
A scheduled pull from the UPS Tracking API and FedEx Track API, writing `carrier_status`, `carrier_eta`, and `delivered_dt` into the Orders table, replaces steps 2-5 with a column read and makes DELIVERED a real stage. Until then this file is the method.
