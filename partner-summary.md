# 파트너 공유용 요약: OpenShift Migration Advisor

## 한 줄 요약

OpenShift Migration Advisor는 VMware 환경을 OpenShift Virtualization으로 이전하기 전에 VM, Host, Datastore, Network, Cluster 정보를 수집하고, migration readiness, warning/error, sizing과 예상 시간을 검토할 수 있게 해주는 assessment 도구입니다.

## 평가 방식

| 방식 | 특징 | 적합한 상황 |
|---|---|---|
| RVTools Upload | 기존 RVTools Excel export 업로드 | 빠른 초기 진단, vCenter 접근이 어려운 경우 |
| Discovery Agent | vCenter에 OVA appliance 배포 | 더 정확하고 지속적인 live inventory 평가