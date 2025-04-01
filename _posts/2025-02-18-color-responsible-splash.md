---
layout: post
title: "Apple Music 스타일 앨범 커버 UI/UX 구현하기"
date: 2025-02-18
tags: [Flutter, UI/UX, Animation]
read_time: 10
subtitle: "이미지의 색상을 추출하여 반응하는 컨테이너 UI를 구현해봅니다."
---

## 개요

유저에게 색다른 경험을 주는 UI/UX에 관심이 많은 저에게, 요새 최대 관심사는 애플 뮤직 앱입니다.

퇴근길 버스에서 음악을 들으며 집에 가는 동안, 업계 최고의 디자이너와 개발자들이 구현해낸 멋진 인터렉션과 UX 디자인을 보며 많은 영감을 받고 있습니다.

그래도 감탄만 할 수는 없겠죠, 직접 만들어봅시다.

## 애플 뮤직 UI 디자인 분석

제가 분석한 애플 뮤직의 전반적인 UI 디자인의 지향점은 **특색 없음** 입니다.

마냥 단조롭다는 뜻이 아니라, 아티스트들이 저마다 자신의 음악 세계를 표현하기 위해 만들어낸 앨범 커버를 수용해야 하는 앱의 특성 상, 앱 자체가 너무 튀는 포인트 컬러 테마나 화려한 인터렉션을 지니면 아티스트들의 의도를 온전히 표현할수 없기 때문으로 보입니다.

마치 예술 작품을 걸어두는 갤러리 같은 느낌을 상상하셔도 좋을 듯 합니다.

<figure>
  <img src="/assets/images/post-250218-01.jpg" alt="인사동 갤러리" class="screenshot">
  <figcaption>출처-한국화랑협회</figcaption>
</figure>


인사동 쌈짓길부터 압구정 외곽까지, 그림을 판매하는 갤러리들의 인테리어는 대개 무채색이나 깔끔한 가구, 동선 배치로 방문객들로 하여금 갤러리가 아닌 작품이 주인공이라는 느낌을 받게끔 합니다.

애플의 디자이너도 비슷한 생각을 가지고 있는 것 같습니다.

<figure>
  <img src="/assets/images/post-250218-02.jpg" alt="애플 뮤직 앱 앨범 커버 반응형 컨테이너" class="screenshot">
  <figcaption>출처-애플 뮤직 앱</figcaption>
</figure>

애플 뮤직의 UI 테마 역시 아티스트들의 앨범 커버가 돋보이기 위해 화이트/블랙의 모던한 컬러를 기반으로, 거기에 최대한 화려함을 덜어낸 미니멀 디자인을 보여줍니다. 애플이 원래도 미니멀한 디자인을 추구하지만, 뮤직 앱은 더욱 그런 특성이 두드러집니다.

허나 마냥 미니멀하기만 하면 앱이 너무 심심하고 단조로워 보일 수 있습니다.


## 앨범 커버 반응형 컨테이너

이에 애플이 제안한 해결 방법은 아래와 같습니다.


<figure>
  <img src="/assets/images/post-250218-03.png" alt="애플 뮤직 앱 앨범 커버 반응형 컨테이너" class="screenshot">
  <figcaption>출처-애플 뮤직 앱</figcaption>
</figure>

위는 애플 뮤직에서 음악을 실행중일 때 볼 수 있는 잠금 화면 UI입니다.

자칫 평범해보일수 있는 뮤직 플레이어 UI지만, 배경 화면을 앨범 커버 이미지의 색상을 추출한 스플래시 UI로 덮어 미니멀한 디자인을 유지하면서도 각각의 앨범 커버 이미지들의 특색을 살려줍니다.

**이미지의 색상을 추출하여 배경에 그려준다**

라는 단순한 발상이지만, 유저들에게 내가 자주 듣는 앨범에 특별한 요소를 부여하고, 앱의 스타일을 해치지 않으면서 앨범 커버 아트의 심미적인 요소를 살려내는 좋은 방법이라고 생각됩니다.

물론, 이 UI를 구현하기 위해서는 생각보다 신경써야 할 요소가 많습니다.

