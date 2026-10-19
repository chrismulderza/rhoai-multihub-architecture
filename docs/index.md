# RHOAI Multi-Cluster Models-as-a-Service

This guide specifies a consolidated Models-as-a-Service platform on Red Hat OpenShift AI. An Advanced Cluster Management hub in Azure publishes the only AI gateway, and only to the corporate network. Data center 1 serves the flagship large language model on NVIDIA H100 GPUs. Data center 2 serves speech, embeddings, and smaller models on NVIDIA L40S PCIe accelerators.

- [Who this guide is for](#who-this-guide-is-for)
- [Platform constraints](#platform-constraints)
- [Documentation map](#documentation-map)

## Who this guide is for

### Executive track

The [executive track](executive/executive-summary.md) is written for CIOs, CTOs, infrastructure VPs, and chief AI architects. It covers total cost of ownership for zero-replica failover, service-level tiers, multi-datacenter disaster recovery, and the utilization economics of mixing MIG partitions with time-sliced GPUs.

### Engineering track

The [engineering track](#platform-prerequisites) is written for platform, infrastructure, and MLOps engineers. It provides verbatim command-line procedures and complete Kubernetes manifests for cluster baselines, GPU partitioning, model serving, gateway policy, validation, and operations.

## Platform constraints

Manifests in this guide follow these rules.

- The flagship model runs as a single replica. Its pod tolerates `node.kubernetes.io/not-ready` and `node.kubernetes.io/unreachable` for 10 seconds.
- The flagship workload uses priority class `flagship-critical-priority` with value `1000000000` and preemption policy `PreemptLowerPriority`.
- Model weights mount from CephFS persistent volumes with access mode `ReadWriteMany`.
- Data center 1 partitions H100 SXM5 GPUs with NVIDIA MIG profiles `3g.40gb` and `1g.10gb`. Data center 2 shares L40S PCIe GPUs with NVIDIA GPU Operator time-slicing.

## Documentation map

### Executive

- [Executive summary](executive/executive-summary.md) covers the hub, the two data centers, and the internal gateway.
- [Business value and TCO](executive/business-value-tco.md) covers the four-H100 cost avoidance and the utilization target.
- [SLA and disaster recovery](executive/sla-dr-strategy.md) covers flagship failover inside data center 1 and the latency bands.

### Architecture

- [Global topology](architecture/global-topology.md) covers private links and the flagship failover state machine.
- [Data center 1 H100](architecture/datacenter-1-h100.md) covers the Azure virtual machines, MIG layout, and H100 memory bandwidth.
- [Data center 2 L40S](architecture/datacenter-2-l40s.md) covers the on-premises hosts and time-slicing.
- [MaaS gateway](architecture/maas-gateway-kuadrant.md) covers path routing from the hub.
- [Storage and networking](architecture/storage-networking.md) covers local CephFS volumes and the private links.

### Platform prerequisites

OpenShift cluster baselines, OpenShift Data Foundation CephFS ReadWriteMany storage, the NVIDIA GPU Operator, and Kueue queues with the flagship and batch priority classes.

### GPU partitioning

H100 MIG profiles for data center 1, L40S time-slicing for data center 2, and OpenShift AI accelerator profiles that expose those slices to model workloads.

### Model serving

The single-replica flagship inference service, node-failure recovery onto reserved H100 capacity, the smaller-model fleet on L40S, and the speech translation pipeline.

### MaaS governance

The unified Gateway API entry point, Authorino and Kuadrant authentication, request and token rate limits, and self-service API key issuance.

### Workflows

Interactive Jupyter development with idle culling, ad-hoc distributed training on lower-priority GPUs, and model-registry promotion through CI/CD.

### Verification

An OpenAI-compatible end-to-end client, failure-injection tests for node loss, and performance benchmarks for the flagship and speech paths.

### Operations

Node-failure runbooks, DCGM metrics in Prometheus, and a troubleshooting guide for serving, storage, and GPU scheduling faults.
