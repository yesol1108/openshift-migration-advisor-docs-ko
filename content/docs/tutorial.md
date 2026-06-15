---
title: "Tutorial"
linkTitle: "Tutorial"
weight: 2
---

# How to Use Migration Assessment

이 튜토리얼은 OpenShift Virtualization으로의 마이그레이션을 위해 VMware 환경을 assessment하는 두 가지 방법을 설명합니다.

**Assessment 방법을 선택하세요:**

- **[RVTools Flow](#rvtools-flow)** - 기존 RVTools Excel export를 업로드하여 즉시 assessment 수행
- **[Discovery Agent Flow](#discovery-agent-flow)** - vCenter에 OVA를 배포하여 live discovery 수행

---

## RVTools Flow

기존 RVTools export 파일을 업로드하여 즉시 assessment를 수행합니다. 아래 비디오 튜토리얼을 확인하세요.

<iframe width="700px" height="400px" src="https://interact.redhat.com/share/pkPLJELVr45LbdXg3InF" title="OpenShift Migration Advisor Using RVtools Flow" frameborder="0" referrerpolicy="unsafe-url" allowfullscreen="true" style="border-radius: 10px"></iframe>

### Prerequisites: RVTools File Requirements

RVTools export를 업로드하기 전에 Excel 파일이 아래 요구사항을 충족하는지 확인하세요. migration assessment 도구는 RVTools export를 검증하고, 포함된 데이터에 따라 error 또는 warning을 보고합니다.

#### Hard Requirements (Errors)

아래 항목이 누락되면 업로드가 **실패**합니다. 모든 항목은 **vInfo** 시트에 있습니다.

| Requirement | Why? |
|-------------|------|
| **vInfo** sheet must have at least 1 row | VM이 없으면 계획할 inventory가 없습니다. |
| Column **VM ID** must not be null/empty | VM의 primary unique identifier로 사용됩니다. |
| Column **VM** (Name) must not be null/empty | VM을 식별하고 표시하는 데 사용됩니다. |
| Column **CLUSTER** must not be null/empty | VM을 cluster별로 그룹화하는 데 사용됩니다. |
| Column **VI SDK UUID** must not be null/empty | vCenter를 식별하는 데 필요합니다. |

#### Soft Requirements (Warnings)

아래 시트는 확인 대상이지만, 비어 있거나 누락되어도 도구는 **warning**만 표시합니다. inventory는 계속 생성되지만, 일부 데이터 포인트가 비어 있을 수 있습니다.

| Sheet | Impact if Missing |
|-------|-------------------|
| **vHost** | Host 관련 정보를 사용할 수 없습니다. |
| **vDatastore** | Storage/datastore 정보가 누락됩니다. |
| **vNetwork** | Network 구성 세부 정보가 누락됩니다. |
| **vCPU** | Core count와 같은 상세 CPU metric이 누락됩니다. |
| **vMemory** | RAM allocation 세부 정보가 누락됩니다. |
| **vDisk** | Disk size와 provisioning data가 누락됩니다. |
| **vNic** | Network interface 세부 정보(IP/MAC)가 누락됩니다. |

가장 포괄적인 migration assessment를 위해 RVTools 데이터를 export할 때 **모든 시트가 포함되도록** 설정하세요.

[If the demo doesn't load, please use this link](https://interact.redhat.com/share/pkPLJELVr45LbdXg3InF)

---

## Discovery Agent Flow

vCenter에 discovery agent를 배포하여 live environment analysis를 수행합니다. 아래 비디오 튜토리얼을 확인하세요.

<iframe width="700px" height="400px" src="https://interact.redhat.com/share/kEinYphM8Zyt7wXLys53?mode=videoOnly" title="Configure OpenShift Migration Advisor Agent Flow in Red Hat" frameborder="0" referrerpolicy="unsafe-url" allowfullscreen="true" allow="clipboard-write" sandbox="allow-popups allow-popups-to-escape-sandbox allow-scripts allow-forms allow-same-origin allow-presentation" style="border-radius: 10px"></iframe>

[If the demo doesn't load, please use this link.](https://interact.redhat.com/share/kEinYphM8Zyt7wXLys53)
