# Migration Advisor

Migration Advisor는 VMware 환경을 OpenShift Virtualization으로 이전하려는 고객과 컨설턴트가 마이그레이션을 시작하기 전에 환경을 평가할 수 있도록 돕습니다. 가상 인프라에 대한 종합적인 관점을 제공하고, 사전에 검토해야 할 migration consideration을 식별합니다.

## What It Does

- VMware vCenter 환경을 분석합니다.
- VM과 VM 구성 정보를 식별합니다.
- migration readiness assessment를 제공합니다.
- infrastructure, storage, network 등에 대한 상세 리포트를 생성합니다.

## Who Should Use This

VMware에서 Red Hat OpenShift Virtualization으로 이전을 계획하는 고객과 컨설턴트에게 적합합니다.

## 환경 평가 방법

### RVTools Upload

이미 VMware 환경에 대한 RVTools export 파일이 있다면 Excel 파일을 업로드하여 즉시 assessment를 받을 수 있습니다.

### Discovery Agent

vCenter에 경량 OVA appliance를 배포하여 live discovery를 수행합니다. Agent는 VMware 환경에서 직접 상세 정보를 수집하고, 실시간 assessment data를 제공합니다.

## Get Started

환경 평가를 시작하려면 다음 단계별 tutorial을 참고합니다.

- **RVTools Flow**: 기존 RVTools Excel export를 업로드하여 즉시 assessment 수행
- **Discovery Agent Flow**: vCenter에 OVA를 배포하여 live discovery 수행

## 관련 문서

- Tutorial
- Migration Forecaster
- Cluster Architecture Sizing in OpenShift Migration Advisor
- Cost Estimation For Partners
- Aggregated Data Report
- OMA Estimations

## Collect Infrastructure Data

Agent는 VM, host, datastore, network, cluster를 포함한 VMware 환경의 익명 데이터를 수집합니다.

## Generate Assessment Reports

워크로드의 migration readiness, compatibility analysis, recommendation에 대한 상세 리포트를 제공합니다.

## Open Source

OpenShift Migration Advisor는 open source 프로젝트로 제공됩니다.
