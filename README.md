# Backend Engineering Evidence

공개 가능한 범위에서 정리한 백엔드 프로젝트의 기술 의사결정·측정 조건·재현 절차입니다. 비공개 팀 저장소의 소스, 운영 설정, 사용자 데이터, 내부 호스트 구성은 포함하지 않습니다.

| 사례 | 공개 증빙 | 핵심 내용 |
| --- | --- | --- |
| [Dailyting faststart](case-studies/dailyting-faststart.md) | [서비스](https://dailyting.cloud) · [공개 기술 페이지](https://dailyting.cloud/engineering/faststart.html) | mp4 `moov` atom 배치를 바꿔 재생 전 선행 다운로드를 구조적으로 줄인 사례 |
| [YEON ranking cache](case-studies/yeon-ranking-cache.md) | [서비스](https://yeon.world) · [공개 저장소](https://github.com/yeon-intergation-platform/yeon) | 동일 k6 시나리오에서 TTL 캐시 전후를 비교한 사례 |
| [PULL-IT async pipeline](case-studies/pull-it-async-pipeline.md) | [서비스](https://pull.it.kr) · [공개 저장소](https://github.com/kakao-tech-campus-3rd-step3/Team2_BE) | Main/Worker 분리와 실제 MariaDB 저장 경로 측정 사례 |
| [ZERO-ONE backend](case-studies/zero-one-backend.md) | [서비스](https://zeroone.it.kr) | 비공개 팀 저장소 프로젝트의 공개 가능한 설계 수준 증빙 |

## 읽는 방법

- 숫자는 각 문서에 적힌 도구·기간·데이터·오류율·한계 안에서만 해석합니다. 서로 다른 장비나 환경의 절대 성능 비교로 사용하지 않습니다.
- 비공개 저장소 프로젝트는 저장소 링크를 대신해 공개 서비스와 기술 문서를 제공합니다. 면접에서는 권한과 보안 범위 안에서 설계 선택과 검증 방법을 설명할 수 있습니다.
- 이 저장소는 포트폴리오 증빙용 문서이며, 비공개 코드나 운영 인프라 목록을 공개하지 않습니다.

