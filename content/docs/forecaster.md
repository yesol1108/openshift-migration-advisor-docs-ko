---
title: "Migration Forecaster"
linkTitle: "Migration Forecaster"
description: "VMware datastore pair 간 storage throughput을 benchmark하여 migration time을 예측합니다."
weight: 4
---

# Migration Forecaster (마이그레이션 소요 시간 예측기)

Migration Forecaster는 vCenter 인프라에서 직접 disk-copy benchmark를 실행하여 VMware datastore pair 간 실제 storage throughput을 측정합니다. 이 결과를 통해 이론적 계산이 아니라 실제 환경에 기반한 data-driven migration time estimate를 제공합니다.

Applies to: V0.11.0  
Last update (dd-mm-yyyy): 03-05-2026

---

## 동작 방식

Forecaster는 source datastore와 target datastore 간 disk copy 속도를 benchmark하기 위해 vCenter 환경에 **temporary resources**를 생성합니다. copy를 여러 번 반복하고 mean, median, confidence interval 등 통계를 계산하여 신뢰할 수 있는 throughput estimate를 산출합니다.

### High-Level Flow (전체 흐름)

```
1. vCenter credentials와 privileges 검증
2. Inventory에서 사용 가능한 datastore 검색
3. Benchmark할 datastore pair 선택
4. 각 pair에 대해:
   a. Source datastore에 temporary disk 생성
   b. Temporary Alpine Linux VM을 사용해 disk를 random data로 채움
   c. Source에서 target으로 disk copy 수행(N iterations)
   d. 각 copy의 wall-clock time 측정
   e. 모든 temporary resource 정리
5. Throughput statistics와 time estimates 계산
```

---

## 중요: vCenter에 생성되는 리소스

{{% alert title="Warning" color="warning" %}}
Forecaster는 vCenter 환경에 temporary virtual machines와 virtual disks를 생성합니다. Benchmark 이후 모든 resource는 자동으로 정리되지만, vCenter 관리자는 이 작업이 수행된다는 점을 알고 있어야 합니다.
{{% /alert %}}

### 데이터스토어 쌍별 생성 리소스

| Resource | Location | Purpose | Lifetime |
|----------|----------|---------|----------|
| **Filler VM** | vCenter inventory | Benchmark disk를 random data로 채우기 위해 Alpine Linux 부팅 | 수 분 수준, disk fill 완료 후 삭제 |
| **Alpine boot disk** | Source datastore | Filler VM OS용 256 MB thin VMDK | Pair 완료 후 삭제 |
| **Seed ISO** | Source datastore | Filler VM용 cloud-init configuration | Pair 완료 후 삭제 |
| **Benchmark disk** | Source datastore | Random data로 채워진 thin VMDK(user-specified size, default 10 GB) | Pair 완료 후 삭제 |
| **Clone disk** | Target datastore | 각 iteration에서 생성되는 benchmark disk copy | 각 iteration 후 삭제 |
| **Directories** | Source and target datastores | temporary file 저장용 `forecaster-*` directories | Pair 완료 후 삭제 |

### Filler VM 사양

Benchmark disk를 채우는 데 사용되는 temporary VM입니다.

- **Name**: `forecaster-filler-{timestamp}`
- **OS**: Alpine Linux (minimal, 약 256 MB)
- **Resources**: 1 vCPU, 256 MB RAM
- **Behavior**: 부팅 후 `dd`로 benchmark disk를 random data로 채우고 자동으로 power off
- **No network access**: VM은 network connectivity를 필요로 하거나 사용하지 않습니다.

### 정리 보장 사항

- Benchmark pair가 완료되면(success or failure) 모든 resource가 자동으로 정리됩니다.
- Agent가 benchmark 도중 강제 종료되면 일부 temporary file 또는 directory가 datastore에 남을 수 있으며 수동 정리가 필요할 수 있습니다.
- Temporary resource name은 쉽게 식별할 수 있도록 `forecaster-` 또는 `filler-image-` prefix를 사용합니다.
- Benchmark 중 error가 발생하더라도 filler VM은 항상 삭제됩니다.

---

## 필요한 vSphere 권한

Forecasting에 사용하는 vCenter credential에는 다음 privilege가 필요합니다.

| Privilege | Why |
|-----------|-----|
| `Datastore.AllocateSpace` | Benchmark disk와 directory 생성 |
| `Datastore.Browse` | Datastore content 조회 |
| `Datastore.DeleteFile` | Temporary file과 directory 정리 |
| `Datastore.FileManagement` | Filler image와 seed ISO upload |
| `VirtualMachine.Inventory.Create` | Temporary filler VM 생성 |
| `VirtualMachine.Inventory.Delete` | 사용 후 filler VM 삭제 |
| `VirtualMachine.Provisioning.Clone` | Datastore 간 disk copy |
| `VirtualMachine.Interact.PowerOn` | Filler VM 부팅 |
| `VirtualMachine.Config.AddRemoveDevice` | Filler VM에 disk attach/detach |

Credential verification endpoint(`PUT /forecaster/credentials`)는 benchmark 시작 전 이러한 privilege를 확인하고 누락된 항목을 보고합니다.

---

## API 워크플로우

### Step 1: Credentials 확인

vCenter credentials가 필요한 권한을 가지고 있는지 검증합니다.

```bash
curl -X PUT http://agent:3443/api/v1/forecaster/credentials   -H "Content-Type: application/json"   -d '{
    "url": "https://vcenter.example.com",
    "username": "administrator@vsphere.local",
    "password": "your-password"
  }'
```

