# [Deep Dive into React Concurrent Mode: Exploring Key Features and Use Cases](https://www.dhiwise.com/post/deep-dive-into-react-concurrent-mode-exploring-key-features-and-use-cases)

### 🗓️ 번역 날짜: 2024.08.04

### 🧚 번역한 크루: 버건디(전태헌)

---

안녕하세요, React 열성 팬 여러분!

React Concurrent Mode의 미스터리를 풀어내는 짜릿한 여정을 준비하세요. React 18에서 도입된 이 강력한 새로운 기능은 우리가 사용자 인터페이스를 구축하는 방식을 혁신할 것입니다. 마치 React 앱이 메인 스레드를 차단하지 않고 여러 작업을 동시에 수행할 수 있는 새로운 차원으로 들어가는 것과 같습니다.

무거운 계산이나 네트워크 요청 중에도 앱이 응답성을 유지할 수 있는 세계를 상상해 보세요. 사용자가 상호작용할 때 UI가 멈추지 않는 세계를 말이죠. 이것이 Concurrent 기능의 약속입니다. 이것은 단순한 새로운 기능이 아니라 React에 대한 새로운 사고방식입니다.

그럼, 안전벨트를 매고 React의 동시성 세계로 뛰어들 준비를 합시다!

```tsx
import { unstable_createRoot } from "react-dom";

const root = unstable_createRoot(document.getElementById("root"));

root.render(<App />);
```

이 코드 스니펫에서 우리는 react-dom의 unstable_createRoot 함수를 사용하여 루트 노드를 생성하고 있습니다. 이 함수는 전체 앱에 대해 동시 렌더링을 가능하게 합니다.

동시 렌더링의 아름다움은 React가 메모리에서 UI의 여러 버전을 준비한 다음 사용자 상호작용 및 네트워크 조건에 따라 빠르게 전환할 수 있게 한다는 것입니다. 이는 무거운 계산이나 네트워크 요청이 있는 복잡한 앱에서도 더 응답성이 좋고 유동적인 사용자 경험을 제공합니다.

그러나 동시 렌더링이 만병통치약은 아닙니다. 이는 새로운 수준의 복잡성을 도입하며 앱이 업데이트되고 렌더링되는 방식을 다르게 생각해야 합니다. 그러나 이를 익히게 되면, React 앱이 한 단계 더 발전할 수 있는 방법을 알게 될 것입니다.

## 리액트 16에 비해서 동시성 모드는 어떻게 다른가

React 18의 등장과 함께 React 팀은 이전 버전인 React 16 및 이전 버전에서 사용된 동기 렌더러와 근본적으로 다른 새로운 동시 렌더러를 도입했습니다. 그렇다면 정확히 무엇이 변경되었을까요?

React 16에서는 렌더링이 항상 동기적이었습니다. 이는 React가 컴포넌트 트리의 렌더링을 시작하면 전체 트리가 렌더링될 때까지 중지하지 않는다는 것을 의미합니다. 이는 큰 렌더링 작업이 메인 스레드를 차단하여 UI가 멈추고 응답하지 않게 되는 문제를 일으킬 수 있었습니다.

반면, React 18은 동시 렌더링을 도입합니다. 이 새로운 모드는 React가 렌더링 프로세스를 중단하고 사용자 입력과 같은 더 긴급한 작업을 처리할 수 있게 합니다. 이는 마치 React가 동시에 여러 곳에 있을 수 있는 초능력을 가진 것과 같습니다.

```ts
// React 16
 ReactDOM.render(<App />, document.getElementById('root'));

// React 18
 const root = ReactDOM.createRoot(document.getElementById('root'));
 root.render(<App />)
```

React 16에서는 ReactDOM.render를 사용하여 앱을 렌더링합니다. 그러나 React 18에서는 ReactDOM.createRoot를 사용하여 루트 노드를 생성한 후 해당 루트 노드에서 render를 호출합니다. 이를 통해 앱 전체에서 동시 렌더링이 가능해집니다.