무작위(혹은 다양한 프리셋 중 선택)으로 렌더링되는 그라데이션 요소나, 앨범 커버 이미지에서 색상을 추출할 때 단순히 이미지의 색상 면적 비율에 비례하는게 아닌 시각적으로 자연스러운 요소를 취사 선택하는 기술 등등... 애플답게 단순해보이는 UI에도 디테일한 요소가 상당히 많이 존재했습니다.

하나하나 분석하여 만들어봅시다.

## 이미지에서 색상을 추출하기

실제 구현에서의 최우선 과제는, 이미지에서 색상을 추출하는 로직입니다.

아래는 색상 추출 로직의 전체 코드입니다.

##### ColorExtractor
~~~dart
part of '../adaptive_aura.dart';

/// Utility class for extracting colors from images
class ColorExtractor {
  /// Extract color palette from an image
  static Future<AuraColorPalette> extractColorsFromImage({
    required ImageProvider imageProvider,
    bool enableLogging = false,
  }) async {
    try {
      if (enableLogging) {
        debugPrint('🎨 Starting color extraction from image...');
      }

      // Load image
      final imageStream = imageProvider.resolve(ImageConfiguration.empty);
      final completer = Completer<ui.Image>();
      final listener = ImageStreamListener(
        (ImageInfo info, bool _) {
          completer.complete(info.image);
        },
        onError: (exception, stackTrace) {
          completer.completeError(exception);
        },
      );

      imageStream.addListener(listener);
      final image = await completer.future;
      imageStream.removeListener(listener);

      // Check image dimensions
      final width = image.width;
      final height = image.height;

      if (enableLogging) {
        debugPrint('📏 Image size: $width x $height');
      }

      // Extract image data
      final byteData =
          await image.toByteData(format: ui.ImageByteFormat.rawRgba);
      if (byteData == null) {
        throw Exception('Unable to extract image data');
      }

      final pixels = byteData.buffer.asUint8List();
      final colors = <Color>[];

      // Sampling pixel count (not processing all pixels for performance optimization)
      final sampleSize = (width * height) ~/ 100;
      final step = (width * height) ~/ sampleSize;

      for (int i = 0; i < pixels.length; i += step * 4) {
        if (i + 3 < pixels.length) {
          final r = pixels[i];
          final g = pixels[i + 1];
          final b = pixels[i + 2];
          final a = pixels[i + 3];

          // Ignore transparent pixels
          if (a > 0) {
            colors.add(Color.fromARGB(a, r, g, b));
          }
        }
      }

      if (enableLogging) {
        debugPrint('🔍 Number of sampled colors: ${colors.length}');
      }

      // Return default palette if no colors extracted
      if (colors.isEmpty) {
        if (enableLogging) {
          debugPrint('⚠️ No colors extracted. Using default palette.');
        }
        return AuraColorPalette.defaultPalette();
      }

      // Sort colors by brightness
      colors.sort((a, b) {
        final brightnessA = (0.299 * a.red + 0.587 * a.green + 0.114 * a.blue);
        final brightnessB = (0.299 * b.red + 0.587 * b.green + 0.114 * b.blue);
        return brightnessB.compareTo(brightnessA);
      });

      // Select main colors
      final primary = colors[colors.length ~/ 3];
      final secondary = colors[colors.length ~/ 2];
      final tertiary = colors[colors.length ~/ 4];
      final light = colors.first;
      final dark = colors.last;

      if (enableLogging) {
        debugPrint('✅ Color extraction complete');
      }

      return AuraColorPalette(
        primary: primary,
        secondary: secondary,
        tertiary: tertiary,
        light: light,
        dark: dark,
      );
    } catch (e) {
      debugPrint('❌ Error during color extraction: $e');
      return AuraColorPalette.defaultPalette();
    }
  }
}
~~~

ColorExtractor 클래스는 이미지에서 조화로운 색상 팔레트를 자동으로 추출하는 역할을 합니다. 이 과정을 단계별로 분석해보겠습니다.


#### 이미지 소스 분석
~~~ dart
final imageStream = imageProvider.resolve(ImageConfiguration.empty);
final completer = Completer<ui.Image>();
final listener = ImageStreamListener(
  (ImageInfo info, bool _) {
    completer.complete(info.image);
  },
  onError: (exception, stackTrace) {
    completer.completeError(exception);
  },
);

