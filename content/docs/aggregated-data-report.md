---
title: "What Happens to Your Data? Understanding Our Aggregated Reporting Process"
linkTitle: "Aggregated Data Report"
description: "Migration Advisor가 수집하는 inventory data와 각 field가 migration planning에서 의미하는 바를 설명합니다."
---

Migration Advisor tool은 OpenShift Virtualization으로의 migration planning을 돕기 위해 VMware 환경에 대한 데이터를 수집합니다. 이 문서는 aggregated inventory report에 포함되는 모든 data field를 설명합니다.

Applies to: V0.4.0  
Last update (dd-mm-yyyy): 02-02-2026

이 문서에서 설명하는 data field와 structure는 release마다 달라질 수 있습니다. Migration Advisor tool version에 따라 새 field가 추가되거나 deprecated field가 제거될 수 있습니다.

## Report Structure Overview

Inventory report는 계층 구조로 구성됩니다.

```
Inventory
├── vcenter_id          # vCenter의 unique identifier
├── vcenter             # vCenter-level aggregated data
└── clusters            # Cluster name을 key로 하는 inventory data map
    └── [cluster_name]
        ├── vms         # Virtual machine statistics
        └── infra       # Infrastructure data
```

---

## Virtual Machine Data

`vms` section에는 환경에서 발견된 모든 VM에 대한 statistics가 포함됩니다.

### VM Counts

| Field | Description |
|-------|-------------|
| `total` | 발견된 총 VM 수 |
| `totalMigratable` | 문제 없이 migration 가능한 VM 수 |
| `totalMigratableWithWarnings` | migration은 가능하지만 해결해야 할 warning이 있는 VM 수 |

### Resource Breakdown Fields

주요 resource(CPU, RAM, Disk)별로 report는 detailed breakdown을 제공합니다.

#### CPU Cores

| Field | Description |
|-------|-------------|
| `total` | 모든 VM에 할당된 total vCPU cores |
| `totalForMigratable` | Migration 가능한 VM의 vCPU cores |
| `totalForMigratableWithWarnings` | Warning이 있지만 migration 가능한 VM의 vCPU cores |
| `totalForNotMigratable` | Migration 불가능한 VM의 vCPU cores |

#### RAM

| Field | Description |
|-------|-------------|
| `total` | 모든 VM에 할당된 total RAM(GB) |
| `totalForMigratable` | Migration 가능한 VM의 RAM(GB) |
| `totalForMigratableWithWarnings` | Warning이 있지만 migration 가능한 VM의 RAM(GB) |
| `totalForNotMigratable` | Migration 불가능한 VM의 RAM(GB) |

#### Disk Storage

| Field | Description |
|-------|-------------|
| `total` | 모든 VM의 total disk storage(GB) |
| `totalForMigratable` | Migration 가능한 VM의 disk storage(GB) |
| `totalForMigratableWithWarnings` | Warning이 있지만 migration 가능한 VM의 disk storage(GB) |
| `totalForNotMigratable` | Migration 불가능한 VM의 disk storage(GB) |

#### Disk Count

| Field | Description |
|-------|-------------|
| `total` | 모든 VM의 total virtual disk 수 |
| `totalForMigratable` | Migration 가능한 VM의 disk count |
| `totalForMigratableWithWarnings` | Warning이 있지만 migration 가능한 VM의 disk count |
| `totalForNotMigratable` | Migration 불가능한 VM의 disk count |

### Distribution Metrics

이 field들은 VM이 resource tier별로 어떻게 분포되어 있는지 이해하는 데 도움을 줍니다.

#### CPU Tier Distribution

CPU tier bucket별 VM 분포:
- `0-4` vCPUs
- `5-8` vCPUs
- `9-16` vCPUs
- `17-32` vCPUs
- `32+` vCPUs

#### Memory Tier Distribution

Memory tier bucket별 VM 분포:
- `0-4` GB
- `5-16` GB
- `17-32` GB
- `33-64` GB
- `65-128` GB
- `129-256` GB
- `256+` GB

#### NIC Count Distribution

Network interface count별 VM 분포:
- `0` NICs
- `1` NIC
- `2` NICs
- `3` NICs
- `4+` NICs

### Disk Information

#### Disk Size Tiers

각 disk size tier에 대해 report는 다음을 제공합니다.

| Field | Description |
|-------|-------------|
| `totalSizeTB` | 해당 tier의 total disk size(TB) |
| `vmCount` | 해당 tier에 속한 VM 수 |

#### Disk Types

각 disk type(thin, thick, RDM 등)에 대해 report는 다음을 제공합니다.

| Field | Description |
|-------|-------------|
| `vmCount` | 해당 type의 disk를 하나 이상 가진 VM 수 |
| `totalSizeTB` | 해당 disk type의 total disk size(TB) |

### Power States

VM power state별 breakdown입니다.

### Operating System Information

감지된 operating system별로 report는 다음을 제공합니다.

| Field | Description |
|-------|-------------|
| `count` | 해당 OS를 실행 중인 VM 수 |
| `supported` | 해당 OS가 MTV migration에 지원되는지 여부 |
| `upgradeRecommendation` | 지원되지 않는 OS 중 supported version으로 upgrade 가능한 경우 권장 upgrade path |

