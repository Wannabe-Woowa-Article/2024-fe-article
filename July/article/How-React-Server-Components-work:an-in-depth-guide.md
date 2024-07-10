## 🔗 [How React server components work: an in-depth guide](https://www.plasmic.app/blog/how-react-server-components-work)

### 🗓️ 번역 날짜: 2024.07.08

### 🧚 번역한 크루: 버건디(전태헌)

---

React 서버 컴포넌트(RSC)는 React 18에서 실험적으로 도입된 새로운 기능으로, 페이지 로드 성능, 번들 크기, 그리고 React 애플리케이션 개발 방식에 엄청난 영향을 미칠 것으로 기대됩니다.

저희 Plasmic은 React용 시각적 빌더를 제공하며, 특히 성능 중요한 마케팅 및 전자상거래 사이트를 구축하는 고객들에게 매우 중요한 React 성능 개선에 큰 관심을 가지고 있습니다.

이에 React 서버 컴포넌트에 대해 깊이 파헤쳐보았으며, 그 동작 원리에 대해 알아보고 공유하려고 합니다.

또한 [트윗 요약](https://www.reddit.com/r/reactjs/comments/s8r0ve/how_react_server_components_work_an_indepth_guide/)을 보실 수 있습니다.

## 리액트 서버 컴포넌트는 무엇일까요 ?

리액트 서버 컴포넌트 (RSC)는 서버와 클라이언트(브라우저)가 협력하여 React 애플리케이션을 렌더링할 수 있게 해줍니다.

일반적으로 React 요소 트리는 여러 개의 React 컴포넌트로 구성되어 페이지를 렌더링하는데, RSC는 이 트리의 일부 컴포넌트를 서버에서 렌더링하고, 일부 컴포넌트를 클라이언트에서 렌더링할 수 있게 만듭니다.

이는 기존의 React 개발 방식을 크게 바꾸게 됩니다.

React 팀이 제시한 간략한 그림에서도 이 목표가 잘 나타납니다: 서버에서 렌더링되는 주황색 컴포넌트와 클라이언트에서 렌더링되는 파란색 컴포넌트가 혼합된 React 트리입니다.

![](https://www.plasmic.app/blog/static/images/react-server-components.png)

### 이건 서버 사이드 렌더링(SSR) 아닌가요 ?

**리액트 서버 컴포넌트는 SSR이 아닙니다!**

두 기술 모두 "서버"라는 용어를 사용하며 서버에서 작업을 수행하지만, 별도의 개념이며 서로 독립적으로 작동합니다.

SSR은 React 트리를 원시 HTML로 렌더링하기 위한 환경을 시뮬레이션하는 반면, RSC는 서버와 클라이언트가 협력하여 특정 컴포넌트를 서버 또는 클라이언트에서 렌더링할 수 있게 합니다.

SSR은 서버와 클라이언트 컴포넌트를 구분하지 않고 동일하게 렌더링하지만, RSC는 서버에서 일부 컴포넌트를 렌더링하고, 나머지는 클라이언트에서 렌더링하는 점에서 차이가 있습니다.

RSC를 사용하기 위해서는 SSR을 사용할 필요는 없으며, 그 반대도 마찬가지입니다.

물론 SSR과 RSC를 결합하여 서버사이드 렌더링과 함께 서버 컴포넌트를 브라우저에서 적절하게 렌더링할 수도 있습니다.

향후 포스트에서는 이 두 기술이 어떻게 함께 작동하는지에 대해 더 자세히 다룰 예정입니다.

하지만 현재는 SSR을 무시하고 RSC에만 집중하는 것이 좋습니다.

## 우리는 이걸 왜 원하게 될까요 ?

리액트 서버 컴포넌트가 도입되기 전에는 모든 리액트 컴포넌트가 "클라이언트" 컴포넌트로, 브라우저에서 실행되었습니다.

브라우저가 리액트 페이지를 방문하면 필요한 모든 리액트 컴포넌트의 코드를 다운로드하고, 리액트 엘리먼트 트리를 구성하여 DOM에 렌더링하거나(SSR을 사용하는 경우 DOM을 하이드레이션) 합니다.

브라우저에서 이 작업을 수행하는 것이 좋은 이유는 리액트 애플리케이션이 상호작용할 수 있게 해주기 때문입니다.

이벤트 핸들러를 설치하고, 상태를 추적하며, 이벤트에 따라 리액트 트리를 변경하고, 효율적으로 DOM을 업데이트할 수 있습니다.

그렇다면 왜 서버에서 렌더링을 해야 할까요?

브라우저를 넘어서서 서버에서 렌더링 하는 것의 몇몇 장점들이 있습니다.

- 서버는 데이터 소스에 직접적으로 접근할 수 있습니다. 예를 들어 데이터베이스, GraphQL 엔드포인트, 파일 시스템 등이 있습니다.
  서버는 공용 API 엔드포인트를 거치지 않고도 직접 필요한 데이터를 가져올 수 있으며, 보통 데이터 소스와 물리적으로 더 가까운 위치에 있어 브라우저보다 데이터를 더 빠르게 가져올 수 있습니다.

- 서버는 마크다운을 HTML로 렌더링하는 npm 패키지와 같은 "무거운" 코드 모듈을 저렴하게 사용할 수 있습니다.
  브라우저는 이러한 의존성을 사용할 때마다 자바스크립트 번들로 다운로드해야 하지만, 서버는 매번 이러한 의존성을 다운로드할 필요가 없기 때문입니다.

간단히 말해, **리액트 서버 컴포넌트는 서버와 브라우저가 각자의 장점을 최대한 발휘할 수 있게 해줍니다.** 서버 컴포넌트는 데이터 가져오기와 콘텐츠 렌더링에 집중하고, 클라이언트 컴포넌트는 상태를 관리하고 상호작용성을 유지하는 데 집중할 수 있습니다.
그 결과, 페이지 로딩 속도가 빨라지고 자바스크립트 번들 크기가 작아지며, 사용자 경험이 개선됩니다.

## 전체적인 그림

이 작업이 어떻게 이루어지는지 직관적으로 이해해봅시다.

제 아이들은 컵케이크 장식을 매우 좋아하지만, 굽는 과정에는 별로 흥미가 없습니다.

만약 아이들에게 처음부터 컵케이크를 만들고 장식하라고 한다면 (귀엽긴 하겠지만) 정말 큰일일 것입니다.

저는 아이들에게 밀가루와 설탕, 버터를 주고 오븐을 사용할 수 있게 해주며, 많은 설명을 읽어주고 하루 종일 걸릴 것입니다.

그러나 제가 미리 일부 작업을 처리해준다면, 즉 컵케이크를 굽고 아이싱을 만들어 아이들에게 주면, 아이들은 더 빨리 재미있는 장식 작업을 시작할 수 있습니다!

게다가 오븐을 사용하는 걱정도 하지 않아도 되죠. 정말 좋은 상황이죠!

![](https://www.plasmic.app/blog/static/images/cake.jpg)

리액트 서버 컴포넌트는 이러한 작업 분담을 가능하게 합니다.

즉, 서버가 잘할 수 있는 작업을 미리 처리한 후, 브라우저가 나머지 작업을 완료하도록 넘기는 것입니다.

이렇게 하면 서버가 넘겨줘야 할 것들이 줄어듭니다.

예를 들어, 밀가루 한 봉지와 오븐 전체를 넘기는 대신, 12개의 작은 컵케이크를 넘기는 것이 훨씬 효율적입니다!

여러분의 페이지에 대한 리액트 트리를 생각해봅시다.

일부 컴포넌트는 서버에서 렌더링되고, 일부는 클라이언트에서 렌더링됩니다.

이러한 고수준이라고 할 수 있는 전략을 단순화해서 생각해보면 다음과 같습니다:

서버는 서버 컴포넌트를 평소처럼 "렌더링"하여 리액트 컴포넌트를 `div`나 `p` 같은 네이티브 HTML 요소로 변환합니다.

그러나 브라우저에서 렌더링해야 하는 "클라이언트" 컴포넌트를 만나면, 서버는 그 자리에 클라이언트 컴포넌트와 속성(props)을 채우기 위한 지침과 함께 `플레이스홀더`를 출력합니다.

그 다음 브라우저가 이 출력을 받아서, 그 플레이스홀더를 클라이언트 컴포넌트로 채우면 끝납니다!

**실제로는 이렇게 간단하지 않지만,** 이제 곧 복잡한 세부 사항을 다룰 예정입니다.

하지만 이런 고수준의 그림을 머릿속에 가지고 있는 것은 유용합니다!

## 서버-클라이언트 컴포넌트 구분

먼저, 서버 컴포넌트란 무엇일까요? 그리고 어떤 컴포넌트가 "서버용"이고 어떤 것이 "클라이언트용"인지 어떻게 구분할 수 있을까요?

리액트 팀은 컴포넌트가 작성된 파일의 확장자에 따라 이를 정의했습니다.

파일이 `.server.jsx`로 끝나면 그 파일에는 서버 컴포넌트가 포함되어 있고, `.client.jsx`로 끝나면 클라이언트 컴포넌트가 포함되어 있습니다.

확장자가 둘 다 아닌 경우, 그 파일에는 서버와 클라이언트 모두에서 사용할 수 있는 컴포넌트가 포함되어 있습니다.

이 정의는 실용적입니다.

사람들과 번들러 모두가 서버 컴포넌트와 클라이언트 컴포넌트를 쉽게 구분할 수 있습니다.

특히 번들러는 파일 이름을 검사하여 클라이언트 컴포넌트를 다르게 처리할 수 있습니다.

곧 알게 되겠지만, 번들러는 리액트 서버 컴포넌트(RSC)를 작동시키는 데 중요한 역할을 합니다.

서버 컴포넌트는 서버에서 실행되고 클라이언트 컴포넌트는 클라이언트에서 실행되기 때문에, 각 컴포넌트가 할 수 있는 것에는 많은 제한이 있습니다.

가장 중요한 것은 **클라이언트 컴포넌트가 서버 컴포넌트를 가져올 수 없다는 점**입니다.

서버 컴포넌트는 브라우저에서 실행될 수 없으며, 브라우저에서 작동하지 않는 코드가 포함될 수 있습니다.

클라이언트 컴포넌트가 서버 컴포넌트에 의존하게 되면, 브라우저 번들에 이러한 허용 되지 않는 의존성을 끌어오게 됩니다.

이 마지막 부분은 헷갈릴 수 있습니다.

이것은 다음과 같은 컴포넌트 형식이 불가능하다는 것을 의미합니다.

```tsx
// ClientComponent.client.jsx
// NOT OK:
import ServerComponent from "./ServerComponent.server";
export default function ClientComponent() {
  return (
    <div>
      <ServerComponent />
    </div>
  );
}
```

클라이언트 컴포넌트가 서버 컴포넌트를 import할 수 없고, 따라서 서버 컴포넌트를 인스턴스화할 수 없는데, 어떻게 서버 컴포넌트와 클라이언트 컴포넌트가 섞여 있는 React 트리가 생길 수 있나요?

어떻게 클라이언트 컴포넌트(파란 점) 아래에 서버 컴포넌트(주황 점)가 있을 수 있나요?

![](https://www.plasmic.app/blog/static/images/react-server-components.png)

클라이언트 컴포넌트가 서버 컴포넌트를 import하고 렌더링할 수는 없지만, 여전히 컴포지션을 사용할 수 있습니다.

즉, 클라이언트 컴포넌트는 불투명한 `ReactNode` 형태의 `props`를 받을 수 있으며, 이 ReactNode는 서버 컴포넌트에 의해 렌더링될 수 있습니다. 예를 들어:

```tsx
// ClientComponent.client.jsx
export default function ClientComponent({ children }) {
  return (
    <div>
      <h1>Hello from client land</h1>
      {children}
    </div>
  );
}

// ServerComponent.server.jsx
export default function ServerComponent() {
  return <span>Hello from server land</span>;
}

// OuterServerComponent.server.jsx
// OuterServerComponent는 클라이언트와 서버 컴포넌트를 모두 인스턴스화할 수 있으며,
// <ServerComponent/>를 ClientComponent의 children prop으로 전달하고 있습니다.

import ClientComponent from "./ClientComponent.client";
import ServerComponent from "./ServerComponent.server";
export default function OuterServerComponent() {
  return (
    <ClientComponent>
      <ServerComponent />
    </ClientComponent>
  );
}
```

이 제한은 RSC를 더 잘 활용하기 위해 컴포넌트를 구성하는 방식에 큰 영향을 미칠 것입니다.

## - RSC 렌더링의 과정

React 서버 컴포넌트를 렌더링하려고 할 때 실제로 어떤 일이 발생하는지 자세히 살펴보겠습니다.

서버 컴포넌트를 사용하기 위해 여기서 모든 것을 이해할 필요는 없지만, 작동 방식을 직관적으로 이해하는 데 도움이 될 것입니다.

## 1. 서버가 렌더링 요청을 받음

서버는 일부 렌더링을 수행해야 하므로 RSC를 사용하는 페이지의 시작은 항상 서버에서 시작됩니다.

이는 React 컴포넌트를 렌더링하기 위한 API 호출에 대한 응답으로 이루어집니다.

이 "루트" 컴포넌트는 항상 서버 컴포넌트이며, 이는 다른 서버 또는 클라이언트 컴포넌트를 렌더링할 수 있습니다.

서버는 요청에 전달된 정보를 기반으로 어떤 서버 컴포넌트와 어떤 props를 사용할지 결정합니다.

이 요청은 일반적으로 특정 URL의 페이지 요청 형태로 들어오지만, Shopify Hydrogen은 더 [세분화된 방법](https://shopify.github.io/hydrogen-v1/tutorials/server-props)을 제공하며, React 팀의 공식 데모에는 [원시 구현](https://github.com/reactjs/server-components-demo/blob/main/server/api.server.js)이 포함되어 있습니다.

## 2. 서버가 루트 컴포넌트 요소를 JSON으로 직렬화함

여기서 최종 목표는 초기 루트 서버 컴포넌트를 기본 HTML 태그와 클라이언트 컴포넌트 "플레이스홀더"의 트리로 렌더링하는 것입니다.

그런 다음 이 트리를 직렬화하여 브라우저로 보내면, 브라우저는 이를 역직렬화하고 클라이언트 플레이스홀더를 실제 클라이언트 컴포넌트로 채워 최종 결과를 렌더링합니다.

따라서, 위의 예를 따르면 — `<OuterServerComponent/>`를 렌더링하려고 한다고 가정해 봅시다.

직렬화된 요소 트리를 얻기 위해 단순히 JSON.stringify(`<OuterServerComponent />`)를 사용할 수 있을까요?

거의 맞지만, 완전히 그렇지는 않습니다! 😅 React 요소가 실제로 무엇인지 상기해 봅시다 — React 요소는 객체로, type 필드는 기본 HTML 태그를 위한 문자열("div"와 같은) 또는 React 컴포넌트 인스턴스를 위한 함수입니다.

```ts
// React element for <div>oh my</div>
> React.createElement("div", { title: "oh my" })
{
  $$typeof: Symbol(react.element),
  type: "div",
  props: { title: "oh my" },
  ...
}

// React element for <MyComponent>oh my</MyComponent>
> function MyComponent({children}) {
    return <div>{children}</div>;
  }
> React.createElement(MyComponent, { children: "oh my" });
{
  $$typeof: Symbol(react.element),
  type: MyComponent  // reference to the MyComponent function
  props: { children: "oh my" },
  ...
}
```

우리가 클라이언트 컴포넌트 함수에 대한 참조를 직렬화 가능한 “모듈 참조” 객체로 변환하는 이 작업은 어디서 발생하고 있습니까?

결국, 이 마술을 수행하는 것은 번들러입니다! React 팀은 `react-server-dom-webpack`에서 [webpack 로더](https://github.com/facebook/react/blob/main/packages/react-server-dom-webpack/src/ReactFlightWebpackNodeLoader.js) 또는 [node-register](https://github.com/facebook/react/blob/main/packages/react-server-dom-webpack/src/ReactFlightWebpackNodeRegister.js)로 공식 RSC 지원을 발표했습니다.

서버 컴포넌트가 \*.client.jsx 파일에서 무언가를 가져올 때 실제로 그 항목을 가져오는 대신 파일 이름과 내보낸 이름을 포함하는 모듈 참조 객체만 가져옵니다.

클라이언트 컴포넌트 함수는 서버에서 구성된 React 트리의 일부가 아닙니다!

다시 위의 예제를 고려해보면, 우리가 `<OuterServerComponent />`를 직렬화하려고 할 때, 우리는 다음과 같은 JSON 트리를 얻게 됩니다:

```ts
{
  // "모듈 참조"가 있는 ClientComponent 요소 플레이스홀더

  type: {
    $$typeof: Symbol(react.module.reference),
    name: 'default',
    filename: './src/ClientComponent.client.js',
  },
  props: {
    // ClientComponent에 전달된 자식, 즉 <ServerComponent />.
    children: {
      // ServerComponent가 직접 html 태그로 렌더링됨;
      // ServerComponent에 대한 참조가 전혀 없음을 주목하세요.
      // 우리는 직접 `span`을 렌더링하고 있습니다.
      $$typeof: Symbol(react.element),
      type: 'span',
      props: {
        children: 'Hello from server land',
      },
    },
  },
};

```

### 직렬화 가능한 React 트리

이 과정이 끝나면, 서버에서 다음과 같은 React 트리를 얻게 되고, 이를 브라우저로 보내 "마무리"를 할 수 있기를 바랍니다:

![](https://www.plasmic.app/blog/static/images/react-server-components-placeholders.png)

## 모든 props는 직렬화가 가능해야 합니다

전체 React 트리를 JSON으로 직렬화하기 때문에, 클라이언트 컴포넌트나 기본 html 태그에 전달하는 모든 props도 직렬화 가능해야 합니다.

이는 서버 컴포넌트에서 이벤트 핸들러를 prop으로 전달할 수 없다는 것을 의미합니다!

```ts
// NOT OK: 함수는 직렬화 될 수 없기 때문에 서버컴포넌트는
// 자식에게 함수를 props로 넘길 수 없습니다.
function SomeServerComponent() {
  return <button onClick={() => alert('OHHAI')}>Click me!</button>;
}
```

하지만 여기서 주목할 점은 RSC 과정 중에 클라이언트 컴포넌트를 만나면, 클라이언트 컴포넌트 함수를 호출하거나 클라이언트 컴포넌트 내부로 들어가지 않는다는 것입니다.

따라서 클라이언트 컴포넌트가 또 다른 클라이언트 컴포넌트를 인스턴스화하는 경우:

```tsx
function SomeServerComponent() {
  return <ClientComponent1>Hello world!</ClientComponent1>;
}

function ClientComponent1({children}) {
  // 클라이언트가 클라이언트에게 props로 함수를 전달하는 것은 가능합니다.
  return <ClientComponent2 onChange={...}>{children}</ClientComponent2>;
}
```

ClientComponent2는 이 RSC JSON 트리에 전혀 나타나지 않습니다.

대신, 우리는 ClientComponent1에 대한 모듈 참조와 props가 있는 요소만 보게 됩니다.

따라서 ClientComponent1이 ClientComponent2에 이벤트 핸들러를 prop으로 전달하는 것은 전혀 문제가 없습니다.

## 3. 브라우저에서 React 트리 재구성

브라우저는 서버로부터 JSON 출력을 받아, 이제 브라우저에서 렌더링될 React 트리를 재구성해야 합니다.

`타입`이 모듈 참조인 요소를 만날 때마다, 이를 실제 클라이언트 컴포넌트 함수의 참조로 교체해야 합니다.

이 작업에는 번들러의 도움이 다시 필요합니다.

번들러가 서버에서 클라이언트 컴포넌트 함수를 모듈 참조로 교체했으며, 이제 브라우저에서 모듈 참조를 실제 클라이언트 컴포넌트 함수로 교체하는 방법을 아는 것도 번들러입니다.

재구성된 React 트리는 다음과 같이 보일 것입니다 — 네이티브 태그와 클라이언트 컴포넌트만 교체된 상태로:

![](https://www.plasmic.app/blog/static/images/react-server-components-client.png)

그런 다음 우리는 이 트리를 평소와 같이 DOM에 렌더링하고 커밋하면 됩니다!

## 그런데 이것이 Suspense와 함께 작동할까요?

네! Suspense는 위의 모든 단계에서 중요한 역할을 합니다.

이 글에서는 Suspense에 대해 의도적으로 자세히 다루지 않았습니다.

Suspense 자체가 매우 큰 주제이며, 별도의 블로그 포스트가 필요하기 때문입니다.

하지만 간단히 말하자면 — Suspense는 React 컴포넌트가 아직 준비되지 않은 무언가를 필요로 할 때(데이터를 가져오거나, 컴포넌트를 지연 로드하는 등) Promise를 던질 수 있게 해줍니다.

이러한 Promise는 "Suspense 경계"에서 잡히며 — Suspense 하위 트리의 렌더링에서 Promise가 던져지면, React는 해당 Promise가 해결될 때까지 해당 하위 트리의 렌더링을 중지하고, 이후 다시 시도합니다.

서버에서 RSC 출력을 생성하기 위해 서버 컴포넌트 함수를 호출할 때, 해당 함수들이 필요한 데이터를 가져오면서 Promise를 던질 수 있습니다.

이러한 [Promise를 만났을 때](https://github.com/facebook/react/blob/42c30e8b122841d7fe72e28e36848a6de1363b0c/packages/react-server/src/ReactFlightServer.js#L416)는 플레이스홀더를 출력합니다.

Promise가 해결되면 서버 컴포넌트 함수를 다시 호출하여 완료된 청크를 출력합니다.

우리는 실제로 RSC 출력의 스트림을 생성하고 있으며, Promise가 던져질 때 멈추고, 해결되면 추가 청크를 스트리밍합니다.

마찬가지로, 브라우저에서도 위의 `fetch()` 호출로부터 RSC JSON 출력을 스트리밍합니다.

이 과정에서도 출력에서 서스펜스 플레이스홀더(서버에서 Promise가 던져진 부분)를 만나고, 아직 스트림에서 해당 플레이스홀더 내용을 보지 못한 경우 Promise를 던질 수 있습니다(여기에는 약간의 [세부 사항](https://github.com/facebook/react/blob/main/packages/react-client/src/ReactFlightClientStream.js)이 있습니다).

또한, 클라이언트 컴포넌트 모듈 참조를 만나지만 브라우저에 해당 클라이언트 컴포넌트 함수가 아직 로드되지 않은 경우에도 Promise를 던질 수 있습니다.

이 경우 번들러 런타임이 필요한 [청크를 동적으로 가져와야 합니다.](https://github.com/facebook/react/blob/main/packages/react-server-dom-webpack/src/ReactFlightClientWebpackBundlerConfig.js)

Suspense 덕분에 서버는 서버 컴포넌트가 데이터를 가져오는 동안 RSC 출력을 스트리밍할 수 있으며, 브라우저는 데이터가 사용 가능해질 때마다 점진적으로 렌더링하고, 필요에 따라 클라이언트 컴포넌트 번들을 동적으로 가져올 수 있습니다.

# RSC 와이어 포맷

하지만 서버가 정확히 무엇을 출력하는 것일까요? "JSON"과 "스트림"이라는 단어를 읽고 의아하게 여겼다면, 너무 당연합니다!

그렇다면 서버가 브라우저로 스트리밍하는 데이터는 무엇일까요?

이것은 간단한 포맷으로, 각 줄에 ID가 태그된 하나의 JSON 블롭이 있습니다.

다음은 <OuterServerComponent/> 예제의 RSC 출력입니다:

```
M1:{"id":"./src/ClientComponent.client.js","chunks":["client1"],"name":""}
J0:["$","@1",null,{"children":["$","span",null,{"children":"Hello from server land"}]}]
```

위의 코드 조각에서 `M`으로 시작하는 줄은 클라이언트 컴포넌트 모듈 참조를 정의하며, 클라이언트 번들에서 컴포넌트 함수를 찾는 데 필요한 정보를 포함합니다.

`J`로 시작하는 줄은 실제 React 요소 트리를 정의하며, `@1`과 같은 참조는 `M` 줄에서 정의된 클라이언트 컴포넌트를 가리킵니다.

이 포맷은 매우 스트리밍 가능하여, 클라이언트가 전체 행을 읽는 즉시 JSON 조각을 구문 분석하고 일부 작업을 진행할 수 있습니다.

서버가 렌더링 중에 서스펜스 경계를 만났다면, 각 청크가 해결될 때마다 여러 `J` 줄을 볼 수 있습니다.

예를 들어서, 더욱 구체적인 예를 하나 더 만들어보겠습니다.

```tsx
// Tweets.server.js
import { fetch } from "react-fetch"; // React's Suspense-aware fetch()
import Tweet from "./Tweet.client";
export default function Tweets() {
  const tweets = fetch(`/tweets`).json();
  return (
    <ul>
      {tweets.slice(0, 2).map((tweet) => (
        <li>
          <Tweet tweet={tweet} />
        </li>
      ))}
    </ul>
  );
}

// Tweet.client.js
export default function Tweet({ tweet }) {
  return (
    <div onClick={() => alert(`Written by ${tweet.username}`)}>
      {tweet.body}
    </div>
  );
}

// OuterServerComponent.server.js
export default function OuterServerComponent() {
  return (
    <ClientComponent>
      <ServerComponent />
      <Suspense fallback={"Loading tweets..."}>
        <Tweets />
      </Suspense>
    </ClientComponent>
  );
}
```

이 경우에선 RSC 스트림이 어떻게 될까요?

```
M1:{"id":"./src/ClientComponent.client.js","chunks":["client1"],"name":""}
S2:"react.suspense"

J0:["$","@1",null,{"children":[["$","span",null,{"children":"Hello from server land"}],["$","$2",null,{"fallback":"Loading tweets...","children":"@3"}]]}]

M4:{"id":"./src/Tweet.client.js","chunks":["client8"],"name":""}

J3:["$","ul",null,{"children":[["$","li",null,{"children":["$","@4",null,{"tweet":{...}}}]}],["$","li",null,{"children":["$","@4",null,{"tweet":{...}}}]}]]}]
```

`J0` 줄에는 이제 추가 자식이 있습니다 — 새로운 `Suspense` 경계로, `children`이 참조 @3을 가리키고 있습니다.

여기서 흥미로운 점은 `@3`이 이 시점에서는 아직 정의되지 않았다는 것입니다!

서버가 트윗을 로드하는 작업을 완료하면, `M4` 줄을 출력하여 `Tweet.client.js` 컴포넌트에 대한 모듈 참조를 정의하고, `J3` 줄을 출력하여` @3`이 있는 위치에 교체될 또 다른 React 요소 트리를 정의합니다.
(또한 `J3`의 자식이 `M4`에서 정의된 `Tweet` 컴포넌트를 참조하고 있다는 점에 주목하세요).

여기서 주목할 또 다른 점은, 번들러가 `ClientComponent`와 `Tweet`을 자동으로 두 개의 별도 번들로 분리하여 브라우저가 `Tweet` 번들을 나중에 다운로드할 수 있도록 했다는 것입니다!

## RSC 포맷 소비하기

이 RSC 스트림을 브라우저에서 실제 React 요소로 어떻게 변환할 수 있을까요? `react-server-dom-webpack`에는 RSC 응답을 받아서 React 요소 트리를 재생성하는 [진입점](https://github.com/facebook/react/blob/main/packages/react-server-dom-webpack/src/ReactFlightDOMClient.js)이 포함되어 있습니다.

다음은 루트 클라이언트 컴포넌트의 단순화된 버전입니다:

```ts
import { createFromFetch } from 'react-server-dom-webpack'
function ClientRootComponent() {
  // 우리의 RSC API 엔드포인트에서 fetch()를 호출합니다. react-server-dom-webpack은
  // fetch 결과를 받아서 React 요소 트리를 재구성할 수 있습니다.
  const response = createFromFetch(fetch('/rsc?...'))
  return <Suspense fallback={null}>{response.readRoot() /* React 요소를 반환합니다! */}</Suspense>
}
```

`react-server-dom-webpack`에게 API 엔드포인트에서 RSC 응답을 읽도록 요청합니다.

그런 다음, `response.readRoot()`는 응답 스트림이 처리됨에 따라 업데이트되는 React 요소를 반환합니다!

스트림이 전혀 읽히지 않은 상태에서는 내용이 준비되지 않았기 때문에 즉시 Promise를 던집니다.

그런 다음 첫 번째 `J0`를 처리하면서 해당하는 React 요소 트리를 생성하고 던져진 Promise를 해결합니다.

React는 렌더링을 재개하지만 아직 준비되지 않은 `@3` 참조를 만나면 또 다른 Promise가 던져집니다.

그리고 `J3`을 읽으면 그 Promise가 해결되고, React는 다시 렌더링을 재개하여 이번에는 완료됩니다.

따라서 RSC 응답을 스트리밍하는 동안 우리는 Suspense 경계로 정의된 청크에서 요소 트리를 계속 업데이트하고 렌더링하게 되며, 끝날 때까지 이 과정을 반복합니다.

## 왜 단순히 HTML을 출력하지 않을까요?

왜 새로운 와이어 포맷을 발명했을까요? 클라이언트의 목표는 React 요소 트리를 재구성하는 것입니다.

HTML을 구문 분석하여 React 요소를 생성해야 하는 경우보다 이 포맷을 사용하는 것이 훨씬 더 쉽습니다.

React 요소 트리를 재구성하는 것이 중요한 이유는, 이를 통해 React 트리에 대한 후속 변경 사항을 최소한의 DOM 커밋으로 병합할 수 있기 때문입니다.

## 클라이언트 컴포넌트에서 데이터를 가져오는 것보다 이것이 더 나은가요?

어차피 이 콘텐츠를 가져오기 위해 서버에 API 요청을 해야 한다면, 오늘날처럼 단순히 데이터를 가져와서 클라이언트에서 렌더링을 완전히 수행하는 것보다 이 방법이 정말 더 나은가요?

궁극적으로, 화면에 무엇을 렌더링하느냐에 따라 다릅니다.

RSC를 사용하면 사용자에게 보여줄 내용에 직접적으로 매핑되는 비정규화된 "처리된" 데이터를 얻을 수 있으므로, 가져올 데이터의 작은 부분만 렌더링하거나 렌더링 자체에 많은 자바스크립트가 필요하여 브라우저로 다운로드하는 것을 피하고 싶은 경우에 유리합니다.

또한 렌더링이 여러 데이터 가져오기를 필요로 하고, 이 가져오기가 계단식으로 서로 의존하는 경우, 데이터 지연이 훨씬 낮은 서버에서 가져오는 것이 브라우저에서 가져오는 것보다 더 좋습니다.

## 하지만… 서버 사이드 렌더링은요?

알고 있습니다.

React 18을 사용하면 SSR과 RSC를 결합하여 서버에서 HTML을 생성하고, 브라우저에서 RSC로 그 HTML을 하이드레이션할 수 있습니다.

이 주제에 대한 더 많은 정보를 기대해주세요!

## 서버 컴포넌트가 렌더링하는 내용을 업데이트하기

서버 컴포넌트가 새로운 내용을 렌더링해야 한다면 — 예를 들어, 한 제품 페이지에서 다른 제품 페이지로 전환하는 경우를 생각해보세요.

다시 말해, 렌더링이 서버에서 발생하기 때문에 새로운 RSC 와이어 포맷의 콘텐츠를 얻기 위해 서버에 또 다른 API 호출이 필요합니다.

좋은 소식은, 브라우저가 새로운 콘텐츠를 받으면 새로운 React 요소 트리를 구성하고, 이전 React 트리와의 차이를 비교하여 DOM에 필요한 최소한의 업데이트를 수행할 수 있다는 것입니다.

이 과정에서 클라이언트 컴포넌트의 상태와 이벤트 핸들러는 그대로 유지됩니다.

클라이언트 컴포넌트 입장에서는 이 업데이트가 브라우저 내에서 전적으로 발생한 것과 다르지 않습니다.

현재는 루트 서버 컴포넌트부터 전체 React 트리를 다시 렌더링해야 하지만, 미래에는 서브 트리에 대해서도 이 작업을 수행할 수 있게 될 것입니다.

## RSC에 메타 프레임워크를 사용해야 하는 이유는 무엇인가요?

React 팀은 RSC가 처음에는 Next.js나 Shopify Hydrogen 같은 [메타 프레임워크](https://github.com/josephsavona/rfcs/blob/server-components/text/0000-server-components.md#adoption-strategy)를 통해 도입되도록 의도했다고 말했습니다. 그렇다면 왜 그럴까요? 메타 프레임워크는 어떤 도움을 주나요?

꼭 사용할 필요는 없지만, 사용하면 훨씬 편리합니다.

메타 프레임워크는 친숙한 래퍼(Wrapper)와 추상화를 제공하여 서버에서 RSC 스트림을 생성하고 브라우저에서 이를 소비하는 것에 대해 신경 쓸 필요가 없도록 해줍니다.

메타 프레임워크는 서버 사이드 렌더링도 지원하며, 서버 컴포넌트를 사용할 경우 서버에서 생성된 HTML을 제대로 하이드레이션할 수 있도록 [작업](https://github.com/vercel/next.js/issues/30994)을 [수행](https://github.com/Shopify/hydrogen/pull/250)합니다.

보셨다시피, 브라우저에서 클라이언트 컴포넌트를 올바르게 제공하고 사용하는 데에는 번들러의 협력이 필요합니다.

이미 webpack 통합이 있으며, Shopify는 [vite 통합 작업](https://github.com/facebook/react/pull/22952)을 진행 중입니다.

이러한 플러그인은 RSC에 필요한 많은 부분이 공개 npm 패키지로 게시되지 않았기 때문에 React 저장소의 일부가 되어야 합니다.

그러나 개발이 완료되면, 이러한 구성 요소는 메타 프레임워크 없이도 사용할 수 있어야 합니다.

## RSC가 준비되었나요?

리액트 서버 컴포넌트는 현재 [Next.js의 실험적 기능](https://nextjs.org/docs/app/building-your-application/rendering/server-components)과 [Shopify Hydrogen의](https://hydrogen.shopify.dev/) 현재 개발자 프리뷰로 제공되고 있지만, 어느 것도 아직 프로덕션에서 사용하기에는 준비되지 않았습니다.

앞으로의 블로그 포스트에서 각 프레임워크가 RSC를 어떻게 사용하는지에 대해 자세히 다룰 예정입니다.

하지만 React Server Components가 React의 미래에 큰 부분을 차지할 것이라는 점은 확실합니다.

이는 더 빠른 페이지 로드, 더 작은 자바스크립트 번들, 그리고 더 짧은 인터랙티브 시간에 대한 React의 해답입니다 — React를 사용하여 멀티 페이지 애플리케이션을 구축하는 방법에 대한 더 포괄적인 논제입니다.

아직 준비되지 않았지만, 곧 주목할 시기가 올 것입니다.

## Plasmic의 React Server Components 사용

Plasmic은 콘텐츠 제작자가 랜딩 페이지 및 고성능 React 웹사이트와 스토어프론트의 다른 부분을 구축할 수 있게 해주며, 개발자는 콘텐츠 페이지 작업에서 자유로워지게 합니다.

Plasmic은 기존 React 컴포넌트를 캔버스에 드래그 앤 드롭할 수 있도록 깊이 있는 React 지원을 제공합니다.

React Server Components와 같은 성능 향상 도구는 우리와 고객 모두에게 매우 중요합니다.

도전적인 사용자들을 위해: [이제 Shopify Hydrogen과 함께 작동하는 Plasmic 데모](https://github.com/plasmicapp/hydrogen-demo)가 있습니다.

이는 Shopify Hydrogen 웹사이트 내에서 시각적으로 페이지를 구축하고, 편집기에서 기존 React 컴포넌트를 드래그 앤 드롭할 수 있음을 의미합니다.

또는 [Plasmic](https://studio.plasmic.app/)을 사용하여 자신의 코드베이스에서 시각적으로 구축을 시작해보세요!

이 글의 초기 초안을 검토해주신 Hassan과 Josh에게 많은 감사를 드립니다!
