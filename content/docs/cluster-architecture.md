---
title: "Cluster Architecture Sizing in OpenShift Migration Advisor"
linkTitle: "Cluster Architecture"
description: "VMware inventory를 기반으로 OpenShift cluster requirement를 계산합니다."
weight: 5
---

# Cluster Architecture Sizing

Cluster Architecture Sizing 기능은 VMware workload를 수용하기 위해 필요한 OpenShift cluster size(nodes, CPU, memory)를 계산합니다. 두 가지 mode를 제공합니다.

- **Assessment-based sizing** — 완료된 assessment의 VMware cluster inventory를 분석합니다.
- **Standalone sizing** — 가상의 inventory total을 입력해 sizing을 계산합니다(assessment 불필요).

두 mode는 동일한 calculation algorithm과 deployment option을 사용하여 workload에 적합한 bare-metal OpenShift configuration을 산정합니다.

Assessment-based sizing은 vCenter report의 UI에서 사용할 수 있습니다.

Applies to: V0.11.0  
Last update (dd-mm-yyyy): 20-05-2026

---

## How It Works

Cluster sizing 기능은 정교한 batching 및 scheduling algorithm을 통해 VMware inventory data를 OpenShift cluster specification으로 변환합니다.

### High-Level Flow

```
1. Inventory에서 VMware cluster 선택
2. Deployment mode 구성(Full HA / Single Node / Hosted Control Plane)
3. Worker node hardware 지정(CPU, memory, optional SMT threads)
4. CPU와 memory over-commit ratio 구성
5. (Optional) Control plane node specification 구성
6. 시스템 수행 작업:
   a. Cluster 내 모든 VM을 service batch로 집계
   b. Kubelet resource overhead 계산 적용
   c. Over-commit ratio를 적용하여 Kubernetes requests와 limits 산정
   d. Batched service를 worker node에 scheduling
   e. 필요한 control plane node 계산(해당되는 경우)
   f. Failover capacity 추가(multi-node cluster의 경우)
7. Node count, resource total, detailed placement가 포함된 sizing recommendation 수신
```

