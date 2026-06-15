# Aggregated Data Report

Aggregated Data Report는 Migration Advisor가 수집한 VMware inventory data를 migration planning 관점에서 정리한 리포트입니다.

## 주요 데이터

- VM 수와 migratable 상태
- CPU, RAM, Disk resource breakdown
- OS 지원 여부와 upgrade recommendation
- migration issue 및 warning
- ESXi host, cluster, datacenter 정보
- network, datastore 정보

## Migration Issues

Migration을 차단할 수 있는 항목에는 unsupported disk type, unsupported guest OS, snapshot, USB device, NVRAM encryption 등이 포함될 수 있습니다.

## 활용 방법

파트너는 이 리포트를 통해 migration scope, risk, sizing, 사전 조치 항목을 고객과 함께 정리할 수 있습니다.