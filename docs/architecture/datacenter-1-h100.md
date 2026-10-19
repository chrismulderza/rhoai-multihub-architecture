# Data center 1, H100

Data center 1 is an Azure OpenShift cluster managed by the hub. It has three worker nodes with no GPUs and two `Standard_ND96isr_H100_v5` virtual machines. GPU pods schedule only on the two virtual machines.

Each virtual machine provides 96 Intel Xeon Sapphire Rapids vCPUs, 1,900 GiB of memory, and eight NVIDIA H100 80GB GPUs. NVLink 4.0 connects those GPUs inside the virtual machine. Each GPU also has a dedicated 400 Gbps NVIDIA Quantum-2 CX7 InfiniBand port. InfiniBand is not the path for flagship tensor parallel. Tensor parallel size 4 stays inside one virtual machine and uses NVLink.

The virtual machines share a size and differ by MIG layout. OpenShift applies that layout with the GPU Operator. This cluster is not an AKS MIG node pool.

| Node       | MIG config             | Layout                                                                     |
| ---------- | ---------------------- | -------------------------------------------------------------------------- |
| GPU node 1 | `all-disabled`         | GPUs 0-7 remain full H100 devices. The flagship replica uses four of them. |
| GPU node 2 | `node2-hybrid-profile` | GPUs 0-3 remain full H100 devices. GPUs 4-7 are MIG-enabled.               |

Each MIG GPU on node 2 hosts one `3g.40gb` instance and four `1g.10gb` instances. NVIDIA documents both profile names for the H100 80GB. That mix uses 7 of 7 compute slices and 8 of 8 memory slices. GPUs 0 through 3 stay whole so the flagship can preempt batch work onto four full GPUs without rebuilding MIG geometry during an outage.

The three workers without GPUs run platform services and the OpenShift Data Foundation CephFS volume for this cluster. Both H100 virtual machines mount that volume with `ReadWriteMany`. Local NVMe on the virtual machines is scratch capacity, not the weight store.

## Memory bandwidth

NVIDIA publishes these accelerator figures:

| Accelerator   | Memory               | Bandwidth | Link used by the flagship            |
| ------------- | -------------------- | --------- | ------------------------------------ |
| H100 SXM 80GB | 80 GB HBM3           | 3.35 TB/s | NVLink, 900 GB/s                     |
| L40S          | 48 GB GDDR6 with ECC | 864 GB/s  | PCIe Gen4 x16, 64 GB/s bidirectional |

H100 SXM memory bandwidth is about 3.9 times the L40S figure (3.35 TB/s compared with 864 GB/s). With sparsity, NVIDIA publishes 3,958 FP8 teraFLOPS for the H100 SXM and 1,466 FP8 teraFLOPS for the L40S. The flagship uses FP8 for the KV cache on the full H100 GPUs. MIG slices on node 2 do not inherit that flagship argument set.

Sources: [NVIDIA H100 product specifications](https://www.nvidia.com/en-us/data-center/h100/) and [NVIDIA L40S specifications](https://www.nvidia.com/en-us/data-center/l40s/).

## Flagship placement

The replica is one pod. Its vLLM process uses `--tensor-parallel-size 4`, `--kv-cache-dtype fp8`, `--gpu-memory-utilization 0.92`, and `--max-model-len 32768`. It tolerates `node.kubernetes.io/not-ready` and `node.kubernetes.io/unreachable` for 10 seconds and runs at `flagship-critical-priority`.

Node 2 GPUs 4 through 7 remain available to smaller models through the whole failover. Their isolation boundary is the MIG instance, not a second copy of the 72B weights.
