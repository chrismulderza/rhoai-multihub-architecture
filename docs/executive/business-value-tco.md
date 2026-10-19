# Business value and total cost of ownership

The platform spends H100 capacity on one serving replica, then reuses the second H100 server for failover and for smaller workloads. L40S servers absorb speech and embeddings. The planning target is sustained GPU use above 80 percent, from MIG partitions on the spare H100s and from time-slicing on every L40S.

## Zero-replica failover

A hot standby for the flagship would be a second 72B replica on another four H100 GPUs, running at all times. This design keeps a single replica. When the primary Azure virtual machine is lost, the replica moves onto four full H100s that already sit in the second virtual machine. Those four GPUs run preemptable batch work until the flagship needs them.

The planning comparison in this guide values those four avoided H100s at about $140,000 or more of capital or cloud cost. The figure is the cost of the GPUs that a dual-replica design would hold idle for failover. It is not a quote for the whole Azure estate. The two `Standard_ND96isr_H100_v5` virtual machines are still required: one for the live replica, one for the failover GPUs, MIG tenants, and batch work.

Weights for that move are already on the data center 1 CephFS volume. Failover does not copy a 72B model across the private link.

## Where the GPUs go

| Pool                             | Hardware                                     | Steady-state job                       | Failover job                                             |
| -------------------------------- | -------------------------------------------- | -------------------------------------- | -------------------------------------------------------- |
| DC1 node 1, four of eight H100s  | Full GPUs, NVLink inside the virtual machine | Flagship 72B replica                   | Returns to this pool when the virtual machine is healthy |
| DC1 node 1, remaining four H100s | Full GPUs                                    | Other full-GPU work on the same server | Stays local to that server                               |
| DC1 node 2, GPUs 0-3             | Full H100s                                   | Preemptable batch and training         | Flagship replica                                         |
| DC1 node 2, GPUs 4-7             | MIG `3g.40gb` and `1g.10gb`                  | Smaller isolated models                | Stays partitioned                                        |
| DC2, 16 L40S GPUs                | 4-way time-slicing                           | Speech, text-to-speech, embeddings     | Stays on premises                                        |

Sixteen L40S GPUs with four-way time-slicing present 64 shared GPU slots for the smaller fleet. MIG on the second H100 server packs several smaller models onto GPUs 4 through 7 without giving each of them a full H100.

## Utilization

MIG is hardware isolation on the H100. A `3g.40gb` slice and four `1g.10gb` slices fill one 80GB H100, so idle silicon on the failover server still earns its keep. Time-slicing on the L40S lets several speech and embedding pods share one GPU when their duty cycles are bursty. Together, those two sharing modes are how the platform targets more than 80 percent sustained GPU use. That target is a planning objective for scheduling and mix, not a measured production report.

NVIDIA publishes 3.35 TB/s of HBM3 bandwidth for the H100 SXM 80GB and 864 GB/s of GDDR6 bandwidth for the L40S. The flagship stays on the H100 because a 72B model is memory-bandwidth bound. The L40S fleet is the right cost band for the smaller models. The [data center 1](../architecture/datacenter-1-h100.md) page records the comparison.

## What the hub does not buy

The Advanced Cluster Management hub does not carry H100 or L40S GPUs and does not store model weights. Its cost is the control-plane and gateway cluster, not a third GPU estate. Callers still see one internal API, so application teams do not fund a gateway per data center.
