---
layout: default
title: OpenShift Migration Advisor 요약
---

# OpenShift Migration Advisor 요약

OpenShift Migration Advisor는 VMware 환경을 OpenShift Virtualization으로 이전하기 전에 VM, Host, Datastore, Network, Cluster 정보를 수집하고 migration readiness, warning/error, sizing, 예상 시간을 검토할 수 있게 해주는 assessment 도구입니다.

## 평가 방식

| 방식 | 특징 | 적합한 상황 |
|---|---|---|
| RVTools Upload | RVTools Excel export 업로드 | 빠른 초기 진단, vCenter 접근이 어려운 경우 |
| Discovery Agent | vCenter에 OVA appliance 배포 | 더 정확한 live inventory 평가가 필요한 경우 |

## 핵심 포인트

1. **초기 진단 진입장벽이 낮음**  
   RVTools 파일만 있어도 assessment를 시작할 수 있습니다.

2. **Discovery Agent는 더 정확한 데이터 확보에 유리**  
   vCenter API에서 직접 데이터를 수집하므로 live environment 기반의 평가가 가능합니다.

3. **Migration Forecaster로 시간 예측 근거 확보**  
   datastore pair 간 실제 copy benchmark를 수행하여 migration time을 계산합니다. 단순 계산이 아니라 실제 환경 throughput 기반입니다.

4. **Aggregated Data Report로 scope와 risk 정리 가능**  
   VM 수, resource allocation, OS 지원 여부, migration block issue, warning, datastore/network 정보를 정리할 수 있습니다.

5. **OMA Estimations는 effort planning에 활용**  
   OS와 disk size 기반 complexity, storage migration time, post-migration troubleshooting time을 산정합니다.

## 설명 예시

Migration Advisor는 VMware에서 OpenShift Virtualization으로 전환을 검토하는 환경에서 현재 환경이 어느 정도 이전 가능한지, 어떤 VM이 차단될 수 있는지, 대략 어느 정도 시간이 걸릴지 사전에 확인하게 해주는 도구입니다. RVTools 업로드 방식은 빠른 초기 assessment에 적합하고, Discovery Agent 방식은 vCenter에 OVA를 배포해 더 정확한 live inventory를 수집하는 방식입니다. 추가로 Migration Forecaster를 사용하면 datastore 간 실제 throughput을 측정해 migration time forecast를 데이터 기반으로 제시할 수 있습니다.

## 주의사항

- Migration Forecaster는 benchmark 수행 중 vCenter에 임시 VM, disk, ISO, directory를 생성합니다.
- Benchmark가 정상 종료되면 리소스는 자동 정리되지만, agent가 중간 종료되면 일부 임시 파일이 남을 수 있습니다.
- vCenter credential에는 datastore 및 VM 생성/삭제/clone/power-on 관련 권한이 필요합니다.
- `unsupported OS`는 migration이 반드시 실패한다는 의미가 아니라, Red Hat에서 virt-v2v conversion을 공식 검증/지원하지 않는다는 의미에 가깝습니다.
