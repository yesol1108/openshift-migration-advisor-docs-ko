---
layout: default
title: OpenShift Migration Advisor 문서 한국어 번역
---

# OpenShift Migration Advisor 문서 한국어 번역

OpenShift Migration Advisor 공식 문서를 한국어로 번역·정리한 웹 문서입니다.

- 원문: <https://kubev2v.github.io/openshift-migration-advisor-docs/docs/>
- 원문 라이선스: CC BY 4.0
- 번역 기준일: 2026-06-16

## 빠른 이해

OpenShift Migration Advisor는 VMware 환경을 OpenShift Virtualization으로 이전하기 전에 현재 vCenter 환경을 평가하고, VM/Host/Datastore/Network/Cluster 정보를 기반으로 migration readiness, sizing, migration issue 및 예상 시간을 검토할 수 있게 해주는 assessment 도구입니다.

## 문서 목록

1. [요약](overview.md)
2. [Migration Advisor 개요](docs/01-migration-advisor.md)
3. [튜토리얼: RVTools Flow / Discovery Agent Flow](docs/02-tutorial.md)
4. [Migration Forecaster](docs/03-migration-forecaster.md)
5. [Aggregated Data Report](docs/04-aggregated-data-report.md)
6. [OMA Estimations](docs/05-oma-estimations.md)

## 주요 흐름

```text
RVTools 업로드 또는 Discovery Agent 배포
        ↓
VMware inventory 수집
        ↓
Migration readiness / issue / warning 평가
        ↓
Sizing, 예상 시간, 사전 조치 항목 검토
        ↓
OpenShift Virtualization migration planning
```
