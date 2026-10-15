# Lane reference

All values from DRI container data, Sep 2026. Revisit after the shadow quarter.

## Discharge ports
| Port | Terminal | IPI to Kansas City | Rail dwell baseline | Notes |
|---|---|---|---|---|
| Los Angeles | multiple | 9 days | 6 (p75), escalate 10 (p90) | On-dock rail. Largest volume. |
| Long Beach | Total Terminals International | 9 days | 6 | On-dock rail. |
| New York | Port Liberty Bayonne | 10 days | 6 | DRI boxes run ~2 days past published averages here. |
| Houston | - | not railed | 5 (truck) | Most Houston FCL trucks to Kansas City. |

IPI = inland point intermodal, the rail leg from port ramp to destination ramp.

## Time constants
| Constant | Value | Used for |
|---|---|---|
| Divert window closes | 72 hours before vessel ETA | After this a container is committed to rail. |
| Truck transit after discharge | 4 days | Divert projection and non-rail warehouse ETA. |
| Rail projection pre-discharge | vessel ETA + 3 + IPI + 1 | 3 = median dwell, 1 = destination handling. |
| Handling after rail discharge | 1 day | Rail discharged + 1 = warehouse. |
| Re-fire tolerance | 2 days | HOLD or notified line returns when ETA slips 2+ days past what was recorded. |
| Stale untracked cutoff | 75 days from GRPO | No Gnosis row and GRPO older than this = assumed delivered. |
| Rail discharged, assumed delivered | 5 days | Drops from scoring. |
| Customs hold threshold | 3 days after discharge | Only at port, only if the forwarder import has a row. |

## Observed distributions
| Measure | p50 | p75 | p90 | p95 | Basis |
|---|---|---|---|---|---|
| Awaiting Rail Load dwell (days) | 3 | 6 | 10 | 11 | 202 containers |
| Vessel ETA drift (days) | 0 | 2 | 5 | 9 | 339 containers |
| Rail ETA drift (days) | 2 | 5 | 11 | 14 | 253 containers |
| Gap between Gnosis snapshots (days) | 2 | 4 | 6 | 8 | 362 containers - sets silence at 4 |
| Ex-factory change lead time (days) | 34 | 62 | - | - | 3,083 changes; 12% inside 7 days; 46% move earlier |

## Origins
Chattogram (Bangladesh) nearly always transships at Colombo, Singapore, Port Klang or Tanjung Pelepas. No direct US service. Vietnam, Thailand, China, Indonesia run direct or single-transship into LA/Long Beach. Final destination is Kansas City; the warehouse is Edwardsville, KS.

## Modes
FCL (40HC, 40GP, 45HC, 20GP) rails from LA/LB/NY, trucks from Houston. ~91% of FCL rail. LCL: about a third rail; the rest deconsolidate at a CFS and truck. Air and parcel are keyed on AWB or tracking number and excluded from this scoring.