> **Note:** "supported" / "unsupported" OS classification은 MTV(Migration Toolkit for Virtualization)가 VM을 OpenShift Virtualization용 KVM으로 migration할 때 내부적으로 사용하는 conversion tool인 [virt-v2v](https://access.redhat.com/articles/1351473)를 기준으로 합니다. "supported" OS는 virt-v2v conversion이 검증되어 공식 지원된다는 의미입니다. "unsupported" OS가 반드시 migration 실패를 의미하는 것은 아니며, Red Hat에서 검증하지 않았고 문제가 발생했을 때 Red Hat expert의 official support를 받을 수 없다는 의미입니다. 많은 경우 unsupported OS conversion도 정상 동작할 수 있습니다. Supported guest operating systems 전체 목록은 [Converting virtual machines from other hypervisors to KVM with virt-v2v](https://access.redhat.com/articles/1351473)를 참고하세요.

### Migration Issues

#### Not Migratable Reasons

VM migration을 막는 issue list입니다.

| Field | Description |
|-------|-------------|
| `id` | Issue type의 unique identifier |
| `label` | 사람이 읽을 수 있는 issue description |
| `assessment` | Assessment category(예: `"error"`) |
| `count` | 이 issue의 영향을 받는 VM 수 |

**Common reasons include:**
- Unsupported disk types(RDM, shared VMDK)
- Unsupported guest operating systems
- Snapshots가 있는 VM
- USB device가 attached된 VM
- NVRAM encryption이 있는 VM

#### Migration Warnings

Migration을 block하지는 않지만 조치가 필요한 issue list입니다.

| Field | Description |
|-------|-------------|
| `id` | Warning type의 unique identifier |
| `label` | 사람이 읽을 수 있는 warning description |
| `assessment` | Assessment category(예: `"warning"`) |
| `count` | 이 warning의 영향을 받는 VM 수 |

**Common warnings include:**
- Changed Block Tracking(CBT) 미활성화
- VMware Tools 미설치 또는 outdated
- NIC가 4개를 초과하는 VM

---

## Infrastructure Data

`infra` section에는 VM을 지원하는 physical infrastructure 정보가 포함됩니다.

### Summary Counts

| Field | Description |
|-------|-------------|
| `totalHosts` | Total ESXi hosts |
| `totalDatacenters` | vSphere datacenter 수 |
| `clustersPerDatacenter` | Datacenter별 cluster 수 |

### Overcommitment Ratios

| Field | Description |
|-------|-------------|
| `cpuOverCommitment` | CPU overcommitment ratio(allocated vCPUs ÷ physical cores) |
| `memoryOverCommitment` | Memory overcommitment ratio(allocated memory ÷ physical memory) |

이 ratio는 현재 infrastructure가 얼마나 heavily utilized되는지 이해하고 OpenShift cluster sizing을 계획하는 데 도움을 줍니다.

### Host Information

각 ESXi host에 대해:

| Field | Description |
|-------|-------------|
| `id` | Host의 unique identifier |
| `vendor` | Hardware vendor(예: Dell, HPE, Cisco) |
| `model` | Server model |
| `cpuCores` | Physical CPU cores 수 |
| `cpuSockets` | CPU socket 수 |
| `memoryMB` | Total host memory(MB) |

### Host Power States

Host power state별 breakdown:
- `poweredOn`
- `standby`
- `maintenance`

### Network Information

환경 내 각 network에 대해:

| Field | Description |
|-------|-------------|
| `name` | Network name |
| `type` | Network type: `standard`, `distributed`, `dvswitch`, or `unsupported` |
| `vlanId` | VLAN identifier |
| `dvswitch` | Distributed vSwitch name(if applicable) |
| `vmsCount` | 이 network에 연결된 VM 수 |

### Datastore Information

각 datastore에 대해:

| Field | Description |
|-------|-------------|
| `type` | Storage type(VMFS, NFS, vSAN 등) |
| `totalCapacityGB` | Total capacity(GB) |
| `freeCapacityGB` | Available free space(GB) |
| `vendor` | Storage vendor |
| `model` | Storage model |
| `diskId` | Disk/LUN identifier |
| `protocolType` | Storage protocol(FC, iSCSI, NFS 등) |
| `hardwareAcceleratedMove` | VAAI/XCOPY 지원 여부 |
| `hostId` | 이 datastore가 attached된 host의 identifier |

---

## Data Report Generation

Data report는 두 가지 방법으로 생성할 수 있습니다.

### 1. Discovery Agent (OVA installation in the VSphare environment)

Discovery Agent를 사용하는 경우:
- Data는 vCenter API에서 직접 수집됩니다.
- 가장 정확하고 complete한 정보를 제공합니다.
- 환경이 변경되면 자동으로 update됩니다.
- vCenter에 OVA appliance를 배포해야 합니다.

### 2. RVTools Upload

RVTools export를 업로드하는 경우:
- Data는 Excel file에서 parsing됩니다.
- Agent 배포 없이 quick assessment가 가능합니다.
- Data accuracy는 RVTools export가 생성된 시점에 따라 달라집니다.
- RVTools configuration에 따라 일부 field를 사용할 수 없을 수 있습니다.

---