UI에서는 1–7단계가 vCenter report에서 **Recommendation** page를 열고, **OpenShift Cluster Architecture** tab을 구성한 뒤 **Generate recommendation**을 클릭하는 흐름으로 매핑됩니다. [Using the UI](#using-the-ui)를 참고하세요.

---

## Using the UI

이 섹션은 Migration Advisor UI에서 cluster architecture sizing을 수행하는 방법을 설명합니다.

### Open cluster architecture sizing

1. Assessment를 완료하고 해당 assessment의 **vCenter report**를 엽니다.
2. Cluster dropdown에서 sizing할 VMware cluster를 선택합니다.
3. **View Recommendation based on vCenter cluster**를 클릭합니다.

![vCenter report — select a cluster and open recommendations](https://kubev2v.github.io/openshift-migration-advisor-docs/images/docs/cluster-architecture/01-vcenter-report-recommendation-entry.png)

### Configure OpenShift cluster architecture

**Recommendation** page에는 관련 planning feature를 위한 tab이 표시됩니다.

| Tab | Purpose |
|-----|---------|
| **OpenShift Cluster Architecture** | Cluster sizing(이 기능) |
| **Migration Time Estimation** | Storage throughput 및 migration duration |
| **Migration Complexity** | Workload complexity scoring |
| **Migration Plan** | Migration planning(prerequisite가 충족될 때까지 사용할 수 없을 수 있음) |

**OpenShift Cluster Architecture** tab에 머무릅니다.

#### Migration preferences

| UI control | API / doc parameter | Notes |
|------------|---------------------|-------|
| **Cluster mode** | Deployment mode | **Full HA (3CP)** → Full HA; **Single Node** → SNO; **Hosted Control Plane** → HCP |
| **Run workloads on control plane nodes** | `controlPlaneSchedulable` | SNO에서는 필수이며, production Full HA에서는 권장되지 않음 |
| **Enable SMT/Hyperthreading** + thread count | `workerNodeThreads` | 활성화 시 worker node당 total CPU threads를 설정 |

![Recommendation page — migration preferences and worker node settings](https://kubev2v.github.io/openshift-migration-advisor-docs/images/docs/cluster-architecture/02-recommendation-config-top.png)

#### Worker node and control plane

아래로 scroll하여 worker와 control plane 설정을 완료한 뒤 **Generate recommendation**을 클릭합니다.

| UI control | API / doc parameter |
|------------|---------------------|
| **Worker node CPU core** | `workerNodeCPU` |
| **Worker node memory (GB)** | `workerNodeMemory` |
| **CPU overcommitment** | `cpuOverCommitRatio` (예: Standard (1:4) → `"1:4"`) |
| **Memory overcommitment** | `memoryOverCommitRatio` |
| **Control plane CPU core** | `controlPlaneCPU` (HCP에서는 표시되지 않음) |
| **Control plane memory (GB)** | `controlPlaneMemory` (HCP에서는 표시되지 않음) |

**Single Node** mode에서는 worker와 control plane CPU/memory를 같은 값으로 설정합니다([SNO sizing fields](#single-node-openshift-sno) 참고).

![Recommendation page — worker node, control plane, and Generate recommendation](https://kubev2v.github.io/openshift-migration-advisor-docs/images/docs/cluster-architecture/03-recommendation-config-bottom.png)

{{% alert title="Note" color="info" %}}
UI에 표시되는 resource requirement는 현재 workload를 기반으로 한 estimate입니다. Procurement 또는 build-out 전에 platform team과 recommended architecture를 확인하세요.
{{% /alert %}}

### Read cluster recommendations

생성 후 **Cluster recommendations** panel이 결과를 요약합니다. **Copy as plain text**를 사용해 architect에게 공유하거나 runbook에 붙여넣을 수 있습니다.

| UI field | Related response / doc fields |
|----------|-------------------------------|
| **Total nodes** (workers + control plane) | `totalNodes`, `workerNodes`, `controlPlaneNodes` |
| **Failover capacity** | `failoverNodes` |
| **Node size** (worker / control plane) | Form에서 구성한 hardware |
| **Overcommitment** | `cpuOverCommitRatio`, `memoryOverCommitRatio` |
| **Workload details** (VMs, ratios) | `inventoryTotals`, effective over-commit |
| **Resources** (requests, limits, physical capacity) | `resourceConsumption` |

![Cluster recommendations results](https://kubev2v.github.io/openshift-migration-advisor-docs/images/docs/cluster-architecture/04-recommendation-results.png)

결과가 표시되면 **Migration preferences**는 접힙니다. Deployment mode, node size, over-commit setting을 변경하려면 **Migration preferences**를 펼치고 값을 수정한 뒤 **Generate recommendation**을 다시 클릭합니다.

Programmatic access가 필요한 경우 동일한 input/output을 [API Workflow](#api-workflow)에서 사용할 수 있습니다.

---

## Deployment Modes

이 기능은 세 가지 OpenShift deployment architecture를 지원합니다. UI에서는 **Migration preferences** 아래 **Cluster mode** dropdown에서 mode를 선택합니다.

| Mode | Cluster mode (UI) | Control plane | Worker nodes | Use case |
| --- | --- | --- | --- | --- |
| **Full HA** | Full HA (3CP) | 3 nodes(default: 6 CPU, 16 GB each) | 사용자가 지정한 count와 size | High availability가 필요한 production cluster |
| **Single Node OpenShift (SNO)** | Single Node | 1 node(`controlPlaneNodeCount: 1`) | 별도 worker pool 없음; workload는 control plane에서 실행. `workerNode*`와 `controlPlane*` 상호작용은 [SNO sizing fields](#single-node-openshift-sno) 참고 | Edge, small footprint, development |
| **Hosted Control Plane (HCP)** | Hosted Control Plane | External(별도 관리, sizing 대상 아님) | 사용자가 지정한 count와 size | Multi-tenant platform, reduced overhead |

### Mode-Specific Configuration

#### Full HA (3 Control Plane Nodes)

- **Default**: 3 control plane nodes, each with 6 CPU cores and 16 GB memory
- **Configurable**: Control plane CPU(2-200 cores), memory(4-512 GB)
- **Control plane scheduling**: Optional. 활성화하면 workload를 control plane node에 schedule할 수 있습니다(production에서는 권장하지 않음).
- **Reserved resources**: Control plane service(etcd, kube-apiserver 등)를 위해 control plane node당 3.5 CPU와 13.39 GB reserved

#### Single Node OpenShift (SNO)

`controlPlaneNodeCount`가 `1`이면 SNO로 선택됩니다. 물리 node는 하나이며 모든 workload가 그 node에 schedule됩니다.

- **`controlPlaneSchedulable`**: 반드시 `true`여야 합니다(`false`이면 API가 error 반환).
- **`controlPlaneNodeCount`**: 반드시 `1`이어야 합니다.
- **Reserved resources**: 해당 node의 Kubernetes control plane service를 위해 3.5 CPU와 13.39 GB reserved

**Sizing fields (two inputs, one node):** UI와 API는 **Worker node** 및 **Control plane** CPU/memory field를 모두 제공합니다. SNO에서는 batching과 placement consistency를 위해 두 값을 **같게** 설정하세요. API는 값이 일치하는지 검증하지 않으므로, 값이 다르면 잘못된 결과가 나올 수 있습니다.

| Field group | Role in SNO |
|-------------|-------------|
| `workerNodeCPU`, `workerNodeMemory`, `workerNodeThreads` | **VM batching** — inventory가 service batch로 나뉘는 방식(SMT는 batch target 계산에서만 effective CPU에 사용) |
| `controlPlaneCPU`, `controlPlaneMemory` | **Node placement** — sizer에 전달되는 single node의 physical size(physical cores; SMT는 이 값에 적용되지 않음). Fit/min-size error message에도 사용 |
| `controlPlaneSchedulable` | 반드시 `true`; workload는 control plane machine set에 배치 |

#### Hosted Control Plane (HCP)

- **Control plane**: 외부에서 관리되며 sizing에 포함되지 않습니다.
- **Worker nodes**: Worker node만 sizing합니다.
- **Restrictions**: Control plane CPU, memory, node count를 지정할 수 없습니다(HCP mode와 호환되지 않음).

---

## Configuration Options

다음 표는 cluster sizing에 사용되는 모든 configuration parameter를 설명합니다.

| Parameter | Type | Required | Default | Constraints | Description |
|-----------|------|----------|---------|-------------|-------------|
| `clusterId` | string | Assessment-based only | — | — | VMware cluster identifier(예: `"domain-c8"`). **Standalone mode에서는 사용하지 않음.** |
| `totalVMs` | integer | Standalone only | — | 1–10,000 | 총 VM 수. **Assessment-based mode에서는 사용하지 않음(cluster inventory에서 산출).** |
| `totalCPU` | integer | Standalone only | — | 1–100,000 | 모든 VM의 총 CPU cores. **Assessment-based mode에서는 사용하지 않음.** |
| `totalMemory` | integer | Standalone only | — | 1–500,000 | 모든 VM의 총 memory(GB). **Assessment-based mode에서는 사용하지 않음.** |
| `cpuOverCommitRatio` | enum | Yes | `"1:4"` | `"1:1"`, `"1:2"`, `"1:4"`, `"1:6"` | CPU over-commit ratio. Kubernetes CPU requests가 limits 대비 어느 정도인지 결정 |
| `memoryOverCommitRatio` | enum | Yes | `"1:2"` | `"1:1"`, `"1:2"`, `"1:4"` | Memory over-commit ratio. Kubernetes memory requests가 limits 대비 어느 정도인지 결정 |
| `workerNodeCPU` | integer | Yes | — | 2–200 | Worker node당 physical CPU cores. SNO에서는 VM batching에만 사용되며 `controlPlaneCPU`와 일치해야 함 |
| `workerNodeMemory` | integer | Yes | — | 4–512 | Worker node당 memory(GB). SNO에서는 VM batching에만 사용되며 `controlPlaneMemory`와 일치해야 함 |
| `workerNodeThreads` | integer | No | — | 2–2000 | Worker node당 total CPU threads(SMT). Batching effective CPU에만 영향. `workerNodeCPU` 이상이어야 함. 생략하면 SMT 없음(threads = cores)으로 가정 |
| `controlPlaneSchedulable` | boolean | No | `false` | — | Control plane node에 workload scheduling 허용. SNO에서는 `true` 필수 |
| `controlPlaneCPU` | integer | No | `6` | 2–200 | Control plane node당 physical CPU cores. SNO에서는 sizer의 single node size를 정의. `workerNodeCPU`와 일치 권장. `hostedControlPlane`과 함께 사용할 수 없음 |
| `controlPlaneMemory` | integer | No | `16` | 4–512 | Control plane node당 memory(GB). SNO에서는 sizer의 single node memory를 정의. `workerNodeMemory`와 일치 권장. `hostedControlPlane`과 함께 사용할 수 없음 |
| `controlPlaneNodeCount` | integer | No | `3` | `1` or `3` | SNO는 `1`, Full HA는 `3`. `hostedControlPlane`과 함께 사용할 수 없음 |
| `hostedControlPlane` | boolean | No | `false` | — | External control plane mode. 모든 control plane configuration field와 호환되지 않음 |

### Over-Commit Ratio Descriptions

**CPU Over-Commit Ratios:**
- `1:1` — over-commit 없음. Kubernetes CPU requests = limits. 최대 isolation.
- `1:2` — moderate over-commit. Requested CPU의 2배까지 burst 가능.
- `1:4` — standard over-commit(default). Density와 performance 균형.
- `1:6` — aggressive over-commit. Density 최대화, bursty workload에 적합.

**Memory Over-Commit Ratios:**
- `1:1` — over-commit 없음. Kubernetes memory requests = limits. 최대 isolation.
- `1:2` — moderate over-commit(default). Requested memory의 2배까지 burst 가능.
- `1:4` — aggressive over-commit. Memory usage가 낮은 것으로 알려진 workload에 적합.

**Note on API formats:** Request에서는 over-commit ratio를 string으로 지정합니다(예: `"1:4"`). Response에서는 `overCommitRatio` field가 numeric multiplier 값을 포함합니다(예: `"1:4"`는 `4.0`, `"1:2"`는 `2.0`).

---

## Calculations

### VM Batching Algorithm

시스템은 cluster 내 모든 VM을 OpenShift node에 schedule 가능한 batched service workload로 집계합니다. 이 algorithm은 resource distribution을 효율화하고 특정 node overload를 방지합니다.

For SNO, batching always uses `workerNodeCPU`, `workerNodeMemory`, and `workerNodeThreads`; the sizer then places batches on a single node sized by `controlPlaneCPU` and `controlPlaneMemory`.

#### Algorithm Steps

1. **Target batch capacity 계산**(worker node capacity의 80%):
   ```
   targetCPU = effectiveWorkerNodeCPU × 0.8
   targetMemory = workerNodeMemory × 0.8
   ```

2. **필요 batch 수 결정**:
   ```
   batchesCPU = ceil(totalCPU / targetCPU)
   batchesMemory = ceil(totalMemory / targetMemory)
   batchesVMCount = ceil(totalVMs / 200)  // Max 200 VMs per node
   numBatches = max(batchesCPU, batchesMemory, batchesVMCount)
   ```

3. **Resource를 batch에 균등 분배**:
   ```
   cpuPerBatch = totalCPU / numBatches
   memoryPerBatch = totalMemory / numBatches
   vmsPerBatch = totalVMs / numBatches (with remainder distributed to first batches)
   ```

4. **Minimum batch constraints 확인**:
   - Minimum CPU per batch: 0.1 cores
   - Minimum memory per batch: 2.0 GB
   - Minimum보다 작으면 constraint를 충족하도록 batch를 재계산

5. **Over-commit ratio 적용**:
   ```
   limitCPU = cpuPerBatch
   limitMemory = memoryPerBatch
   requiredCPU = limitCPU / cpuOverCommitMultiplier
   requiredMemory = limitMemory / memoryOverCommitMultiplier
   ```

   Where:
   - `cpuOverCommitMultiplier`: 1.0 (1:1), 2.0 (1:2), 4.0 (1:4), 6.0 (1:6)
   - `memoryOverCommitMultiplier`: 1.0 (1:1), 2.0 (1:2), 4.0 (1:4)

#### Example

**Input:**
- Total VMs: 100
- Total CPU: 400 cores
- Total Memory: 800 GB
- Worker node: 32 CPU, 64 GB memory
- SMT enabled: 128 threads(effective: 80 cores)
- CPU over-commit: 1:4
- Memory over-commit: 1:2

**Calculation:**
```
Target capacity:
  targetCPU = 80 × 0.8 = 64 cores
  targetMemory = 64 × 0.8 = 51.2 GB

Batches needed:
  batchesCPU = ceil(400 / 64) = 7
  batchesMemory = ceil(800 / 51.2) = 16
  batchesVMCount = ceil(100 / 200) = 1
  numBatches = max(7, 16, 1) = 16

Resources per batch:
  cpuPerBatch = 400 / 16 = 25 cores
  memoryPerBatch = 800 / 16 = 50 GB
  vmsPerBatch = 100 / 16 = 6 (4 batches get 7 VMs)

Kubernetes resources per batch:
  limitCPU = 25 cores
  limitMemory = 50 GB
  requiredCPU = 25 / 4.0 = 6.25 cores
  requiredMemory = 50 / 2.0 = 25 GB
```

Result: 16 service batches, each requesting 6.25 CPU / 25 GB with limits of 25 CPU / 50 GB.

---

### SMT/Hyperthreading Effective CPU

SMT(Simultaneous Multithreading) 또는 Hyperthreading이 활성화되어 있으면 physical core를 초과하는 thread에 대해 50% efficiency factor를 적용하여 effective CPU capacity를 계산합니다.

#### Formula

```
effectiveCPU = physicalCores + ((threads - physicalCores) × 0.5)
```

`threads`가 제공되지 않거나 `physicalCores`와 같으면 formula는 `physicalCores`를 반환합니다(SMT 없음).

#### Examples

| Physical Cores | Threads | Calculation | Effective CPU |
|----------------|---------|-------------|---------------|
| 16 | 32 | 16 + ((32 - 16) × 0.5) | 24 |
| 32 | 64 | 32 + ((64 - 32) × 0.5) | 48 |
| 64 | 128 | 64 + ((128 - 64) × 0.5) | 96 |
| 16 | 16 | 16 (no SMT) | 16 |

**Rationale**: SMT thread는 core resource(execution units, cache)를 공유하므로 physical core 대비 약 50%의 추가 throughput을 제공한다고 가정합니다.

---

### Kubelet Resource Overhead

OpenShift는 각 node에서 kubelet, container runtime, system service를 위해 CPU와 memory를 reserve합니다. 이 reservation은 workload pod에 할당 가능한 capacity를 줄입니다.

#### Memory Reservation

Memory overhead는 total node memory를 기준으로 tiered percentage를 적용해 계산합니다.

| Node Memory | Reservation Formula |
|-------------|---------------------|
| **< 1 GB** | Flat 255 MiB |
| **≥ 1 GB** | 25% of first 4 GB<br>+ 20% of next 4 GB<br>+ 10% of next 8 GB<br>+ 6% of next 112 GB<br>+ 2% of memory above 128 GB |

Reservation은 tax bracket처럼 bracketed calculation을 사용합니다. 1 GB 이상의 memory를 가진 node에서는 각 tier가 해당 범위의 memory에만 적용됩니다.

| Memory Tier | Percentage | Max Reserved |
|-------------|------------|--------------|
| First 4 GB | 25% | 1.0 GB |
| Next 4 GB (4–8 GB) | 20% | 0.8 GB |
| Next 8 GB (8–16 GB) | 10% | 0.8 GB |
| Next 112 GB (16–128 GB) | 6% | 6.72 GB |
| Above 128 GB | 2% | — |

**Example calculation for 64 GB node:**
```
Reserved = (0.25 × 4) + (0.20 × 4) + (0.10 × 8) + (0.06 × 48)
         = 1.0 + 0.8 + 0.8 + 2.88
         = 5.48 GB
```

#### CPU Reservation

CPU overhead는 total node CPU를 기준으로 tiered percentage를 적용해 계산합니다.

| Node CPU Range | Formula | Example (32 CPU node) |
|----------------|---------|----------------------|
| First core | 0.06 | 0.06 |
| 1–2 cores | 0.06 + 0.01 × (cpu - 1) | 0.06 + 0.01 = 0.07 |
| 2–4 cores | [previous] + 0.005 × (cpu - 2) | 0.07 + 0.01 = 0.08 |
| > 4 cores | [previous] + 0.0025 × (cpu - 4) | 0.08 + (0.0025 × 28) = **0.15 cores** |

**Cumulative formula** for 32 cores:
```
reserved = 0.06 + (0.01 × 1) + (0.005 × 2) + (0.0025 × 28)
         = 0.06 + 0.01 + 0.01 + 0.07
         = 0.15 cores
```

이 reservation은 scheduling algorithm에서 자동으로 고려됩니다.

---

### Over-Commit Ratios

Over-commit ratio는 Kubernetes resource **requests**(guaranteed resources)와 **limits**(maximum burst resources)의 관계를 제어합니다. 이를 통해 pod가 guaranteed allocation을 초과해 burst할 수 있으므로 workload density를 높일 수 있습니다.

#### How It Works

VM이 Kubernetes pod specification으로 변환될 때:

1. **Limit** = VM의 전체 resource allocation(CPU cores, memory GB)
2. **Request** = Limit / over-commit multiplier

**Example** (1 VM with 4 CPU / 8 GB, 1:4 CPU, 1:2 memory over-commit):
```yaml
resources:
  requests:
    cpu: 1 core       # 4 / 4
    memory: 4 GB      # 8 / 2
  limits:
    cpu: 4 cores
    memory: 8 GB
```

이 pod는 1 CPU와 4 GB를 **guaranteed**로 받지만, node에 capacity가 있으면 4 CPU와 8 GB까지 **burst**할 수 있습니다.

#### Scheduling Impact

Scheduler는 **requests**를 기준으로 node placement를 결정합니다. 1:4 CPU over-commit을 사용하면 32-core node는 theoretically 128 CPU cores에 해당하는 limit을 가진 pod를 schedule할 수 있습니다. 단, combined request가 node allocatable capacity를 초과하지 않아야 합니다.

#### Trade-offs

| Over-Commit Level | Density | Performance Risk |
|-------------------|---------|------------------|
| **1:1 (none)** | Low | 없음. Pod가 항상 full resources를 받음 |
| **1:2** | Medium | 낮음. 대부분의 workload에 적합 |
| **1:4** | High | 중간. 평균 utilization이 낮은 bursty workload에 적합 |
| **1:6** | Very High | 높음. known-bursty 또는 batch workload에만 적합. CPU throttling risk |

---

### Failover Capacity

Multi-node cluster(Full HA 및 HCP mode)의 경우, 시스템은 node failure 시 service disruption 없이 대응할 수 있도록 extra worker nodes를 추가합니다.

#### Formula

```
failoverNodes = max(2, ceil(workerNodes × 0.10))
```

Failover capacity는 다음 중 **더 큰 값**입니다.
- 2 nodes(의미 있는 HA를 위한 최소값)
- worker node count의 10%(올림)

#### Examples

| Worker Nodes | 10% | Failover Nodes |
|--------------|-----|----------------|
| 5 | 0.5 | **2** (minimum) |
| 15 | 1.5 | **2** (10% rounds to 2) |
| 25 | 2.5 | **3** (10% rounds to 3) |
| 50 | 5.0 | **5** |
| 100 | 10.0 | **10** |

**Total nodes** = worker nodes + failover nodes + control plane nodes(if applicable)

**Rationale**: Worker node 하나가 실패하더라도 workload가 surviving node에 reschedule되고 target 80% node capacity를 초과하지 않도록 보장합니다.

---

### Minimum Node Size Calculation

UI에 "minimum recommended node size" message가 표시되는 경우, 이는 cluster가 operational limit(max 100 nodes) 내에 들어가도록 보장하기 위한 계산입니다.

#### Formula

```
minEffectiveCPU = inventoryCPU / (100 × 0.8)
minNodeCPU = ceil(minEffectiveCPU / smtMultiplier)
minNodeCPU = ceil(minNodeCPU / 2) × 2  // Round to even

minNodeMemory = ceil(inventoryMemory / (100 × 0.8))
minNodeMemory = ceil(minNodeMemory / 4) × 4  // Round to multiple of 4
```

Where:
- `inventoryCPU` = all VMs의 total CPU cores
- `inventoryMemory` = all VMs의 total memory(GB)
- `smtMultiplier` = effectiveCPU / physicalCores(예: 2:1 SMT의 경우 1.5)
- Max nodes = 100(architectural limit)
- Capacity multiplier = 0.8(target 80% utilization)

#### Example

**Input:**
- Inventory: 3200 CPU cores, 6400 GB memory
- SMT multiplier: 1.5(예: 32 cores, 64 threads → 48 effective)

**Calculation:**
```
minEffectiveCPU = 3200 / (100 × 0.8) = 40 effective cores
minNodeCPU = ceil(40 / 1.5) = ceil(26.67) = 27 cores
minNodeCPU = ceil(27 / 2) × 2 = 14 × 2 = 28 cores (rounded to even)

minNodeMemory = ceil(6400 / 80) = ceil(80) = 80 GB
minNodeMemory = ceil(80 / 4) × 4 = 20 × 4 = 80 GB (already multiple of 4)
```

**Result**: Minimum recommended node size는 28 CPU / 80 GB memory입니다.

사용자가 더 작은 node size를 선택하면 cluster가 100 nodes를 초과하게 되어 sizing request가 실패합니다.

---

## API Workflow

Cluster sizing 기능은 cluster requirement 계산을 위해 두 endpoint를 제공합니다.

1. **Assessment-based sizing** — 완료된 assessment의 기존 VMware cluster를 기준으로 sizing 계산
2. **Standalone sizing** — Inline inventory data를 사용해 hypothetical sizing 계산(assessment 불필요)

두 endpoint는 동일한 calculation algorithm을 사용하고 동일한 configuration parameter를 공유합니다. 주요 차이는 inventory data source입니다.

### Assessment-Based Sizing

기존 assessment의 VMware cluster에 대한 cluster sizing을 계산합니다.

**Endpoint:**
```
POST /api/v1/assessments/{assessmentId}/cluster-requirements
```

**Request Body Example** (Full HA with SMT):
```bash
curl -X POST http://localhost:3000/api/v1/assessments/123e4567-e89b-12d3-a456-426614174000/cluster-requirements   -H "Content-Type: application/json"   -d '{
    "clusterId": "domain-c8",
    "cpuOverCommitRatio": "1:4",
    "memoryOverCommitRatio": "1:2",
    "workerNodeCPU": 32,
    "workerNodeMemory": 64,
    "workerNodeThreads": 128,
    "controlPlaneSchedulable": false,
    "controlPlaneCPU": 6,
    "controlPlaneMemory": 16,
    "controlPlaneNodeCount": 3,
    "hostedControlPlane": false
  }'
```

**Request Body Example** (Single Node OpenShift):
```bash
curl -X POST http://localhost:3000/api/v1/assessments/123e4567-e89b-12d3-a456-426614174000/cluster-requirements   -H "Content-Type: application/json"   -d '{
    "clusterId": "domain-c8",
    "cpuOverCommitRatio": "1:4",
    "memoryOverCommitRatio": "1:2",
    "workerNodeCPU": 64,
    "workerNodeMemory": 128,
    "workerNodeThreads": 128,
    "controlPlaneSchedulable": true,
    "controlPlaneCPU": 64,
    "controlPlaneMemory": 128,
    "controlPlaneNodeCount": 1,
    "hostedControlPlane": false
  }'
```

**Request Body Example** (Hosted Control Plane):
```bash
curl -X POST http://localhost:3000/api/v1/assessments/123e4567-e89b-12d3-a456-426614174000/cluster-requirements   -H "Content-Type: application/json"   -d '{
    "clusterId": "domain-c8",
    "cpuOverCommitRatio": "1:4",
    "memoryOverCommitRatio": "1:2",
    "workerNodeCPU": 32,
    "workerNodeMemory": 64,
    "workerNodeThreads": 128,
    "hostedControlPlane": true
  }'
```

---

### Standalone Sizing

Inline inventory data로 hypothetical cluster sizing을 계산합니다. Full VMware assessment를 실행하지 않고 "what-if" scenario, capacity planning, sizing estimate에 사용할 수 있습니다.

**Endpoint:**
```
POST /api/v1/cluster-requirements
```

**Key Differences from Assessment-Based Sizing:**

| Aspect | Assessment-Based | Standalone |
|--------|------------------|------------|
| **Requires assessment** | Yes — 먼저 VMware data collection 완료 필요 | No — inventory total을 직접 제공 |
| **Cluster selection** | `clusterId` field(예: `"domain-c8"`) | 해당 없음 — cluster ID 불필요 |
| **Inventory source** | Assessment database에서 자동 조회 | Inline: `totalVMs`, `totalCPU`, `totalMemory` fields |
| **Use case** | 실제 migration 대상 VMware cluster sizing | Hypothetical sizing, capacity planning, pre-assessment estimates |

**Additional Required Parameters (Standalone Only):**

| Parameter | Type | Required | Constraints | Description |
|-----------|------|----------|-------------|-------------|
| `totalVMs` | integer | Yes | 1–10,000 | Sizing 대상 VM 총 수 |
| `totalCPU` | integer | Yes | 1–100,000 | 모든 VM의 총 CPU cores |
| `totalMemory` | integer | Yes | 1–500,000 | 모든 VM의 총 memory(GB) |

기타 configuration parameter(worker node specs, control plane settings, over-commit ratios)는 두 endpoint에서 **동일**합니다.

**Request Body Example** (Standalone Full HA):
```bash
curl -X POST http://localhost:3000/api/v1/cluster-requirements   -H "Content-Type: application/json"   -d '{
    "totalVMs": 100,
    "totalCPU": 400,
    "totalMemory": 800,
    "cpuOverCommitRatio": "1:4",
    "memoryOverCommitRatio": "1:2",
    "workerNodeCPU": 32,
    "workerNodeMemory": 64,
    "workerNodeThreads": 128,
    "controlPlaneSchedulable": false,
    "controlPlaneCPU": 6,
    "controlPlaneMemory": 16,
    "controlPlaneNodeCount": 3,
    "hostedControlPlane": false
  }'
```

**Request Body Example** (Standalone Single Node OpenShift):
```bash
curl -X POST http://localhost:3000/api/v1/cluster-requirements   -H "Content-Type: application/json"   -d '{
    "totalVMs": 20,
    "totalCPU": 80,
    "totalMemory": 160,
    "cpuOverCommitRatio": "1:4",
    "memoryOverCommitRatio": "1:2",
    "workerNodeCPU": 128,
    "workerNodeMemory": 256,
    "workerNodeThreads": 256,
    "controlPlaneSchedulable": true,
    "controlPlaneCPU": 128,
    "controlPlaneMemory": 256,
    "controlPlaneNodeCount": 1
  }'
```

**Request Body Example** (Standalone Hosted Control Plane):
```bash
curl -X POST http://localhost:3000/api/v1/cluster-requirements   -H "Content-Type: application/json"   -d '{
    "totalVMs": 150,
    "totalCPU": 600,
    "totalMemory": 1200,
    "cpuOverCommitRatio": "1:4",
    "memoryOverCommitRatio": "1:2",
    "workerNodeCPU": 32,
    "workerNodeMemory": 64,
    "workerNodeThreads": 128,
    "hostedControlPlane": true
  }'
```

---

### Response Format

두 endpoint는 동일한 response structure를 반환합니다.

**Response Example** (200 OK):
```json
{
  "clusterSizing": {
    "totalNodes": 20,
    "workerNodes": 15,
    "controlPlaneNodes": 3,
    "failoverNodes": 2,
    "totalCPU": 960,
    "totalMemory": 1280
  },
  "resourceConsumption": {
    "cpu": 100,
    "memory": 400,
    "limits": {
      "cpu": 400,
      "memory": 800
    },
    "overCommitRatio": {
      "cpu": 4.0,
      "memory": 2.0
    }
  },
  "inventoryTotals": {
    "totalVMs": 100,
    "totalCPU": 400,
    "totalMemory": 800
  }
}
```

---

## Understanding Results

### ClusterSizing Fields

| Field | Type | Description |
|-------|------|-------------|
| `totalNodes` | integer | 필요한 총 OpenShift nodes(workers + control plane + failover) |
| `workerNodes` | integer | Workload scheduling에 필요한 worker node 수 |
| `controlPlaneNodes` | integer | Control plane node 수(HCP=0, SNO=1, Full HA=3) |
| `failoverNodes` | integer | HA failover capacity를 위한 추가 node 수(SNO=0) |
| `totalCPU` | integer | 모든 node의 total physical CPU cores |
| `totalMemory` | integer | 모든 node의 total memory(GB) |

### ResourceConsumption Fields

| Field | Type | Description |
|-------|------|-------------|
| `cpu` | number | 모든 workload pod의 total Kubernetes CPU requests(cores) |
| `memory` | number | 모든 workload pod의 total Kubernetes memory requests(GB) |
| `limits.cpu` | number | 모든 workload pod의 total Kubernetes CPU limits(cores) |
| `limits.memory` | number | 모든 workload pod의 total Kubernetes memory limits(GB) |
| `overCommitRatio.cpu` | number | 적용된 CPU over-commit multiplier(예: `"1:4"` ratio는 4.0) |
| `overCommitRatio.memory` | number | 적용된 memory over-commit multiplier(예: `"1:2"` ratio는 2.0) |

**Note**: `cpu`와 `memory`는 over-commit 적용 후 Kubernetes resource requests(guaranteed resources)를 의미합니다. `limits`는 over-commit 이전의 실제 VM resource allocation을 의미합니다.

### InventoryTotals Fields

| Field | Type | Description |
|-------|------|-------------|
| `totalVMs` | integer | Source VMware cluster의 총 VM 수 |
| `totalCPU` | integer | 모든 VM CPU allocation 합계(cores) |
| `totalMemory` | integer | 모든 VM memory allocation 합계(GB) |

이 field들은 OpenShift transformation 이전의 original VMware inventory를 나타냅니다.

---

## Examples

### Example 1: Full HA Cluster with SMT

**Scenario**: 200 VMs(800 CPU cores, 1600 GB memory)를 SMT enabled(128 threads)인 32-core / 64 GB worker node 기반 Full HA OpenShift cluster로 migration.

**Configuration:**
- Cluster ID: `domain-c16`
- Worker node: 32 CPU, 64 GB, 128 threads
- SMT effective CPU: 32 + ((128 - 32) × 0.5) = 80 cores
- CPU over-commit: 1:4
- Memory over-commit: 1:2
- Control plane: 3 nodes, 6 CPU / 16 GB each(default)
- Control plane schedulable: No

**Calculation:**

1. **VM Batching**:
   ```
   Target capacity: 80 × 0.8 = 64 CPU, 64 × 0.8 = 51.2 GB
   Batches: max(ceil(800/64), ceil(1600/51.2), ceil(200/200)) = max(13, 32, 1) = 32
   Resources per batch: 800/32 = 25 CPU, 1600/32 = 50 GB, 200/32 = 6-7 VMs
   After over-commit: requests = 6.25 CPU / 25 GB, limits = 25 CPU / 50 GB
   ```

2. **Scheduling**:
   - 32 batches are placed using **requests** against each worker's allocatable CPU and memory(after kubelet overhead). **This example uses 30 workers** so failover and totals below match; real assessment에서는 cluster-requirements를 호출하세요.

3. **Failover**:
   ```
   failoverNodes = max(2, ceil(30 × 0.10)) = max(2, 3) = 3
   ```

4. **Total**:
   ```
   totalNodes = 30 (workers) + 3 (failover) + 3 (control plane) = 36 nodes
   totalCPU = 33 × 32 = 1056 cores (30+3 workers) + 3 × 6 = 18 (CP) = 1074 cores
   totalMemory = 33 × 64 = 2112 GB (workers) + 3 × 16 = 48 GB (CP) = 2160 GB
   ```

**Result:**
- Total nodes: 36(30 workers + 3 failover + 3 control plane)
- Total CPU: 1074 cores
- Total memory: 2160 GB
- Resource consumption: 200 CPU requests, 800 GB requests / 800 CPU limits, 1600 GB limits

---

### Example 2: Single Node OpenShift

**Scenario**: 20 VMs(80 CPU cores, 160 GB memory)를 Single Node OpenShift에 배치하는 small edge deployment.

**Configuration:**
- Cluster ID: `domain-c8`
- `controlPlaneNodeCount`: 1, `controlPlaneSchedulable`: true
- Worker node(batching): 128 CPU, 256 GB, 256 threads
- Control plane(sizer의 single node): 128 CPU, 256 GB — worker와 **동일**해야 함(consistent result 필요)
- SMT effective CPU(batching only): 128 + ((256 - 128) × 0.5) = 192 cores
- CPU over-commit: 1:4
- Memory over-commit: 1:2

**Calculation:**

1. **VM batching**(`workerNode*`와 threads 기반 effective CPU 사용):
   ```
   Target capacity: 192 × 0.8 = 153.6 CPU, 256 × 0.8 = 204.8 GB
   Batches: max(ceil(80/153.6), ceil(160/204.8), ceil(20/200)) = max(1, 1, 1) = 1
   Resources per batch: 80 CPU, 160 GB, 20 VMs
   After over-commit: requests = 20 CPU / 80 GB, limits = 80 CPU / 160 GB
   ```

2. **Single-node placement**(sizer는 `controlPlaneCPU` / `controlPlaneMemory` — 128 physical cores를 사용하며 SMT의 192 effective CPU는 사용하지 않음):
   ```
   Control plane reserved: 3.5 CPU, 13.39 GB
   Kubelet reserved (128-core node): 0.39 CPU, 11.88 GB memory
   Allocatable: 128 - 3.5 - 0.39 = 124.11 CPU, 256 - 13.39 - 11.88 = 230.73 GB
   Workload requests fit: 20 CPU / 80 GB < 124.11 / 230.73 ✓
   ```

**Result:**
- Total nodes: 1
- Total CPU: 128 cores
- Total memory: 256 GB
- Resource consumption: 20 CPU requests, 80 GB requests / 80 CPU limits, 160 GB limits

---

## Tips

- **Choose the right path**: VMware assessment가 완료되어 real inventory 기반 sizing이 필요하면 [UI](#using-the-ui) 또는 **assessment-based sizing** API를 사용합니다. Full assessment 없이 hypothetical scenario, capacity planning, quick estimate가 필요하면 **standalone sizing**(API only)을 사용합니다.
- **Start with defaults**: 특별한 workload profile이 없다면 1:4 CPU 및 1:2 memory over-commit ratio로 시작합니다. 대부분의 mixed workload에서 density와 performance의 균형을 제공합니다.
- **Enable SMT when available**: Hyperthreading은 일반적으로 20-50%의 추가 throughput을 제공합니다. Hardware가 SMT를 지원하면 정확한 sizing을 위해 항상 `workerNodeThreads`를 지정하세요. SNO에서는 SMT가 batching에만 영향을 주며, sizer는 `controlPlaneCPU`(physical cores)를 사용해 node를 모델링합니다.
- **Match worker and control plane sizes for SNO**: **Worker node**와 **Control plane** field에 동일한 CPU와 memory를 사용하세요. `workerNode*`는 batching을 결정하고 `controlPlane*`는 workload가 single node에 들어가는지를 결정합니다.
- **Plan for failover**: 자동 10% failover capacity(minimum 2 nodes)는 single worker node loss 시 resource exhaustion을 방지합니다. Mission-critical workload라면 recommendation보다 추가 node를 고려하세요.
- **Validate node sizes against inventory**: [UI](#using-the-ui)가 계획한 hardware보다 큰 minimum node size를 제안한다면 cluster가 100-node architectural limit 안에 들어가지 못한다는 의미입니다. 더 큰 node를 사용하거나 migration을 여러 OpenShift cluster로 나누세요.
- **Don't schedule workloads on control plane in production**: `controlPlaneSchedulable=true`는 Full HA에서 지원되지만 production에서는 권장되지 않습니다. Reliability를 위해 control plane node는 Kubernetes control service 전용으로 유지하세요.
- **Use Hosted Control Plane for multi-tenant platforms**: HCP mode는 control plane을 외부화하여 cluster당 overhead를 줄이므로 많은 small tenant cluster를 hosting하는 데 적합합니다.
- **Compare with actual migrations**: Sizing calculation은 static inventory 기반 estimate입니다. 첫 migration 후 실제 resource consumption을 recommendation과 비교하여 향후 over-commit ratio를 조정하세요.
- **Consider growth**: Recommendation은 current inventory를 기준으로 sizing됩니다. Migration 후 workload 추가가 계획되어 있으면 node count를 수동으로 늘리거나 planned VM을 inventory에 추가한 뒤 sizing을 다시 실행하세요.
- **Storage is separate**: 이 기능은 compute resources(CPU, memory, nodes)를 sizing합니다. Storage는 VMware inventory의 disk provisioning requirement를 기준으로 별도 계획하세요(total provisioned disk는 Aggregated Data Report 참고).
- **Retrieve stored inputs for consistency**: 동일 cluster의 sizing calculation을 다시 실행하거나 deployment mode별 결과를 비교할 때 consistency를 위해 stored input endpoint를 사용하세요.
