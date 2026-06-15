---
title: "Cost Estimation For Partners"
linkTitle: "Cost Estimation For Partners"
description: "VMware에서 OpenShift Virtualization으로의 migration cost estimation을 계산합니다."
weight: 5
---

# Cost Estimation For Partners

Cost Estimation 기능은 VMware 환경을 OpenShift Virtualization으로 migration할 때 필요한 cost analysis를 제공합니다. VMware VCF, VMware VVF, OpenShift Virtualization scenario의 3-year cost projection을 비교하여 partner가 migration decision을 안내할 수 있도록 지원합니다.

{{% alert title="Partner-Only Feature" color="info" %}}
Cost estimation은 Red Hat partner에게만 제공됩니다. 이 기능에 접근하려면 partner authentication이 필요합니다.
{{% /alert %}}

Applies to: V0.11.0  
Last update (dd-mm-yyyy): 12-05-2026

---

## How It Works

Cost estimation service는 VMware cluster inventory를 분석하여 3-year cost projection을 계산합니다. Assessment에서 environment data를 추출하고 proprietary cost model을 적용하여 comparative cost breakdown을 생성합니다.

### High-Level Flow

```
1. Partner가 API에 authenticate
2. Cost analysis 대상 assessment와 cluster 선택
3. Optional: vendor별 discount percentage 적용
4. Service가 cluster inventory에서 customer environment data 추출:
   - Total ESXi hosts
   - CPU sockets per host
   - CPU cores per socket
   - Total virtual machines
5. External cost estimation service가 세 가지 scenario 비용 계산:
   - VMware VCF (VMware Cloud Foundation)
   - VMware VVF (VMware vSphere Foundation)
   - OpenShift Virtualization
6. Response에 cost breakdown과 savings analysis 포함
```

---

## Prerequisites

### Partner Access

- Valid Red Hat partner credentials
- Partner authentication token
- Target assessment에 대한 read permission

### Assessment Requirements

- Assessment에는 최소 하나의 snapshot이 있어야 합니다.
- Snapshot에는 inventory data가 포함되어야 합니다.
- Inventory에 지정한 cluster ID를 가진 cluster가 존재해야 합니다.

---

## API Workflow

### Step 1: Authenticate as Partner

Valid partner credentials와 authentication token이 있는지 확인합니다. Cost estimation endpoint는 request 처리 전에 partner status를 검증합니다.

### Step 2: Calculate Cost Estimation

Assessment 내 특정 cluster에 대해 cost estimation calculation을 요청합니다.

```bash
curl -X POST http://planner:3443/api/v1/assessments/{assessment-id}/cost-estimation   -H "Content-Type: application/json"   -H "Authorization: Bearer <partner-token>"   -d '{
    "clusterId": "cluster-1",
    "discounts": {
      "vcfDiscountPct": 0,
      "vvfDiscountPct": 0,
      "redhatDiscountPct": 0,
      "aapDiscountPct": 0
    }
  }'
```

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `clusterId` | Yes | - | Assessment inventory의 VMware cluster ID |
| `discounts.vcfDiscountPct` | No | 0 | VMware VCF discount percentage(0-100) |
| `discounts.vvfDiscountPct` | No | 0 | VMware VVF discount percentage(0-100) |
| `discounts.redhatDiscountPct` | No | 0 | Red Hat discount percentage(0-100) |
| `discounts.aapDiscountPct` | No | 0 | Ansible Automation Platform discount percentage(0-100) |

### Response Codes

| Code | Meaning |
|------|---------|
| **200** | Cost estimation calculation successful |
| **400** | Bad request(missing clusterId, invalid discounts) |
| **401** | Unauthorized(invalid or missing authentication) |
| **403** | Forbidden(requires partner access) |
| **404** | Assessment or cluster not found |
| **500** | Internal error |
| **503** | Cost estimation service unavailable |

---

## Understanding Results

### Response Structure

Cost estimation response에는 다음이 포함됩니다.

```json
{
  "calculatorVersion": "1.0.0",
  "results": {
    "vmwareVcf": { ... },
    "vmwareVvf": { ... },
    "openshiftVirtualization": { ... }
  },
  "savings": {
    "vsVcf": { ... },
    "vsVvf": { ... }
  }
}
```

### Calculator Version

`calculatorVersion` field는 사용된 cost calculation model version을 나타냅니다. 이를 통해 estimate methodology의 변화를 추적할 수 있습니다.

### Cost Scenarios

각 scenario는 다음을 제공합니다.

| Field | Description |
|-------|-------------|
| `totalThreeYearCostEstimation` | 3년간 projected total cost(USD) |
| `breakdown` | Detailed cost components(아래 참고) |

### Cost Breakdown Components

