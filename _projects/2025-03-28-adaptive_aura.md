---
layout: project
title: "Adaptive_Aura"
description: "Adaptive_Aura는 모든 Flutter 플랫폼에서 사용 가능한 이미지 반응형 UI 패키지입니다.
사용자가 제공하는 이미지의 색상을 추출하여 여러가지 스타일의 컨테이너 그래픽을 생성해주며, 파라미터를 통해 커스터마이징 요소도 제공합니다."
type: "Flutter Package" # 프로젝트 유형 
date: 2025-03-28 #대략적인 개발 기간
thumbnail: "/assets/images/adative_aura_title.001.jpeg" # 썸네일 이미지 링크 or path
posts: 
  - 2025-02-18-color-responsible-splash.md

---

## 프로젝트 소개

`adaptive_aura`는 이미지(혹은 내가 지정한 팔레트)의 색을 읽어, 화면 뒤에서 은은하게 살아 움직이는 “오라” 배경을 만들어주는 Flutter 패키지입니다.
음악 플레이어·갤러리·디테일 화면 같은 곳에서 “이미지와 잘 어울리는” 배경을 자동으로 생성해주는 도구입니다.

- 지원 플랫폼: Android / iOS / Web / macOS / Windows / Linux
- 라이선스: MIT
- 배포: [pub.dev/adaptive_aura](https://pub.dev/packages/adaptive_aura)
- 작업 일지: [/color-responsible-splash/](/color-responsible-splash/)


## 개발 동기

Apple Music의 앨범 커버 기반 배경 생성 기능에 영감을 얻어 작업을 시작하게 되었습니다.

유기적인 Flutter UI 공부를 위해 작업이 진전되다가, 다른 플러터 유저들도 이 기능을 써보았으면 좋겠다고 느껴, 색상 추출 기능을 비롯한 다양한 테마를 추가하여 패키지화하게 되었습니다.


## 패키지 특징

- 이미지를 바꿔도 어색하지 않은, “이미지 친화적” 배경 자동 생성
- 채도 낮은 이미지나 흑백 이미지에서도 밋밋하지 않게 보정
- 앱 전반에서 재사용 가능한 API와 안정적인 성능
- 꼭 정해진 테마가 아니더라도, 색상 추출기를 함수화하여 커스터마이징 가능


## 개발 과정

자세한 과정은 작업 일지에 정리했습니다: [/color-responsible-splash/](/color-responsible-splash/)

1) 색 추출
- 모든 픽셀을 돌지 않고 대표 샘플만 뽑아 성능과 품질을 맞췄습니다.
- 밝기 가중치(0.299R + 0.587G + 0.114B)로 정렬해 primary/secondary/tertiary, light/dark를 뽑습니다.
- 실패하면 기본 팔레트로 폴백합니다.

2) 팔레트 성격 판별(HSL)
- 평균 채도/명도와 명도 범위를 보고 Grayscale / Vivid / Dark / Bright / Medium으로 성격을 나눕니다.
- 성격에 따라 그라디언트 색 조합과 분포 전략을 다르게 적용합니다.

3) 레이어 구조
- Base Linear Gradient → Radial Gradient Points → Highlight → Blur를 순서대로 쌓아 자연스러운 깊이를 만듭니다.


## 주요 기능

- 스타일 3종: Gradient / Blob / Sunray
- 파라미터: animationValue, variety(복잡도), blurStrength(X/Y), colorIntensity, blurLayerOpacity, 전환 Duration 등
- 전환: 이미지 교체 시 컬러 트랜지션과 효과 애니메이션
- 콜백: onPaletteGenerated로 팔레트 활용 가능


## 간단한 사용법

```dart
AdaptiveAuraContainer(
  image: const AssetImage('assets/images/album.jpg'),
  auraStyle: AuraStyle.gradient,   // gradient | blob | sunray
  animationValue: 0.8,
  variety: 0.6,
  colorIntensity: 0.85,
  blurStrength: 15.0,
  blurLayerOpacity: 0.1,
  animationDuration: const Duration(milliseconds: 800),
  colorTransitionDuration: const Duration(milliseconds: 300),
  onPaletteGenerated: (palette) {
    // palette.primary / secondary / tertiary / light / dark
  },
  child: YourContentWidget(),
)
```

## 성능/품질 고려

- 대용량 이미지에서도 버벅임을 최소화하기 위해 대표 샘플링 적용
- 알파값 0(투명) 픽셀 제외, 이상치 대비 폴백 팔레트 제공
- Web/저사양 환경을 위해 애니메이션·블러 상한선 가이드

## 링크

- 패키지: [pub.dev/adaptive_aura](https://pub.dev/packages/adaptive_aura)
- 작업 일지(구현 상세): [/color-responsible-splash/](/color-responsible-splash/)
- 관련 포스트: [/guitar-app-2/](/guitar-app-2/) 외