하지만 차이점은 단순히 앱을 렌더링하는 방식에 그치지 않습니다. 동시 모드를 통해 React 18은 여러 상태 업데이트의 자동 일괄 처리, 스트리밍 서버 렌더링, 그리고 컴포넌트의 부드러운 로딩 상태를 만들 수 있는 React Suspense와 같은 새로운 기능들을 도입합니다.

본질적으로, React 18의 동시 모드는 React에서 렌더링에 대한 사고방식을 변화시키는 패러다임의 전환입니다. 이는 단순히 앱을 더 빠르게 만드는 것뿐만 아니라 더 응답성이 좋고 사용자 친화적으로 만드는 것을 목표로 합니다.

## 리액트 18에서의 동시성 기능들 : 깊이 파헤치기

React 18에는 앱을 더 응답성 있고 사용자 친화적으로 만들기 위한 다양한 동시 기능이 포함되어 있습니다. 이러한 기능 중 일부를 깊이 살펴보고 어떻게 React 앱을 향상시킬 수 있는지 알아보겠습니다.

1. 자동 배칭:

이전 버전의 React에서는 React 이벤트 외부에서 트리거된 상태 업데이트가 함께 배칭되지 않았습니다. 이는 불필요한 재렌더링과 비효율적인 앱을 초래할 수 있었습니다. React 18은 여러 상태 업데이트를 함께 배칭하여 단일 재렌더링을 수행하는 자동 배칭을 도입합니다.

다음은 예제입니다:

```tsx
const [count, setCount] = useState(0);

// In React 16, these would cause two re-renders
setCount(count + 1);
setCount(count + 1);

// In React 18, these are batched together and cause a single re-render
```

2. 스트리밍 서버 렌더링:

React 18은 서버에서 클라이언트로 HTML을 스트리밍할 수 있는 새로운 서버 렌더링 API를 도입합니다. 이는 브라우저가 전체 응답을 기다리지 않고 첫 번째 데이터 청크를 받자마자 HTML 렌더링을 시작할 수 있음을 의미합니다. 이를 통해 앱의 인식된 로드 시간을 크게 개선할 수 있습니다.

3. React Suspense:

React Suspense는 컴포넌트에 대해 부드러운 로딩 상태를 만들 수 있는 React 18의 새로운 기능입니다. Suspense를 사용하면 일부 코드가 로드될 때까지 "대기"하고 선언적으로 로딩 상태를 지정할 수 있습니다. 이는 코드 분할 또는 데이터 페칭을 처리할 때 특히 유용할 수 있습니다.

```tsx
<Suspense fallback={<div>Loading...</div>}>
  <MyComponent />
</Suspense>
```

이 예제에서 컴포넌트는 MyComponent를 감싸고 있습니다. MyComponent가 아직 준비되지 않은 경우(예: 데이터를 페칭하거나 코드 분할 청크를 로드하는 중일 때), React는 fallback 속성으로 지정된 내용을 표시합니다. 이 경우 간단한 "Loading..." div를 표시합니다.

이것은 React 18의 동시 기능 중 일부에 불과합니다. 또한 startTransition 기능이 있어 특정 업데이트를 "전환"으로 표시하고 우선순위를 낮추어 처리할 수 있으며, useDeferredValue 기능을 사용하면 느린 상태 업데이트를 빠른 업데이트가 완료될 때까지 연기할 수 있습니다.

이러한 동시 기능들을 함께 사용하면 React는 복잡한 렌더링 작업을 더 효율적으로 처리하고 더 부드럽고 응답성이 뛰어난 사용자 경험을 제공할 수 있습니다.

## 동시성 모드가 가능하도록하기 : 차근치근 해보기

이제 동시 렌더링과 그 기능에 대해 알아보았으니, 실제로 React 앱에서 동시 모드를 활성화하는 방법을 살펴보겠습니다.

단계 1: React 18로 업그레이드

첫 번째 단계는 앱을 React 18로 업그레이드하는 것입니다. 이를 위해 터미널에서 다음 명령어를 실행하면 됩니다:

이 명령어를 실행하면 React와 ReactDOM 패키지가 최신 버전으로 업데이트됩니다.

```
npm install react@18 react-dom@18
```

단계 2: Strict Mode 활성화

다음으로, 앱에서 Strict Mode를 활성화해야 합니다. 이는 동시 모드에서 문제가 될 수 있는 잠재적인 이슈를 잡아내는 데 도움이 됩니다. 이를 위해 앱의 루트 컴포넌트를 StrictMode로 감싸면 됩니다.

```tsx
import React, { StrictMode } from "react";

<StrictMode>
  <App />
</StrictMode>;
```

단계 3: 동시 모드 활성화

이제 동시 모드를 활성화할 차례입니다. ReactDOM.render를 사용하는 대신 ReactDOM.createRoot를 사용합니다.

```tsx
import { createRoot } from "react-dom";

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

그리고 보세요! 이제 React 앱에서 동시 모드를 활성화했습니다. 이제 자동 배칭, 스트리밍 서버 렌더링 및 React Suspense와 같은 React 18의 동시 기능을 최대한 활용할 수 있습니다.

기억하세요, 동시 모드는 강력한 도구이지만 큰 힘에는 큰 책임이 따릅니다. 이는 새로운 수준의 복잡성을 도입하며 앱의 업데이트와 렌더링 방식을 다르게 생각해야 합니다. 그러나 이를 익히게 되면, React 앱이 한 단계 더 발전할 수 있는 방법을 알게 될 것입니다.

## 동시 React: 실제 예제

이제 동시 모드의 이론과 설정에 대해 다루었으니, 실제로 동시 React가 사용자 경험을 어떻게 향상시킬 수 있는지 몇 가지 예제를 살펴보겠습니다.

### 예제 1: 사용자 입력 우선 처리

동시 모드의 주요 이점 중 하나는 특정 작업을 다른 작업보다 우선 처리할 수 있다는 것입니다. 예를 들어, 앱에 검색창이 있다고 가정해봅시다. 사용자가 검색창에 입력하면 앱은 사용자의 입력을 기반으로 항목 목록을 필터링합니다.

전통적인 동기 렌더링 모델에서는 항목 목록이 크다면, 목록을 필터링하는 작업이 메인 스레드를 차단하여 입력이 사용자의 타이핑 속도에 뒤처질 수 있습니다. 이는 사용자를 불편하게 만들 수 있습니다.

동시 모드를 사용하면, React는 필터링 프로세스를 중단하고 사용자 입력을 처리하여 입력이 항상 응답성을 유지하도록 할 수 있습니다.

### 예제 2: Suspense를 통한 부드러운 로딩 상태

동시 모드의 또 다른 강력한 기능은 React Suspense입니다. 이를 통해 컴포넌트의 부드러운 로딩 상태를 만들 수 있습니다.

예를 들어, API에서 데이터를 가져오는 컴포넌트가 있다고 가정해봅시다. 전통적인 모델에서는 로딩 상태를 수동으로 관리해야 하며, 제대로 관리되지 않으면 사용자 경험이 불편해질 수 있습니다.

Suspense를 사용하면 선언적으로 로딩 상태를 지정하고 나머지는 React가 처리하도록 할 수 있습니다. 이렇게 하면 React가 데이터를 로드하는 동안 "기다리고" 그 사이에 로딩 상태를 보여줌으로써 훨씬 더 부드러운 사용자 경험을 제공할 수 있습니다.

```tsx
<Suspense fallback={<div>Loading...</div>}>
  <DataFetchingComponent />
