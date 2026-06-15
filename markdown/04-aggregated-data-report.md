# Aggregated Data Report: 수집 데이터와 의미

이 문서는 Migration Advisor가 수집하는 inventory data와 각 field가 마이그레이션 계획에서 어떤 의미를 갖는지 설명합니다.

Migration Advisor tool은 VMware 환경 데이터를 수집하여 OpenShift Virtualization으로의 migration planning을 지원합니다. 이 문서는 aggregated inventory report에 포함되는 모든 data field를 설명합니다.

- 적용 버전: V0.4.0
- 마지막 업데이트: 02-02-2026

> 이 문서에 설명된 data field와 구조는 release마다 달라질 수 있습니다. Migration Advisor tool version에 따라 새로운 field가 추가되거나 deprecated field가 제거될 수 있습니다.

## Report Structure Overview

Inventory report는 계층 구조로 구성됩니다.

---

## Virtual Machine Data

`vms` section에는 환경에서 발견된 모든 VM에 대한 통계가 포함됩니다.

### VM Counts

| Field | Description |
|---|---|
| `total` | 발견된 전체 VM 수 |
| `totalMigratable` | issue 없이 migration 가능한 VM 수 |
| `totalMigratableWithWarnings` | migration 가능하지만 조치해야 할 warning이 있는 VM 수 |

### Resource Breakdown Fields

CPU, RAM, Disk 등 주요 resource별 상세 breakdown을 제공합니다.

#### CPU Cores

| Field | Description |
|---|---|
| `total` | 전체 VM에 할당된 총 vCPU core 수 |
| `totalForMigratable` | migration 가능한 VM의 vCPU core 수 |
| `totalForMigratableWithWarnings` | warning이 있는 migration 가능 VM의 vCPU core 수 |
| `totalForNotMigratable` | migration 불가 VM의 vCPU core 수 |

#### RAM

| Field | Description |
|---|---|
| `total` | 전체 VM에 할당된 총 RAM, GB |
| `totalForMigratable` | migration 가능한 VM의 RAM, GB |
| `totalForMigratableWithWarnings` | warning이 있는 migration 가능 VM의 RAM, GB |
| `totalForNotMigratable` | migration 불가 VM의 RAM, GB |

#### Disk Storage

| Field | Description |
|---|---|
| `total` | 전체 VM의 총 disk storage, GB |
| `totalForMigratable` | migration 가능한 VM의 disk storage, GB |
| `totalForMigratableWithWarnings` | warning이 있는 migration 가능 VM의 disk storage, GB |
| `totalForNotMigratable` | migration 불가 VM의 disk storage, GB |

#### Disk Count

| Field | Description |
|---|---|
| `total` | 전체 VM의 virtual disk 총 개수 |
| `totalForMigratable` | migration 가능한 VM의 disk count |
| `totalForMigratableWithWarnings` | warning이 있는 migration 가능 VM의 disk count |
| `totalForNotMigratable` | migration 불가 VM의 disk count |

### Distribution Metrics

VM이 서로 다른 resource tier에 어떻게 분포되어 있는지 파악할 수 있습니다.

#### CPU Tier Distribution

VM을 CPU tier bucket별로 분포시킵니다.

- `0-4` vCPU
- `5-8` vCPU
- `9-16` vCPU
- `17-32` vCPU
- `32+` vCPU

#### Memory Tier Distribution

VM을 memory tier bucket별로 분포시킵니다.

- `0-4` GB
- `5-16` GB
- `17-32` GB
- `33-64` GB
- `65-128` GB
- `129-256` GB
- `256+` GB

#### NIC Count Distribution

VM을 network interface 개수별로 분포시킵니다.

- `0` NIC
- `1` NIC
- `2` NICs
- `3` NICs
- `4+` NICs

### Disk Information

#### Disk Size Tiers

각 disk size tier에 대해 다음 정보를 제공합니다.

| Field | Description |
|---|---|
| `totalSizeTB` | 해당 tier의 총 disk size, TB |
| `vmCount` | 해당 tier에 속한 VM 수 |

#### Disk Types

각 disk type, 예: thin, thick, RDM에 대해 다음 정보를 제공합니다.

| Field | Description |
|---|---|
| `vmCount` | 해당 disk type을 하나 이상 가진 VM 수 |
| `totalSizeTB` | 해당 disk type의 총 size, TB |

### Power States

VM power state별 breakdown을 제공합니다.

### Operating System Information

감지된 operating system별로 다음 정보를 제공합니다.

| Field | Description |
|---|---|
| `count` | 해당 OS를 실행 중인 VM 수 |
| `supported` | OS가 MTV migration에 지원되는지 여부 |
| `upgradeRecommendation` | 지원되지 않는 OS를 지원 버전으로 올리기 위한 권장 upgrade path |

> `supported`/`unsupported` OS classification은 MTV가 VM을 KVM/OpenShift Virtualization으로 migration할 때 내부적으로 사용하는 conversion tool인 `virt-v2v` 기준입니다. `supported`로 표시된 OS는 `virt-v2v` conversion에 대해 검증되고 공식 지원된다는 의미입니다.
>
> `unsupported` OS가 반드시 migration 실패를 의미하지는 않습니다. Red Hat에서 검증하지 않았고 이슈 발생 시 공식 지원을 제공하지 않는다는 의미에 가깝습니다. 많은 경우 unsupported OS conversion이 동작할 수 있습니다.

