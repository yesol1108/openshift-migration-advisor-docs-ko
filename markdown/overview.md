# OpenShift Migration Advisor 전체 개요

OpenShift Migration Advisor는 VMware 환경을 OpenShift Virtualization으로 이전하기 전에 현재 vCenter 환경을 평가하고, VM/Host/Datastore/Network/Cluster 정보를 바탕으로 migration readiness, sizing, migration issue, 예상 시간을 검토할 수 있게 해주는 assessment 도구입니다.

## 평가 방식

| 방식 | 특징 | 적합한 상황 |
|---|---|---|
| RVTools Upload | RVTools Excel export 업로드 | 빠른 초기 진단, vCenter 직접 접근이 어려운 경우 |
| Discovery Agent | vCenter에 OVA appliance 배포 | 더 정확한 live inventory 평가가 필요한 경우 |
| Migration Forecaster | datastore pair 간 disk-copy benchmark | 실제 storage throughput 기반으로 migration time을 예측해야 하는 경우 |

## 주요 결과물

- VMware inventory 기반 VM/Host/Datastore/Network/Cluster 현황
- migration 가능/경고/불가 VM 집계
- OS, disk, network, datastore 관련 issue 및 warning
- OpenShift Virtualization migration planning에 필요한 sizing 및 estimation 근거
- Migration Forecaster를 통한 datastore pair별 실제 throughput 및 1TB 기준 예상 시간

## 문서별 역할

1. **Migration Advisor**: 도구 목적과 평가 방식 개요
2. **튜토리얼**: RVTools Upload와 Discovery Agent 사용 흐름
3. **Migration Forecaster**: 실제 storage throughput benchmark와 API workflow
4. **Aggregated Data Report**: 수집되는 data field와 리포트 의미
5. **OMA Estimations**: complexity score와 migration time 산정 방식
