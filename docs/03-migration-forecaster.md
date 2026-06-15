# Migration Forecaster

VMware datastore pair 간 storage throughput을 벤치마크하여 마이그레이션 시간을 예측합니다.

Migration Forecaster는 vCenter 인프라에서 직접 disk-copy benchmark를 실행하여 VMware datastore pair 간 실제 storage throughput을 측정합니다. 이 결과를 통해 이론값이 아니라 실제 환경 기반의 데이터 중심 마이그레이션 시간 예측을 제공합니다.

- 적용 버전: V0.11.0
- 마지막 업데이트: 03-05-2026

## 동작 방식

Forecaster는 vCenter 환경에 임시 리소스를 생성하여 source datastore와 target datastore 간 disk copy 속도를 벤치마크합니다. Copy 작업을 여러 번 반복하고 평균, 중앙값, 신뢰구간 등 통계를 계산하여 신뢰 가능한 throughput 추정치를 산출합니다.

## 중요: vCenter에 생성되는 리소스

Forecaster는 vCenter 환경에 임시 VM과 virtual disk를 생성합니다. 벤치마크가 끝나면 모든 리소스는 자동 정리되지만, vCenter 관리자는 해당 작업이 발생한다는 점을 알고 있어야 합니다.

### Datastore pair별 생성 리소스

| 리소스 | 위치 | 목적 | 수명 |
|---|---|---|---|
| Filler VM | vCenter inventory | Alpine Linux를 부팅하여 benchmark disk를 random data로 채움 | 수 분 내외, disk fill 완료 후 삭제 |
| Alpine boot disk | Source datastore | Filler VM OS용 256 MB thin VMDK | pair 완료 후 삭제 |
| Seed ISO | Source datastore | Filler VM용 cloud-init 구성 | pair 완료 후 삭제 |
| Benchmark disk | Source datastore | 사용자 지정 크기, 기본 10 GB thin VMDK. random data로 채움 | pair 완료 후 삭제 |
| Clone disk | Target datastore | 각 iteration마다 생성되는 benchmark disk copy | 각 iteration 후 삭제 |
| Directories | Source/target datastore | 임시 파일 저장용 `forecaster-*` directory | pair 완료 후 삭제 |

### Filler VM 사양

- 이름: `forecaster-filler-{timestamp}`
- OS: Alpine Linux, 최소 구성, 약 256 MB
- 리소스: 1 vCPU, 256 MB RAM
- 동작: 부팅 후 `dd`로 benchmark disk를 random data로 채운 뒤 자동 power off
- 네트워크 미사용: 네트워크 연결이 필요하지 않으며 사용하지 않음

### Cleanup 보장

- benchmark pair가 성공하거나 실패해도 완료 시 모든 리소스가 자동 정리됩니다.
- Agent가 benchmark 중간에 종료되면 일부 임시 파일이나 디렉터리가 datastore에 남을 수 있으며 수동 정리가 필요할 수 있습니다.
- 임시 리소스 이름은 식별이 쉽도록 `forecaster-` 또는 `filler-image-` prefix를 사용합니다.
- benchmark 오류가 발생하더라도 Filler VM은 항상 삭제됩니다.

## 필요한 vSphere 권한

| 권한 | 필요한 이유 |
|---|---|
| `Datastore.AllocateSpace` | benchmark disk 및 directory 생성 |
| `Datastore.Browse` | datastore contents 조회 |
| `Datastore.DeleteFile` | 임시 파일 및 directory 정리 |
| `Datastore.FileManagement` | filler image 및 seed ISO upload |
| `VirtualMachine.Inventory.Create` | 임시 Filler VM 생성 |
| `VirtualMachine.Inventory.Delete` | 사용 후 Filler VM 삭제 |
| `VirtualMachine.Provisioning.Clone` | datastore 간 disk copy |
| `VirtualMachine.Interact.PowerOn` | Filler VM 부팅 |
| `VirtualMachine.Config.AddRemoveDevice` | Filler VM에 disk attach/detach |

Credential verification endpoint인 `PUT /forecaster/credentials`는 benchmark 시작 전에 권한을 확인하고 누락 권한을 보고합니다.

## API Workflow

### Step 1: Credential 검증

vCenter credential에 필요한 권한이 있는지 확인합니다.

```bash
curl -X PUT http://agent:3443/api/v1/forecaster/credentials \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://vcenter.example.com",
    "username": "administrator@vsphere.local",
    "password": "your-password"
  }'
```

- `200 OK`: credential이 유효함
- `403`: 필요한 권한이 누락됨. 응답에는 `missingPrivileges` 목록 포함

### Step 2: Datastore 목록 조회

환경 인벤토리에서 발견된 datastore 목록을 가져옵니다.

