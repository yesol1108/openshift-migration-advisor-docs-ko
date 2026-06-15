# OMA Estimations

OMA Estimations는 migration complexity와 예상 시간을 계산하는 logic을 설명합니다.

## Complexity Estimation

복잡도는 OS type과 disk size 기준으로 산정됩니다.

- OS score: 0–4, unknown은 0
- Disk score: 1–4
- 점수가 높을수록 수동 개입이나 특수 handling 가능성이 큽니다.

## Time Estimation

주요 calculator는 다음과 같습니다.

| Calculator | 의미 |
|---|---|
| Storage Migration | 총 disk size와 transfer rate 기반 data transfer 시간 |
| Post-Migration Checks | VM 수와 engineer 수 기반 post-check/troubleshooting 시간 |

## 기본 가정

- Transfer rate 기본값: 620 Mbps
- Troubleshooting 기본값: VM당 60분
- Engineer 기본값: 10명
- Work hours 기본값: 하루 8시간