### Migration Issues

#### Not Migratable Reasons

VM migration을 막는 issue 목록입니다.

| Field | Description |
|---|---|
| `id` | issue type의 unique identifier |
| `label` | 사람이 읽을 수 있는 issue 설명 |
| `assessment` | assessment category, 예: `error` |
| `count` | 해당 issue 영향을 받는 VM 수 |

일반적인 사유는 다음과 같습니다.

- 지원되지 않는 disk type, 예: RDM, shared VMDK
- 지원되지 않는 guest operating system
- snapshot이 있는 VM
- USB device가 attached된 VM
- NVRAM encryption이 적용된 VM

#### Migration Warnings

Migration을 차단하지는 않지만 사전에 조치하는 것이 좋은 issue 목록입니다.

| Field | Description |
|---|---|
| `id` | warning type의 unique identifier |
| `label` | 사람이 읽을 수 있는 warning 설명 |
| `assessment` | assessment category, 예: `warning` |
| `count` | 해당 warning 영향을 받는 VM 수 |

일반적인 warning은 다음과 같습니다.

- Changed Block Tracking, CBT 비활성화
- VMware Tools 미설치 또는 구버전
- NIC가 4개를 초과하는 VM

---

## Infrastructure Data

`infra` section에는 VM을 지원하는 물리 인프라 정보가 포함됩니다.

### Summary Counts

| Field | Description |
|---|---|
| `totalHosts` | 전체 ESXi host 수 |
| `totalDatacenters` | vSphere datacenter 수 |
| `clustersPerDatacenter` | datacenter별 cluster 수 |

### Overcommitment Ratios

| Field | Description |
|---|---|
| `cpuOverCommitment` | CPU overcommitment ratio, 할당 vCPU ÷ 물리 core |
| `memoryOverCommitment` | Memory overcommitment ratio, 할당 memory ÷ 물리 memory |

이 ratio는 현재 infrastructure가 얼마나 많이 사용되고 있는지 이해하고, 적절한 OpenShift cluster sizing을 계획하는 데 도움이 됩니다.

### Host Information

각 ESXi host에 대해 다음 정보를 제공합니다.

| Field | Description |
|---|---|
| `id` | host의 unique identifier |
| `vendor` | hardware vendor, 예: Dell, HPE, Cisco |
| `model` | server model |
| `cpuCores` | physical CPU core 수 |
| `cpuSockets` | CPU socket 수 |
| `memoryMB` | host total memory, MB |

### Host Power States

Host power state별 breakdown을 제공합니다.

- `poweredOn`
- `standby`
- `maintenance`

### Network Information

각 network에 대해 다음 정보를 제공합니다.

| Field | Description |
|---|---|
| `name` | network name |
| `type` | network type: `standard`, `distributed`, `dvswitch`, `unsupported` 등 |
| `vlanId` | VLAN identifier |
| `dvswitch` | distributed vSwitch name, 해당되는 경우 |
| `vmsCount` | 이 network에 연결된 VM 수 |

### Datastore Information

각 datastore에 대해 다음 정보를 제공합니다.

| Field | Description |
|---|---|
| `type` | storage type, 예: VMFS, NFS, vSAN 등 |
| `totalCapacityGB` | 총 capacity, GB |
| `freeCapacityGB` | 사용 가능한 free space, GB |
| `vendor` | storage vendor |
| `model` | storage model |
| `diskId` | disk/LUN identifier |
| `protocolType` | storage protocol, 예: FC, iSCSI, NFS 등 |
| `hardwareAcceleratedMove` | VAAI/XCOPY 지원 여부 |
| `hostId` | 이 datastore가 attached된 host identifier |

---

## Data Report Generation

Data report는 두 가지 방식으로 생성할 수 있습니다.

### 1. Discovery Agent, vSphere 환경에 OVA 설치

Discovery Agent를 사용하는 경우:

- vCenter API에서 직접 data를 수집합니다.
- 가장 정확하고 완전한 정보를 제공합니다.
- 환경 변경에 따라 자동으로 update됩니다.
- vCenter에 OVA appliance 배포가 필요합니다.

### 2. RVTools Upload

RVTools export를 업로드하는 경우:

- Excel file에서 data를 parsing합니다.
- Agent 배포 없이 빠른 assessment가 가능합니다.
- data accuracy는 RVTools export가 생성된 시점에 의존합니다.
- RVTools configuration에 따라 일부 field는 제공되지 않을 수 있습니다.

---

## Collect Infrastructure Data

Agent는 VM, host, datastore, network, cluster를 포함한 VMware 환경의 익명 데이터를 수집합니다.

## Generate Assessment Reports

워크로드의 migration readiness, compatibility analysis, recommendation에 대한 상세 리포트를 제공합니다.

## Open Source

OpenShift Migration Advisor는 open source 프로젝트입니다.
