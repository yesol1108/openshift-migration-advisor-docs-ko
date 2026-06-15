# 튜토리얼: Migration Assessment 사용 방법

이 tutorial은 VMware 환경을 OpenShift Virtualization으로 이전하기 전에 assessment를 수행하는 두 가지 방법을 설명합니다.

평가 방법은 다음 중 하나를 선택할 수 있습니다.

- **RVTools Flow**: 기존 RVTools Excel export를 업로드하여 즉시 assessment 수행
- **Discovery Agent Flow**: vCenter에 OVA를 배포하여 live discovery 수행

---

## RVTools Flow

기존 RVTools export 파일을 업로드하여 즉시 assessment를 수행합니다.

### Prerequisites: RVTools File Requirements

RVTools export를 업로드하기 전에 Excel 파일이 아래 요구사항을 충족하는지 확인합니다. Migration Assessment tool은 RVTools export를 검증하고, 포함된 데이터에 따라 error 또는 warning을 표시합니다.

#### Hard Requirements, Errors

아래 항목이 누락되면 upload가 실패합니다. 모든 항목은 `vInfo` sheet에 있어야 합니다.

| Requirement | Why |
|---|---|
| `vInfo` sheet에 최소 1개 row가 있어야 함 | VM이 없으면 계획할 inventory가 없음 |
| `VM ID` column이 null/empty가 아니어야 함 | VM의 primary unique identifier로 사용 |
| `VM` 또는 `Name` column이 null/empty가 아니어야 함 | VM 식별 및 화면 표시용 |
| `CLUSTER` column이 null/empty가 아니어야 함 | VM을 cluster 기준으로 grouping |
| `VI SDK UUID` column이 null/empty가 아니어야 함 | vCenter 식별에 필요 |

#### Soft Requirements, Warnings

아래 sheet는 확인 대상이지만, 비어 있거나 누락되어도 tool은 warning만 표시합니다. Inventory는 계속 생성되지만, 특정 data point가 비어 있을 수 있습니다.

| Sheet | Impact if Missing |
|---|---|
| `vHost` | Host 관련 정보 사용 불가 |
| `vDatastore` | Storage/datastore 정보 누락 |
| `vNetwork` | Network configuration detail 누락 |
| `vCPU` | core count 등 상세 CPU metric 누락 |
| `vMemory` | RAM allocation detail 누락 |
| `vDisk` | Disk size 및 provisioning data 누락 |
| `vNic` | Network interface detail, IP/MAC 정보 누락 |

가장 포괄적인 migration assessment를 위해 RVTools data를 export할 때 모든 sheet를 활성화하는 것이 좋습니다.

---

## Discovery Agent Flow

Discovery Agent를 vCenter에 배포하여 live environment analysis를 수행합니다.

## Collect Infrastructure Data

Agent는 VM, host, datastore, network, cluster를 포함한 VMware 환경의 익명 데이터를 수집합니다.

## Generate Assessment Reports

워크로드에 대한 migration readiness, compatibility analysis, recommendation을 포함한 상세 리포트를 제공합니다.

## Open Source

OpenShift Migration Advisor는 open source 프로젝트입니다.
