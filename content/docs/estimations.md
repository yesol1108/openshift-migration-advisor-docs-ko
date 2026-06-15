---
title: "Beyond the Numbers: OMA Estimations"
linkTitle: "OMA Estimations"
description: "OMA recommendation 뒤의 methodology를 자세히 설명합니다."
---
# Migration Estimations

이 문서는 OMA tool에 구현된 estimation logic을 설명합니다. 주요 영역은 **complexity estimation**(OS와 disk tier)과 **time estimation**(storage migration 및 post-migration checks)입니다.

Applies to: V0.7.0  
Last update (dd-mm-yyyy): 12-03-2026

---

## 1. Complexity Estimation

Complexity estimation은 migration 난이도를 두 가지 독립 dimension인 **OS type**과 **disk size**를 기준으로 분류합니다. 이 score는 planning과 prioritization에 사용되며, time calculation에는 직접 영향을 주지 않습니다.

### 1.1 Score Scale

Numeric range는 1–4이며, 1이 가장 쉽고 4가 가장 복잡합니다.

| Score | Meaning |
|-------|---------|
| 0 | OS가 인식되지 않음; complexity assessment 불가 |
| 1 | 단순 migration, 최소 개입 예상 |
| 2 | 표준 migration, 일부 확인 필요 |
| 3 | 복잡한 migration, 더 많은 effort 예상 |
| 4 | Manual intervention 및 special handling이 필요할 수 있음 |

- **OS scores**: 0–4(0 = unknown)
- **Disk scores**: 1–4(unknown은 disk size에 적용되지 않음)

---

### 1.2 OS Map (OS Difficulty Scores)

OS complexity는 VMware가 보고한 VM OS name을 OS difficulty scores map과 **substring matching**하여 산정합니다. Matching은 **case-insensitive**입니다. Keyword가 match되지 않으면 OS는 score **0(unknown)**을 받습니다.

Classify OS function은 map을 순회하면서 첫 번째 matching keyword의 score를 반환합니다. 여러 key가 동일 OS name에 match될 수 있는 경우에도 동일 score를 갖도록 설계되어 iteration order가 결과에 영향을 주지 않습니다.

#### Full OS Map

| Score | OS Family | Matched Versions / Notes |
|-------|-----------|--------------------------|
| **1** | Red Hat Enterprise Linux | 6, 7, 8, 9, 10 |
| | Red Hat Fedora | all |
| | CentOS | 6, 7, 8, 9 |
| | Oracle Linux | 6, 7, 8, 9 |
| | Rocky Linux | 8, 9 |
| **2** | Red Hat Enterprise Linux | 4, 5 |
| | CentOS | 4, 5 |
| | Oracle Linux | 4, 5 |
| | SUSE Linux Enterprise | 12, 15 |
| | Microsoft Windows Server | 2016, 2019, 2022, 2025 |
| | Microsoft Windows | 10, 11 |
| **3** | SUSE Linux Enterprise | 8, 9, 10, 11 |
| | SUSE openSUSE | all |
| | AlmaLinux | 8, 9 |
| | Ubuntu Linux | 18.04, 20.04, 22.04, 24.04, generic |
| | Debian GNU/Linux | 5–12 |
| | Microsoft Windows Server | 2000, 2003, 2008, 2008 R2, 2012, 2012 R2 |
| | Microsoft Windows | XP, Vista, 7, 8 |
| | Oracle Solaris | 10, 11 |
| | FreeBSD | all |
| | VMware Photon OS | all |
| | Amazon Linux 2 | all |
| | CoreOS Linux | all |
| | Apple macOS | all |
| **4** | Microsoft SQL | all(database workload) |
| **0** | *(any other)* | Unknown — 예: "Other (64-bit)", unrecognised OSes |

---

### 1.3 Disk Tiers (Disk Size Scores)

Disk complexity는 user input에서 산정되며, 각 tier는 numeric score에 mapping됩니다.

