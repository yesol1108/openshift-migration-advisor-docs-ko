# OMA Estimations: 권장 사항 산정 방식

OMA recommendation methodology를 설명하는 문서입니다.

## Migration Estimations

이 문서는 OMA tool에 구현된 estimation logic을 설명합니다. 주요 영역은 OS 및 disk tier 기반의 complexity estimation과, storage migration 및 post-migration check 기반의 time estimation입니다.

- 적용 버전: V0.7.0
- 마지막 업데이트: 12-03-2026

---

## 1. Complexity Estimation

Complexity estimation은 OS type과 disk size라는 두 가지 독립적인 기준을 바탕으로 migration이 얼마나 어려울 것으로 예상되는지 breakdown을 제공합니다. 이 score는 planning과 priority 설정에 사용되며, time calculation에는 직접 영향을 주지 않습니다.

### 1.1 Score Scale

숫자 범위는 1–4이며, 1이 가장 쉽고 4가 가장 복잡합니다.

| Score | Meaning |
|---|---|
| 0 | OS를 인식할 수 없어 complexity 평가 불가 |
| 1 | 비교적 단순한 migration, 최소 개입 예상 |
| 2 | 표준적인 migration, 일부 확인 필요 |
| 3 | 복잡한 migration, 더 많은 effort 예상 |
| 4 | 수동 개입 및 특수 handling 가능성 |

- OS score: 0–4, 0은 unknown
- Disk score: 1–4, disk size에는 unknown이 적용되지 않음

---

### 1.2 OS Map, OS Difficulty Scores

OS complexity는 VMware가 보고한 VM OS 이름을 OS difficulty scores map과 substring matching하여 산정합니다. Matching은 대소문자를 구분하지 않습니다. Keyword가 매칭되지 않으면 OS는 score 0, unknown을 받습니다.

`classify OS` function은 map을 순회하면서 처음 matching되는 keyword의 score를 반환합니다. 여러 key가 같은 OS name과 매칭될 수 있는 경우에도 동일한 score를 갖도록 설계되어 있어 iteration order가 결과에 영향을 주지 않습니다.

#### Full OS Map

| Score | OS Family | Matched Versions / Notes |
|---|---|---|
| 1 | Red Hat Enterprise Linux | 6, 7, 8, 9, 10 |
| 1 | Red Hat Fedora | all |
| 1 | CentOS | 6, 7, 8, 9 |
| 1 | Oracle Linux | 6, 7, 8, 9 |
| 1 | Rocky Linux | 8, 9 |
| 2 | Red Hat Enterprise Linux | 4, 5 |
| 2 | CentOS | 4, 5 |
| 2 | Oracle Linux | 4, 5 |
| 2 | SUSE Linux Enterprise | 12, 15 |
| 2 | Microsoft Windows Server | 2016, 2019, 2022, 2025 |
| 2 | Microsoft Windows | 10, 11 |
| 3 | SUSE Linux Enterprise | 8, 9, 10, 11 |
| 3 | SUSE openSUSE | all |
| 3 | AlmaLinux | 8, 9 |
| 3 | Ubuntu Linux | 18.04, 20.04, 22.04, 24.04, generic |
| 3 | Debian GNU/Linux | 5–12 |
| 3 | Microsoft Windows Server | 2000, 2003, 2008, 2008 R2, 2012, 2012 R2 |
| 3 | Microsoft Windows | XP, Vista, 7, 8 |
| 3 | Oracle Solaris | 10, 11 |
| 3 | FreeBSD | all |
| 3 | VMware Photon OS | all |
| 3 | Amazon Linux 2 | all |
| 3 | CoreOS Linux | all |
| 3 | Apple macOS | all |
| 4 | Microsoft SQL | all, database workload |
| 0 | any other | Unknown, 예: `Other (64-bit)`, 인식되지 않는 OS |

---

### 1.3 Disk Tiers, Disk Size Scores

Disk complexity는 사용자 입력에서 산정하며, 각 tier는 numeric score로 mapping됩니다.