**200 OK** — credentials가 유효합니다. **403** — privilege가 부족합니다(response에 `missingPrivileges` list 포함).

### Step 2: Datastores 목록 조회

Environment inventory에서 검색된 datastore 목록을 조회합니다.

```bash
curl -X POST http://agent:3443/api/v1/forecaster/datastores
```

Datastore type, capacity, vendor, offload capability가 포함된 목록을 반환합니다.

### Step 3: Check Pair Capabilities (Optional)

특정 datastore pair에서 사용 가능한 storage offload capability를 확인합니다.

```bash
curl -X POST http://agent:3443/api/v1/forecaster/capabilities   -H "Content-Type: application/json"   -d '{
    "pairs": [
      {
        "name": "local-to-nfs",
        "sourceDatastore": "datastore1",
        "targetDatastore": "datastore2"
      }
    ]
  }'
```

### Step 4: Benchmark 시작

하나 이상의 datastore pair에 대한 benchmark를 시작합니다.

```bash
curl -X POST http://agent:3443/api/v1/forecaster   -H "Content-Type: application/json"   -d '{
    "pairs": [
      {
        "name": "local-to-nfs",
        "sourceDatastore": "datastore1",
        "targetDatastore": "datastore2",
        "host": "esxi-01.example.com"
      }
    ],
    "diskSizeGb": 10,
    "iterations": 5,
    "concurrency": 1
  }'
```

| Parameter | Default | Description |
|-----------|---------|-------------|
| `pairs` | *(required)* | Benchmark할 datastore pair list |
| `pairs[].host` | *(auto-selected)* | 특정 ESXi host에 고정(optional) |
| `diskSizeGb` | 10 | Benchmark disk size(GB) |
| `iterations` | 5 | Pair별 copy iteration 수 |
| `concurrency` | 1 | 병렬로 benchmark할 pair 수 |

초기 status와 함께 **202 Accepted**를 반환합니다. Benchmark가 이미 실행 중이면 **409 Conflict**를 반환합니다.

### Step 5: Poll Status

Benchmark 진행 상황을 모니터링합니다.

```bash
curl http://agent:3443/api/v1/forecaster
```

각 pair는 현재 state를 보고합니다.

| State | Meaning |
|-------|---------|
| `pending` | Queue 대기 중, 아직 시작 전 |
| `preparing` | Benchmark disk 생성 및 fill 중 |
| `running` | Disk copy 수행 중(iteration 진행) |
| `completed` | 모든 iteration 완료 |
| `error` | 실패(`error` field 확인) |
| `canceled` | 사용자가 중지 |

`preparing` 중에는 `prepBytesTotal`과 `prepBytesUploaded`가 disk fill progress를 추적합니다.  
`running` 중에는 `completedRuns`와 `totalRuns`가 iteration progress를 추적합니다.

### Step 6: 결과 조회

Benchmark run data를 조회합니다.

```bash
curl "http://agent:3443/api/v1/forecaster/runs?pairName=local-to-nfs"
```

계산된 statistics와 추측 예상 시간을 조회합니다.

```bash
curl "http://agent:3443/api/v1/forecaster/stats?pairName=local-to-nfs"
```

Stats response에는 mean/median/min/max throughput, standard deviation, 95% confidence interval, 1 TB당 estimated migration time(best case, expected, worst case)이 포함됩니다.

### Benchmark 취소하기

실행 중인 모든 pair를 취소합니다.

```bash
curl -X DELETE http://agent:3443/api/v1/forecaster
```

단일 pair를 취소합니다.

```bash
curl -X DELETE http://agent:3443/api/v1/forecaster/pairs/local-to-nfs
```

Canceling은 active benchmark를 중지합니다. Operation 완료 전에 resource가 정리됩니다.

---

## 결과 이해하기

### Throughput Statistics

Benchmark가 완료되면 stats endpoint에서 다음 정보를 제공합니다.

| Metric | Description |
|--------|-------------|
| `meanMbps` | 성공한 모든 iteration의 average throughput |
| `medianMbps` | 중앙값(outlier 영향을 덜 받음) |
| `minMbps` / `maxMbps` | 관측된 throughput 범위 |
| `stddevMbps` | Standard deviation(consistency measure) |
| `ci95LowerMbps` / `ci95UpperMbps` | 실제 throughput에 대한 95% confidence interval |

### 예상 시간

`estimatePer1TB` field는 1 TB data에 대한 마이그레이션 예측 시간을 제공합니다.
| Estimate | Based On |
|----------|----------|
| `bestCase` | 95% confidence interval upper bound |
| `expected` | Median throughput |
| `worstCase` | 95% confidence interval lower bound |

실제 data volume에 맞춰 선형으로 scale하면 됩니다.

---

## Tips

- **Disk size**: 큰 benchmark disk는 더 현실적인 결과를 제공하지만 준비 시간이 길어집니다. 10 GB는 빠른 estimate에 적합한 default입니다.
- **Iterations**: Iteration이 많을수록 statistical confidence가 향상됩니다. 5회는 합리적인 default이며, production planning에는 10회 이상을 권장합니다.
- **Host pinning**: Migration에 특정 ESXi host를 사용할 예정이라면, 더 정확한 결과를 위해 benchmark를 해당 host에 pin하세요.
- **Off-peak testing**: 대표적인 load condition에서 benchmark를 수행해야 현실적인 throughput 수치를 얻을 수 있습니다.
- **Multiple pairs**: 서로 다른 storage backend 간 migration이 있다면 각 unique source-target combination을 별도로 benchmark하세요.
