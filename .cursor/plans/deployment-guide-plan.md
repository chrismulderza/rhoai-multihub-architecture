# AI Agent Execution Plan: Enterprise Multi-Cluster RHOAI Deployment Guide

**Plan Identifier:** `PLAN-RHOAI-H100-L40S-MAAS-V1`

**Execution Objective:** Generate a publication-ready, enterprise-grade deployment guide and architectural blueprint for a consolidated Models-as-a-Service (MaaS) AI platform across heterogeneous data centers using Red Hat OpenShift AI (RHOAI), NVIDIA H100 SXM5, and NVIDIA L40S PCIe accelerators.

**Output Target:** Static site documentation built with `mkdocs` (Material theme).

**Target Audiences:**

1. **Executive Leadership & Enterprise Architects:** High-level topology, ROI/TCO analysis, GPU utilization economics, multi-datacenter SLA/DR posture, and enterprise governance.
2. **Platform, Infrastructure & MLOps Engineers:** Verbatim CLI commands, production-grade Kubernetes/OpenShift YAML manifests (zero placeholders), validation scripts, and disaster-recovery runbooks.

---

## 1. Multi-Agent Team Structure & RACI Matrix

The execution harness delegates tasks across specialized autonomous subagents coordinated by the Lead Orchestrator:

```
                          ┌───────────────────────────┐
                          │     Lead Orchestrator     │
                          │   (Workflow & Handoffs)   │
                          └─────────────┬─────────────┘
                                        │
     ┌──────────────────┬───────────────┴───────────────┬──────────────────┐
     ▼                  ▼                               ▼                  ▼
┌──────────────┐ ┌──────────────┐               ┌──────────────┐ ┌──────────────┐
│  Subagent 1  │ │  Subagent 2  │               │  Subagent 3  │ │  Subagent 4  │
│ Documentation│ │  Research &  │               │  Platform &  │ │   MLOps &    │
│  Architect   │ │ Schema Valid │               │ Infra Engine │ │ Model Serving│
└──────────────┘ └──────────────┘               └──────────────┘ └──────────────┘
                                        │
                                        ▼
                                 ┌──────────────┐
                                 │  Subagent 5  │
                                 │ QA, Runbooks │
                                 │ & Validation │
                                 └──────────────┘

```

| Subagent Role | Designation | Primary Responsibilities |
| --- | --- | --- |
| **Lead Orchestrator** | `agent-orchestrator` | Coordinates phased execution, verifies gate pass criteria, manages inter-agent artifacts. |
| **Documentation Architect** | `subagent-doc-architect` | Manages `mkdocs.yml`, layouts, navigation hierarchy, admonitions, and Executive/Architect tracks. |
| **Research & Schema Validator** | `subagent-schema-validator` | Validates API versions, CRDs (Gateway API, KServe, Kuadrant, NVIDIA), and hardware constraints. |
| **Platform & Infra Engineer** | `subagent-infra-engineer` | Produces Day 0/Day 1 manifests: GPU Operator, MIG ConfigMaps, Time-Slicing, ODF/CephFS, Kueue. |
| **MLOps & Serving Engineer** | `subagent-serving-engineer` | Produces KServe InferenceServices, vLLM flags, Audio pipelines, Model Registry, Kuadrant MaaS Gateway. |
| **QA & Verification Engineer** | `subagent-qa-verifier` | Writes Python/cURL verification suites, failure injection tests, troubleshooting guides, runs `mkdocs build --strict`. |

---

## 2. Target MkDocs Site Structure

The documentation must be organized into this exact repository structure:

```
rhoai-multihub-architecture/
├── mkdocs.yml
├── requirements.txt
└── docs/
    ├── index.md
    ├── executive/
    │   ├── executive-summary.md
    │   ├── business-value-tco.md
    │   └── sla-dr-strategy.md
    ├── architecture/
    │   ├── global-topology.md
    │   ├── datacenter-1-h100.md
    │   ├── datacenter-2-l40s.md
    │   ├── maas-gateway-kuadrant.md
    │   └── storage-networking.md
    ├── platform-prerequisites/
    │   ├── cluster-baselines.md
    │   ├── odf-cephfs-rwx-setup.md
    │   ├── nvidia-gpu-operator.md
    │   └── kueue-priority-classes.md
    ├── gpu-partitioning/
    │   ├── dc1-h100-mig-profiles.md
    │   ├── dc2-l40s-time-slicing.md
    │   └── accelerator-profiles.md
    ├── model-serving/
    │   ├── flagship-llm-qwen72b.md
    │   ├── failover-resiliency-mechanisms.md
    │   ├── small-models-fleet-l40s.md
    │   └── speech-translation-pipeline.md
    ├── maas-governance/
    │   ├── unified-gateway-api.md
    │   ├── authorino-kuadrant-auth.md
    │   ├── rate-limiting-quotas.md
    │   └── self-service-api-keys.md
    ├── workflows/
    │   ├── interactive-jupyter-dev.md
    │   ├── adhoc-distributed-training.md
    │   └── model-registry-cicd.md
    ├── verification/
    │   ├── end-to-end-validation-suite.md
    │   ├── failure-injection-testing.md
    │   └── performance-benchmarking.md
    └── operations/
        ├── runbooks-node-failure.md
        ├── monitoring-dcgm-prometheus.md
        └── troubleshooting-faq.md

```

