---
title: "Migration Advisor"
linkTitle: "Documentation"
weight: 1
---

Migration Advisor는 VMware 환경을 OpenShift Virtualization으로 이전하려는 고객과 컨설턴트가 현재 환경을 평가할 수 있도록 지원합니다. 가상 인프라 전반을 종합적으로 확인하고, 마이그레이션을 시작하기 전에 고려해야 할 항목을 식별할 수 있습니다.

## What It Does

- VMware vCenter 환경을 분석합니다.
- VM과 해당 구성을 식별합니다.
- migration readiness assessment를 제공합니다.
- 인프라, 스토리지, 네트워크 등에 대한 상세 리포트를 생성합니다.

## Who Should Use This

VMware에서 Red Hat OpenShift Virtualization으로 마이그레이션을 계획 중인 고객과 컨설턴트를 위한 도구입니다.

## There is few Ways to Assess Your Environment

### RVTools Upload

이미 VMware 환경에 대한 RVTools export 파일이 있다면, Excel 파일을 업로드하는 것만으로 즉시 assessment를 받을 수 있습니다.

### Discovery Agent

vCenter에 경량 OVA appliance를 배포하여 live discovery를 수행합니다. 에이전트는 VMware 환경에서 직접 상세 정보를 수집하고 실시간 assessment data를 제공합니다.

## Get Started

환경 assessment를 시작할 준비가 되었다면 다음 단계별 튜토리얼을 확인하세요.

- **[RVTools Flow](tutorial/#rvtools-flow)** - 기존 RVTools Excel export를 업로드하여 즉시 assessment 수행
- **[Discovery Agent Flow](tutorial/#discovery-agent-flow)** - vCenter에 OVA를 배포하여 live discovery 수행
