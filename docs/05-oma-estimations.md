# OMA Estimations: 권장 사항 산정 방식

이 문서는 OMA tool에 구현된 estimation logic을 설명합니다. 주요 영역은 complexity estimation, 즉 OS와 disk tier 기반 복잡도 산정, 그리고 time estimation, 즉 storage migration과 post-migration check 시간 산정입니다.

- 적용 버전: V0.7.0
- 마지막 업데이트: 12-03-2026

## 1. Complexity Estimation

Complexity estimation은 migration이 얼마나 어려울 것으로 예상되는지를 OS type과 disk size라는 두 가지 독립적인 기준으로 나누어 보여줍니다. 이 score는 계획과 우선순위 지정에 사용되며, 시간 계산에는 직접 영향을 주지 않습니다.

### 1.1 Score Scale

| Score | 의미 |
|---|---|
| 0 | OS를 인식할 수 없어 complexity 평가 불가 |
| 1 | 비교적 단순한 migration, 최소 개입 예상 |
| 2 | 일반적인 migration, 일부 확인 필요 |
| 3 | 복잡한 migration, 더 많은 effort 예상 |
| 4 | 수동 개입 및 특수 handling 가능성 |

- OS score: 0–4, 0은 unknown
- Disk score: 1–4, disk size에는 unknown이 적용되지 않음

### 1.2 OS Map: OS Difficulty Scores

OS complexity는 VMware가 보고한 VM OS 이름을 OS difficulty score map과 substring matching하여 산정합니다. 대소문자는 구분하지 않으며, keyword가 매칭되지 않으면 score 0, unknown으로 처리합니다.

| Score | OS Family / Version |
|---|---|
| 1 | Red Hat Enterprise Linux 6, 7, 8, 9, 10 / Fedora / CentOS 6–9 / Oracle Linux 6–9 / Rocky Linux 8–9 |
| 2 | Red Hat Enterprise Linux 4–5 / CentOS 4–5 / Oracle Linux 4–5 / SUSE Linux Enterprise 12, 15 / Windows Server 2016, 2019, 2022, 2025 / Windows 10, 11 |
| 3 | SLES 8–11 / openSUSE / AlmaLinux 8–9 / Ubuntu 18.04–24.04 / Debian 5–12 / Windows Server 2000–2012 R2 / Windows XP, Vista, 7, 8 / Solaris 10–11 / FreeBSD / Photon OS / Amazon Linux 2 / CoreOS / macOS |
| 4 | Microsoft SQL 등 database workload |
| 0 | 기타 인식되지 않는 OS, 예: Other 64-bit |

### 1.3 Disk Tiers: Disk Size Scores

Disk complexity는 사용자 input 기반으로 산정되며, 각 tier는 numeric score에 매핑됩니다.

| Tier | Size Range, Provisioned | Score |
|---|---|---|
| 1, 0-10TB | ≤ 10 TB | 1 |
| 2, 11-20TB | ≤ 20 TB | 2 |
| 3, 21-50TB | ≤ 50 TB | 3 |
| 4, >50TB | > 50 TB | 4 |

## 2. Time Estimation

Time estimation은 migration의 각 phase를 모델링하는 여러 calculator를 결합합니다. 등록된 calculator를 모두 실행하고 duration을 합산하여 전체 migration 시간을 산정합니다.

### 2.1 Calculators Overview

| Calculator | Phase | Required Params | Optional Params |
|---|---|---|---|
| Storage Migration | Data transfer | Total disk in GB | Transfer rate Mbps |
| Post-Migration Checks | Post-migration troubleshooting | VM count | VM당 troubleshooting minutes, engineer 수, 하루 work hours |

### 2.2 Storage Migration Calculator

Source에서 target cluster로 VM storage data를 network를 통해 전송하는 데 필요한 시간을 산정합니다.

#### Assumptions

- Transfer rate 기본값: 620 Mbps, 즉 77.5 MB/s
- 이 값은 500 GB당 약 110분이라는 baseline과 일치합니다.
- Data size: cluster 내 모든 VM의 총 provisioned disk size, GB 기준
- Linear transfer: 지속 throughput을 가정하며 parallelism이나 다른 phase와의 overlap은 고려하지 않습니다.

#### Formula

- `totalDiskGB` = 총 disk size, GB
- `1024` = GB당 MB
- `60` = minute당 second

예시:

- `total_disk_gb`: 1000
- `transfer_rate_mbps`: 620, 기본값
- 결과: 약 3h 40m
- 사유: `1000.00 GB at 620 Mbps (110 min/500GB)`

### 2.3 Post-Migration Troubleshooting Calculator

VM별 post-migration check와 troubleshooting에 engineer가 투입하는 시간을 산정합니다.

#### Assumptions

- Troubleshooting time 기본값: VM당 60분
- Engineers 기본값: 10명 병렬 작업
- Work hours 기본값: 하루 8시간, duration 계산이 아니라 reason string에만 사용
- Parallelism: 총 man-minutes를 engineer 수로 나누어 wall-clock time 산정

예시:

- `vm_count`: 50
- `troubleshoot_mins_per_vm`: 60, 기본값
- `post_migration_engineers`: 10, 기본값
- `work_hours_per_day`: 8, 기본값
- 결과: 5h 0m

### 2.4 Combined Example

Cluster inventory input:

- Total disk: 2000 GB
- VM count: 100
- Optional params는 모두 기본값 사용

Storage Migration:

```text
(2000 × 1024) / (620/8) / 60 ≈ 440.4 min → 약 7h 20m
```

Post-Migration Checks:

```text
100 × 60 / 10 = 600 min → 10h 0m
```

총 migration time은 약 17h 20m입니다.