---

## 3. Phased Execution Plan & Quality Gates

```
┌──────────────────┐     Gate 1     ┌──────────────────┐     Gate 2     ┌──────────────────┐
│  Phase 1: Setup  │ ─────────────> │ Phase 2: Schema  │ ─────────────> │ Phase 3: Exec &  │
│ & Scaffolding    │                │ & Architecture   │                │ Architecture Doc │
└──────────────────┘                └──────────────────┘                └──────────────────┘
                                                                                  │
┌──────────────────┐     Gate 5     ┌──────────────────┐     Gate 4     ┌─────────┴────────┐
│ Phase 6: Build,  │ <───────────── │  Phase 5: MaaS,  │ <───────────── │ Phase 4: Infra & │
│ QA & Package     │                │ Serving & Ops    │                │ GPU Partitioning │
└──────────────────┘                └──────────────────┘                └──────────────────┘

```

### Phase 1: Environment Scaffolding & Design Framework

* **Responsible:** `subagent-doc-architect`
* **Tasks:**
1. Generate `requirements.txt` containing `mkdocs`, `mkdocs-material`, `mkdocs-mermaid2-plugin`, `pymdown-extensions`.
2. Author `mkdocs.yml` with Material theme, syntax highlighting, code copy buttons, tabbed content containers, and navigation structure.
3. Generate stub index and landing page (`docs/index.md`).


* **Quality Gate 1:** `mkdocs build` executes cleanly with zero syntax errors.

### Phase 2: Schema Research, Verification & Architecture Validation

* **Responsible:** `subagent-schema-validator`
* **Tasks:**
1. Validate OpenShift AI 2.x / 3.x CRDs: `serving.kserve.io/v1beta1`, `serving.kserve.io/v1alpha1` (`ServingRuntime`), `gateway.networking.k8s.io/v1` (`HTTPRoute`, `Gateway`), `kuadrant.io/v1beta2` (`RateLimitPolicy`, `AuthPolicy`), and `kueue.x-k8s.io/v1beta1`.
2. Compile hardware matrix: Confirm MIG configurations (`3g.40gb`, `1g.10gb`) and verify that which GPUs do not support MIG, enforcing GPU time-slicing via NVIDIA GPU Operator ConfigMaps if required.
  1. Ask the following: How Many Data Centres?, How Many Nodes?, Number and Type of GPU's in each node.
3. Produce a consolidated Technical Schema Reference document for Subagents 3 and 4.


* **Quality Gate 2:** Schema reference includes zero deprecated APIs and confirms all hardware-specific flags (e.g., `--kv-cache-dtype fp8`, `--tensor-parallel-size 4`).

### Phase 3: Executive Strategy & Architecture Documentation

* **Responsible:** `subagent-doc-architect` (with inputs from `subagent-schema-validator`)
* **Tasks:**
1. Author `docs/executive/` files: Executive Summary, Business Value/TCO, and SLA/DR Strategy.
2. Create Mermaid diagrams illustrating:
* Global Multi-Datacenter routing topology.
* Unified MaaS Gateway routing tree.


3. Author `docs/architecture/` files detailing memory bandwidth comparisons, inter-datacenter networking, storage replication (ODF), and security posture.


* **Quality Gate 3:** Executive documentation clearly articulates GPU utilization ROI, multi-tenant isolation, zero-replica failover cost benefits, and latency profiles.

### Phase 4: Platform Infrastructure & GPU Partitioning Directives

* **Responsible:** `subagent-infra-engineer`
* **Tasks:**
1. Write `docs/platform-prerequisites/`:
* ODF CephFS ReadWriteMany (RWX) storage manifests for weight caching.
* Node Feature Discovery (NFD) and NVIDIA GPU Operator installation.
* Kueue `ClusterQueue`, `ResourceFlavor`, `LocalQueue`, and `PriorityClass` (`flagship-critical-priority` vs. `batch-low-priority`).


