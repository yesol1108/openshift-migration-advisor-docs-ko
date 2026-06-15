# Migration Advisor

OpenShift Migration Advisor는 VMware 환경을 Red Hat OpenShift Virtualization으로 이전하기 전에 현재 vCenter 환경을 평가하는 assessment 도구입니다.

## 주요 기능

- VMware vCenter 환경 분석
- VM 및 VM 구성 정보 식별
- migration readiness 평가
- 인프라, 스토리지, 네트워크 리포트 생성

## 평가 방식

### RVTools Upload
RVTools Excel export 파일을 업로드해 빠르게 초기 평가를 수행합니다.

### Discovery Agent
vCenter에 경량 OVA appliance를 배포해 live environment 기반으로 VM, Host, Datastore, Network, Cluster 정보를 수집합니다.

## 활용 포인트

파트너 미팅에서는 “빠른 사전 진단 → 상세 inventory 수집 → migration risk 및 sizing 검토” 흐름으로 설명하면 좋습니다.