| Tier Label | Size Range (Provisioned) | Score |
|-------------|---------------------------|-------|
| 1 (0-10TB) | ≤ 10 TB | 1 |
| 2 (11-20TB) | ≤ 20 TB | 2 |
| 3 (21-50TB) | ≤ 50 TB | 3 |
| 4 (>50TB) | > 50 TB | 4 |

---

## 2. Time Estimation

Time estimation은 migration의 각 phase를 model하는 여러 **calculator**를 조합합니다. Estimation은 등록된 모든 calculator를 실행하고 duration을 합산하여 total migration time을 산출합니다.

### 2.1 Calculators Overview

| Calculator | Phase | Required Params | Optional Params |
|-------------------------|--------------------------|-----------------|----------------|
| Storage Migration | Data transfer | Total disk in GB | Transfer rate mbps |
| Post-Migration Checks | Post-migration troubleshooting | vm count | troubleshoot mins per vm, number of post migration engineers, work hours per day |

---

### 2.2 Storage Migration Calculator

Source에서 target cluster로 VM storage data를 network를 통해 transfer하는 데 필요한 시간을 estimate합니다.

#### Assumptions

- **Transfer rate**: Default 620 Mbps(77.5 MB/s). 이는 500 GB당 약 110분 baseline과 일치합니다.
- **Data size**: Cluster 내 모든 VM의 total provisioned disk size(GB)를 사용합니다.
- **Linear transfer**: Sustained throughput을 가정하며, parallelism 또는 다른 phase와의 overlap은 고려하지 않습니다.

#### Formula

```
transferRateMBps = transferRateMbps / 8
totalMinutes = (totalDiskGB × 1024) / transferRateMBps / 60
```

Where:
- `totalDiskGB` = total disk size in gigabytes
- `1024` = MB per GB
- `60` = seconds per minute

#### Example

**Input:**
- `total_disk_gb`: 1000
- `transfer_rate_mbps`: 620(default)

**Calculation:**
```
transferRateMBps = 620 / 8 = 77.5 MB/s
totalMinutes = (1000 × 1024) / 77.5 / 60 ≈ 220.2 minutes
```

**Output:**
- Duration: ~3h 40m
- Reason: `"1000.00 GB at 620 Mbps (110 min/500GB)"`

---

### 2.3 Post-Migration Troubleshooting Calculator

VM별 post-migration checks와 troubleshooting에 engineer가 투입하는 시간을 estimate합니다.

#### Assumptions

- **Troubleshooting time**: Default 60 minutes per VM.
- **Engineers**: Default 10 engineers working in parallel.
- **Work hours**: Default 8 hours per day(reason string에만 사용, duration에는 직접 사용하지 않음).
- **Parallelism**: Total man-minutes를 engineer count로 나누어 wall-clock time 산정.

#### Formula

```
totalManMins = vmCount × troubleshootMinsPerVM
realTimeMins = totalManMins / engineerCount
duration = realTimeMins minutes
```

#### Example

**Input:**
- `vm_count`: 50
- `troubleshoot_mins_per_vm`: 60(default)
- `post_migration_engineers`: 10(default)
- `work_hours_per_day`: 8(default)

**Calculation:**
```
totalManMins = 50 × 60 = 3000
realTimeMins = 3000 / 10 = 300 minutes
workDays = ceil(300 / (8 × 60)) = ceil(300 / 480) = 1
```

**Output:**
- Duration: 5h 0m
- Reason: `"50 VMs @ 60.0 mins each / 10 engineers working 8 h/day for a total of 1 work days"`

---

### 2.4 Combined Example

**Input (from cluster inventory):**
- Total disk: 2000 GB
- VM count: 100
- Defaults for all optional params

**Storage Migration:**
- `(2000 × 1024) / (620/8) / 60 ≈ 440.4 min` → **~7h 20m**

**Post-Migration Checks:**
- `100 × 60 / 10 = 600 min` → **10h 0m**

**Total migration time:** ~17h 20m