```bash
curl -X POST http://agent:3443/api/v1/forecaster/datastores
```

응답에는 datastore type, capacity, vendor, offload capability 등이 포함됩니다.

### Step 3: Pair capability 확인, 선택 사항

특정 datastore pair에 대해 어떤 storage offload capability를 사용할 수 있는지 확인합니다.

```bash
curl -X POST http://agent:3443/api/v1/forecaster/capabilities \
  -H "Content-Type: application/json" \
  -d '{
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

하나 이상의 datastore pair에 대해 benchmark를 시작합니다.

```bash
curl -X POST http://agent:3443/api/v1/forecaster \
  -H "Content-Type: application/json" \
  -d '{
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

| 파라미터 | 기본값 | 설명 |
|---|---|---|
| `pairs` | 필수 | benchmark할 datastore pair 목록 |
| `pairs[].host` | 자동 선택 | 특정 ESXi host에 고정, 선택 사항 |
| `diskSizeGb` | 10 | benchmark disk 크기, GB |
| `iterations` | 5 | pair별 copy 반복 횟수 |
| `concurrency` | 1 | 병렬 benchmark pair 수 |

- `202 Accepted`: 초기 상태와 함께 시작됨
- `409 Conflict`: 이미 benchmark가 실행 중임

### Step 5: Status Polling

Benchmark 진행 상황을 모니터링합니다.

```bash
curl http://agent:3443/api/v1/forecaster
```

| 상태 | 의미 |
|---|---|
| `pending` | Queue에 대기 중, 아직 시작 전 |
| `preparing` | benchmark disk 생성 및 fill 중 |
| `running` | disk copy iteration 진행 중 |
| `completed` | 모든 iteration 완료 |
| `error` | 실패. `error` field 확인 필요 |
| `canceled` | 사용자에 의해 중단됨 |

`preparing` 중에는 `prepBytesTotal`, `prepBytesUploaded`로 disk fill progress를 확인할 수 있습니다. `running` 중에는 `completedRuns`, `totalRuns`로 iteration 진행 상황을 확인할 수 있습니다.

### Step 6: 결과 조회

Benchmark run data를 조회합니다.

```bash
curl "http://agent:3443/api/v1/forecaster/runs?pairName=local-to-nfs"
```

계산된 통계와 시간 예측을 조회합니다.

```bash
curl "http://agent:3443/api/v1/forecaster/stats?pairName=local-to-nfs"
```

Stats 응답에는 평균/중앙값/최소/최대 throughput, 표준편차, 95% 신뢰구간, 1 TB당 예상 마이그레이션 시간(best case, expected, worst case)이 포함됩니다.

## Benchmark 취소

전체 실행 중인 pair를 취소합니다.

```bash
curl -X DELETE http://agent:3443/api/v1/forecaster
```

단일 pair만 취소합니다.

```bash
curl -X DELETE http://agent:3443/api/v1/forecaster/pairs/local-to-nfs
```

취소하면 active benchmark가 중단되고, 작업 완료 전에 리소스가 정리됩니다.

## 결과 이해하기

### Throughput 통계

| Metric | 설명 |
|---|---|
| `meanMbps` | 성공한 모든 iteration의 평균 throughput |
| `medianMbps` | 중앙값, outlier 영향을 덜 받음 |
| `minMbps` / `maxMbps` | 관측된 throughput 범위 |
| `stddevMbps` | 표준편차, 일관성 측정 |
| `ci95LowerMbps` / `ci95UpperMbps` | 실제 throughput에 대한 95% 신뢰구간 |

### 시간 예측

`estimatePer1TB` field는 1 TB 데이터 기준의 마이그레이션 시간 예측을 제공합니다.

| Estimate | 기준 |
|---|---|
| `bestCase` | 95% 신뢰구간 상한 |
| `expected` | 중앙값 throughput |
| `worstCase` | 95% 신뢰구간 하한 |

실제 데이터 용량에 맞춰 선형적으로 scale하면 됩니다.

## Tips

- Disk size: benchmark disk가 클수록 더 현실적인 결과를 제공하지만 준비 시간이 길어집니다. 빠른 추정에는 10 GB가 적절한 기본값입니다.
- Iterations: 반복 횟수가 많을수록 통계적 신뢰도가 높아집니다. 기본 5회가 합리적이며, production planning에는 10회 이상을 권장합니다.
- Host pinning: 실제 migration에 특정 ESXi host를 사용할 예정이라면 해당 host에 benchmark를 고정하는 것이 더 정확합니다.
- Off-peak testing: 현실적인 throughput을 얻으려면 실제 운영 부하를 대표할 수 있는 시간대에 benchmark를 수행합니다.
- Multiple pairs: 서로 다른 storage backend 간 migration이 있다면 source-target 조합별로 각각 benchmark해야 합니다.
