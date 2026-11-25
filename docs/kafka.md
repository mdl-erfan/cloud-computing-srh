# Session 5: Kafka on GKE

Three-broker Strimzi Kafka cluster (`freight`, namespace `kafka`) on the `freight` GKE cluster, topic `ocean-events` (3 partitions, replication factor 3, `min.insync.replicas=2`, retention 30 days). Producer image: `europe-west3-docker.pkg.dev/srhuni/freight/generator:v2`.

## 1. Topic description while broker 1 was down

I deleted the pod `freight-dual-1` while the producer was running, then described the topic straight away:

```
Topic: ocean-events     TopicId: UcpY5nmLQMWhJ7xd5WORkA PartitionCount: 3       ReplicationFactor: 3    Configs: min.insync.replicas=2,retention.ms=2592000000
        Topic: ocean-events     Partition: 0    Leader: 2       Replicas: 2,0,1 Isr: 0,2        Elr:    LastKnownElr:
        Topic: ocean-events     Partition: 1    Leader: 0       Replicas: 0,1,2 Isr: 0,2        Elr:    LastKnownElr:
        Topic: ocean-events     Partition: 2    Leader: 2       Replicas: 1,2,0 Isr: 0,2        Elr:    LastKnownElr:
```

Before the failure, partition 2 was led by broker 1. After broker 1 came back, all three `Isr` lists were `0,1,2` again and partition 2 was led by broker 1 again.

## 2. Consumer group `ops-dashboard` before and after the offset reset

Before the reset (`kafka-consumer-groups.sh --describe --group ops-dashboard`):

```
GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
ops-dashboard   ocean-events    0          0               217             217
ops-dashboard   ocean-events    1          0               286             286
ops-dashboard   ocean-events    2          50              198             148
```

Reset with `--reset-offsets --to-earliest --execute`, then described again:

```
GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
ops-dashboard   ocean-events    0          0               235             235
ops-dashboard   ocean-events    1          0               320             320
ops-dashboard   ocean-events    2          0               213             213
```

The group's position went back to 0 on every partition, so its lag became the whole log. The events themselves were not touched. (The log-end offsets are higher in the second table only because the producer kept writing between the two commands.)

## 3. Which partition did my shipment use?

Shipment `LIVE-SHP05-000046` had 7 events (BOOKED, GATE_IN, LOADED, DEPARTED, ARRIVED, DISCHARGED, GATE_OUT), and all of them were on partition 0. The producer uses the shipment ID as the message key, and Kafka picks the partition by hashing the key, so the same key always maps to the same partition. A partition is an append-only log that is read in offset order, so all events of one shipment stay in the order they were written. Across different shipments (different partitions) there is no ordering guarantee.

## 4. What happened when the broker died, and why didn't the producer fail?

Deleting `freight-dual-1` took broker 1 offline. It was the leader of partition 2, so Kafka elected broker 2, which had an in-sync copy, as the new leader. Broker 1 dropped out of the in-sync replica set of all three partitions (`Isr: 0,2`). The producer did not fail because every partition still had two in-sync replicas, which satisfies `min.insync.replicas=2`, so writes were still accepted, and the producer client refreshed its metadata and sent to the new leader on its own. The producer showed 0 restarts and its event count kept rising (480 to 660 events). When the pod restarted, broker 1 caught up from the other replicas, rejoined the `Isr` and became leader of partition 2 again. Nothing was lost and nobody restarted anything by hand.