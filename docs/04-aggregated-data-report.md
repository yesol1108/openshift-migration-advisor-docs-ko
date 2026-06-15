# Aggregated Data Report: 수집 데이터와 의미

이 문서는 Migration Advisor가 수집하는 inventory data와 각 field가 마이그레이션 계획에서 어떤 의미를 갖는지 설명합니다.

Migration Advisor는 VMware 환경 데이터를 수집하여 OpenShift Virtualization으로의 마이그레이션 계획을 지원합니다. Aggregated inventory report에 포함되는 주요 데이터 field는 아래와 같습니다.

- 적용 버전: V0.4.0
- 마지막 업데이트: 02-02-2026

> 문서에 설명된 데이터 field와 구조는 릴리스마다 달라질 수 있습니다. 새 field가 추가되거나 deprecated field가 제거될 수 있습니다.

## Report Structure Overview

Inventory report는 계층 구조로 구성됩니다.

## Virtual Machine Data

`vms` section에는 환경에서 발견된 모든 VM에 대한 통계가 포함됩니다.

### VM Counts

| Field | 설명 |
|---|---|
| `total` | 발견된 전체 VM 수 |
| `totalMigratable` | 문제 없이 migration 가능한 VM 수 |
| `totalMigratableWithWarnings` | migration 가능하지만 조치해야 할 warning이 있는 VM 수 |

### Resource Breakdown Fields

CPU, RAM, Disk 등 주요 resource별 상세 breakdown을 제공합니다.

#### CPU Cores

| Field | 설명 |
|---|---|
| `total` | 전체 VM에 할당된 총 vCPU core 수 |
| `totalForMigratable` | migration 가능한 VM의 vCPU core 수 |
| `totalForMigratableWithWarnings` | warning이 있는 migration 가능 VM의 vCPU core 수 |
| `totalForNotMigratable` | migration 불가 VM의 vCPU core 수 |

#### RAM

| Field | 설명 |
|---|---|
| `total` | 전체 VM에 할당된 총 RAM, GB |
| `totalForMigratable` | migration 가능한 VM의 RAM, GB |
| `totalForMigratableWithWarnings` | warning이 있는 migration 가능 VM의 RAM, GB |
| `totalForNotMigratable` | migration 불가 VM의 RAM, GB |

#### Disk Storage / Disk Count

Disk storage는 VM 전체 disk 용량을 GB 기준으로 집계하고, Disk count는 VM 전체 virtual disk 개수를 집계합니다. 각각 migration 가능, warning 포함 가능, migration 불가 상태별로 나뉩니다.

### Distribution Metrics

VM이 resource tier별로 어떻게 분포되어 있는지 확인할 수 있습니다.

- CPU tier: `0-4`, `5-8`, `9-16`, `17-32`, `32+` vCPU
- Memory tier: `0-4`, `5-16`, `17-32`, `33-64`, `65-128`, `129-256`, `256+` GB
- NIC count: `0`, `1`, `2`, `3`, `4+`

### Disk Information

Disk size tier별로 총 disk size TB와 VM count를 제공합니다. Disk type, 예: thin, thick, RDM별로도 VM count와 total size TB를 제공합니다.

### Power States

VM power state별 breakdown을 제공합니다.

### Operating System Information

각 detected OS별로 다음 정보를 제공합니다.

| Field | 설명 |
|---|---|
| `count` | 해당 OS를 실행 중인 VM 수 |
| `supported` | MTV migration 지원 여부 |
| `upgradeRecommendation` | 지원되지 않는 OS에 대한 권장 upgrade path |

`supported`/`unsupported` OS 구분은 MTV가 내부적으로 VM을 KVM/OpenShift Virtualization으로 변환할 때 사용하는 `virt-v2v` 기준입니다. `unsupported`라고 해서 반드시 migration이 실패한다는 의미는 아니며, Red Hat에서 검증 및 공식 지원하지 않는다는 의미에 가깝습니다.

### Migration Issues

#### Not Migratable Reasons

Migration을 차단하는 issue 목록입니다.

| Field | 설명 |
|---|---|
| `id` | issue type의 고유 식별자 |
| `label` | 사람이 읽을 수 있는 issue 설명 |
| `assessment` | 평가 category, 예: error |
| `count` | 해당 issue 영향을 받는 VM 수 |

일반적인 사유는 다음과 같습니다.

- 지원되지 않는 disk type: RDM, shared VMDK
- 지원되지 않는 guest OS
- snapshot이 있는 VM
- USB device가 연결된 VM
- NVRAM encryption이 적용된 VM

#### Migration Warnings

Migration을 막지는 않지만 사전에 조치하면 좋은 issue 목록입니다.

- CBT, Changed Block Tracking, 미활성화
- VMware Tools 미설치 또는 구버전
- NIC가 4개를 초과하는 VM

## Infrastructure Data

`infra` section에는 VM을 지원하는 물리 인프라 정보가 포함됩니다.

### Summary Counts

| Field | 설명 |
|---|---|
| `totalHosts` | 전체 ESXi host 수 |
| `totalDatacenters` | vSphere datacenter 수 |
| `clustersPerDatacenter` | datacenter별 cluster 수 |

### Overcommitment Ratios

| Field | 설명 |
|---|---|
| `cpuOverCommitment` | 할당 vCPU ÷ 물리 core |
| `memoryOverCommitment` | 할당 memory ÷ 물리 memory |

이 비율은 현재 인프라가 얼마나 많이 사용되고 있는지 파악하고 OpenShift cluster sizing을 계획하는 데 활용됩니다.

### Host Information

각 ESXi host별로 id, vendor, model, cpuCores, cpuSockets, memoryMB 정보를 제공합니다.

### Network Information

각 network별로 name, type, vlanId, dvswitch, vmsCount 정보를 제공합니다. Network type은 `standard`, `distributed`, `dvswitch`, `unsupported` 등이 될 수 있습니다.

### Datastore Information

각 datastore별로 type, totalCapacityGB, freeCapacityGB, vendor, model, diskId, protocolType, hardwareAcceleratedMove, hostId 정보를 제공합니다.

## Data Report Generation

### 1. Discovery Agent

Discovery Agent를 사용하는 경우:

- vCenter API에서 직접 데이터 수집
- 가장 정확하고 완전한 정보 제공
- 환경 변경 시 자동 update
- vCenter에 OVA appliance 배포 필요

### 2. RVTools Upload

RVTools export를 업로드하는 경우:

- Excel file에서 data parsing
- Agent 배포 없이 빠른 assessment 가능
- data accuracy는 RVTools export 시점에 의존
- RVTools 구성에 따라 일부 field는 제공되지 않을 수 있음