imageStream.addListener(listener);
final image = await completer.future;
imageStream.removeListener(listener);
~~~

첫 번째 과정은 이미지 소스를 분석하는 것입니다.

Flutter의 ImageProvider 시스템을 활용하여 다양한 소스(asset, network, file 등)의 이미지를 일관되게 입력 받고, Completer 패턴으로 비동기 이미지 로딩을 처리합니다.

에러 핸들링을 통한 안정적인 이미지 로딩을 제공하도록 하고 있습니다.


#### 색상 정보 추출
~~~ dart
final byteData = await image.toByteData(format: ui.ImageByteFormat.rawRgba);
if (byteData == null) {
  throw Exception('Unable to extract image data');
}

final pixels = byteData.buffer.asUint8List();
~~~

이후, 이미지를 RGBA 형식의 바이트 데이터로 변환, 각 필셀의 색상 정보에 접근할수 있도록 제공합니다.


#### 이미지 픽셀 최적화
~~~ dart
final sampleSize = (width * height) ~/ 100;
final step = (width * height) ~/ sampleSize;

for (int i = 0; i < pixels.length; i += step * 4) {
  if (i + 3 < pixels.length) {
    final r = pixels[i];
    final g = pixels[i + 1];
    final b = pixels[i + 2];
    final a = pixels[i + 3];

    // 투명한 픽셀 무시
    if (a > 0) {
      colors.add(Color.fromARGB(a, r, g, b));
    }
  }
}
~~~

이미지를 픽셀 데이터로, 픽셀 데이터에서 색상 정보로 추출하는 과정에서는 추가적인 최적화 처리도 필요합니다.

대용량 이미지에 대비하여, 모든 픽셀을 처리하는 대신 일정 간격으로 샘플링합니다. 

또한, 투명 픽셀에 대한 색상 계산은 제외합니다. 사각 형태의 앨범 커버 이미지 뿐만 아니라, 투명 픽셀을 포함하고 있는 PNG 이미지(흔히들 말하는 누끼 없는 이미지 등)도 대비하기 위함입니다.


#### 컬러 팔레트 생성하기
~~~dart
colors.sort((a, b) {
  final brightnessA = (0.299 * a.red + 0.587 * a.green + 0.114 * a.blue);
  final brightnessB = (0.299 * b.red + 0.587 * b.green + 0.114 * b.blue);
  return brightnessB.compareTo(brightnessA);
});

// 주요 색상 선택
final primary = colors[colors.length ~/ 3];
final secondary = colors[colors.length ~/ 2];
final tertiary = colors[colors.length ~/ 4];
final light = colors.first;
final dark = colors.last;
~~~

우리의 눈은 녹색에 더 민감하게 반응합니다.
따라서 자연스러운 색상 추출을 위해 3원색의 색상 중 청색광을 최대한 줄이고, 녹색의 비중을 높히도록 가중치를 적용합니다.

추출한 색상을 바탕으로, 밝은 색상, 어두운 색상, 중간 톤(그중에서도 Primary, secondart, tertiary) 로 구성된 색상 팔레트를 만들어냅니다.


## 컬러 팔레트 활용하기...?

이제 생성된 컬러 팔레트를 배경에 입력하여 배경 UI를 그려봅시다.

헌데, 한가지 준비과정이 더 남아있습니다.

위 팔레트를 그대로 그라데이션 배경 UI에 넣어도 괜찮겠지만, 컬러 팔레트의 성격에 따라 약간의 추가 처리가 필요합니다.

<figure>
  <img src="/assets/images/post-250218-01.jpeg" alt="애플 뮤직 앱 앨범 커버 반응형 컨테이너" class="screenshot">
  <figcaption>애플 뮤직 잠금화면 캡쳐</figcaption>
</figure>

사람이 인지하는 앨범 커버의 이미지 컬러 구성과, 코드를 통해 컴퓨터가 인식하는 이미지의 컬러 구성은 상이합니다. 