각 scenario는 비용을 여섯 가지 category로 분해합니다.

| Component | Description |
|-----------|-------------|
| `softwareSubscriptions` | Core platform subscription costs |
| `ansibleAutomationPlatform` | AAP licensing and subscription |
| `migrationConsultingServices` | Migration을 위한 professional services |
| `swingHardwareUpgrades` | Migration swing을 위한 temporary hardware |
| `additionalStorageCosts` | Storage infrastructure costs |
| `thirdPartyIsvCosts` | Third-party vendor software costs |

{{% alert title="Note" color="warning" %}}
이 cost component를 계산하는 구체적인 algorithm과 multiplier는 proprietary and confidential입니다. Cost estimate는 Red Hat internal pricing model과 industry analysis를 기반으로 합니다.
{{% /alert %}}

### Savings Analysis

OpenShift Virtualization이 VMware option보다 cost-effective한 경우 `savings` field는 다음을 제공합니다.

| Savings Field | Description |
|---------------|-------------|
| `vsVcf.absoluteThreeYearUsd` | VMware VCF 대비 3년간 절감액(USD) |
| `vsVcf.percentage` | VMware VCF cost 대비 절감률 |
| `vsVvf.absoluteThreeYearUsd` | VMware VVF 대비 3년간 절감액(USD) |
| `vsVvf.percentage` | VMware VVF cost 대비 절감률 |

---

## Customer Environment Data Extraction

Service는 cost calculation에 반영하기 위해 cluster inventory에서 다음 데이터를 자동 추출합니다.

| Data Point | Source | Notes |
|------------|--------|-------|
| Total ESXi Hosts | Cluster infrastructure inventory | Inventory의 total host count |
| Sockets per Host | First host CPU configuration from inventory | `cpuSockets` field에서 추출, 없으면 2로 default |
| Cores per Socket | First host CPU data에서 계산 | `cpuCores / cpuSockets`로 계산, unavailable 또는 invalid이면 16으로 default |
| Total Virtual Machines | Cluster VM count from inventory | Cluster 내 total VM count |

Service는 inventory의 first host에서 실제 CPU configuration을 추출하려고 시도합니다. CPU data가 없거나 invalid이면 industry-standard default(2 sockets per host, 16 cores per socket)로 fallback합니다.

---

## Cost Estimation Methodology

Cost estimation 기능은 Red Hat이 개발한 proprietary calculation model을 사용하여 3-year cost를 project합니다. Methodology는 다음을 고려합니다.

- Customer environment sizing(hosts, sockets, cores, VMs)
- Vendor subscription models and pricing tiers
- Migration consulting service requirements
- Migration을 위한 infrastructure upgrade needs
- Third-party software dependencies
- User가 적용한 discount percentages

{{% alert title="Classified Information" color="warning" %}}
구체적인 pricing formulas, cost multipliers, algorithmic details는 classified information이며 public documentation에 공개할 수 없습니다. Cost estimates는 Red Hat confidential pricing intelligence와 market analysis를 기반으로 합니다.
{{% /alert %}}

---

## Error Handling

### Common Errors

| Error | Cause | Resolution |
|-------|-------|------------|
| **403 Forbidden** | Non-partner user가 접근 시도 | Partner credentials와 authentication 확인 |
| **404 Not Found** | Assessment 또는 cluster가 존재하지 않음 | Inventory의 assessment ID와 cluster ID 확인 |
| **400 Bad Request - clusterId required** | Request에 clusterId 누락 | Request body에 clusterId 포함 |
| **400 Bad Request - no snapshots** | Assessment에 inventory data 없음 | Valid inventory가 포함된 assessment 생성 |
| **400 Bad Request - empty inventory** | Latest snapshot에 inventory 없음 | Inventory collection 재실행 |
| **503 Service Unavailable** | Cost estimation service 중단 | System administrator에게 문의 |

---

## Tips

- **Use Latest Inventory**: Cost calculation은 latest snapshot을 기준으로 합니다. Estimate 요청 전 assessment inventory가 최신인지 확인하세요.
- **Accurate Discount Data**: 현실적인 cost comparison을 위해 정확한 discount percentage를 적용하세요. Current discount rate는 vendor account team과 확인하세요.
- **Multiple Clusters**: 여러 VMware cluster가 서로 다른 configuration을 가진 경우 cluster별 cost estimate를 생성하세요.
- **Partner Context**: Cost estimate는 partner-led customer conversation을 위해 설계되었습니다. 결과는 final pricing이 아니라 discussion point로 사용하세요.
- **Version Tracking**: 결과의 `calculatorVersion`을 기록하세요. 서로 다른 calculator version의 estimate는 직접 비교가 어려울 수 있습니다.
