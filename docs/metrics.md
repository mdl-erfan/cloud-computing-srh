# Metrics: delay per lane (launch customer SHP04, Greenleaf Foods)

Reliability target: 65% on time on every lane. A shipment is on time if it arrived no more than 24 hours after its scheduled arrival.

| Lane | Trade lane | Shipments | Avg delay (h) | On-time % | Target | Result |
|---|---|---|---|---|---|---|
| CNYTN → USLAX | Transpacific | 24 | 47.3 | 45.8% | 65% | Misses |
| CNSHA → NLRTM | Asia-North Europe | 31 | 33.5 | 54.8% | 65% | Misses |
| NLRTM → USHOU | Transatlantic | 30 | 48.4 | 56.7% | 65% | Misses |
| GBFXT → CNSHA | North Europe-Asia | 29 | 38.5 | 79.3% | 65% | Meets |

Three of four lanes miss the target. The worst is CNYTN → USLAX at 45.8%, 19.2 points below target.

## Which carriers cause the misses

On-time % per carrier on each lane (shipments in brackets):

| Carrier | CNYTN → USLAX | CNSHA → NLRTM | NLRTM → USHOU | GBFXT → CNSHA |
|---|---|---|---|---|
| ATL | 0.0% (5) | 33.3% (6) | 50.0% (4) | 16.7% (6) |
| KST | 25.0% (4) | n/a | 33.3% (3) | 100% (4) |
| NWL | 25.0% (4) | 0.0% (9) | 71.4% (7) | 100% (7) |
| BLC | 33.3% (3) | 75.0% (4) | 100% (3) | 75.0% (4) |
| ORS | 100% (5) | 100% (5) | 33.3% (9) | 100% (4) |
| HRZ | 100% (3) | 100% (7) | 75.0% (4) | 100% (4) |

Findings:
- **ATL is below the 65% target on every lane it serves**, and is the worst carrier on Transpacific (0 of 5 on time).
- **HRZ is the most dependable carrier**: 100% on three lanes and 75% on Transatlantic.
- **Transpacific (45.8%)**: only HRZ and ORS are reliable (8 of 8 on time). The other four carriers had 3 of 16 on time.
- **Asia-North Europe (54.8%)**: NWL (0 of 9) and ATL (2 of 6) cause the miss. The other three carriers had 15 of 16 on time.
- **Transatlantic (56.7%)**: ORS, KST and ATL are weak (6 of 16 on time). The other carriers had 11 of 14 on time.

Caveat: each carrier-lane cell has only 3 to 9 shipments, so these are signals, not proof.

Implication for the platform: Greenleaf's reliability problem is mostly about **which carriers it books**. A platform that compares carrier reliability at booking time (batch is enough for this) would address it.