위 이미지의 예시로, 우리가 레드 핫 칠리 페퍼스의 Californication 앨범 커버(우측)를 바라보면 **주황색과 파란색의 대비가 인상적**이라고 자연스럽게 느끼지만, 

픽셀 단위로 분석하는 컴퓨터는 우리의 눈에는 중요도가 낮다고 느껴지는 수영장 영역 양 옆의 살구색 영역, 나무 그림의 어두운 초록색 영역도 팔레트에 포함할수 있습니다.

무채색의 경우에도 마찬가지입니다.
악틱 몽키즈의 AM 앨범 커버(좌측)의 경우에도, 대부분이 검은색으로 이루어져있기에 앨범 커버 이미지를 바라보는 우리는 자연스레 **검은색 위주의 배경**을 연상할수 있습니다.

허나 색상 추출 코드에서는 중간의 그래프 형상 포인트 그래픽의 하얀색마저 추출해버려, 마치 판다 무늬같은 배경이 생성될 가능성이 존재합니다.

## 컬러 팔레트의 색감을 판별하기
~~~dart
/// 아우라 색상 특성 enum
enum AuraColorCharacteristic {
  /// 생동감 있는 색상 (원색)
  VIVID,

  /// 그레이스케일 (무채색)
  GRAYSCALE,

  /// 어두운 톤
  DARK,

  /// 밝은 톤
  BRIGHT,

  /// 중간 톤
  MEDIUM,
}
~~~

~~~dart
/// 색상 팔레트의 특성 판별
AuraColorCharacteristic _determineColorCharacteristic() {
  // HSL 색상 공간 기반 분석 (색상, 채도, 명도)
  final primaryHSL = HSLColor.fromColor(colorPalette.primary);
  final secondaryHSL = HSLColor.fromColor(colorPalette.secondary);
  final tertiaryHSL = HSLColor.fromColor(colorPalette.tertiary);
  final lightHSL = HSLColor.fromColor(colorPalette.light);
  final darkHSL = HSLColor.fromColor(colorPalette.dark);

  // 평균 채도 계산
  final avgSaturation = (primaryHSL.saturation + 
                        secondaryHSL.saturation + 
                        tertiaryHSL.saturation) / 3;

  // 평균 명도 계산
  final avgLightness = (primaryHSL.lightness + 
                       secondaryHSL.lightness + 
                       tertiaryHSL.lightness + 
                       lightHSL.lightness) / 4;

  // 명도 범위 계산 (최대-최소)
  final lightnessRange = lightHSL.lightness - darkHSL.lightness;

  // 채도 임계값 (이 값보다 낮으면 그레이스케일로 간주)
  const saturationThreshold = 0.15;

  // 그레이스케일 판별: 채도가 낮은 경우
  if (avgSaturation < saturationThreshold) {
    return AuraColorCharacteristic.GRAYSCALE;
  }

  // 생동감 판별: 채도가 높고 명도 범위가 넓은 경우
  if (avgSaturation > 0.5 && lightnessRange > 0.5) {
    return AuraColorCharacteristic.VIVID;
  }

  // 어두움 판별: 평균 명도가 낮은 경우
  if (avgLightness < 0.35) {
    return AuraColorCharacteristic.DARK;
  }

  // 밝음 판별: 평균 명도가 높은 경우
  if (avgLightness > 0.65) {
    return AuraColorCharacteristic.BRIGHT;
  }

  // 기본값: 중간 톤
  return AuraColorCharacteristic.MEDIUM;
}
~~~

이러한 문제를 해결하기 위해서는 컬러 팔레트가 어떠한 구성으로 이루어져 있는지(비비드한 색감인지, 무채색 계열인지, 중간 톤인지 등등) 판별하기 위한 별도의 로직이 필요합니다.

위의 코드를 HSL 색상 공간을 사용하여 팔레트의 특성을 분석하게 되며, 과정은 다음과 같습니다:
1. 팔레트 색상들의 HSL 값을 계산합니다.
2. 평균 채도와 명도를 계산합니다.
3. 명도 범위(가장 밝은 색과 가장 어두운 색의 차이)를 계산합니다.
4. 다음 규칙에 따라 색상 특성을 판별합니다:
- 채도가 매우 낮으면 **GRAYSCALE**
- 채도가 높고 명도 범위가 넓으면 **VIVID**
- 평균 명도가 낮으면 **DARK**
- 평균 명도가 높으면 **BRIGHT**
- 그 외의 경우는 **MEDIUM**

