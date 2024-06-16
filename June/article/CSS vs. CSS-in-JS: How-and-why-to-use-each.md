## [CSS vs. CSS-in-JS: How and why to use each](https://blog.logrocket.com/css-vs-css-in-js/)

### 🗓️ 번역 날짜: 2024.06.10

### 🧚 번역한 크루: 버건디(전태헌)

---

# CSS vs. CSS-in-JS: 각각 사용하는 방법과 이유

JavaScript 프레임워크를 사용하면서 개발자들이 흔히 직면하는 딜레마 중 하나는 CSS-in-JS를 사용할지 여부입니다.

React 개발자라면 아마 CSS-in-JS를 사용해본 경험이 있을 것입니다.

![](https://blog.logrocket.com/wp-content/uploads/2022/11/How-why-use-modern-CSS-vs-CSS-in-JS.png)

요즘 CSS vs CSS-in-JS는 뜨거운 주제입니다.

주로 CSS-in-JS가 성능 문제로 지적받기 때문입니다.

그러나 이러한 문제를 해결할 수 있는 새로운 CSS 기능들도 개발 중에 있습니다.

이 글의 목적은 현대 CSS의 현재 상태와 미래에 어떻게 변화할 가능성이 있는지를 고려하여, 다가오는 프로젝트에서 CSS와 CSS-in-JS 중에서 선택하는 데 도움을 주기 위함입니다.

1. [Render-blocking and CSS](https://blog.logrocket.com/css-vs-css-in-js/#render-blocking-css)
2. [What CSS-in-JS offers](https://blog.logrocket.com/css-vs-css-in-js/#what-css-in-js-offers)
3. [Pros of CSS-in-JS](https://blog.logrocket.com/css-vs-css-in-js/#pros-css-in-js)
4. [Cons of CSS-in-JS](https://blog.logrocket.com/css-vs-css-in-js/#cons-css-in-js)
5. [Recommendations for where to use CSS-in-JS](https://blog.logrocket.com/css-vs-css-in-js/#recommendations-where-use-css-in-js)
6. [Overview of CSS Module](https://blog.logrocket.com/css-vs-css-in-js/#overview-css-module)
7. [Pros of CSS Module](https://blog.logrocket.com/css-vs-css-in-js/#pros-css-module)
8. [Cons of CSS Module](https://blog.logrocket.com/css-vs-css-in-js/#cons-css-module)
9. [Recommendations for where to use CSS Module](https://blog.logrocket.com/css-vs-css-in-js/#recommendations-where-use-css-module)
10. [Modern CSS features to watch](https://blog.logrocket.com/css-vs-css-in-js/#modern-css-features-watch)

이 글에서 제시하는 모든 코드 예시와 데모는 React와 CSS를 사용합니다.

따라서 계속 진행하기 전에 이 두 기술에 익숙해지시기 바랍니다.

모든 JavaScript 프론트엔드 프레임워크나 라이브러리는 CSS-in-JS의 아이디어를 구현할 수 있습니다.

이 글에서는 CSS-in-JS의 적용과 그 장단점을 논의하기 위해 가장 인기 있는 JavaScript 프론트엔드 라이브러리인 React를 사용합니다.

## - 렌더링 차단 및 CSS

본격적으로 어떤 것이 좋고 좋지 않은지 논하기 전에, CSS로 인해 발생하는 렌더링 문제에 대해 조금 이야기해보겠습니다.

전통적으로, 브라우저는 먼저 HTML을 로드하고, 그 다음에 모든 외부 리소스로부터 CSS를 로드합니다.

이후 브라우저는 외부 및 내부 CSS 정보를 사용하여 CSSOM(CSS Object Model)을 생성합니다.

이제 브라우저는 CSS 계층 규칙에 따라 렌더된 HTML에 스타일을 적용할 준비가 됩니다.

이 과정은 CSS가 [페이지 렌더링을 차단](https://blog.logrocket.com/11-best-practices-eliminate-render-blocking-resources/)하고 요청된 페이지의 첫 페인트(first paint)를 지연시킵니다.

첫 페인트는 브라우저가 요청된 페이지의 첫 번째 픽셀을 화면에 그리는 이벤트입니다.

첫 페인트가 0.5초 이상 지연되면 사용자 불만족의 위험이 커지고, 이는 애플리케이션의 목표에 부정적인 영향을 미칠 수 있습니다.

CSS를 클라이언트에게 더 빨리 제공할 수록 페이지의 첫 페인트 시간을 최적화할 수 있습니다.

## - 렌더링 차단에 대응하는 법

HTTP/2를 사용하는 애플리케이션에서는 여러 HTML, CSS, 및 JS 파일을 병렬로 로드할 수 있습니다.

이러한 기능은 HTTP/1.1에서는 제한적이었습니다.

대부분의 최신 브라우저와 웹사이트는 이제 HTTP/2를 지원하며, 이는 다른 파일을 로드하기 위해 기다리는 동안 발생하는 렌더링 차단을 최소화합니다.

![](https://blog.logrocket.com/wp-content/uploads/2022/11/img1-Graphic-showing-difference-file-loading-HTTP1-HTTP2.png)

그러나 렌더링 차단에는 파일 로딩 속도 외에도 다른 요인이 있습니다.

예를 들어, 우리 애플리케이션의 한 페이지에 많은 CSS가 있다고 가정해 봅시다.

페이지마다 하나의 마스터 CSS 파일을 가져오기 때문에 사용되지 않는 셀렉터들도 포함될 수 있습니다.

위의 시나리오는 기본적으로 우리가 CSS UI 프레임워크나 빠르게 디자인 시스템을 구축하기 위해 만든 UI 라이브러리를 직접 소비하는 방식에 익숙해져 있는 상황을 설명합니다.

그 프레임워크나 라이브러리에서 참조된 모든 스타일이 모든 페이지에서 사용되는 것은 아닙니다.

결과적으로, 최종 페이지의 CSS 스타일에는 더 많은 불필요한 코드가 포함됩니다.

CSS가 많을수록 브라우저가 CSSOM을 구성하는 데 시간이 더 오래 걸리며, 이는 완전히 불필요한 렌더링 차단을 초래합니다.

이를 해결하기 위해 CSS를 작은 청크로 분할하는 것이 매우 유용합니다.

다시 말해, 전역 스타일과 중요한 CSS는 하나의 범용 CSS 파일에 유지하고, 나머지는 모두 컴포넌트화합니다.

이 전략은 훨씬 더 합리적이며 불필요한 차단 문제를 해결합니다.

![](https://blog.logrocket.com/wp-content/uploads/2022/11/img2-Project-structure-componentized-CSS.png)

위 그림은 React에서 다른 컴포넌트에 대해 개별 CSS 파일을 생성하고 관리하는 전통적인 방법을 보여줍니다.

각 CSS 파일이 해당 컴포넌트에 직접 연결되어 있어, 관련 컴포넌트가 임포트될 때만 로드되고 해당 컴포넌트가 제거되면 사라집니다.

이 방법에는 한 가지 단점이 있습니다.

예를 들어, 우리의 애플리케이션에 100개의 컴포넌트가 있고, 같은 프로젝트를 작업하는 다른 개발자들이 일부 CSS 파일에서 같은 클래스 이름을 실수로 사용했다고 가정해봅시다.

여기서 각 컴포넌트의 CSS 파일의 범위는 글로벌이기 때문에, 이러한 실수로 중복된 스타일은 서로를 계속 덮어쓰고 전역으로 적용됩니다.

이러한 상황은 심각한 레이아웃 및 디자인 불일치를 초래할 수 있습니다.

CSS-in-JS는 이 스코핑 문제를 해결한다고 알려져 있습니다.

다음 세그먼트에서는 CSS-in-JS를 높은 수준에서 검토하고, 그것이 이 스코핑 문제를 한 번에 효과적으로 해결하는지 여부를 논의합니다.

## - CSS-in-JS는 무엇을 제안하는가

간단히 말해, CSS-in-JS는 JavaScript를 통해 컴포넌트에 대한 CSS 속성을 작성할 수 있게 해주는 외부 기능 계층입니다.

이 모든 것은 2015년에 [JSS라는 JavaScript 라이브러리](https://cssinjs.org/)에서 시작되었습니다. 이 라이브러리는 여전히 활발하게 유지보수되고 있습니다. JavaScript 문법을 사용하여 셀렉터에 CSS 속성을 제공하면, 페이지가 로드될 때 자동으로 해당 셀렉터에 속성을 적용합니다.

React와 같은 라이브러리를 통해 JavaScript가 프론트엔드 렌더링 및 관리를 장악하면서, [styled-components](https://blog.logrocket.com/how-style-react-router-links-styled-components/)라는 CSS-in-JS 솔루션이 등장했습니다. 또 다른 점점 인기를 얻고 있는 방법은 [Emotion 라이브러리](https://blog.logrocket.com/styled-components-vs-emotion-for-handling-css/)를 사용하는 것입니다.

CSS-in-JS의 예제 사용 사례를 가장 인기 있는 방식인 스타일드 컴포넌트 라이브러리를 사용하여 시연할 것입니다.

## - 스타일드 컴포넌트를 사용한 CSS-in-JS의 사용 예시

React 애플리케이션에서 아래 Yarn 명령어를 사용하여 styled-components 라이브러리를 설치하세요. [다른 패키지 매니저](https://blog.logrocket.com/javascript-package-managers-compared/)를 사용하는 경우, [styled-components 설치 문서](https://styled-components.com/docs/basics#installation)를 참조하여 적절한 설치 명령어를 찾으세요:

```
yarn add styled-components
```

styled-components 라이브러리를 설치한 후, styled 함수를 임포트하여 아래 코드와 같이 사용하세요:

```ts
import styled from "styled-components";

const StyledButton = styled.a`
  padding: 0.75em 1em;
  background-color: ${({ primary }) => (primary ? "#07c" : "#333")};
  color: white;

  &:hover {
    background-color: #111;
  }
`;

export default StyledButton;
```

React 환경에 접근할 수 없는 경우, 위 코드가 작동하는 모습을 볼 수 있도록 [CodePen 데모](https://codepen.io/_rahul/pen/oNywWXR)를 제공해드립니다:

> 원글에 코드펜이 아예 삽입되어있는데, md파일에 코드펜을 삽입하는 법을 몰라서 일단 링크만 넣어놓았습니다.. 🥲

위 코드는 React에서 버튼-링크 컴포넌트를 스타일링하는 방법을 보여줍니다.

이제 이 스타일된 컴포넌트는 어디서든 임포트하여 직접 사용하여 스타일에 대해 걱정하지 않고 기능적인 컴포넌트를 구축할 수 있습니다.

```ts
import StyledButton from './components/styles/Button.styled';

function App() {
  return (
    <div className="App">
      ...
      <StyledButton href="...">Default Call-to-action</StyledButton>
      <StyledButton primary href="...">Primary Call-to-action</StyledButton>
    </div>
  );
}

export default App;
```

스타일된 컴포넌트에 적용된 스타일은 로컬 범위로 한정되어 있어 CSS 클래스 명명과 글로벌 범위에 대해 신경 쓸 필요가 없어집니다.

또한, 컴포넌트에 제공된 props나 애플리케이션 기능이 요구하는 다른 논리에 따라 CSS를 동적으로 추가하거나 제거할 수 있습니다.

## - CSS-in-JS의 장점

JavaScript 개발자는 CSS 클래스를 사용하는 대신 CSS-in-JS로 스타일을 정의하는 것을 선호할 수 있습니다.

CSS-in-JS 접근 방식이 해결하는 가장 큰 문제는 글로벌 스코프 문제입니다.

또한, JavaScript 개발자에게 매우 유용한 몇 가지 다른 장점도 있습니다.

이제 이러한 장점 중 일부를 살펴보겠습니다.

## - 스코핑 및 우선순위 문제 없음

스타일이 로컬 범위 내에서 사용되므로 다른 컴포넌트의 스타일과 충돌할 가능성이 없습니다.

스타일 충돌을 피하기 위해 엄격하게 이름을 정할 필요도 없습니다.

스타일은 자식 셀렉터를 앞에 붙이지 않고 한 컴포넌트에만 독점적으로 작성되므로, 우선순위 문제도 거의 발생하지 않습니다.

## - 동적 스타일링

조건부 CSS는 CSS-in-JS의 또 다른 중요한 특징입니다.

위의 버튼 예제에서 알 수 있듯이, prop 값을 확인하고 적절한 스타일을 추가하는 것이 각 변형마다 별도의 CSS 스타일을 작성하는 것보다 훨씬 더 효율적입니다.

## CSS 우선순위 감소

CSS-in-JS는 CSS 선언의 우선순위를 최소화하는 데 도움이 됩니다.

스타일링하는 유일한 대상이 요소 자체이기 때문입니다. 동일한 원칙이 컴포넌트 변형을 만들 때도 적용됩니다.

prop 객체 값을 확인하고 필요할 때 동적 스타일을 추가할 수 있습니다.

## 쉬운 테마 적용

[사용자 정의 CSS 속성을 사용하여 애플리케이션에 테마를 적용하는 것](https://blog.logrocket.com/a-guide-to-theming-in-css/)은 합리적입니다.

결국, 사용자 입력에 따라 테마를 전환하고 기억하는 로직을 작성하려면 JavaScript 쪽으로 이동해야 합니다.

CSS-in-JS를 사용하면 테마 로직을 완전히 JavaScript로 작성할 수 있습니다.

styled-components의 ThemeProvider 래퍼를 사용하면 컴포넌트에 대한 테마를 빠르게 색상 코드로 적용할 수 있습니다.

styled-components를 사용한 컴포넌트 테마 적용을 직접 확인하려면 이 [CodePen 예제](https://codepen.io/_rahul/pen/qBKXevo)를 참조하세요:

> 이것도 코드펜 첨부 안되면 그냥 링크만 삽입하겠습니다~

## 간편한 유지보수

CSS-in-JS가 제공하는 기능과 장점을 고려하면, JavaScript 개발자는 수백 개의 CSS 파일을 관리하는 것보다 CSS-in-JS를 사용하는 것이 더 편리할 수 있습니다.

그러나 여전히 CSS-in-JS로 구동되는 대규모 프로젝트를 효과적으로 관리하고 유지하려면 JavaScript와 CSS 모두에 대한 충분한 이해가 필요하다는 사실은 변함없습니다.

## CSS-in-JS의 단점

CSS-in-JS는 스코핑 문제를 매우 잘 해결합니다.

그러나 초기 논의에서 언급했듯이, 사용자 경험에 직접적으로 영향을 미치는 렌더링 차단과 같은 더 큰 도전 과제가 있습니다.

이와 함께, CSS-in-JS 개념이 여전히 해결해야 할 몇 가지 다른 문제들도 있습니다.

## 렌더링 지연

CSS-in-JS는 JavaScript를 실행하여 JavaScript 컴포넌트에서 CSS를 구문 분석한 다음, 이 구문 분석된 스타일을 DOM에 삽입합니다.

컴포넌트가 많을수록 브라우저가 첫 페인트(first paint)를 수행하는 데 더 많은 시간이 걸립니다

## 캐싱 문제

CSS 캐싱은 연속적인 페이지 로드 시간을 개선하는 데 자주 사용됩니다.

CSS-in-JS를 사용할 때는 CSS 파일이 포함되지 않기 때문에 캐싱이 큰 문제로 작용합니다.

또한, 동적으로 생성된 CSS 클래스 이름은 이 문제를 더욱 복잡하게 만듭니다.

## CSS 전처리기 지원 부족

일반적인 컴포넌트화된 CSS 접근 방식에서는 SASS, Less, PostCSS 등과 같은 전처리기를 쉽게 추가할 수 있습니다.

하지만 CSS-in-JS에서는 동일한 지원이 불가능합니다.

## 지저분한 DOM

CSS-in-JS는 모든 스타일 정의를 JavaScript에서 일반 CSS로 구문 분석한 다음, 스타일 블록을 사용하여 DOM에 삽입하는 아이디어를 기반으로 합니다.

CSS-in-JS로 스타일링된 각 컴포넌트마다 100개의 스타일 블록이 구문 분석되고 삽입될 수 있습니다.

간단히 말해, 추가적인 오버헤드 비용이 발생할 것입니다.

## 라이브러리 의존성

CSS-in-JS 기능을 외부 라이브러리로 추가할 수 있습니다.

그러나 JavaScript에서 CSS 스타일로 구문 분석하기 위해 styled-components와 같은 라이브러리에 의존하므로, 실제 CSS 구문 분석 전에 많은 JavaScript가 포함되고 실행됩니다.

## 학습 곡선

CSS-in-JS에는 많은 기본 CSS 및 SCSS 기능이 부족합니다.

CSS와 SCSS에 익숙한 개발자들이 CSS-in-JS에 적응하는 것은 매우 어려울 수 있습니다.

## 광범위한 지원 부족

대부분의 UI 및 컴포넌트 라이브러리는 현재 CSS-in-JS 접근 방식을 지원하지 않습니다.

이는 여전히 해결해야 할 많은 문제가 있기 때문입니다.

위에서 논의한 문제들은 종합적으로 저성능, 유지보수 어려움, 여러 UI 및 UX 불일치로 이어질 수 있습니다.

## CSS-in-JS 사용 추천 사항

CSS-in-JS 솔루션은 성능이 낮은 우선순위를 가지는 작은 애플리케이션을 다룰 때 이상적입니다.

성능이 중요한 대규모 디자인 시스템을 가진 애플리케이션에서는 이상적이지 않을 수 있습니다.

애플리케이션이 커질수록, CSS-in-JS의 모든 단점을 고려할 때 복잡해질 가능성이 큽니다.

디자인 시스템을 CSS-in-JS로 변환하는 데 많은 작업이 필요하며, 제 생각에는 어떤 JavaScript 개발자도 그것을 다루고 싶어하지 않을 것입니다.

## CSS Module 에 관하여

CSS Module은 렌더된 CSS에서 모든 속성이 기본적으로 로컬 범위로 지정되는 [CSS 파일](https://blog.logrocket.com/a-deep-dive-into-css-modules/)입니다.

JavaScript는 CSS Module 파일을 추가로 처리하여 스타일 선언을 캡슐화하고 스코핑 문제를 해결합니다.

CSS Module을 사용하려면 CSS 파일 이름을 .module.css 확장자로 지정하고 이를 JavaScript 파일에 임포트해야 합니다. 아래 코드 예시는 CSS Module을 사용하는 기본 예제를 제공합니다:

```ts
import styles from './Button.module.css';

export default function Button(props) {
  return (
    <a
      href={props.href ? props.href : '#'}
      className={styles.btn}
    >
      {props.name}
    </a>
  );
}
```

이 [StackBlitz 예제](https://stackblitz.com/edit/react-hbivvp?file=src%2Fcomponents%2FButton%2FButton.module.css)를 참고하여 React에서 CSS Modules을 구현하는 방법을 확인해보세요.

이 예제는 CSS Modules을 사용하여 스코핑 문제를 해결하는 방법을 보여줍니다.

StackBlitz 예제에서 Button.module.css와 AnotherButton.module.css에서 동일한 클래스 이름이 처리되고 최적화되어 명명 충돌을 방지하는 방법을 확인해보세요.

## CSS Module의 장점

CSS Module이 제공하는 가장 큰 이점은 스코핑과 우선순위 문제를 해결하기 위해 CSS-in-JS에 의존할 필요를 없애준다는 것입니다.

CSS를 가능한 전통적인 방식으로 유지하면서 스코핑과 우선순위 문제를 해결할 수 있다면, CSS-in-JS는 불필요한 작업이 될 것입니다.

## 스코핑 및 우선순위 문제 없음

위의 예제에서 보듯이, CSS Module은 전통적인 CSS에서 발생하는 스코핑 문제를 성공적으로 해결합니다.

CSS Module 파일에 규칙이 느슨하게 작성되어 있기 때문에 우선순위 문제가 발생하는 경우도 드뭅니다.

## 조직된 코드

별도의 CSS 파일을 유지하는 것은 제한처럼 보일 수 있습니다.

그러나 이 방법은 실제로 더 나은 구조를 구성합니다.

예를 들어, 저는 컴포넌트를 각각의 폴더로 분리하여 다음과 같이 구성합니다:

```
-Project -
  src -
  components -
  Button -
  Button.jsx -
  Button.modules.css -
  Carousel -
  Carousel.jsx -
  Carousel.modules.css;
```

## 캐싱 가능성

최종 빌드와 함께 생성된 최소화된 CSS 파일은 브라우저에 의해 캐싱되어 연속적인 페이지 로드 시간을 개선할 수 있습니다.

## CSS 전처리

PostCSS, SASS, Less 등과 같은 CSS 전처리기를 추가로 지원하기 쉽습니다.

그러나 이를 위해서는 추가 패키지에 의존해야 합니다.

## 학습 곡선 없음

CSS가 어떻게 작동하는지 알고 있다면, 앞서 소개한 몇 가지 사항을 제외하고는 새로운 것을 배우지 않고도 CSS Module을 사용할 수 있습니다.

## 뛰어난 지원

CSS Modules를 사용하기 위해 추가 패키지를 설치할 필요가 없습니다.

모든 주요 프레임워크와 라이브러리는 기본적으로 CSS Modules를 지원합니다.

## CSS Module의 단점

CSS Module은 많은 이점을 제공하지만 완벽한 솔루션은 아닙니다.

아래는 고려해야 할 몇 가지 사항입니다.

## 비표준속성

글로벌 범위의 셀렉터를 대상으로 할 때는
규칙을 사용해야 합니다.

이는 CSS 사양의 일부가 아니며, 글로벌 스타일을 표시하기 위해 JavaScript에서 사용됩니다.

## 동적 스타일 불가능

CSS Module을 사용하면 모든 선언이 별도의 CSS 파일로 들어갑니다.

따라서 CSS 파일에서 JavaScript를 구현할 수 없으므로 CSS-in-JS처럼 동적 스타일을 구현하는 것이 불가능합니다.

## 외부 CSS 파일

CSS Module을 사용하면 컴포넌트에서 CSS 파일 사용을 생략할 수 없습니다.

CSS Module을 사용하는 유일한 방법은 외부 CSS 파일을 유지하고 임포트하는 것입니다.

## TypeScript 제한 사항

TypeScript에서 CSS Modules을 사용하려면, index.d.ts 파일에 모듈 정의를 추가하거나 웹팩 로더를 사용해야 합니다:

```ts
/** index.d.ts **/
declare module "*.module.css"; // CSS Module 파일용 TS 모듈
declare module "*.module.scss"; // SCSS 형식의 CSS Module 파일용 TS 모듈
```

## CSS Module 추천 경우

CSS Module을 사용하는 것은 대규모 UI를 갖춘 성능이 중요한 애플리케이션에 좋은 선택입니다.

CSS Module이 제공하는 모든 것이 궁극적으로 전통적이고 실험적이지 않은 사용법을 기반으로 하기 때문에, 이 방법은 성능을 모니터링하고 수정하기 더 쉽습니다.

CSS Module 파일은 CSS만 다루기 때문에, 원하는 CSS 프레임워크에서 코드를 간단하게 적용할 수 있습니다.

앞서 논의했듯이, 기본적인 CSS 지식만 있으면 이 작업을 수행하기에 충분합니다.

## 주목할 현대 CSS 기능

서론에서, CSS Module, CSS-in-JS 또는 다른 JavaScript 솔루션에 의존하지 않고도 스코핑 문제를 해결할 수 있는 몇 가지 현대적인 CSS 기능이 미래에 도움이 될 수 있다고 언급했습니다.

새로운 및 계획된 기능들 - 예를 들어, 스코핑 지시자 및 @scope 가상 요소와 같은 것들은 전통적인 CSS의 오래된 문제들을 해결하는 것을 목표로 합니다.

이는 개발자들이 이러한 문제에 대한 해결책으로 CSS-in-JS와 같은 방법에 의존할 필요를 줄일 수 있습니다.

현재 초안에 있는 스코프된 CSS가 CSS-in-JS와 심지어 CSS Module의 문제를 어떻게 해결할 수 있는지 살펴보겠습니다.

다른 현대 CSS 기능의 전체 목록은 [State of CSS 2022](https://web.dev/blog/state-of-css-2022?hl=ko)를 참조하세요.

## CSS 스코핑의 잠재적 미래

[<style scope>](https://github.com/whatwg/html/issues/552)가 CSS 사양에서 이상하게 도입되고 제거된 후, [현재의 스코프된 CSS 초안](https://drafts.csswg.org/css-cascade-6/#scoped-styles)은 CSS 규칙을 작성하여 요소의 스코핑 전제를 정의하기에 충분히 좋아 보입니다.

현재 상태는 주어진 요소에 대해 스코핑을 제공하기 위해 지시자와 가상 클래스를 사용하는 것을 포함합니다.

아래는 요소의 스코프를 경계 내에 잠그고 캐스케이드 규칙에 상관없이 유지하는 방법에 대한 대략적인 그림입니다:

```js
<div class="card">
  <img src="...">
  <div class="content">
    <p>...</p>
  </div><!-- .content -->
</div><!-- .card -->

<style>
@scope (.card) {
  :scope {
    display: grid;
    ...
  }
  img {
    object-fit: cover;
    ...
  }
  .content { ... }
}
</style>
```

이 새로운 기능은 스코핑 문제를 해결하기 위해 CSS Module이나 CSS-in-JS의 필요성을 제거할 수 있습니다.

이 기능이 브라우저에서 사용 가능해질 때까지 기다려야 합니다.

## 결론

위에서 우리는 CSS 렌더링 차단이 웹 애플리케이션의 주요 성능 문제일 수 있다는 점을 논의했습니다.

그런 다음 이 문제를 해결하기 위한 몇 가지 솔루션을 논의하면서 CSS-in-JS, CSS Module, 그리고 진행 중인 새로운 스코프된 CSS 기능의 공식 초안 상태를 탐구했습니다.

JavaScript를 좋아하는 개발자들은 CSS-in-JS를 선호하는데, 이는 JavaScript로 거의 모든 스타일링 측면을 다룰 수 있기 때문입니다.

반면에 CSS를 좋아하고 현재 기술이 개발자와 최종 사용자 모두를 지원하기를 원하는 사람들은 CSS Module을 선호할 수 있습니다.

이 기사를 즐겁게 읽으셨기를 바랍니다. 댓글로 여러분의 생각, 질문, 제안을 알려주세요.
