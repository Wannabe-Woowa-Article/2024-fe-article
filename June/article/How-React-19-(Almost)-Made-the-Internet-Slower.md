# [How React 19 (Almost) Made the Internet Slower](https://blog.codeminer42.com/how-react-19-almost-made-the-internet-slower/?utm_source=newsletter.reactdigest.net&utm_medium=referral&utm_campaign=react-19-and-suspense)

### 🗓️ 번역 날짜: 2024.06.26

### 🧚 번역한 크루: 마스터위(명재위)

---

Henrique Yuji  
Updated On Jun 17, 2024  
Keyword : **Frontend**, **React**

리액트(React)는 여전히 [가장 인기 있고 널리 사용되는](https://gist.github.com/tkrotoff/b1caa4c3a185629299ec234d2314e190) 사용되는 UI 프레임워크로, 넷플릭스, 에어비앤비, 디스코드, 메타(페이스북, 인스타그램, 왓츠앱) 등 대형 웹사이트에서 사용됩니다. 수십억 명이 사용하는 사용자 인터페이스를 만드는 데 사용되므로, 인터넷 트래픽의 상당 부분이 React에 의해 처리된다고 볼 수 있습니다.

올해 초 발표된 기대를 모았던 React 19는 많은 새로운 기능과 개발자 경험(DX) 개선을 포함하고 있지만, 최근까지 주목받지 못한 작은 변경 사항이 있었습니다. 이는 여러 React 기반 웹사이트의 성능을 크게 저하시킬 수 있는 잠재력을 가지고 있었습니다.

모든 것은 다음 트윗에서 시작되었습니다:

<img src="https://i0.wp.com/d604h6pkko9r0.cloudfront.net/wp-content/uploads/2024/06/17094424/tk-dodo-first-post.png?w=1200&ssl=1" width=400>

[@TkDodo](https://x.com/TkDodo/status/1800501040766144676): "내가 상상하는 건가요, 아니면 React 18과 19에서 Suspense가 병렬 페칭을 처리하는 방식에 차이가 있나요? 18에서는 각 컴포넌트가 병렬로 분리되었지만, 19에서는 연속적으로 실행되는 것 같습니다."
이는 두 개의 쿼리를 병렬로 실행하고, 둘 다 해결될 때까지 기다린 다음 전체 서브 트리를 보여줍니다.
React 19에서는 내가 보기엔 쿼리들이 이제 폭포수처럼 실행됩니다. @rickhanlonii가 이런 얘기를 했던 것 같은데 지금은 evidence를 찾을 수 없습니다.

---

<img src="https://i0.wp.com/d604h6pkko9r0.cloudfront.net/wp-content/uploads/2024/06/17094427/adam-rackis-first-post-response.png?w=1200&ssl=1" width=400>

[@AdamRackis](https://x.com/AdamRackis/status/1800588094560772224) : 이는 이해할 수 없는, 미친 변화입니다. 댓글에 따르면 클라이언트 컴포넌트에 적용되는 것 같습니다만, 병렬 페칭은 여전히 RSC에서 작동합니다.
이는 react-query를 망치며, React로 데이터를 관리하는 유일한 좋은 방법입니다.
냉정한 판단이 있기를 바라지만, 그러지 않을 것 같네요.

---

<img src="https://i0.wp.com/d604h6pkko9r0.cloudfront.net/wp-content/uploads/2024/06/17094454/tanner-response.png?w=1200&ssl=1" width=400>

@TkDodo : 이제 확실히 React 19의 suspense와 형제 사전 렌더링 변경 사항에 대해 블로그에 써야겠어요. React 핵심 팀 외에는 이 거래가 옳다고 생각하는 사람을 아직 본 적이 없어요. v19.0.0은 아직 출시되지 않았으니, 그들을 재고하도록 할 수 있는 가능성이 있을지도 몰라요.

---

[@tannerlinsley](https://x.com/tannerlinsley/status/1800903098464096664) : "그래요, 이건 나쁜 결정처럼 느껴져요, 특히 기존 애플리케이션과 사용 사례를 자동으로 더 나쁘게 만들기 때문에."

---

<img src='https://i0.wp.com/d604h6pkko9r0.cloudfront.net/wp-content/uploads/2024/06/17094430/adam-rackis-vercel.png?w=1200&ssl=1' width=400>

[@AdamRackis](https://x.com/AdamRackis/status/1800663066922963264) : 저는 간절히 바라건대 Vercel, 그리고 이제는 더욱 우려되는 *React 핵심 팀*이 모든 사람이 콘텐츠를 가능한 빨리 앞에 보여주어 지루한 사용자가 뒤로 가기 버튼을 누르지 않도록 하는 것이 최우선인 전자 상거래 사이트를 배포하는 것은 아니라는 점을 이해하기를 바랍니다.

---

<br>

도미닉(TkDodo)을 모르는 분들을 위해 설명하자면, 그는 널리 사용되는 TanStack Query의 주요 유지 관리자 중 한 명이며, 전설적인 태너 린즐리(Tanner Linsley)와 함께 일하고 있습니다.

하지만 다시 본론으로 돌아가서, 여기서 논의되는 변경 사항은 React 19에서 동일한 Suspense 경계 내 형제 요소의 병렬 렌더링을 비활성화한다는 것입니다. 이는 형제 요소 내부에서 가져오는 데이터에 대한 데이터 페칭 폭포수를 도입하게 됩니다.

다음은 그 예시입니다:

<img src='https://i0.wp.com/d604h6pkko9r0.cloudfront.net/wp-content/uploads/2024/06/17094440/real-case-waterfall-1.png?w=1200&ssl=1' width=400>

<img src='https://i0.wp.com/d604h6pkko9r0.cloudfront.net/wp-content/uploads/2024/06/17094445/real-case-waterfall-2.png?w=1200&ssl=1' width=400>

가장 큰 문제는 이 변화가 성능 측면에서 중요한 변경 사항이지만, 이 패턴에 의존하는 많은 사람들에게 영향을 미칠 것이라는 점입니다. 그럼에도 불구하고 이 변경 사항이 단 한 줄의 불명확한 글머리로 언급되었다는 점입니다.

<img src='https://i0.wp.com/d604h6pkko9r0.cloudfront.net/wp-content/uploads/2024/06/17094436/react-19-migration.png?w=1200&ssl=1' width=400>

제가 방금 말한 내용이 헷갈리셨다면, 당신만 그런 것이 아닙니다. 저도 처음 이 글을 접했을 때 그렇게 느꼈습니다. 걱정하지 마세요. 곧 모든 것이 이해될 것입니다.

## Suspense 복습

이 내용을 이해하려면 먼저 React의 Suspense에 대해 간략히 복습해야 합니다.

Suspense는 자식 컴포넌트가 로딩을 마칠 때까지 대체 내용을 표시할 수 있게 하는 React 컴포넌트입니다. 자식 컴포넌트가 지연 로드되거나, Suspense를 사용한 데이터 페칭 메커니즘을 사용할 때 유용합니다.

사용 예시는 다음과 같습니다:

```jsx
<Suspense fallback={<Loading />}>
  <ComponentThatFetchesDataOrIsLazyLoaded />
</Suspense>
```

Suspense는 오랫동안 React API의 일부였지만, 공식적으로 승인된 사용법은 React.lazy와 함께 지연 로딩을 사용하는 것이었습니다. 이는 앱을 코드 분할하고 필요한 경우에만 분할된 부분을 로드하는 데 매우 유용합니다.

React.lazy를 사용하면 처음 지연 로드된 컴포넌트를 렌더링하려고 할 때 Suspense 경계를 트리거하고, 컴포넌트 코드를 가져올 때까지 대체 내용을 렌더링한 다음 컴포넌트 자체를 렌더링합니다.

오랫동안 클라이언트 측에서 Suspense를 위한 공식 데이터 페칭 지원이 약속되었지만, 지금까지 실현되지 않았습니다. 그럼에도 불구하고 많은 라이브러리(예: TanStack Query)는 React 내부를 조사하여 이를 구현했습니다. 결과적으로 현재 많은 프로덕션 애플리케이션이 클라이언트 측 데이터 페칭에 Suspense를 사용하고 있습니다.

## 변화 이해하기

현재(React 18.3.1 기준) 동일한 Suspense 경계 내에서 여러 컴포넌트와 함께 Suspense 지원 데이터 페칭 또는 지연 로딩을 사용할 때, React는 첫 번째 형제 컴포넌트가 중단되더라도 모든 형제를 렌더링하려고 시도합니다.

실제로 이는 이러한 형제 컴포넌트 내에서 발생하는 데이터 페칭 또는 지연 로딩이 모두 병렬로 시작된다는 것을 의미합니다.

이 생각에 대한 예시를 보시죠 :

```jsx
function App() {
  return (
    <>
      <Suspense fallback={"Loading..."}>
        <ComponentThatFetchesData val={1} />
        <ComponentThatFetchesData val={2} />
        <ComponentThatFetchesData val={3} />
      </Suspense>
    </>
  );
}

const ComponentThatFetchesData = ({ val }) => {
  const result = fetchSomethingSuspense(val);

  return <div>{result}</div>;
};
```

Demo: https://stackblitz.com/edit/vitejs-vite-x3nv7r?file=src%2FApp.jsx

이 예시에서는 (React 18에서) `fetchSomethingSuspense`가 첫 번째 `ComponentThatFetchesData`를 중단시켜도, React는 여전히 형제 컴포넌트를 렌더링하려고 시도하여 각각의 데이터 페칭이 병렬로 시작됩니다.

이를 확인하기 위해 콘솔에서 각 데이터 페칭이 언제 트리거되는지 로그를 확인할 수 있습니다:

<img src='https://i0.wp.com/d604h6pkko9r0.cloudfront.net/wp-content/uploads/2024/06/17094450/stack-blitz-1.png?w=1200&ssl=1' width=400>

모든 데이터 페칭이 거의 동시에 시작됩니다.

이제 동일한 코드를 React 19 (canary)에서 실행했을 때 어떤 일이 발생하는지 살펴보겠습니다:

Demo: https://stackblitz.com/edit/vitejs-vite-55rddj?file=src%2FApp.jsx

<img src='https://i0.wp.com/d604h6pkko9r0.cloudfront.net/wp-content/uploads/2024/06/17094453/stack-blitz-2.png?w=1200&ssl=1' width=400>

콘솔을 다시 확인해 보면 이제는 폭포수 형태가 나타나며, 각 데이터 페칭이 이전 페칭이 완료된 후에야 시작되는 것을 알 수 있습니다.

이것은 다음 PR에서 일어났습니다 :
https://github.com/facebook/react/pull/26380

이 PR에 의해 도입된 변경 사항 이후, React는 동일한 Suspense 경계 내의 모든 형제 요소를 렌더링하려고 시도하는 대신, 첫 번째로 중단되는 요소에서 멈추게 됩니다. 이러한 경우, 첫 번째 컴포넌트를 렌더링하려고 시도한 후 중단되면, 데이터 페칭이 완료된 후에야 다음 형제 요소로 넘어가고, 다시 중단됩니다.

이 새로운 동작은 데이터 페칭을 위한 Suspense 사용뿐만 아니라, 오래된 패턴으로 더 널리 사용되는 React.lazy 사용에도 영향을 미칩니다.

<img src='https://i0.wp.com/d604h6pkko9r0.cloudfront.net/wp-content/uploads/2024/06/17094434/lazy-loading.png?w=1200&ssl=1' width=400>

[@bentonnnnnn](https://x.com/bentonnnnnn/status/1800940807618171270) : 예상대로 이 변경 사항은 지연 로드된 컴포넌트(react.lazy)에도 영향을 미칩니다.
이제 더 이상 병렬로 가져오지 않기 때문에 로딩 시간은 두 페치의 합이 되며, 이는 큰 후퇴로 간주되어야 합니다.

---

## 근거와 DX 영향

이 변경의 근거는, 실제로 중단되기 전에 모든 형제 요소를 렌더링하려고 시도하는 것은 비용이 들며, 본질적으로 fallback을 표시하는 것을 지연시킨다는 점입니다. 또한 이 변경은 React 팀이 React 18 이전에 Suspense를 도입하면서부터 추진해 온 "fetch as you render" 접근 방식과 일치합니다.

이상적으로는, 컴포넌트 렌더링 시 데이터 페칭을 시작하는 대신, 가능한 빨리 데이터를 가져오는 것이 좋습니다.

비록 성능 면에서는 최상의 접근 방식이지만, 이는 컴포넌트와 데이터 요구 사항을 함께 배치하는 것을 어렵게 만들어 개발자 경험(DX)에 큰 단점을 안겨줍니다.

이 주제에 대해 이미 많은 논의가 있었고, 이 문제를 해결하기 위해 특별히 만들어진 라이브러리도 존재하기 때문에 여기서 깊이 다루지는 않겠습니다. 다만, 이 특정 문제에 대해 이야기하는 몇 가지 트윗을 남기겠습니다.

<img src='https://i0.wp.com/d604h6pkko9r0.cloudfront.net/wp-content/uploads/2024/06/17094418/teemu-taskula-1.png?w=1200&ssl=1' width=400>

[@teemu_taskula](https://x.com/teemu_taskula/status/1800770818097754509) : Dominik의 고통이 느껴지네요 😟 이건 React의 주요 설계 원칙인 composition에 반하는 것 아닌가요? 제가 제대로 이해한 게 맞다면, 이제 복잡한 사전 페칭 트릭 없이는 데이터 요구를 구성할 수 없게 되었습니다. 아니면 모든 데이터 페칭을 공통 부모로 올려야 하죠.
데이터 페칭을 공통 부모로 올려야 한다면 Suspense를 사용하는 의미가 무엇인가요? 저는 비동기 작업 조정을 더 쉽게 만드는 것이 Suspense의 목적이라고 생각했습니다. 만약 view의 루트에서 데이터 페칭을 시작해야 한다면, 차라리 Suspense 없이 데이터를 쿼리하는 것이 낫습니다.

---

여기서 중요한 점은 컴포넌트와 그 데이터 요구 사항의 최적의 성능 특성과 공동 배치를 동시에 달성하는 것이 **컴파일러** 없이 불가능하다는 것입니다. 이는 Relay가 수행하는 작업입니다.

## 결론

다행히도 이 이야기는 해피엔딩으로 끝났습니다. 많은 공개 반발, 열띤 토론, 그리고 아마도 배후에서의 상당한 논의 끝에, React 팀은 이 변경 사항을 보류하기로 결정했습니다.

<img src='https://i0.wp.com/d604h6pkko9r0.cloudfront.net/wp-content/uploads/2024/06/17094448/sophie.png?w=1200&ssl=1' width=400>

[@sophiebits](https://x.com/sophiebits/status/1801663976973209620) : 좋은 소식입니다. Suspense와 관련된 문제에 대해
@rickhanlonii, @en_JS, @acdlite와 만났습니다.

우리는 단일 페이지 애플리케이션(SPA)에 대해 많은 관심을 가지고 있으며, 팀이 오늘날 많은 사람들이 이 기능에 의존하고 있다는 것을 잘못 판단했습니다.
여전히 프리로딩을 권장하지만, 항상 실용적이지 않다는 것을 인식하고 있습니다.
좋은 해결책을 찾을 때까지 React 19.0 릴리스를 보류할 계획입니다.
추가 소식이 있을 예정입니다.

---

이것은 커뮤니티가 Meta와 Vercel 외부에서 React가 어떻게 사용되는지에 대한 고려 없이 도입된 변경 사항에 반발한 첫 번째 사례가 아닙니다. React 팀과 특히 Vercel이 RSCs를 React로 구축하는 기본 요소로 만들려는 추진이 그 한 사례입니다.

React의 유지 관리자들이 React의 미래를 위해 최선이라고 생각하는 것과 커뮤니티의 의견 사이에 불일치가 명확히 존재합니다. 이러한 커뮤니케이션 문제들이 더 심화될지는 아직 지켜봐야 합니다.