~~~dart
switch (colorCharacteristic) {
  case AuraColorCharacteristic.VIVID:
    // 생생한 테마: 생동감 있는 색상만 사용, 그레이스케일 완전 제거
    colors.add(colorPalette.primary);
    colors.add(colorPalette.secondary);
    colors.add(colorPalette.tertiary);

    // 추가 생동감 있는 색상 믹스
    colors.add(Color.lerp(colorPalette.primary, colorPalette.secondary, 0.3)!);
    colors.add(Color.lerp(colorPalette.secondary, colorPalette.tertiary, 0.3)!);
    colors.add(Color.lerp(colorPalette.tertiary, colorPalette.primary, 0.3)!);
    // ... 더 많은 색상 믹스
    break;

  case AuraColorCharacteristic.GRAYSCALE:
    // 그레이스케일 테마: 주로 그레이스케일 색상 사용
    colors.add(colorPalette.primary);
    colors.add(colorPalette.dark);
    colors.add(colorPalette.light);

    // 추가 그레이스케일 믹스
    colors.add(Color.lerp(colorPalette.primary, colorPalette.dark, 0.5)!);
    colors.add(Color.lerp(colorPalette.primary, colorPalette.light, 0.5)!);

    // 색감 살짝 추가 (variety가 높을 때만)
    if (variety > 0.7) {
      colors.add(colorPalette.secondary.withOpacity(0.3));
    }
    break;

  case AuraColorCharacteristic.DARK:
    // 어두운 테마: 주로 어두운 색상 사용
    colors.add(colorPalette.primary);
    colors.add(colorPalette.dark);
    colors.add(colorPalette.secondary);

    // 추가 어두운 색상 믹스
    colors.add(Color.lerp(colorPalette.primary, colorPalette.dark, 0.7)!);
    colors.add(Color.lerp(colorPalette.secondary, colorPalette.dark, 0.7)!);

    // 밝은 색상 살짝 추가 (variety가 높을 때만)
    if (variety > 0.6) {
      colors.add(Color.lerp(colorPalette.primary, colorPalette.light, 0.2)!);
    }
    break;

  // ... BRIGHT 및 MEDIUM 특성에 대한 처리
}
~~~

위의 코드는 추후 소개드릴 실제 UI 코드에서 활용되는 코드입니다.

컬러 팔레트의 특성을 판별하게 되면, 비비드한 색감의 팔레트는 주요 컬러를 위주로 사용하고, 무채색 계열 팔레트는 주로 그레이스케일 색상을 사용하는 등- 색감에 따른 그라디언트 배경을 생성합니다.

이를 통해 컬러 팔레트의 색감별로 구성 컬러들을 조금 더 자연스럽게 배정하고, 색감에 따라 지향하는 UI를 더 세밀하게 조정할수 있습니다.



## 그라디언트 배경 생성하기

이제는 진짜로 배경 UI를 구현해볼 차례입니다.

애플 뮤직 스타일의 배경을 구현하기 위해서는, 다음의 요소들이 필요합니다.

- 기본 배경: 팔레트 색상을 사용한 선형 그라디언트
- 그라디언트 포인트: 화면 전체에 분산된 다양한 크기와 불투명도의 방사형 그라디언트
- 하이라이트 포인트: 특정 위치에 추가되는 강조 효과

위 세 개의 요소를 각각의 위젯으로 레이어화하여, Stack 구조로 그라디언트 배경을 완성해봅시다.

#### 기본 배경 레이어

~~~dart
@override
Widget buildBackgroundLayer() {
  // 애니메이션 값에 따라 그라디언트 위치 조정
  final animationOffset = Alignment(0, 0.2 - (0.2 * animationValue));

  // 팔레트 색상을 사용한 그라디언트 생성
  return Container(
    decoration: BoxDecoration(
      gradient: LinearGradient(
        begin: Alignment.topLeft + animationOffset,
        end: Alignment.bottomRight - animationOffset,
        colors: [
          colorPalette.primary.withOpacity(0.9 * animationValue),
          colorPalette.secondary.withOpacity(0.8 * animationValue),
          colorPalette.tertiary.withOpacity(0.7 * animationValue),
        ],
      ),
    ),
  );
}
~~~