</Suspense>
```

이 예제에서 DataFetchingComponent가 여전히 데이터를 가져오는 중이라면, React는 fallback 속성을 표시합니다. 이 경우 간단한 "Loading..." div를 보여줍니다.

이것은 동시 React가 사용자 경험을 향상시키는 방법에 대한 몇 가지 예에 불과합니다. 가능성은 무궁무진하며, 이러한 강력한 새로운 기능으로 여러분이 무엇을 만들지 기대됩니다!

## React 18: 손쉽게 동시성 처리

React 18의 동시 모드는 애플리케이션에서 동시성을 처리하는 방식에 큰 변화를 가져옵니다.
이는 새로운 동시 렌더러를 도입하여 메인 스레드를 차단하지 않고 동시에 여러 작업을 수행할 수 있게 합니다. 이는 한 번에 하나의 작업만 처리할 수 있었던 이전의 동기 렌더러에 비해 상당한 개선입니다.

하지만 React는 동시성을 어떻게 처리할까요? 그 비밀은 React가 업데이트를 스케줄링하는 방식에 있습니다. 동시 모드에서는 React가 진행 중인 렌더링 프로세스를 중단하고 사용자 입력과 같은 더 긴급한 작업을 처리할 수 있습니다. 이를 통해 앱이 무거운 계산이나 네트워크 요청 중에도 항상 응답성을 유지할 수 있습니다.

이것을 설명하기 위한 예제는 다음과 같습니다:

```tsx
import { startTransition } from "react";

