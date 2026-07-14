# YEON: 랭킹 API TTL 캐시 부하테스트

- 역할: Independent Developer / Project Lead
- 공개 서비스: https://yeon.world
- 공개 저장소: https://github.com/yeon-intergation-platform/yeon

## 문제와 해결

게임 좋아요 랭킹 API가 요청마다 전체 좋아요 데이터를 `GROUP BY`로 집계하는 병목을 보였습니다. 느리게 변하는 랭킹 결과를 5초 TTL의 인메모리 캐시로 전환하고, 좋아요 토글 시에는 즉시 무효화하도록 했습니다. 따라서 최대 stale window는 5초이며, 쓰기 직후에는 캐시를 비워 정합성을 보완합니다.

## 측정 조건

- 대상: `GET /game-service/likes/ranking?limit=100`
- 도구와 시나리오: k6 `constant-vus`, 50 VU, 30초, before/after 동일 시나리오
- 환경: 로컬에서 실행한 실제 애플리케이션과 PostgreSQL. 라이브 서비스나 CDN을 경유하지 않았습니다.
- 데이터: synthetic load data 475,700 likes / 200 games. 측정 후 제거했습니다.
- 오류 기준: HTTP 200 check와 `http_req_failed < 1%`; 두 실행 모두 오류율 0.00%였습니다.
- warm-up: 별도 warm-up 단계는 구성하지 않았습니다. 이 결과는 30초 constant-VU 실행의 비교값입니다.

## 결과

| 지표 | Before: 요청마다 집계 | After: TTL 캐시 | 변화 |
| --- | ---: | ---: | ---: |
| 총 요청수 | 13,000 | 1,704,861 | — |
| 처리량 | 431.6 TPS | 56,827 TPS | 약 131배 |
| p95 | 151.9ms | 1.4ms | 약 99% 감소 |
| 오류율 | 0.00% | 0.00% | — |

## 재현 절차

실제 애플리케이션과 PostgreSQL을 실행한 뒤, 동일한 데이터와 환경에서 아래처럼 전후 결과를 export했습니다.

```bash
k6 run --summary-export=baseline.json ranking.js
# TTL cache 적용 후 같은 시나리오 실행
k6 run --summary-export=after.json ranking.js
```

`ranking.js`의 기본값은 `VUS=50`, `DURATION=30s`입니다.

## 해석 범위와 한계

- 수치는 같은 로컬 환경에서의 전후 비교입니다. 장비 사양을 기록하지 않았으므로 범용적인 서버 처리량으로 일반화하지 않습니다.
- 캐시 hit 비율과 캐시 워밍 상태는 별도의 운영 메트릭으로 측정하지 않았습니다.
- 결과의 핵심은 절대 TPS가 아니라, 동일 시나리오에서 매 요청 집계를 제거했을 때의 상대 변화와 p95 개선입니다.

