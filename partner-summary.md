# 파트너 공유용 요약: OpenShift Migration Advisor

OpenShift Migration Advisor는 VMware 환경을 OpenShift Virtualization으로 이전하기 전에 VM, Host, Datastore, Network, Cluster 정보를 수집하고 migration readiness, warning/error, sizing, 예상 시간을 검토할 수 있게 해주는 assessment 도구입니다.

## 평가 방식

| 방식 | 특징 | 적합한 상황 |
|---|---|---|
| RVTools Upload | RVTools Excel export 업로드 | 빠른 초기 진단, vCenter 접근이 어려운 경우 |
| Discovery Agent | vCenter에 OVA appliance 배포 | 더 정확한 live inventory 평가가 필요한 경우 |

## 파트너 메시지

RVTools로 빠르게 진단하고, Discovery Agent로 정확도를 높인 뒤, Migration Forecaster로 datastore 간 실제 throughput 기반 migration time을 산정하는 흐름으로 설명하면 좋습니다.