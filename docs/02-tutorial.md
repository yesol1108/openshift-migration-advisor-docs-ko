# 튜토리얼: Migration Assessment 사용 방법

Migration Assessment는 VMware 환경을 OpenShift Virtualization으로 이전하기 전에 두 가지 방식으로 평가할 수 있습니다.

## RVTools Flow

기존 RVTools Excel export 파일을 업로드하여 즉시 평가합니다. 필수 데이터는 주로 `vInfo` 시트에 있으며, VM ID, VM/Name, Cluster, VI SDK UUID 같은 식별 정보가 필요합니다.

## Discovery Agent Flow

Discovery Agent OVA를 vCenter에 배포하여 live inventory를 수집합니다. Agent는 VM, Host, Datastore, Network, Cluster 정보를 수집하고 migration readiness, compatibility, warning/error를 포함한 리포트를 제공합니다.

## 선택 기준

- 빠른 초기 진단: RVTools Flow
- 더 정확한 live 평가: Discovery Agent Flow