2. Write `docs/gpu-partitioning/`:
* DC1 Node 1 (`all-disabled`) and Node 2 (`node2-hybrid-profile` with 4 full + 4 MIG GPUs).
* DC2 L40S 4x time-slicing ConfigMap and `ClusterPolicy` patches.
* OpenShift AI AcceleratorProfiles for suggested MIG configurations, and unshared GPUs.




* **Quality Gate 4:** Every Kubernetes manifest must be completely functional, including explicit `metadata.namespace`, `spec.template`, labels, annotations, and zero ellipses (`# ...`).

### Phase 5: Model Serving, MaaS Gateway & Operational Runbooks

* **Responsible:** `subagent-serving-engineer` and `subagent-qa-verifier`
* **Tasks:**
1. Write `docs/model-serving/`:
* Ask which model should be considered for flagship general purpose work. Suggest most capable models for the architecture provided.
* Flagship KServe manifest: 1 replica, Node 1 affinity, 10s node-eviction tolerations, CephFS RWX volume mount, vLLM TP=4 FP8.
* Node failure recovery dynamics and preemption on Node 2.
* Whisper STT (vLLM transcription) and TTS (Kokoro-82M / Piper custom runtime) manifests with Serverless scale-to-zero.


2. Write `docs/maas-governance/`:
* Gateway API `Gateway`, `HTTPRoute` mapping paths to internal services.
* Kuadrant `AuthPolicy` (API keys, OIDC) and `RateLimitPolicy` (RPM/TPM tiers).
* Self-service subscription workflows and token showback dashboards.


3. Write `docs/workflows/`, `docs/verification/`, and `docs/operations/`:
* Jupyter workbench setup with idle culling.
* Ad-hoc 4-GPU RayCluster training manifests with CephFS checkpointing.
* End-to-end Python test client using OpenAI SDK.
* Failure injection script simulating Node 1 cordoning, hard failure, and failover validation.
* DCGM Prometheus metrics and troubleshooting guides.




* **Quality Gate 5:** End-to-end Python test scripts, cURL commands, and failover runbooks pass dry-run linting.

### Phase 6: QA, Build Validation & Final Packaging

* **Responsible:** `agent-orchestrator` and `subagent-qa-verifier`
* **Tasks:**
1. Run `mkdocs build --strict` to verify that there are no dead links, missing anchors, or markdown syntax violations.
2. Validate that all bash, python, and yaml code snippets feature clean copy-paste formatting and explicit variable declarations.
3. Output deployment summary and verify all project requirements are met.


* **Quality Gate 6:** Build passes with exit code `0`. Site generates cleanly in `./site`.

---

## 4. Subagent Specification & Execution Directives

### Directives for `subagent-doc-architect`

```yaml
agent_name: subagent-doc-architect
target_files:
  - mkdocs.yml
  - requirements.txt
  - docs/index.md
  - docs/executive/*.md
  - docs/architecture/*.md
style_requirements:
  theme: material
  features:
    - navigation.instant
    - navigation.tracking
    - navigation.tabs
    - navigation.sections
    - navigation.expand
    - content.code.copy
    - content.tabs.link
  palette:
    - scheme: slate
      primary: red
      accent: red
executive_writing_rules:
  - "Target audience: CIO, CTO, VP of Infrastructure, Chief AI Architect."
  - "Articulate the financial advantage of zero-replica active failover over running dual 72B replicas (saving 4x H100s = ~$140k+ capital/cloud cost)."
  - "Highlight GPU utilization economics: combining MIG and time-slicing to achieve >80% sustained utilization."
  - "Include clear Mermaid diagrams for multi-datacenter topology and failover state machines."

```

### Directives for `subagent-schema-validator`

```yaml
agent_name: subagent-schema-validator
verification_targets:
  - "KServe API Version: serving.kserve.io/v1beta1 (InferenceService)"
  - "KServe ServingRuntime: serving.kserve.io/v1alpha1"
  - "Gateway API: gateway.networking.k8s.io/v1"
  - "Kuadrant RateLimitPolicy & AuthPolicy: kuadrant.io/v1beta2"
  - "Kueue ClusterQueue & LocalQueue: kueue.x-k8s.io/v1beta1"
  - "NVIDIA GPU Operator ClusterPolicy: nvidia.com/v1"
technical_rules:
  - "H100 SXM5 supports MIG. Validate exact profile strings: '3g.40gb', '1g.10gb'."
  - "L40S PCIe does NOT support MIG. Ensure time-slicing is used exclusively in DC2."
  - "Flagship vLLM flags MUST include: --tensor-parallel-size 4, --kv-cache-dtype fp8, --gpu-memory-utilization 0.92, --max-model-len 32768."
  - "Whisper vLLM flags MUST include: --task transcription."
  - "Verify that all PVC mounts use 'ReadWriteMany' with CephFS storage classes."

```

