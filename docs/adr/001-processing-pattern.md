# ADR 001: Lean Lambda, not Kappa, for the Greenleaf platform

## Status
Accepted

Date: 2026-11-23

## Context
Shipper: Greenleaf Foods (SHP04). Targets: freshness 60 seconds, reliability 65% on time on every lane, budget $30 per month.

Evidence (docs/metrics.md): 3 of 4 lanes miss the reliability target (45.8%, 54.8% and 56.7% against 65%). The misses come mostly from carrier choice: ATL is below target on every lane it serves, and NWL had 0 of 9 on time on CNSHA to NLRTM. Warehouse teams re-plan the moment a vessel slips, so live tracking must reach the ops view within 60 seconds. The carrier analysis only needs daily data.

## Options considered
1. **Lambda (reference platform):** a speed layer for live events (Pub/Sub, Kafka, Dataflow) and a batch layer for files (Cloud Storage, Dataproc). Meets freshness, but two code paths and several always-on services.
2. **Kappa:** one streaming path for everything. The daily files would also be streamed, and history recomputed by replaying the stream. Simpler to reason about, but Pub/Sub only keeps messages for a few days, so replaying history would need the raw files anyway, and an always-on streaming job costs more than the budget.
3. **Lean Lambda:** keep the two layers, but use the cheapest tool in each. Live events go Pub/Sub to a BigQuery table through a BigQuery subscription (no Kafka, no Dataflow). Daily files go Cloud Storage to BigQuery load jobs, plus a scheduled query for the carrier scorecard (no Dataproc).

## Decision
We chose **Option 3, lean Lambda**, because only the ops view needs to be live. The carrier analysis, which is Greenleaf's actual reliability problem, only needs daily data. Splitting by freshness need lets each path use the cheapest tool that meets its target.

## Consequences
- **Cost:** rough estimate of a few dollars a month (Pub/Sub, BigQuery storage and queries, Cloud Storage and a scheduler are all small at this volume), well under $30. Kafka, an always-on Dataflow streaming worker or a Dataproc cluster would each cost close to or above the budget on their own. To be checked with the Google Cloud pricing calculator.
- **Freshness:** seconds for live events through the BigQuery subscription, against the 60-second target. Batch data is one day old, which is fine for the carrier scorecard.
- **Complexity:** two paths to build, run and monitor. Definitions such as "on time" (no more than 24 hours late) must be the same in both, so they should live in one shared SQL view.
- **What gets harder:** with no Dataflow, live events can only be cleaned or enriched in BigQuery SQL, not in-stream. If Greenleaf later needs windowed or stateful processing, we add Dataflow and write a new ADR that replaces this one.