function MyComponent() {
  const [state, setState] = useState(initialState);

  function handleClick() {
    startTransition(() => {
      setState(newState);
    });
  }

  // ...
}
```

이 예제에서는 startTransition 함수를 사용하여 상태 업데이트를 "전환"으로 표시하고 있습니다. 이는 React에게 이 업데이트가 긴급하지 않으며, 더 긴급한 작업이 있을 경우 중단될 수 있음을 알려줍니다.

작업의 우선순위를 지정할 수 있는 이 기능이 동시 모드를 매우 강력하게 만듭니다. 이를 통해 React는 복잡한 렌더링 작업을 더 효율적으로 처리하고 더 부드럽고 응답성이 좋은 사용자 경험을 제공합니다.

그러나 큰 힘에는 큰 책임이 따릅니다. 동시 모드는 새로운 수준의 복잡성을 도입하며, 앱이 업데이트되고 렌더링되는 방식을 다르게 생각해야 합니다. 따라서 동시 모드를 사용하기 전에 개념을 철저히 이해하는 것이 중요합니다.

## React 18: 더블 렌더 미스터리 설명

React 18의 동시 모드에서 가장 흥미로운 측면 중 하나는 소위 "더블 렌더" 현상입니다. React 18을 실험해보셨다면 일부 컴포넌트가 한 번만 렌더링되어야 하는데 두 번 렌더링되는 것처럼 보이는 것을 발견했을지도 모릅니다. 도대체 무슨 일이 벌어지고 있는 걸까요?

더블 렌더는 사실 버그가 아니라 기능입니다. 이는 새로운 동시 렌더러의 작동 방식에서 비롯됩니다. 동시 모드에서는 React가 DOM을 업데이트하기 전에 메모리에서 새로운 화면(또는 업데이트된 컴포넌트 상태)을 준비합니다. 이를 "메모리 내 렌더링" 또는 "오프스크린 렌더링"이라고 합니다.

이 단계에서 React는 일부 컴포넌트를 사용자에게 보이지 않더라도 한 번 이상 렌더링할 수 있습니다. 이것이 "더블 렌더"를 초래하는 원인입니다. 그러나 이러한 추가 렌더링은 일반적으로 빠르게 이루어지며 메인 스레드를 차단하지 않으므로 앱의 성능에 영향을 미치지 않습니다.

이를 설명하기 위한 예제는 다음과 같습니다:

```tsx
function MyComponent() {
  const [state, setState] = useState(initialState);

  console.log("Render");

  // ...
}
```

이 예제에서 이 컴포넌트를 동시 모드에서 실행하면, 컴포넌트가 한 번만 업데이트되더라도 콘솔에 "Render"가 두 번 기록되는 것을 볼 수 있습니다. 이것이 바로 "더블 렌더" 현상입니다.

따라서 동시 모드에서 컴포넌트가 두 번 렌더링되는 것을 보더라도 놀라지 마세요. 이는 React가 새로운 화면을 메모리에서 준비하여 더 부드러운 사용자 경험을 제공하기 위한 과정일 뿐입니다.

## React Experimental: 무엇에 관한 것일까?

React 개발을 주시하고 있다면 "React Experimental"이라는 용어를 접했을 것입니다. 하지만 이것이 무엇을 의미하며 동시 모드와 어떤 관련이 있을까요?

"React Experimental"은 안정적인 릴리스에서는 아직 사용할 수 없는 기능을 포함하는 실험적인 React 빌드를 가리키는 용어입니다. 이러한 실험 빌드는 React 팀이 새로운 기능을 테스트하고, 커뮤니티로부터 피드백을 받고, 안정적인 릴리스에 포함하기 전에 디자인을 반복적으로 개선하는 방법입니다.

동시 모드는 처음에 React Experimental에서 도입되었습니다. 이는 광범위한 테스트와 피드백이 필요한 급진적인 새로운 기능이었기 때문입니다. React Experimental에서 먼저 출시함으로써, React 팀은 커뮤니티로부터 귀중한 피드백을 수집하고 동시 모드의 디자인과 구현에 필요한 조정을 할 수 있었습니다.

따라서 최신의 React 기능을 공식 출시 전에 사용해보고 싶다면 React Experimental을 주목하세요. 하지만 React Experimental의 기능은 변경될 수 있으며, 프로덕션 환경에서 사용해서는 안 됩니다.

그리고 React 개발에 기여하고 싶다면, React Experimental을 시도해보고 React 팀에 피드백을 제공하는 것이 좋은 시작점입니다. 여러분의 피드백은 React의 미래를 형성하고 모든 사람에게 더 나은 React를 만드는 데 도움이 될 수 있습니다.

## React의 지속적인 개발: 미래 전망

React는 지속적으로 개발되고 개선되는 살아 숨쉬는 프로젝트입니다. React 팀과 오픈 소스 커뮤니티에 의해 끊임없이 발전하고 있으며, React 18의 출시로 React의 미래는 밝고 흥미로운 가능성들로 가득 차 있음을 알 수 있습니다.

React의 지속적인 개발에서 가장 흥미로운 측면 중 하나는 동시성에 대한 집중입니다. React 18에서 동시 모드의 도입으로 React는 더 응답성이 좋고 사용자 친화적인 미래를 향한 큰 발걸음을 내디뎠습니다. 하지만 이는 시작에 불과합니다. React 팀은 이미 동시 렌더링의 강력함을 한층 더 발전시키기 위한 새로운 동시 기능과 개선 작업을 진행하고 있습니다.

또 다른 흥미로운 개발 영역은 서버 컴포넌트입니다. 이는 서버에서 렌더링되고 HTML로 클라이언트에 전송될 수 있는 새로운 유형의 컴포넌트입니다. 서버 컴포넌트는 API 계층 없이도 서버 측 데이터를 직접 액세스할 수 있어, 앱의 데이터 페칭 로직을 크게 단순화하고 성능을 향상시킬 수 있습니다.

그리고 React Suspense도 잊지 말아야 합니다. 컴포넌트의 부드러운 로딩 상태를 만들 수 있는 강력한 기능입니다. Suspense는 이미 React 18에서 사용할 수 있지만, React 팀은 이를 더욱 강력하고 유연하게 만들기 위한 새로운 API와 추가 개선 작업을 진행하고 있습니다.

따라서 React 개발자로서 앞으로 기대할 것이 많습니다. React의 미래는 앱을 더 응답성 높고 사용자 친화적이며, 개발하기 재미있게 만드는 흥미로운 새로운 기능과 개선으로 가득할 것입니다. React 팀이 우리에게 어떤 것을 준비하고 있을지 정말 기대가 됩니다!

## 왜 React가 개발자들 사이에서 인기가 많은가

React는 프론트엔드 개발의 세계를 강타했습니다. 2013년 Facebook에 의해 출시된 이후, React는 개발자들 사이에서 엄청난 인기를 얻었으며 사용자 인터페이스를 구축하는 데 가장 널리 사용되는 JavaScript 라이브러리 중 하나가 되었습니다. 그렇다면 왜 React가 이렇게 인기가 많을까요?

주요 이유 중 하나는 그 단순함과 유연성입니다. React는 재사용 가능한 코드 조각인 컴포넌트 기반으로 구성되어 있으며, 이러한 컴포넌트를 조합하여 복잡한 사용자 인터페이스를 구축할 수 있습니다. 이 컴포넌트 기반 아키텍처는 대규모 애플리케이션을 구축하고 유지보수하기 쉽게 만듭니다.

또 다른 이유는 성능입니다. React는 가상 DOM과 디핑(diffing) 알고리즘을 사용하여 실제 DOM에 대한 업데이트 수를 최소화합니다. 이는 실제 DOM 업데이트가 느릴 수 있기 때문입니다. 이를 통해 React 애플리케이션은 복잡한 UI를 가진 대규모 애플리케이션에서도 빠르고 응답성이 높습니다.

React 18에서 도입된 동시 모드는 이러한 성능을 한 단계 더 끌어올립니다. React가 여러 작업을 동시에 수행하고 그 중요도에 따라 우선 순위를 지정할 수 있게 함으로써, 동시 모드는 React 애플리케이션을 더욱 응답성이 높고 사용자 친화적으로 만듭니다.

하지만 아마도 React의 인기가 많은 가장 중요한 이유는 그 활기찬 커뮤니티일 것입니다. 전 세계 수백만 명의 개발자가 React를 사용하고 있어, 언제나 도움을 받을 수 있습니다. 문제 해결, 개발을 단순화할 라이브러리, 또는 영감이 필요할 때, React 커뮤니티가 항상 도와줄 준비가 되어 있습니다.

그리고 동시 모드, 서버 컴포넌트, React Suspense와 같은 새로운 기능의 도입으로, React는 현재에 안주하지 않고 계속해서 진화하고 개선되고 있습니다. 이는 React가 개발자들에게 흥미롭고 보람 있는 선택이 되게 만듭니다.

## React Suspense: 프로덕션 준비가 되었을까? 분석

React Suspense는 React 18에서 도입된 가장 흥미로운 기능 중 하나입니다. 이는 컴포넌트의 로딩 상태를 선언적으로 처리할 수 있는 방법을 제공하여 부드럽고 응답성 좋은 로딩 경험을 만들기 쉽게 해줍니다. 하지만 이것이 프로덕션에서 사용하기에 준비가 되었을까요?

답은 "예"이지만 주의할 점이 있습니다. React Suspense는 React 18에 포함되어 있으며 프로덕션에서 사용할 수 있지만, 여전히 실험적인 기능으로 간주됩니다. 이는 일반적으로 안정적이고 신뢰할 수 있지만, React 팀이 해결해야 할 몇 가지 엣지 케이스와 버그가 있을 수 있음을 의미합니다.

Suspense를 사용할 때 염두에 두어야 할 주요 사항 중 하나는 로딩 상태에 대한 사고방식의 변화가 필요하다는 점입니다. 컴포넌트에서 로딩 상태를 수동으로 관리하는 대신, Suspense를 사용하면 선언적으로 로딩 상태를 지정하고 나머지는 React가 처리하도록 할 수 있습니다.

이렇게 하면 코드가 간소화되고 로딩 상태가 일관되게 유지될 수 있지만, 기존 코드를 일부 리팩토링해야 할 수도 있습니다. 따라서 Suspense를 대규모 기존 코드베이스에 사용하려고 한다면, 어느 정도의 작업을 준비해야 합니다.

결론적으로, React Suspense는 부드러운 로딩 상태를 만드는 강력한 도구이지만, 여전히 실험적인 기능이며 기존 코드에 통합하는 데 약간의 작업이 필요할 수 있습니다. 그러나 새로운 프로젝트를 시작하거나 기존 코드를 리팩토링할 의지가 있다면, 로딩 경험을 크게 개선할 수 있는 게임 체인저가 될 수 있습니다.

## React 18: 주요 변경 사항과 대처 방법

라이브러리나 프레임워크의 주요 릴리스마다 항상 호환성 문제가 발생할 가능성이 있습니다. React 18도 예외는 아닙니다. React 팀은 하위 호환성을 유지하기 위해 많은 노력을 기울였지만, 기존 코드를 잠재적으로 중단시킬 수 있는 몇 가지 변경 사항이 있습니다.

가장 중요한 변경 사항 중 하나는 동시 모드의 도입입니다. 동시 모드는 많은 이점을 제공하지만, React에서 렌더링과 상태 업데이트에 대한 새로운 사고방식을 도입합니다. 이는 기존 코드에서 가정한 내용을 잠재적으로 중단시킬 수 있습니다.

예를 들어, 동시 모드에서는 React가 사용자에게 보이지 않더라도 컴포넌트를 여러 번 렌더링할 수 있습니다. 이는 이전 버전의 React에서 컴포넌트가 정확히 한 번 렌더링되는 동기 렌더링 모델과는 다릅니다.

또 다른 잠재적인 문제는 자동 배칭의 도입입니다. React 18에서는 여러 상태 업데이트가 배치되어 단일 재렌더링이 발생합니다. 이는 상태 업데이트가 즉각적인 재렌더링을 유발하는 기존 코드에 영향을 줄 수 있습니다.

이러한 변경 사항에 어떻게 대처할 수 있을까요? 다음 몇 가지 팁을 참고하세요:

1. 앱 테스트하기: 변경 사항에 대처하는 첫 번째 단계는 앱을 철저히 테스트하는 것입니다. 변경 사항에 영향을 받을 것이라고 생각되는 부분뿐만 아니라 앱의 모든 부분을 테스트하세요.

2. 엄격 모드 사용하기: 엄격 모드는 앱에서 잠재적인 문제를 잡아내는 데 도움이 되는 React 도구입니다. React의 새로운 버전으로 마이그레이션할 때 특히 유용하며, 문제가 발생하기 전에 변경 사항을 감지할 수 있습니다.

3. 필요에 따라 리팩토링하기: React 18의 변경 사항으로 인해 앱의 일부가 깨지는 경우, 코드를 리팩토링해야 할 수 있습니다. 이는 상태 업데이트를 처리하는 방식을 변경하거나 앱에서 렌더링을 처리하는 방식을 재고하는 것을 포함할 수 있습니다.

4. 도움 구하기: 막히면 주저하지 말고 도움을 구하세요. React 커뮤니티는 활기차고 지원적이며, React 18의 변경 사항을 탐색하는 데 도움이 되는 많은 리소스가 있습니다.

기억하세요, 호환성 문제는 도전적일 수 있지만, 이는 코드를 개선하고 React에 대해 더 많이 배울 수 있는 기회이기도 합니다. 도전을 받아들이고 즐겁게 코딩하세요!

## React 18: 동시 기능의 힘

React 18과 그 동시 모드를 탐험하면서 React에서 렌더링에 대한 새로운 사고방식을 어떻게 도입했는지 알 수 있었습니다. 이는 단순히 앱을 더 빠르게 만드는 것이 아니라 더 응답성 있고 사용자 친화적으로 만드는 것입니다. React 18의 동시 기능의 힘은 과장할 수 없습니다.

자동 배칭: 이 기능은 여러 상태 업데이트를 함께 배치하여 단일 재렌더링을 수행할 수 있게 합니다. 이는 상태 업데이트가 빈번한 앱의 성능을 크게 향상시킬 수 있습니다.

**스트리밍 서버 렌더링:** 이 기능은 서버에서 클라이언트로 HTML을 스트리밍할 수 있게 하여 앱의 인식된 로드 시간을 개선합니다. 브라우저는 전체 응답을 기다리지 않고 첫 번째 데이터 청크를 받자마자 HTML 렌더링을 시작할 수 있습니다.

**React Suspense:** 이 기능은 컴포넌트에서 로딩 상태를 처리하는 선언적인 방법을 제공합니다. Suspense를 사용하면 일부 코드가 로드될 때까지 "기다리고" 선언적으로 로딩 상태를 지정할 수 있습니다. 이를 통해 코드가 간소화되고 사용자에게 부드러운 로딩 경험을 제공할 수 있습니다.

**StartTransition:** 이 기능을 사용하면 특정 업데이트를 "전환"으로 표시하고 우선 순위를 낮출 수 있습니다. 이는 긴급하지 않고 더 긴급한 작업이 있을 때 중단할 수 있는 업데이트에 유용할 수 있습니다.

**UseDeferredValue:** 이 기능을 사용하면 느린 상태 업데이트를 더 빠른 상태 업데이트가 완료될 때까지 연기할 수 있습니다. 이는 무거운 계산이나 네트워크 요청 중에도 UI의 응답성을 유지하는 데 유용할 수 있습니다.

이러한 동시 기능들은 React 18의 핵심입니다. 이들은 응답성이 뛰어나고 사용자 친화적인 앱을 구축하기 위한 새로운 도구와 기술을 제공합니다. 그러나 큰 힘에는 큰 책임이 따릅니다. 이러한 기능들은 새로운 수준의 복잡성을 도입하며 앱이 업데이트되고 렌더링되는 방식을 다르게 생각해야 합니다. 따라서 이러한 개념을 철저히 이해한 후에 사용하시기 바랍니다.

## 동시 모드와 함께하는 React의 미래

React 동시 모드에 대한 탐험을 통해 우리는 많은 것을 배웠습니다. 동시 렌더링의 기본을 이해하는 것부터 React 18의 새로운 기능을 깊이 파고드는 것까지, 동시 모드가 사용자 인터페이스 구축 방식을 혁신할 준비가 되어 있음을 알 수 있었습니다.

동시 모드는 단순한 새로운 기능이 아니라 React에 대한 완전히 새로운 사고방식입니다. 이를 통해 React는 메인 스레드를 차단하지 않고 동시에 여러 작업을 수행할 수 있습니다. 자동 배칭, 스트리밍 서버 렌더링, React Suspense와 같은 강력한 새로운 기능을 도입합니다. 그리고 React에서 렌더링 및 상태 업데이트에 대한 사고방식을 변화시킵니다.

하지만 이러한 새로운 힘에는 새로운 도전 과제가 따릅니다. 동시 모드는 새로운 수준의 복잡성을 도입하며 앱이 업데이트되고 렌더링되는 방식을 다르게 생각해야 합니다. 기존의 가정을 재고하고 코드를 리팩토링해야 할 수 있습니다. 그리고 React 개발자로서 계속 배우고 성장해야 합니다.

하지만 이것이 바로 React의 매력입니다. React는 지속적으로 진화하고 개선되는 살아있는 프로젝트입니다. 더 나은 사용자 인터페이스를 구축하기 위해 열정을 가진 수백만 명의 개발자로 구성된 커뮤니티입니다. 그리고 학습, 성장, 발견의 여정입니다.

따라서 동시 모드와 함께하는 React의 미래를 바라보며, 여러분이 무엇을 구축할지 기대됩니다. 이러한 새로운 기능을 사용하여 더 응답성이 뛰어나고 사용자 친화적인 앱을 어떻게 만들지 기대됩니다. 그리고 이 여정을 여러분과 함께 계속하게 되어 기쁩니다.

React 개발의 모든 잠재력을 발휘할 준비가 되셨나요?

React 애플리케이션을 구축하는 것은 흥미롭지만 시간이 많이 소요될 수 있습니다. [DhiWise React Builder](https://app.dhiwise.com/sign-up?utm_campaign=blog&utm_source=seo&utm_medium=website&utm_term=education&utm_content=deep-dive-into-react-concurrent-mode-exploring-key-features)는 더 적은 시간에 더 많은 것을 이룰 수 있도록 도와줍니다.

오늘 DhiWise 계정을 등록하고 React 애플리케이션 구축의 미래를 경험해보세요.