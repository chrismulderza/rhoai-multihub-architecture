# Technical Schema Reference

Inter-agent artifact for the platform and serving authors. This file is not a MkDocs page. Authoring target is Red Hat OpenShift AI 3.5 on OpenShift Container Platform 4.19.9 or later, with Red Hat Connectivity Link 1.4.x.

Later manifests copy the current rows below. A rejected row names the replacement and is not an authoring API.

## Sites

| Site | Platform | Nodes | Accelerators |
| --- | --- | --- | --- |
| DC1 | Azure subscription | 3 workers with no GPUs, plus 2 `Standard_ND96isr_H100_v5` virtual machines | 8x NVIDIA H100 80GB per virtual machine |
| DC2 | On-premises bare metal | 4 hosts | 4x NVIDIA L40S 48GB PCIe per host |

GPU pods schedule only on the accelerator nodes. The three Azure workers expose no `nvidia.com/gpu` capacity. Platform services, including the CephFS weight store, may schedule on those workers.

## DC1 GPU virtual machines

`Standard_ND96isr_H100_v5` provides 96 Intel Xeon Sapphire Rapids vCPUs, 1,900 GiB of memory, and 8 NVIDIA H100 80GB GPUs. NVLink 4.0 connects the GPUs inside one virtual machine. Each GPU also has a dedicated 400 Gbps NVIDIA Quantum-2 CX7 InfiniBand link. Those InfiniBand links are not the flagship tensor-parallel path. Tensor parallel stays inside one virtual machine and uses NVLink.

The two virtual machines share this size and differ by the GPU Operator MIG label. OpenShift on these virtual machines applies MIG through the operator. This is not an AKS MIG node pool.

| Node | MIG config | GPU layout |
| --- | --- | --- |
| DC1 GPU node 1 | `all-disabled` | GPUs 0-7 remain full H100 devices. The flagship model uses four of them. |
| DC1 GPU node 2 | `node2-hybrid-profile` | GPUs 0-3 remain full H100 devices with MIG disabled. GPUs 4-7 are MIG-enabled. |

Each MIG-enabled GPU on node 2 uses one `3g.40gb` instance and four `1g.10gb` instances. NVIDIA documents both profile strings for the H100 80GB. That geometry consumes 7/7 compute slices and 8/8 memory slices. The GPU Operator device map for each of GPUs 4, 5, 6, and 7 is `3g.40gb: 1` and `1g.10gb: 4`.

The virtual machine local NVMe disks are not the model-weight store.

## DC2 hosts

Each on-premises host has 4 L40S 48GB PCIe GPUs. The L40S is absent from the NVIDIA MIG-capable product list. DC2 uses GPU Operator time-slicing only, with `migStrategy: none` and `replicas: 4` on `nvidia.com/gpu`. Four hosts, four GPUs, and four replicas produce 64 shared GPU slots.

## API catalog

| Kind | Authoring apiVersion | Status | Consumer flag |
| --- | --- | --- | --- |
| `InferenceService` | `serving.kserve.io/v1beta1` | Current. OpenShift AI 3.5 single-model serving examples use it. | One flagship replica. vLLM arguments are listed under runtime flags. |
| `ServingRuntime` | `serving.kserve.io/v1alpha1` | Current. OpenShift AI 3.5 vLLM runtime examples use it. This release has no newer stable ServingRuntime version. | Container name `kserve-container`. Model path `/mnt/models`. |
| `Gateway`, `HTTPRoute` | `gateway.networking.k8s.io/v1` | Current. Models-as-a-Service requires Gateway API 1.2 or later. | OpenAI paths: `/v1/chat/completions` to the flagship in DC1, `/v1/audio/transcriptions` to Whisper in DC2, `/v1/audio/speech` to TTS in DC2, `/v1/embeddings` to TEI in DC2. |
| `AuthPolicy`, `RateLimitPolicy` | `kuadrant.io/v1` | Current. Connectivity Link 1.4 authors both kinds at `v1`. | `targetRef.group` is `gateway.networking.k8s.io`. API-key and OIDC authentication, with RPM and TPM tiers, attach to the Gateway or HTTPRoute. |
| `ClusterQueue`, `ResourceFlavor`, `LocalQueue` | `kueue.x-k8s.io/v1beta1` | Current for OpenShift AI 3.5. Distributed-workload examples in that release apply `v1beta1`. | Covered GPU resource is `nvidia.com/gpu`. Upstream Kueue 0.20 removed `v1beta1` and stores `kueue.x-k8s.io/v1beta2`. Install the Kueue API served by OpenShift AI 3.5. Do not apply these objects to upstream Kueue 0.20. |
| `ClusterPolicy` | `nvidia.com/v1` | Current GPU Operator API. | MIG and time-slicing are ConfigMaps selected by the policy. DC1 node 1 selects `all-disabled`. DC1 node 2 selects `node2-hybrid-profile`. DC2 selects the 4-way time-slicing ConfigMap. |
| `PriorityClass` | `scheduling.k8s.io/v1` | Current Kubernetes API. | `flagship-critical-priority` value `1000000000`, `preemptionPolicy: PreemptLowerPriority`. `batch-low-priority` value `10000`. |

