# Migration Forecaster

Migration Forecaster는 VMware datastore pair 간 실제 storage throughput을 benchmark하여 migration time을 예측합니다.

## 동작 방식

vCenter 환경에 임시 VM과 virtual disk를 생성하고, source datastore에서 target datastore로 disk-copy benchmark를 여러 번 수행합니다. 결과는 평균, 중앙값, 최소/최대, 표준편차, 95% 신뢰구간으로 계산됩니다.

## 생성되는 임시 리소스

- Filler VM
- Alpine boot disk
- Seed ISO
- Benchmark disk
- Clone disk
- `forecaster-*` directory

Benchmark 완료 시 자동 정리되지만, agent가 중간 종료되면 일부 파일이 datastore에 남을 수 있습니다.

## 결과 활용

`estimatePer1TB` 기준으로 best case, expected, worst case migration time을 확인하고 실제 데이터 용량에 맞춰 선형적으로 계산합니다.