# Migration Advisor

OpenShift Migration Advisor는 VMware 환경을 Red Hat OpenShift Virtualization으로 이전하려는 고객과 컨설턴트가 사전에 환경을 평가할 수 있도록 돕는 도구입니다. 가상 인프라 전반을 종합적으로 확인하고, 실제 마이그레이션을 시작하기 전에 고려해야 할 사항을 식별할 수 있습니다.

## 주요 기능

- VMware vCenter 환경 분석
- VM 및 VM 구성 정보 식별
- 마이그레이션 준비 상태 평가 제공
- 인프라, 스토리지, 네트워크 등에 대한 상세 리포트 생성

## 사용 대상

VMware에서 Red Hat OpenShift Virtualization으로 이전을 계획하는 고객 및 컨설턴트에게 적합합니다.

## 환경 평가 방법

### RVTools Upload

이미 VMware 환경에 대한 RVTools export 파일이 있다면, Excel 파일을 업로드하여 즉시 평가를 받을 수 있습니다.

### Discovery Agent

경량 OVA appliance를 vCenter에 배포하여 라이브 환경을 탐색합니다. Agent는 VMware 환경에서 직접 상세 정보를 수집하고 실시간 평가 데이터를 제공합니다.

## 시작하기

환경 평가를 시작하려면 다음 흐름을 사용할 수 있습니다.

- RVTools Flow: 기존 RVTools Excel export 파일을 업로드하여 즉시 평가
- Discovery Agent Flow: vCenter에 OVA를 배포하여 라이브 탐색 수행

## 추가 문서

- Tutorial
- Migration Forecaster
- Cluster Architecture Sizing in OpenShift Migration Advisor
- Cost Estimation For Partners
- Aggregated Data Report
- OMA Estimations

## 핵심 가치

- 인프라 데이터 수집: Agent는 VM, Host, Datastore, Network, Cluster를 포함한 VMware 환경 데이터를 수집합니다.
- 평가 리포트 생성: 마이그레이션 준비 상태, 호환성 분석, 워크로드별 권장 사항을 제공합니다.
- Open Source: 프로젝트는 오픈소스로 제공됩니다.