## Rejected APIs

| Rejected | Replacement |
| --- | --- |
| `serving.kserve.io/v1alpha1` `InferenceService` | `serving.kserve.io/v1beta1` `InferenceService` |
| `kuadrant.io/v1beta2` `AuthPolicy` | `kuadrant.io/v1` `AuthPolicy` |
| `kuadrant.io/v1beta2` `RateLimitPolicy` | `kuadrant.io/v1` `RateLimitPolicy` |
| `kueue.x-k8s.io/v1beta1` on upstream Kueue 0.20 | `kueue.x-k8s.io/v1beta2` when the installed Kueue is upstream 0.20. OpenShift AI 3.5 examples remain on `v1beta1`. |

## Runtime flags

Flagship vLLM, on four full H100 GPUs of DC1 GPU node 1:

- `--tensor-parallel-size 4`
- `--kv-cache-dtype fp8`
- `--gpu-memory-utilization 0.92`
- `--max-model-len 32768`

`--kv-cache-dtype fp8` is a Hopper flag for this full-GPU flagship. MIG slices on DC1 GPU node 2 do not inherit that flagship argument set.

Whisper vLLM, on DC2 time-sliced L40S capacity:

- `--task transcription`

The flagship pod is a single replica. It tolerates `node.kubernetes.io/not-ready` and `node.kubernetes.io/unreachable` for 10 seconds. Its priority class is `flagship-critical-priority`.

## Storage

Every model-weight volume uses access mode `ReadWriteMany` and the OpenShift Data Foundation CephFS StorageClass `ocs-storagecluster-cephfs`. That class is the default created by a StorageCluster named `ocs-storagecluster`. The Azure virtual-machine local NVMe disks are scratch devices, not the shared weight store.

## Sources

- Azure ND H100 v5 sizes: `https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/gpu-accelerated/ndh100v5-series`
- NVIDIA H100 80GB MIG profiles: `https://docs.nvidia.com/datacenter/tesla/mig-user-guide/latest/supported-mig-profiles.html`
- NVIDIA MIG-capable products: `https://docs.nvidia.com/datacenter/tesla/mig-user-guide/latest/supported-gpus.html`
- OpenShift AI 3.5 supported components, KServe 0.19.0: `https://access.redhat.com/articles/rhoai-supported-configs-3.x`
- OpenShift AI 3.5 ServingRuntime and InferenceService: `https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/configuring_your_model-serving_platform/configuring-your-model-serving-platform_rhoai-admin`
- OpenShift AI 3.5 Kueue examples: `https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/managing_openshift_ai/managing-distributed-workloads_managing-rhoai`
- Connectivity Link 1.4 AuthPolicy and RateLimitPolicy: `https://docs.redhat.com/en/documentation/red_hat_connectivity_link/1.4/html/deploy_red_hat_connectivity_link/rhcl-config-deploy-gateway-policies`
- Upstream Kueue 0.20 removal of `v1beta1`: `https://github.com/kubernetes-sigs/kueue/releases/tag/v0.20.0`