### Directives for `subagent-infra-engineer`

```yaml
agent_name: subagent-infra-engineer
target_files:
  - docs/platform-prerequisites/*.md
  - docs/gpu-partitioning/*.md
engineering_mandates:
  - "ZERO PLACEHOLDERS: Prohibit '# <insert config here>' or '...'. All YAMLs must be full, valid manifests."
  - "Include exact namespace definitions and pre-requisite operator subscriptions."
  - "Provide complete ConfigMaps for DC1 (MIG) and DC2 (Time-Slicing)."
  - "Define exact PriorityClasses (flagship-critical-priority = 1000000000 with PreemptLowerPriority; batch-low-priority = 10000)."
  - "Provide CephFS RWX PVC definitions and exact commands to pre-seed weights from S3 to CephFS."

```

### Directives for `subagent-serving-engineer`

```yaml
agent_name: subagent-serving-engineer
target_files:
  - docs/model-serving/*.md
  - docs/maas-governance/*.md
  - docs/workflows/*.md
engineering_mandates:
  - "Deploy Flagship LLM as a single replica with 10s eviction toleration (not-ready, unreachable)."
  - "Include full Kuadrant HTTPRoute mapping: /v1/chat/completions -> Qwen-72B (DC1), /v1/audio/transcriptions -> Whisper (DC2), /v1/audio/speech -> TTS (DC2), /v1/embeddings -> TEI (DC2)."
  - "Provide complete AuthPolicy and RateLimitPolicy YAMLs with tiered token quotas."
  - "Provide RayCluster and RayJob YAMLs configured to run on DC1 Node 2 (GPUs 0-3) under low priority."
  - "Provide KServe Serverless scale-to-zero configurations for Whisper and Kokoro TTS."

```

### Directives for `subagent-qa-verifier`

```yaml
agent_name: subagent-qa-verifier
target_files:
  - docs/verification/*.md
  - docs/operations/*.md
testing_mandates:
  - "Write complete Python validation script using the official 'openai' library demonstrating: (1) streaming chat completion, (2) audio transcription, (3) text-to-speech audio saving, (4) embedding generation."
  - "Write a shell-based chaos script that drains/cordons Node 1, logs failover timing, monitors eviction and preemption of batch jobs on Node 2, and validates endpoint recovery under 60 seconds."
  - "Include DCGM Prometheus recording rules and Grafana dashboard metrics for GPU compute, VRAM, and MIG instance health."
  - "Execute 'mkdocs build --strict' and resolve any broken cross-references or syntax issues."

```

## 5. Verification Criteria & Deliverable Checklist

Before delivering the generated documentation, the Lead Orchestrator must verify that all the following deliverables are present and complete:

* [ ] **Dual-Audience Coverage:**
* Executive/Architect track articulates total cost of ownership (TCO), multi-datacenter governance, hardware selection economics (H100 vs. L40S), and high availability strategy.
* Engineering track provides copy-pasteable CLI commands and production-grade manifests with zero placeholder comments.


* [ ] **Hardware Partitioning Configuration:**
* Data Center 1: Node 1 (8x full H100s) and Node 2 (GPUs 0–3 full, GPUs 4–7 MIG partitioned into `3g.40gb` and `1g.10gb`).
* Data Center 2: 4 nodes $\times$ 4x L40S configured with 4x time-slicing via NVIDIA GPU Operator ConfigMaps.


* [ ] **Failover & High Availability:**
* Single-replica flagship Qwen-72B model on Node 1 configured with a 10-second node eviction toleration.
* Preemption priority class (`flagship-critical-priority`) that automatically frees GPUs 0–3 on Node 2 during node failure.
* CephFS ReadWriteMany (RWX) storage volume configuration to eliminate model weight download bottlenecks.


* [ ] **Models-as-a-Service (MaaS) Gateway:**
* Single OpenAI-compatible gateway (`/v1`) using Kuadrant, Istio, and Authorino.
* Path-based routing rules directing reasoning to DC1, and STT, TTS, and embeddings to DC2.
* Token-per-minute (TPM) and request-per-minute (RPM) rate limiting policies with self-service API key provisioning.


* [ ] **Technical Validation:**
* `mkdocs build --strict` executes with exit code `0` and zero warnings.
* Complete, end-to-end Python test scripts using the OpenAI SDK.
* Node-failure chaos injection scripts and operational troubleshooting runbooks.
