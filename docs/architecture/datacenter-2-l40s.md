# Data center 2, L40S

Data center 2 is an on-premises OpenShift cluster managed by the hub. It has four bare-metal hosts. Each host has four NVIDIA L40S 48GB PCIe GPUs, for 16 GPUs in the site.

The L40S is not a MIG-capable GPU. NVIDIA lists MIG support as absent for this product, and the card has no NVLink. The GPU Operator shares each GPU with time-slicing: `migStrategy` is `none`, and `nvidia.com/gpu` is advertised at `replicas: 4`. Sixteen GPUs at four replicas present 64 shared slots.

Time-slicing shares the GPU without the memory and fault isolation MIG provides on the H100. Tenant separation for this site is the hub authentication and quota policy, plus the queue that admits the pod. It is not a hardware memory partition.

## What runs here

| Path                       | Workload                                  |
| -------------------------- | ----------------------------------------- |
| `/v1/audio/transcriptions` | Whisper, with vLLM `--task transcription` |
| `/v1/audio/speech`         | Text-to-speech                            |
| `/v1/embeddings`           | Embedding runtime                         |

Whisper and text-to-speech are allowed to scale to zero when idle. The flagship 72B model does not run here. L40S memory is 48 GB of GDDR6 at 864 GB/s, against 80 GB of HBM3 at 3.35 TB/s on the H100 SXM. The interconnect is PCIe Gen4 x16 at 64 GB/s bidirectional.

## Local weights

OpenShift Data Foundation for this site runs on the four bare-metal hosts, because those are the only nodes in the cluster. Model pods mount that cluster's CephFS volume with `ReadWriteMany` on StorageClass `ocs-storagecluster-cephfs`. They do not mount the data center 1 volume across the private link.

The private link to the hub carries inference requests after the gateway has already accepted them. The private link to data center 1 is cluster connectivity, not a weight-serving path.
