# 집앞 조형물 3D 뷰어

집 앞에 있는 조형물 공간을 휴대폰 동영상 한 편으로 찍어서, AI로 3D Gaussian Splat으로 복원하고 웹에서 바로 돌려볼 수 있게 만든 결과물입니다.

**보기: https://bogop8-star.github.io/my-room-3d/**

## 촬영

- 대상: 집 앞 조형물 (걸어서 한 바퀴 돌 수 있는 공간이라 패럴랙스 확보에 유리했습니다)
- 방식: 휴대폰으로 36초 분량 동영상 촬영, 조형물 주변을 걸어다니며 촬영
- 카메라가 걸어다닌 범위 / 장면 전체 크기 비율: 약 0.73 (충분히 넓게 돌면서 찍었다는 뜻)

## 복원 파이프라인

1. 동영상에서 24프레임 균등 추출 (긴 변 1024px로 리사이즈)
2. [MapAnything](https://github.com/facebookresearch/map-anything) (`facebook/map-anything-apache`)로 멀티뷰 3D 복원 — 카메라 포즈, 포인트, 마스크를 한 번에 추론 (GPU에서 약 12초)
3. 신뢰도 낮은 영역 마스킹 + 상하위 0.5% 아웃라이어 제거 → 약 343만 개 포인트
4. 포인트 하나당 Gaussian 속성(위치·색·불투명도·스케일·회전) 부여 후 바이너리 PLY로 저장 (233.5MB)
5. [`@playcanvas/splat-transform`](https://github.com/playcanvas/splat-transform)으로 `.sog` 포맷으로 압축

## 포맷 비교

| 포맷 | 크기 |
| --- | --- |
| PLY (비압축) | 233.5MB |
| SOG (압축) | 15.1MB |

실제 웹에 올린 건 SOG 쪽입니다. 휴대폰 회선에서도 부담 없이 받아지는 크기입니다.

## 뷰어

- [Spark](https://github.com/sparkjsdev/spark) (`@sparkjsdev/spark`) + three.js를 CDN(importmap)으로 불러오는 순수 정적 페이지라 별도 빌드 없이 GitHub Pages에서 바로 돌아갑니다.
- 화면 좌상단에 불러오는 단계(라이브러리 → 렌더러 → 파일 받기 → 알갱이로 풀기 → 크기 재기)를 체크리스트로 보여줍니다.
- 위아래가 뒤집혀 보이면 우측 하단 "위아래 뒤집기" 버튼으로 보정할 수 있습니다.
- 마우스/터치 드래그로 회전, 스크롤로 확대·축소가 가능합니다 (OrbitControls).

## 저장소

- 뷰어 페이지: [index.html](index.html)
- 압축 스플랫 데이터: [scene.sog](scene.sog)
