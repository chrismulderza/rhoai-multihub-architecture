# Storage and networking

Each workload cluster has its own OpenShift Data Foundation CephFS volume. Pods mount weights only inside their own cluster. The hub does not run that volume. A private link is a cluster-connectivity path, not a Ceph data path for inference.

## CephFS placement

| Cluster | Where the volume runs             | Who mounts it                                           |
| ------- | --------------------------------- | ------------------------------------------------------- |
| ACM hub | No model-weight CephFS            | Nobody for model weights                                |
| DC1     | Three Azure workers with no GPUs  | Both H100 virtual machines, access mode `ReadWriteMany` |
| DC2     | Four on-premises bare-metal hosts | L40S model pods, access mode `ReadWriteMany`            |

The StorageClass on each workload cluster is `ocs-storagecluster-cephfs`, the default created by a StorageCluster named `ocs-storagecluster`. Data center 1 uses the three non-GPU workers so the H100 virtual machines are not the Ceph storage nodes. Data center 2 has no separate storage nodes, so the four GPU hosts also host that site's CephFS volume.

Local NVMe on `Standard_ND96isr_H100_v5` is scratch. It is not the shared weight store. A flagship failover from node 1 to node 2 mounts the same data center 1 volume the replica already used.

Weights are seeded into each local volume. Serving a model in data center 1 does not read data center 2's Ceph, and the reverse is also true. A stretched Ceph cluster across Azure and the on-premises site would put the inference data path on the private link and would couple the two failure domains. This design keeps those domains separate.

## Private links

Private links carry cluster traffic:

- Corporate network to the hub gateway.
- Hub to data center 1, for the flagship request path.
- Hub to data center 2, for speech and embeddings.
- Data center 1 to data center 2, for cluster connectivity between the managed clusters.

The corporate path ends at the hub. Inference services do not listen on the internet, and they are not a second entry point on the corporate network.

## What stays off the private link

- Flagship tensor parallel, which stays on NVLink inside one H100 virtual machine.
- Flagship weight reads during failover, which stay on data center 1 CephFS.
- L40S weight reads, which stay on data center 2 CephFS.

The private link from the hub to data center 2 is on the speech and embedding request path after the gateway accepts the call. It is not on the flagship weight path.