기본 배경은 팔레트의 주요 색상들을 사용하여 선형 그라디언트를 생성합니다. 
animationValue에 따라 그라디언트의 위치와 불투명도가 동적으로 변화하도록 설계했습니다.


#### 그라디언트 포인트 (방사형)

~~~dart
/// 그라디언트 포인트 생성
List<_GradientPoint> _generateGradientPoints() {
  final points = <_GradientPoint>[];
  final pointCount = _effectiveGradientPointCount;

  // 기본 그라디언트 포인트 (항상 포함)
  points.add(_generateBaseGradientPoint());

  // variety > 0인 경우 추가 그라디언트 포인트 생성
  if (variety > 0.0 && pointCount > 2) {
    // 색상 팔레트 특성에 따른 색상 구성
    final colors = <Color>[];
    
    switch (colorCharacteristic) {
      case AuraColorCharacteristic.VIVID:
        // 생동감 있는 테마: 원색 위주
        colors.add(colorPalette.primary);
        colors.add(colorPalette.secondary);
        colors.add(colorPalette.tertiary);
        // 추가 색상 믹스...
        break;
      case AuraColorCharacteristic.GRAYSCALE:
        // 그레이스케일 테마: 무채색 위주
        colors.add(colorPalette.primary);
        colors.add(colorPalette.dark);
        colors.add(colorPalette.light);
        // 추가 그레이스케일 믹스...
        break;
      // 다른 특성들에 대한 처리...
    }

    // 그리드 기반 포인트 분산 배치
    final gridSize = math.sqrt(pointCount).ceil();
    final cellWidth = containerSize.width / gridSize;
    final cellHeight = containerSize.height / gridSize;

    // 추가 포인트 생성
    for (int i = 1; i < pointCount; i++) {
      // 색상 선택 및 포인트 생성 로직...
      points.add(_GradientPoint(
        color: gradientColor,
        position: position,
        size: size,
        opacity: opacity * animationValue,
      ));
    }
  }

  return points;
}
~~~

두 번째 레이어는 방사형 그라디언트입니다.
특정 포인트를 지정하여, variety 값에 따라 단계적으로 표기됩니다.


#### 레이어 구조
~~~dart
@override
Widget buildAuraLayer() {
  if (containerSize.width <= 0 ||
      containerSize.height <= 0 ||
      animationValue <= 0.01) {
    return Container();
  }

  return ClipRect(
    child: Stack(
      children: [
        // 하이라이트 레이어
        CustomPaint(
          size: containerSize,
          painter: _HighlightPainter(
            highlightPoints: _highlightPoints,
            variety: variety,
            animationValue: animationValue,
            colorCharacteristic: colorCharacteristic,
            colorPalette: colorPalette,
          ),
        ),

        // 블러 레이어
        BackdropFilter(
          filter: ui.ImageFilter.blur(
            sigmaX: 15.0 + (25.0 * variety),
            sigmaY: 15.0 + (25.0 * variety),
          ),
          child: Container(color: Colors.transparent),
        ),
      ],
    ),
  );
}
~~~

최종적으로 모든 레이어가 Stack으로 쌓여 렌더링됩니다:
1. 가장 아래에 기본 배경 (선형 그라디언트)
2. 그 위에 그라디언트 포인트들
3. 맨 위에 하이라이트 포인트들
4. 전체적으로 블러 효과 적용
이러한 레이어 구조와 각 요소들의 특성별 처리를 통해 애플 뮤직과 유사한 자연스러운 그라디언트 배경이 구현됩니다.


## 완성 예시

<figure>
  <img src="/assets/images/post-250218-04.jpeg" alt="" class="screenshot">
  <figcaption> </figcaption>
</figure>

이미지 데이터로부터 색상을 추출하고, 추출한 색상 팔레트를 기반으로