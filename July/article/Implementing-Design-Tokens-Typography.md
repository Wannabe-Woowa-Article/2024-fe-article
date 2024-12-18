## 🔗 Implementing Design Tokens: Typography

### 🗓️ 번역 날짜: 2024.07.06

### 🧚 번역한 크루: 렛서(김다은)

---

<br/>

## [디자인 토큰 구현하기: 타이포그래피](https://medium.com/@slava.karablikov/implementing-design-tokens-typography-47091602abf8)

> 규모 있는 제품에 타이포그래피 토큰을 구현하기 위한 실용적인 가이드

<br/>

![썸네일](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*U7IqEsky245e79YYd0rURQ.png)

<br/>

디자인 토큰이라는 개념은 다양한 디자인 시스템에서 현실이 되었습니다. 구글의 [Material Design 3](https://m3.material.io/foundations/design-tokens/overview)와 [MUI](https://mui.com/)를 비롯한 많은 디자인 시스템의 필수 도구인 토큰은 UI 요소 구현, 관리 및 업데이트를 용이하게 합니다.

디자인 토큰은 색상, 글꼴, 간격, 애니메이션, 에셋과 같은 중요한 UI 데이터를 저장하여 크로스 플랫폼 UI를 만들고 스타일을 지정하는 데 유용합니다. DNA의 체인 구조처럼, 토큰은 디자인 시스템의 비주얼 음성(원문: visual voice) 역할을 합니다. 디자인 토큰은 각 플랫폼에 대해 고정된 값을 넣는 대신 다양한 형식을 저장하고 있기 때문에, 프론트엔드 개발자가 iOS, Android 또는 웹 애플리케이션을 구축할 때 단일 변수를 사용할 수 있습니다.

이번 글에서는 [이전 시리즈](https://medium.com/@slava.karablikov/implementing-color-design-tokens-practical-guide-2ee1d46a1392?source=post_page-----47091602abf8--------------------------------)에서 컬러 토큰을 통해 살펴 본 주제를 좀 더 깊게 살펴보고자 합니다.

우리는 오랜 시간 개발한 대형 디지털 제품의 디자인 시스템 구현에 중점을 두고 있습니다. 디자인 시스템 구축에 관심이 있지만 관련 작업에 부담을 느낀다면 이 문서가 도움이 될 것입니다.

우리의 경험이 중요한 이유는 무엇인가요? Captiv8 플랫폼은 8년 이상의 스토리를 가진 인플루언서와 브랜드를 연결해주는 여러 수상 경력이 있는 플랫폼이자 업계에서 가장 강력한 도구입니다. 빠른 성장의 대가는 엄청난 수의 스타일, 중복된 요소, 끝없는 패턴입니다. 저는 점진적인 업데이트를 통해 이러한 문제를 어떻게 극복했는지 디자인 팀의 경험을 보여드리고자 합니다.

<br/>

### 준비

#### 재고 수집하기

디자인 시스템 구축에 착수할 때는 재고를 수집하는 것부터 시작하는 것이 중요합니다. 여기에는 플랫폼 전체에서 사용되는 모든 타이포그래피 스타일이 포함됩니다. 정확성을 보장하고 가장 눈에 띄는 사용 사례를 파악하기 위해 이 정보를 수동으로 수집했습니다.

우리는 두 가지 주요 문제를 발견했습니다:

- **두 가지 다른 서체**가(원문: type famillies)가 플랫폼에서 동시에 사용되고 있습니다.
- 명확한 시스템 없이 **40개 이상의 타이포그래피 스타일**이 사용되고 있습니다.

<br/>

![재고 수집](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*JLuAxH2kSLKfjAs4T8OX4Q.png)

<br/>

위 이슈들은 이전에 겪었던 색상 다양성 문제보다는 덜 심각했습니다. 또한 잘못된 글꼴 크기나 줄 길이를 선택하는 등의 글꼴 사용 문제는 많지 않았습니다.

모든 글꼴 스타일을 조사한 후, 이를 18개로 줄였고, 이는 99%의 케이스를 만족했습니다. 모든 폰트 크기는 [메이저 서드 타입 스케일](https://designcode.io/typographic-scales)의 1.2 비율을 가집니다. 이는 각 크기가 이전 크기에서 1.2로 곱하거나 나누어진다는 것을 의미하며, 기본 크기에서 시작하여 4픽셀의 배수로 반올림됩니다.

우리의 타이포그래피 스타일에는 두 가지 범주가 있습니다: 제목과 본문. 각 범주는 유연성을 제공하고 사용자 인터페이스의 다양한 응용 프로그램에 적합한 기본 변형 세트와 옵션을 제공합니다. 이들은 단일 스케일을 사용해 모든 화면 크기에 쉽게 적용할 수 있습니다.

<br/>

![폰트 사이즈 목록](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*OVrEz-S5kUcQnLiYVV4d8w.png)

<br/>

### 구현 전략

먼저, 폰트 토큰과 믹스인에 대한 몇 가지 설명을 추가해야 합니다. 첫 번째로 폰트 토큰에 대해 이야기해보겠습니다.

토큰은 네 가지 그룹으로 나누어집니다:

1. **폰트 패밀리**: 우리는 두 가지 관련 폰트 패밀리를 사용합니다: Roboto와 Roboto Mono가 그것입니다. 첫 번째는 일반적인 경우에 사용하고, 두 번째는 인터페이스에서 코드 사용을 표현해야 할 때 사용합니다.
2. **크기**: 단순 폰트 크기입니다. 8가지 옵션을 사용합니다.
3. **높이**: 또한 8가지 옵션이 크기와 함께 작동해 타이포그래피 시스템을 구성합니다.
4. **굵기**: 여기엔 3가지 옵션만 사용합니다: regular, medium, 그리고 bold입니다.

<br/>

![토큰 구현](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*rfSRw4EEBEkUjT4c8OopEw.png)

<br/>

폰트 토큰을 구성할 때는 매개변수에 의존해야 합니다. 우리는 밑줄이 있는 몇 가지 스타일을 가지고 있지만, 중복 방지를 위해 이를 위한 고유한 토큰은 가지고 있지 않습니다.

이 모든 기본 폰트 스타일은 SASS 문법으로 결합되어 믹스인을 만듭니다. Figma의 관점에서 믹스인은 텍스트 스타일과 유사합니다. 예를 들어, SASS에 따르면 우리 시스템의 제목 스타일 중 하나는 다음과 같습니다.

<br/>

```css
@mixin heading-xl {
  font-family: $font-family-sans;
  font-size: $font-size-400;
  font-weight: $font-weight-medium;
  line-height: $font-line-height-4;
}
```

<br/>

플랫폼의 현재 상태를 분석한 후, 우리는 첫 단계에서 믹스인을 구현하지 않기로 결정했습니다. 대신, 우리는 적은 자원 소비로 점진적 개선을 한다는 원칙에 따라 리팩토링된 요소와 패턴과 함께 믹스인을 구현할 것입니다.

<br/>

### 구현

우리는 기존 스타일에 폰트 토큰을 매핑하는 작업부터 시작했습니다. 믹스인을 폰트 토큰과 함께 구현하지 않기로 한 결정 때문에, 전체 스타일이 아닌 각 스타일에 필요한 부분 집합의 토큰만 할당했습니다.

<br/>

![스타일 매핑](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*XUNEVZG88tSzFi0P_iSMIw.png)

<br/>

왜 그럴까요? 가장 큰 이유는 행 간격 때문입니다. 이를 조정하면 전체 페이지 레이아웃이 깨질 수 있습니다. 또한, 텍스트 필드나 버튼과 같은 요소도 위헙합니다. 우리는 변경사항을 최소화하는 것을 목표로 하고 있습니다. 어떻게 그게 가능했을까요? 매우 간단합니다. 스타일을 변경할 때마다 폰트 패밀리, 크기, 굵기, 매개변수에 가장 가까운 토큰을 찾아 적용합니다. 그러나 행 간격 매개변수에 정확한 토큰을 찾을 수 없는 경우, 그대로 둡니다. 따라서 추가적인 수정이 필요하지 않습니다.

다음은 예시입니다. 우리는 다음과 같은 폰트 스타일을 가지고 있습니다:

<br/>

```css
h4 {
  font-family: Source Sans Pro;
  font-size: 24px;
  line-height: 30px;
  font-weight: semibold;
}
```

<br/>

이 경우는 다음과 같이 대체될 수 있습니다.

```css
h4 {
  font-family: var(--font-family-sans); /* Roboto */
  font-size: var(--font-size-400); /* 24px */
  line-height: 30px;
  font-weight: var(--font-weight-medium); /* Medium */
}
```

우리는 이전 접근 방식에서 성공을 거두었고, 이를 더욱 발전시키기로 결정했습니다. 이번 반복에서는 기본 폰트 크기를 16px에서 14px로 줄여 타이포그래피 시스템을 업데이트했습니다. 시각적 혼란의 잠재적 위험이 있었지만, 우리는 구현 계획을 일주일 앞당길 수 있었습니다.

물론 몇 가지 문제가 있었지만, 이전 단계 덕분에 타이포그래피를 충분히 제어할 수 있었기 때문에 쉽게 해결할 수 있었습니다. 일주일 동안의 사소한 버그 수정과 시각적 개선 작업 후, 우리는 성공적으로 폰트 토큰을 플랫폼에 구현했습니다.

사용자들로부터의 피드백은 긍정적이었습니다. 상당한 변화를 겪었음에도 불구하고 인터페이스는 더 리드미컬하고 예측 가능해졌습니다. 기본 폰트 크기를 2px 줄이는 것에 대해 처음에는 걱정했지만, 부정적인 피드백은 없었습니다. 게다가 이는 모바일 해상도에서 작업을 더 쉽게 만들어 주었습니다.

<br/>

![타이포그래피 이슈에 관한 트래커](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*M2Yn-SY9uwWBolnkqGOgeA.png)

<br/>

### 결론

우리는 디자인 시스템을 구축하는 데 상당한 진전을 이루었습니다. 다행히 플랫폼의 타이포그래피 측면에서는 큰 어려움을 겪지 않아 이 단계를 순조롭게 마칠 수 있었고, 최종 사용자에게도 혼란을 주지 않았습니다. 하지만 팀에게는 몇 가지 이점이 있었습니다. 첫째, 디자인 시스템을 구축하면서 개발자와 디자이너 간의 협업과 의사소통이 촉진되었습니다. 또한, 구현 후에야 비로소 그 이점을 명확히 느낄 수 있었습니다. 인터페이스는 큰 변화가 없었지만, 앞으로의 수정 작업에 대한 더 큰 제어권을 얻을 수 있었습니다. 이는 향후 업데이트의 길을 열어주었고, 프론트엔드 개발자들에게 코드베이스를 구성하고 리팩토링하는 새로운 관점을 제공했습니다.

문자와 텍스트를 시각적으로 매력적이고 읽기 쉽게 배열하는 것은 타이포그래피의 예술입니다. 다양한 글꼴, 스타일, 구조를 통해 감정과 중요한 메시지를 전달할 수 있습니다. 타이포그래피가 사용자 경험에 미치는 영향은 결코 과소평가될 수 없으며, 이는 인터페이스의 모든 측면에 영향을 미치고 브랜딩과도 긴밀히 연관됩니다. 우리 팀은 폰트 토큰을 우선시하여 색상 토큰 구현 직후에 이를 적용했습니다. 구현 순서는 유연하지만, 이러한 토큰 하위 집합을 조기에 우선적으로 도입하는 것이 권장됩니다.