| Tier Label | Size Range, Provisioned | Score |
|---|---:|---:|
| 1, 0–10TB | ≤ 10 TB | 1 |
| 2, 11–20TB | ≤ 20 TB | 2 |
| 3, 21–50TB | ≤ 50 TB | 3 |
| 4, >50TB | > 50 TB | 4 |

---

## 2. Time Estimation

Time estimation은 migration의 각 phase를 model하는 여러 calculator를 결합합니다. Estimation은 등록된 모든 calculator를 실행하고 각 duration을 합산하여 total migration time을 산출합니다.

### 2.1 Calculators Overview

| Calculator | Phase | Required Params | Optional Params |
|---|---|---|---|
| Storage Migration | Data transfer | Total disk in GB | Transfer rate mbps |
| Post-Migration Checks | Post-migration troubleshooting | VM count | troubleshoot mins per VM, number of post migration engineers, work hours per day |

---

### 2.2 Storage Migration Calculator

Source에서 target cluster로 VM storage data를 network를 통해 transfer하는 데 필요한 시간을 산정합니다.

#### Assumptions

- Transfer rate: 기본값 620 Mbps, 77.5 MB/s. 이는 500 GB당 약 110분이라는 baseline과 일치합니다.
- Data size: cluster 내 모든 VM의 total provisioned disk size, GB를 사용합니다.
- Linear transfer: 지속적인 throughput을 가정하며, parallelism이나 다른 phase와의 overlap은 고려하지 않습니다.

#### Formula

```text
storage_migration_minutes = (totalDiskGB × 1024) / (transferRateMbps / 8) / 60
```

Where:

- `totalDiskGB` = total disk size, GB
- `1024` = GB당 MB
- `60` = 분당 초

#### Example

Input:

- `total_disk_gb`: 1000
- `transfer_rate_mbps`: 620, default

Calculation:

```text
(1000 × 1024) / (620 / 8) / 60 ≈ 220.2 min
```

Output:

- Duration: 약 3h 40m
- Reason: `1000.00 GB at 620 Mbps (110 min/500GB)`

---

### 2.3 Post-Migration Troubleshooting Calculator

VM별 post-migration check와 troubleshooting에 engineer가 투입하는 시간을 산정합니다.

#### Assumptions

- Troubleshooting time: 기본값 VM당 60분
- Engineers: 기본값 10명 병렬 작업
- Work hours: 기본값 하루 8시간, duration 계산이 아니라 reason string에만 사용
- Parallelism: 총 man-minutes를 engineer count로 나누어 wall-clock time 산정

#### Formula

```text
post_migration_minutes = (vm_count × troubleshoot_mins_per_vm) / post_migration_engineers
```

#### Example

Input:

- `vm_count`: 50
- `troubleshoot_mins_per_vm`: 60, default
- `post_migration_engineers`: 10, default
- `work_hours_per_day`: 8, default

Calculation:

```text
50 × 60 / 10 = 300 min
```

Output:

- Duration: 5h 0m
- Reason: `50 VMs @ 60.0 mins each / 10 engineers working 8 h/day for a total of 1 work days`

---

### 2.4 Combined Example

Input, cluster inventory 기준:

- Total disk: 2000 GB
- VM count: 100
- 모든 optional params는 default 사용

Storage Migration:

```text
(2000 × 1024) / (620/8) / 60 ≈ 440.4 min → 약 7h 20m
```

Post-Migration Checks:

```text
100 × 60 / 10 = 600 min → 10h 0m
```

Total migration time: 약 17h 20m

## Collect Infrastructure Data

Agent는 VM, host, datastore, network, cluster를 포함한 VMware 환경의 익명 데이터를 수집합니다.

## Generate Assessment Reports

워크로드의 migration readiness, compatibility analysis, recommendation에 대한 상세 리포트를 제공합니다.

## Open Source

OpenShift Migration Advisor는 open source 프로젝트입니다.
