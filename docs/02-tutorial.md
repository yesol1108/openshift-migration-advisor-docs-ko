# 튜토리얼: Migration Assessment 사용 방법

이 튜토리얼은 VMware 환경을 OpenShift Virtualization으로 이전하기 전에 평가하는 두 가지 방법을 설명합니다.

평가 방법은 다음 중 하나를 선택할 수 있습니다.

- **RVTools Flow**: 기존 RVTools Excel export 파일을 업로드하여 즉시 평가
- **Discovery Agent Flow**: vCenter에 OVA를 배포하여 라이브 탐색 수행

---

## RVTools Flow

기존 RVTools export 파일을 업로드하여 즉시 평가를 수행합니다.

### 사전 조건: RVTools 파일 요구사항

RVTools export 파일을 업로드하기 전에 Excel 파일이 다음 요구사항을 충족하는지 확인해야 합니다. Migration Assessment 도구는 RVTools export 파일을 검증하고, 포함된 데이터에 따라 오류 또는 경고를 표시합니다.

#### 필수 요구사항: 오류로 처리되는 항목

다음 항목이 누락되면 업로드가 실패합니다. 모두 `vInfo` 시트에 있어야 합니다.

| 요구사항 | 이유 |
|---|---|
| `vInfo` 시트에 최소 1개 행이 있어야 함 | VM이 없으면 계획할 인벤토리가 없음 |
| `VM ID` 컬럼이 null/empty가 아니어야 함 | VM의 기본 고유 식별자로 사용 |
| `VM` 또는 `Name` 컬럼이 null/empty가 아니어야 함 | VM 식별 및 화면 표시용 |
| `CLUSTER` 컬럼이 null/empty가 아니어야 함 | VM을 Cluster 기준으로 그룹화 |
| `VI SDK UUID` 컬럼이 null/empty가 아니어야 함 | vCenter 식별에 필요 |

#### 권장 요구사항: 경고로 처리되는 항목

다음 시트는 확인 대상이지만, 비어 있거나 누락되어도 도구는 경고만 표시합니다. 인벤토리는 계속 생성되지만 일부 데이터 포인트는 비어 있을 수 있습니다.

| 시트 | 누락 시 영향 |
|---|---|
| `vHost` | Host 관련 정보 사용 불가 |
| `vDatastore` | Storage/Datastore 정보 누락 |
| `vNetwork` | Network 구성 상세 정보 누락 |
| `vCPU` | Core 수 등 상세 CPU 메트릭 누락 |
| `vMemory` | RAM 할당 상세 정보 누락 |
| `vDisk` | Disk 크기 및 provisioning 데이터 누락 |
| `vNic` | Network interface, IP, MAC 정보 누락 |

가장 완전한 마이그레이션 평가를 위해 RVTools 데이터를 export할 때 모든 시트를 활성화하는 것이 좋습니다.

---

## Discovery Agent Flow

Discovery Agent를 vCenter에 배포하여 라이브 환경 분석을 수행합니다.

### 인프라 데이터 수집

Agent는 VMware 환경에서 VM, Host, Datastore, Network, Cluster를 포함한 익명 데이터를 수집합니다.

### 평가 리포트 생성

워크로드에 대한 마이그레이션 준비 상태, 호환성 분석, 권장 사항을 포함한 상세 리포트를 제공합니다.
