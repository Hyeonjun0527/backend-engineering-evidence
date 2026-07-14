# Dailyting: mp4 faststart로 영상 시작 조건 개선

- 역할: 2인 제품팀의 Backend Owner
- 공개 서비스: https://dailyting.cloud
- 공개 기술 페이지: https://dailyting.cloud/engineering/faststart.html
- 저장소: 비공개 팀 저장소. 이 문서는 공개 가능한 측정 조건과 기술 판단만 제공합니다.

## 문제와 판단

정상적인 영상 URL에서도 첫 프레임 표시가 늦어지는 현상을 확인했습니다. 앱 렌더링이나 스토리지 속도로 바로 단정하지 않고, 영상 파일의 top-level atom 배치를 확인했습니다. 원인은 `moov`(재생 인덱스)가 파일 끝에 놓인 non-faststart mp4였습니다. progressive 재생기는 인덱스를 읽을 때까지 재생을 시작할 수 없습니다.

해결은 업로드 확정 단계에서 `ffmpeg -c copy -movflags +faststart`로 `moov` atom을 파일 앞쪽으로 옮기는 remux입니다. 재인코딩이 아니므로 코덱을 다시 변환하지 않습니다. 처리에 실패하면 원본을 확정하고, 상태 전환은 중복 실행되지 않도록 설계했습니다.

## 측정 조건과 결과

동일한 생성 영상(1280×720, 10초, 약 829KB)의 atom 배치를 비교했습니다. 값은 네트워크 속도나 단말 성능이 아니라 **파일 구조상 재생 인덱스를 읽기 전 필요한 바이트 수**입니다.

| 조건 | atom 순서 | `moov` 위치 | 재생 시작 전 필요한 선행 다운로드 |
| --- | --- | ---: | ---: |
| Before: non-faststart | `ftyp → mdat → moov` | offset 824,577 | 828,968B (100%) |
| After: faststart | `ftyp → moov → mdat` | offset 32 | 4,423B (0.5%) |

따라서 같은 파일 구조에서 재생 인덱스에 도달하기 전 필요한 다운로드가 100%에서 0.5%로 줄었습니다(약 99%).

## 재현

```bash
ffmpeg -y -f lavfi -i testsrc=size=1280x720:rate=30:duration=10 \
  -pix_fmt yuv420p -b:v 3M a.mp4
ffmpeg -y -i a.mp4 -c copy -movflags +faststart b.mp4
# 각 파일의 top-level atom 순서와 moov offset을 확인
```

## 해석 범위와 한계

- 이 수치는 atom 배치에 따른 결정적 구조 측정입니다. 실제 단말의 첫 프레임 시간, 네트워크 RTT, 플레이어별 버퍼 정책은 별도로 측정하지 않았습니다.
- 그러므로 “사용자 첫 프레임 시간이 정확히 99% 빨라졌다”는 뜻이 아닙니다. “재생기가 인덱스를 얻기 위해 기다려야 하는 파일 선행 다운로드 조건이 약 99% 줄었다”는 뜻입니다.
- 운영 파일과 사용자 데이터, 내부 경로 및 배포 구성은 공개하지 않습니다.

