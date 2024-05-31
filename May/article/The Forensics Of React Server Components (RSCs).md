## 🔗 [The Forensics Of React Server Components (RSCs)](https://www.smashingmagazine.com/2024/05/forensics-react-server-components/?utm_source=newsletter.reactdigest.net&utm_medium=newsletter&utm_campaign=the-forensics-of-react-server-components)

### 🗓️ 번역 날짜: 2024.05.26

### 🧚 번역한 크루: 버건디(전태헌)

---

# 리액트 서버 컴포넌트 포렌식하기

서버의 작업 부담을 덜어준다는 점에서 클라이언트 측 렌더링을 좋아하지만, 빈 HTML 페이지를 제공하면 초기 페이지 로드 시 사용자 경험에 부담을 주는 경우가 많습니다.

서버 측 렌더링은 빠른 CDN에서 정적 자산을 제공할 수 있기 때문에 좋아하지만 동적 콘텐츠가 포함된 대규모 프로젝트에는 적합하지 않습니다.

React Server Components(RSCs)는 이러한 두 가지 방식의 장점을 결합한 기술입니다.

Lazar Nikolov은 RSCs가 어떻게 현재의 기술적 위치에 도달했는지, 그리고 리액트 서버 컴포넌트가 페이지 로드 타임라인에 미치는 영향을 심도 있게 분석합니다.

이 기사에서는 React 서버 컴포넌트(RSCs)에 대해 깊이 있게 살펴보겠습니다.

RSCs는 React 생태계의 최신 혁신으로, 서버 사이드 렌더링과 클라이언트 사이드 렌더링을 모두 활용하며 [HTML 스트리밍](https://en.wikipedia.org/wiki/Chunked_transfer_encoding)을 통해 가능한 한 빠르게 콘텐츠를 전달합니다.

우리는 React 서버 컴포넌트(RSCs)에 대해 깊이 있게 이해하기 위해 자세히 탐구할 것입니다.

이를 통해 RSCs가 React 생태계에서 어떤 역할을 하는지, 컴포넌트의 렌더링 라이프사이클에 대해 얼마나 많은 제어를 제공하는지, 그리고 RSCs를 사용할 때 페이지 로드가 어떻게 이루어지는지를 살펴보겠습니다.

그러나 이러한 내용을 깊이 탐구하기 전에, 왜 우리가 React 서버 컴포넌트(RSCs)가 필요한지에 대한 배경을 설정하기 위해 지금까지 React가 웹사이트를 렌더링해온 방식을 되돌아보는 것이 먼저라고 생각합니다.

# 초기 시대: React 클라이언트 사이드 렌더링

초기 React 애플리케이션은 클라이언트 측, 즉 브라우저에서 렌더링되었습니다.

개발자로서 우리는 JavaScript 클래스를 컴포넌트로 사용하여 애플리케이션을 작성하고, Webpack과 같은 번들러를 사용하여 모든 코드를 잘 컴파일하고 트리 쉐이킹하여 프로덕션 환경에 배포할 준비가 된 코드 뭉치로 패키징했습니다.

서버에서 반환된 HTML은 다음과 같은 몇 가지 요소를 포함하고 있었습니다:

- `<head>`에 메타데이터가 포함된 HTML 문서와, 애플리케이션을 DOM에 주입하기 위한 훅으로 사용되는 `<body>`의 빈 `<div>`

- React의 핵심 코드와 웹 애플리케이션의 실제 코드가 포함된 JavaScript 리소스.
  이 리소스들은 사용자 인터페이스를 생성하고 빈 `<div>` 내부에 애플리케이션을 채워 넣었습니다.

![](https://files.smashing.media/articles/forensics-react-server-components/1-client-side-rendering-process.jpg)

이 과정에서 웹 애플리케이션은 JavaScript가 모든 작업을 완료한 후에야 비로소 완전히 상호작용이 가능합니다.

**여기서 개발자 경험(DX)이 개선되지만 동시에 사용자 경험(UX)이 안좋아지는 느낌이 들수 있습니다.**

사실, React의 클라이언트 사이드 렌더링(CSR)에는 장단점이 있었습니다.

긍정적인 측면을 살펴보면, 반응형 컴포넌트 덕분에 페이지 새로고침 없이 사용자 상호작용에 따라 업데이트되므로 웹 애플리케이션은 부드럽고 빠른 전환을 제공하여 페이지 로드 시간을 단축시켰습니다

CSR은 서버 부하를 줄이고, CDN을 통해 자산을 제공할 수 있어, 사용자의 지리적으로 가까운 서버 위치에서 콘텐츠를 제공함으로써 페이지 로드를 더욱 최적화할 수 있습니다.

CSR에는 그리 좋지 않은 결과도 따르는데, 특히 컴포넌트가 독립적으로 데이터를 가져올 수 있어 [워터폴 네트워크 요청](https://blog.sentry.io/fetch-waterfall-in-react/)이 발생해 속도가 크게 저하될 수 있습니다.

이는 UX 측면에서 사소한 불편처럼 들릴 수 있지만, 실제로는 큰 피해를 초래할 수 있습니다.

Eric Bailey의 “[Modern Health, frameworks, performance, and harm](https://ericwbailey.design/published/modern-health-frameworks-performance-and-harm/)” 글은 CSR 작업에 대해 경고를 주는 이야기입니다.

다른 CSR의 부정적인 결과는 그리 심각하지는 않지만 여전히 피해를 초래합니다.

예를 들어, 메타데이터와 빈 `<div>`만 포함된 HTML 문서는 완전히 렌더링된 경험을 제공하지 못합니다.

그리하여 검색 엔진 크롤러가 이를 판독하는 것이 불가능했습니다.

오늘날에는 이 문제가 해결되었지만, 당시에는 검색 엔진 트래픽에 의존해 수익을 창출하는 회사 사이트에 SEO(검색 엔진 최적화) 영향을 미쳐 큰 제약이 되었습니다.

# 변화: 서버 사이드 렌더링(SSR)

변화가 필요했습니다.

CSR은 개발자에게 빠르고 상호작용이 가능한 인터페이스를 구축하는 강력한 새로운 접근 방식을 제공했지만, 사용자들은 빈 화면과 로딩 표시를 마주하는 상황이 많았습니다.

이에 대한 해결책은 렌더링을 클라이언트에서 서버로 이동시키는 것이었습니다.

과거의 방식으로 돌아가서 개선해야 했다는 것이 흥미롭게 들릴 수 있습니다.

그래서 React는 서버 사이드 렌더링(SSR) 기능을 갖추게 되었습니다.

[한때 SSR은 React 커뮤니티에서 큰 화제가 되어 주목받기도 했습니다.](https://sentry.io/resources/moving-to-server-side-rendering/)

SSR로의 전환은 애플리케이션 개발에 중요한 변화를 가져왔으며, 특히 React의 동작 방식과 콘텐츠가 브라우저가 아닌 서버를 통해 전달되는 방식에 큰 영향을 미쳤습니다.

![](https://files.smashing.media/articles/forensics-react-server-components/2-diagram-server-side-rendering-process.jpg)

# CSR의 한계 해결

SSR을 사용하면 빈 HTML 문서를 보내는 대신, 초기 HTML을 서버에서 렌더링하여 브라우저로 전송했습니다.

이를 통해 브라우저는 로딩 표시기를 표시하지 않고 즉시 콘텐츠를 표시할 수 있었습니다.

이 방식은 Web Vitals에서 중요한 성능 지표인 [First Contentful Paint(FCP)를 크게 향상시킵니다.](https://docs.sentry.io/product/performance/web-vitals/web-vitals-concepts/#first-paint-fp)

서버 사이드 렌더링(SSR)은 CSR과 함께 발생했던 SEO 문제도 해결했습니다.

크롤러가 웹사이트의 콘텐츠를 직접 받기 때문에 즉시 인덱싱할 수 있었습니다.

초기 데이터 페칭도 서버에서 이루어지는데, 이는 데이터 소스와 가까워서 적절히 수행되면 [워터폴 페칭 문제를 제거할 수 있는 장점](https://blog.sentry.io/fetch-waterfall-in-react/#fetch-data-on-server-to-avoid-a-fetch-waterfall)이 있습니다.

# - Hydration

SSR에는 자체적인 복잡성이 있습니다.

서버에서 받은 정적 HTML을 React가 상호작용 가능하게 만들기 위해서는 "Hydration"이 필요합니다.

Hydration은 React가 초기 HTML의 DOM에 있는 내용을 기반으로 클라이언트 측에서 가상 DOM(Virtual DOM)을 재구성하는 과정입니다.

참고: React는 실제 DOM 대신 [가상 DOM(Virtual DOM)](https://legacy.reactjs.org/docs/faq-internals.html)을 유지합니다.

이는 실제 DOM보다 가상 DOM에서 업데이트를 파악하는 것이 더 빠르기 때문입니다.

React는 UI를 업데이트할 때 가상 DOM에서 차이(diff)를 계산하고, 실제 DOM을 가상 DOM과 동기화합니다.

현재 우리는 두 가지 유형의 React를 갖게 되었습니다:

1. 서버 사이드 React: 컴포넌트 트리에서 정적 HTML을 렌더링할 수 있는 버전.

2. 클라이언트 사이드 React: 페이지를 상호작용 가능하게 만드는 버전.

우리는 여전히 React와 애플리케이션 코드를 브라우저에 전송하고 있습니다.

왜냐하면 초기 HTML을 수화(hydrate)하려면 클라이언트 측에서도 서버에서 사용한 동일한 컴포넌트가 필요하기 때문입니다.

hydration 과정에서 React는 [재조정(reconciliation)](https://css-tricks.com/how-react-reconciliation-works/)이라는 과정을 수행하는데, 이는 서버에서 렌더링된 DOM과 클라이언트에서 렌더링된 DOM을 비교하여 두 DOM 간의 차이점을 식별하려고 시도합니다.

만약 두 DOM 간에 차이점이 있다면, React는 컴포넌트 트리를 다시 hydration하고, 서버에서 렌더링된 구조와 일치하도록 컴포넌트 계층을 업데이트하여 문제를 해결하려고 합니다.

만약 해결할 수 없는 불일치가 여전히 존재한다면, React는 문제를 나타내는 오류를 발생시킵니다.

이 문제는 일반적으로 `hydration error`로 알려져 있습니다.

# SSR의 단점

SSR은 CSR의 한계를 해결하는 만능 솔루션이 아닙니다.

SSR 자체에도 단점이 있습니다.

초기 HTML 렌더링과 데이터 페칭을 서버로 옮겼기 때문에, 이제 서버는 모든 것을 클라이언트에서 로드할 때보다 훨씬 더 큰 부하를 경험하게 됩니다.

SSR이 일반적으로 First Contentful Paint(FCP) 성능 지표를 개선한다고 언급한 것을 기억하시나요?

이는 사실이지만, SSR로 인해 [Time to First Byte(TTFB)](https://docs.sentry.io/product/performance/web-vitals/web-vitals-concepts/#first-contentful-paint-fcp) 성능 지표는 부정적인 영향을 받습니다.

브라우저는 서버가 필요한 데이터를 페칭하고, 초기 HTML을 생성하여 첫 번째 바이트를 전송할 때까지 실제로 기다려야 합니다.

TTFB 자체는 핵심 웹 바이탈(Core Web Vitals) 지표는 아니지만, 다른 지표에 영향을 미칩니다.

TTFB가 부정적이면 Core Web Vitals 지표도 부정적인 영향을 받게 됩니다.

SSR의 또 다른 단점은 클라이언트 측 React가 hydration를 완료하기 전까지 전체 페이지가 응답하지 않는다는 점입니다.

상호작용 요소는 React가 이를 hydration하기 전까지, 즉 React가 의도된 이벤트 리스너를 연결하기 전까지 사용자 상호작용에 응답할 수 없습니다.

hydration 과정은 일반적으로 빠르지만, 인터넷 연결 속도와 사용 중인 장치의 하드웨어 성능에 따라 렌더링 속도가 눈에 띄게 느려질 수 있습니다.

# 현재: 하이브리드 접근법

지금까지 우리는 React 렌더링의 두 가지 방식인 CSR(클라이언트 사이드 렌더링)과 SSR(서버 사이드 렌더링)에 대해 다루었습니다.

두 가지 방식은 서로를 개선하려는 시도로 등장했지만, 이제는 CSR과 SSR의 단점을 줄이고 장점을 모두 살리기 위해 SSR이 세 가지 추가적인 React 방식으로 발전하게 되었습니다.

이제 첫 번째와 두 번째 방식인 **정적 사이트 생성(Static Site Generation 즉, SSG)**과 **증분 정적 재생성(Incremental Static Regeneration 즉, ISR)**에 대해 살펴본 후, React 서버 컴포넌트라는 세 번째 방식에 대해 전체적으로 논의해보겠습니다.

## 정적 사이트 생성 (SSG)

매 요청마다 동일한 HTML 코드를 다시 생성하는 대신, 우리는 SSG(정적 사이트 생성)를 고안해냈습니다.

이 React 방식은 빌드 시점에 전체 애플리케이션을 컴파일하고 빌드하여 정적(순수 HTML과 CSS) 파일을 생성합니다.

그런 다음 이러한 파일들은 빠른 CDN(Content Delivery Network)에 호스팅됩니다.

이 하이브리드 렌더링 방식은 콘텐츠 변화가 적은 마케팅 사이트나 개인 블로그에 적합합니다.

이는 사용자 상호작용으로 인해 콘텐츠가 자주 변경되는 대규모 프로젝트, 예를 들어 전자상거래 사이트와는 맞지 않습니다.

SSG는 서버의 부담을 줄이는 동시에 TTFB(Time To First Byte)와 관련된 성능 지표를 개선합니다.

왜냐하면 서버가 페이지를 다시 렌더링하기 위해 무겁고 비용이 많이 드는 작업을 수행할 필요가 없어지기 때문입니다.

## 증분 정적 재생성 (ISR)

SSG(정적 사이트 생성)의 한 가지 단점은 콘텐츠 변경이 필요할 때 애플리케이션의 모든 코드를 다시 빌드해야 한다는 점입니다.

콘텐츠는 고정되어 있어(정적이므로) 그 일부만 변경할 수 없고 전체를 다시 빌드해야 합니다.

이 문제를 해결하기 위해 Next.js 팀은 두 번째 하이브리드 React 방식을 만들어냈습니다.

**증분 정적 재생성(ISR, Incremental Static Regeneration)**

ISR은 이름 그대로 전체를 다시 빌드하는 대신 필요한 부분만 재생성하는 접근 방식입니다.

빌드 시점에 페이지의 "초기 버전"을 정적으로 생성하지만, 사용자가 방문한 후 데이터가 오래된 페이지는 서버 요청이 데이터 검사를 트리거하여 다시 빌드할 수 있습니다.

그 시점부터 서버는 필요할 때마다 새 버전의 페이지를 점진적으로 정적으로 제공하게 됩니다. ISR은 SSG와 전통적인 SSR 사이에 위치한 하이브리드 접근 방식입니다.

하지만 ISR은 "오래된 콘텐츠" 문제를 해결하지 못합니다.

사용자가 페이지를 방문할 때 해당 페이지가 아직 생성 중일 수 있습니다.

SSG와 달리 ISR은 사용자의 브라우저가 서버 요청을 할 때 개별 페이지를 다시 생성하기 위해 실제 서버가 필요합니다.

이는 ISR 기반 앱을 CDN에 배포하여 최적화된 자료들을 전달하는 능력을 잃게 됨을 의미합니다.

## - 미래: 리액트 서버 컴포넌트

지금까지 우리는 CSR(클라이언트 사이드 렌더링), SSR(서버 사이드 렌더링), SSG(정적 사이트 생성), ISR(증분 정적 재생성) 접근 방식을 사용해왔습니다.

이들은 모두 어느 정도의 트레이드오프를 가지며, 성능, 개발 복잡도, 사용자 경험에 부정적인 영향을 미칠 수 있습니다.

새롭게 도입된 [React 서버 컴포넌트(RSC, React Server Components)](https://nextjs.org/docs/app/building-your-application/rendering/server-components)는 이러한 단점 대부분을 해결하려고 합니다. RSC는 각 개별 React 컴포넌트에 대해 적절한 렌더링 전략을 선택할 수 있게 해줍니다.

RSC는 클라이언트로 전송되는 JavaScript의 양을 크게 줄일 수 있습니다.

서버에서 정적으로 제공할 컴포넌트와 클라이언트 측에서 렌더링할 컴포넌트를 선택적으로 결정할 수 있기 때문입니다.

이를 통해 특정 프로젝트에 맞는 적절한 균형을 찾기 위한 더 많은 제어와 유연성을 제공합니다.

> 알아두어야 할점
> 참고: RSC와 같은 더 발전된 아키텍처를 채택할 때 [모니터링 솔루션](https://docs.sentry.io/product/performance/)은 매우 중요합니다.
> Sentry는 RSC 기반 애플리케이션의 실제 성능을 모니터링하고 오류를 추적할 수 있는 강력한 성능 모니터링 및 오류 추적 기능을 제공합니다.
> Sentry는 또한 릴리스가 어떻게 성능을 발휘하고 얼마나 안정적인지에 대한 통찰을 제공하여, 기존 애플리케이션을 RSC로 마이그레이션하는 동안 중요한 기능을 제공합니다.
> [Next.js](https://sentry.io/for/nextjs/)와 같은 RSC 지원 프레임워크에서 Sentry를 구현하는 것은 단일 터미널 명령을 실행하는 것만큼 간단합니다.

그러나 서버 컴포넌트라는 것은 정확히 무엇일까요 ?

한번 내부적으로 어떻게 동작하는지 살펴보겠습니다.

## - 리액트 서버 컴포넌트의 구조

이 새로운 접근법은 두 가지 유형의 렌더링 컴포넌트를 도입합니다.

바로 **서버 컴포넌트(Server Components)**와 **클라이언트 컴포넌트(Client Components)**입니다.

이 두 컴포넌트의 차이점은 기능이 아닌 실행되는 위치와 설계된 환경입니다.

이 글을 작성하는 시점에서 RSC를 사용하는 유일한 방법은 React 프레임워크를 통해서입니다.

현재 RSC를 지원하는 프레임워크는 [Next.js](https://nextjs.org/docs/app/building-your-application/rendering/server-components), [Gatsby](https://www.gatsbyjs.com/docs/conceptual/partial-hydration/), [RedwoodJS](https://redwoodjs.com/blog/rsc-now-in-redwoodjs) 세 가지입니다.

![](https://files.smashing.media/articles/forensics-react-server-components/3-wire-diagram-server-client-components.jpg)

## - 서버 컴포넌트

서버 컴포넌트(Server Components)는 서버에서 실행되도록 설계되었으며, 그 코드는 브라우저로 전송되지 않습니다.

HTML 출력과 그들이 받을 수 있는 props만 제공됩니다.

이 접근 방식은 여러 가지 성능상의 이점과 사용자 경험 향상을 가져옵니다:

- **서버 컴포넌트(Server Components)는 큰 의존성을 서버 측에 남겨둘 수 있습니다.**

큰 라이브러리를 컴포넌트에 사용하는 경우를 상상해보세요.

컴포넌트를 클라이언트 측에서 실행하면 전체 라이브러리를 브라우저로 전송해야 합니다.

하지만 서버 컴포넌트를 사용하면 정적 HTML 출력만 가져가고 JavaScript는 브라우저로 전송할 필요가 없습니다.

서버 컴포넌트는 진정으로 정적이며, hydration 단계를 제거합니다.

- **서버 컴포넌트는 코드 생성을 위해 필요한 데이터 소스(예: 데이터베이스 또는 파일 시스템)와 훨씬 가깝게 위치합니다.**

또한 서버의 계산 능력을 활용하여 계산 집약적인 렌더링 작업을 가속화하고, 생성된 결과만 클라이언트에 전송합니다.

서버 컴포넌트는 한 번에 생성되므로 [요청 폭포(request waterfalls)와 HTTP 왕복](https://blog.sentry.io/fetch-waterfall-in-react/#fetch-data-on-server-to-avoid-a-fetch-waterfall)을 피할 수 있습니다.

- **서버 컴포넌트(Server Components)는 민감한 데이터와 로직을 브라우저에서 안전하게 보호합니다.**

이는 개인 토큰과 API 키가 클라이언트가 아닌 안전한 서버에서 실행되기 때문입니다.

- **렌더링 결과는 캐시되어 이후 요청 간 및 다른 세션 간에도 재사용될 수 있습니다.**

이는 렌더링 시간을 크게 줄이며, 각 요청마다 가져오는 전체 데이터 양도 줄입니다.

이 아키텍처는 **HTML 스트리밍**도 사용합니다.

즉, 서버는 특정 컴포넌트에 대한 HTML 생성을 지연시키고, 대신 생성된 HTML을 보내는 동안 해당 컴포넌트의 자리에는 대체 요소를 렌더링합니다.

스트리밍 서버 컴포넌트는 [<Suspense>](https://react.dev/reference/react/Suspense) 태그로 컴포넌트를 감싸 대체 값을 제공합니다.

구현 프레임워크는 초기에는 이 대체 요소를 사용하지만, 새로 생성된 콘텐츠가 준비되면 이를 스트리밍합니다.

스트리밍에 대해 더 자세히 이야기하겠지만, 먼저 클라이언트 컴포넌트와 서버 컴포넌트를 비교해보겠습니다.

## - 클라이언트 컴포넌트

클라이언트 컴포넌트(Client Components)는 우리가 이미 잘 알고 사랑하는 컴포넌트입니다.

이들은 클라이언트 측에서 실행됩니다.

따라서 클라이언트 컴포넌트는 사용자 상호작용을 처리할 수 있고, `localStorage`나 지리 위치(geolocation)와 같은 브라우저 API에 접근할 수 있습니다.

"클라이언트 컴포넌트"라는 용어는 새로운 것을 설명하는 것이 아닙니다.

단지 이 용어는 기존의 CSR(Client-Side Rendering) 컴포넌트와 서버 컴포넌트(Server Components)를 구별하기 위해 사용됩니다.

클라이언트 컴포넌트는 파일 상단에 ["use client"](https://react.dev/reference/rsc/use-server) 지시어를 포함하여 정의됩니다.

```tsx
"use client";
export default function LikeButton() {
  const likePost = () => {
    // ...
  };
  return <button onClick={likePost}>Like</button>;
}
```

Next.js에서는 모든 컴포넌트가 기본적으로 서버 컴포넌트(Server Components)로 간주됩니다.

그래서 클라이언트 컴포넌트(Client Components)를 명시적으로 정의하기 위해 "use client" 지시어를 사용해야 합니다.

또한 "use server" 지시어도 있지만, 이는 서버 액션(Server Actions)에 사용됩니다.

서버 액션은 클라이언트에서 호출되지만 서버에서 실행되는 RPC(원격 프로시저 호출)와 유사한 동작을 합니다.

서버 컴포넌트를 정의할 때는 이 지시어를 사용하지 않습니다.

클라이언트 컴포넌트는 클라이언트에서만 렌더링된다고 생각할 수 있지만, Next.js는 초기 HTML을 생성하기 위해 서버에서 클라이언트 컴포넌트를 렌더링합니다.

그 결과 브라우저는 클라이언트 컴포넌트를 즉시 렌더링할 수 있으며, 이후에 하이드레이션(hydration)을 수행하게 됩니다.

## 서버 컴포넌트와 클라이언트 컴포넌트의 관계

클라이언트 컴포넌트는 다른 클라이언트 컴포넌트만 명시적으로 가져올 수 있습니다.

즉, 클라이언트 컴포넌트에서 서버 컴포넌트를 가져올 수 없습니다.

이는 재렌더링 문제 때문입니다.

그러나 클라이언트 컴포넌트의 서브 트리에 서버 컴포넌트를 포함할 수는 있습니다.

이는 children prop을 통해 전달됩니다.

클라이언트 컴포넌트는 브라우저에서 실행되며 사용자 상호작용을 처리하거나 자체 상태를 정의하기 때문에 자주 재렌더링됩니다.

클라이언트 컴포넌트가 재렌더링될 때, 그 서브 트리도 재렌더링됩니다.

하지만 서브 트리에 서버 컴포넌트가 포함되어 있다면 어떻게 재렌더링될까요?

서버 컴포넌트는 클라이언트 측에서 실행되지 않습니다.

이러한 이유로 React 팀은 이러한 제한을 두었습니다.

하지만 잠깐만요! 사실 클라이언트 컴포넌트에서 서버 컴포넌트를 가져올 수 있습니다.

다만, 이는 일대일 관계가 아니기 때문에 서버 컴포넌트가 클라이언트 컴포넌트로 변환됩니다.

만약 브라우저에서 사용할 수 없는 서버 API를 사용 중이라면 에러가 발생할 것이며, 그렇지 않다면 서버 컴포넌트의 코드가 브라우저로 "노출"됩니다.

이것은 RSCs(React Server Components)를 작업할 때 매우 중요한 차이점입니다.

이 미묘한 점을 반드시 염두에 두어야 합니다.

## 렌더링 라이프사이클

다음은 Next.js가 콘텐츠를 스트리밍하는 순서입니다:

1. 앱 라우터는 페이지의 URL을 서버 컴포넌트(Server Component)에 매칭시키고, 컴포넌트 트리를 구축한 후, 서버 측 React에 서버 컴포넌트와 모든 자식 컴포넌트를 렌더링하라고 지시합니다.

2. 렌더링 중에 React는 "RSC 페이로드(RSC Payload)"를 생성합니다. RSC 페이로드는 페이지에 대한 정보와 반환될 내용을 Next.js에 알리며, <Suspense> 동안 대체할 내용을 포함합니다.

3. React가 서스펜드된 컴포넌트를 만나면, 해당 서브 트리의 렌더링을 일시 중지하고 서스펜드된 컴포넌트의 대체 값을 사용합니다.

4. React가 마지막 정적 컴포넌트를 처리하면, Next.js는 생성된 HTML과 RSC 페이로드를 준비하여 하나 이상의 청크로 나누어 클라이언트로 스트리밍합니다.

5. 클라이언트 측 React는 RSC 페이로드와 클라이언트 측 컴포넌트에 대한 지시 사항을 사용하여 UI를 렌더링합니다. 또한, 클라이언트 컴포넌트가 로드될 때마다 이를 하이드레이션(hydration)합니다.

6. 서버는 서스펜드된 서버 컴포넌트가 사용 가능해지면 이를 RSC 페이로드로 스트리밍합니다. 클라이언트 컴포넌트의 자식 컴포넌트도 이때 하이드레이션됩니다(서스펜드된 컴포넌트가 포함된 경우).

RSC 렌더링 라이프사이클을 브라우저 관점에서 잠시 후에 살펴보겠습니다.

다음 그림이 우리가 다룬 단계를 설명합니다.

![](https://files.smashing.media/articles/forensics-react-server-components/4-wire-diagram-rsc-rendering-lifecycle.jpg)

# RSC 페이로드

RSC 페이로드는 서버가 컴포넌트 트리를 렌더링할 때 생성하는 특별한 데이터 형식으로, 다음을 포함합니다:

- 렌더링된 HTML

- 클라이언트 컴포넌트가 렌더링될 자리 표시자

- 클라이언트 컴포넌트의 자바스크립트 파일에 대한 참조

- 호출해야 하는 자바스크립트 파일에 대한 지시 사항

- 서버 컴포넌트에서 클라이언트 컴포넌트로 전달되는 모든 props.

RSC 페이로드에 대해 크게 걱정할 필요는 없지만, RSC 페이로드에 무엇이 포함되는지 이해할 필요는 있습니다.

[제가 만든 데모 앱](https://github.com/nikolovlazar/rsc-forensics)의 예시를 살펴보겠습니다. (간결하게 하기 위해 일부를 생략했습니다).

```ts
1:HL["/_next/static/media/c9a5bc6a7c948fb0-s.p.woff2","font",{"crossOrigin":"","type":"font/woff2"}]
2:HL["/_next/static/css/app/layout.css?v=1711137019097","style"]
0:"$L3"
4:HL["/_next/static/css/app/page.css?v=1711137019097","style"]
5:I["(app-pages-browser)/./node_modules/next/dist/client/components/app-router.js",["app-pages-internals","static/chunks/app-pages-internals.js"],""]
8:"$Sreact.suspense"
a:I["(app-pages-browser)/./node_modules/next/dist/client/components/layout-router.js",["app-pages-internals","static/chunks/app-pages-internals.js"],""]
b:I["(app-pages-browser)/./node_modules/next/dist/client/components/render-from-template-context.js",["app-pages-internals","static/chunks/app-pages-internals.js"],""]
d:I["(app-pages-browser)/./src/app/global-error.jsx",["app/global-error","static/chunks/app/global-error.js"],""]
f:I["(app-pages-browser)/./src/components/clearCart.js",["app/page","static/chunks/app/page.js"],"ClearCart"]
7:["$","main",null,{"className":"page_main__GlU4n","children":[["$","$Lf",null,{}],["$","$8",null,{"fallback":["$","p",null,{"children":"🌀 loading products..."}],"children":"$L10"}]]}]
c:[["$","meta","0",{"name":"viewport","content":"width=device-width, initial-scale=1"}]...
9:["$","p",null,{"children":["🛍️ ",3]}]
11:I["(app-pages-browser)/./src/components/addToCart.js",["app/page","static/chunks/app/page.js"],"AddToCart"]
10:["$","ul",null,{"children":[["$","li","1",{"children":["Gloves"," - $",20,["$...
```

데모 앱에서 이 코드를 찾으려면, 브라우저의 개발자 도구에서 Elements 탭을 열고 페이지 하단의 `<script> 태그`를 확인하세요.

거기에는 다음과 같은 줄이 포함되어 있을 것입니다:

```ts
self.__next_f.push([1,"PAYLOAD_STRING_HERE"]).
```

위의 스니펫의 각 줄은 개별 RSC 페이로드입니다.

각 줄은 숫자 또는 문자로 시작하고, 콜론이 뒤따르며, 때로는 문자로 시작하는 배열이 이어집니다. 이들이 의미하는 바를 깊이 들어가진 않겠지만, 일반적으로:

- HL 페이로드는 "힌트"라고 불리며, CSS나 폰트와 같은 특정 리소스에 링크합니다.

- I 페이로드는 "모듈"이라고 불리며, 특정 스크립트를 호출합니다.

이는 클라이언트 컴포넌트가 로드되는 방식이기도 합니다.

클라이언트 컴포넌트가 메인 번들의 일부이면, 즉시 실행됩니다.

그렇지 않다면(즉, lazy loading 되는 경우), 메인 번들에 페처(fetcher) 스크립트가 추가되어 렌더링 시 컴포넌트의 CSS와 자바스크립트 파일을 가져옵니다.

서버에서 필요한 경우 페처 스크립트를 호출하는 I 페이로드가 전송됩니다.

- "$" 페이로드는 특정 서버 컴포넌트에 대해 생성된 DOM 정의입니다.

이는 보통 서버에서 스트리밍된 실제 정적 HTML과 함께 제공됩니다.

서스펜드된 컴포넌트가 렌더링 준비가 되었을 때 서버가 해당 컴포넌트의 정적 HTML과 RSC 페이로드를 생성하여 브라우저로 스트리밍하는 방식입니다.

## 스트리밍

스트리밍은 서버에서 UI를 점진적으로 렌더링할 수 있게 해줍니다.

RSCs를 사용하면 각 컴포넌트가 자체 데이터를 가져올 수 있습니다.

일부 컴포넌트는 완전히 정적이며 클라이언트로 즉시 전송될 준비가 되어 있는 반면, 다른 컴포넌트는 로드 전에 더 많은 작업이 필요합니다.

이를 기반으로 Next.js는 작업을 여러 청크로 나누어 준비되는 대로 브라우저로 스트리밍합니다.

따라서 사용자가 페이지를 방문하면 서버는 모든 서버 컴포넌트를 호출하고, 페이지의 초기 HTML(즉, 페이지 셸)을 생성하며, "서스펜드된" 컴포넌트의 내용을 대체 값으로 바꾼 후, 이를 한 개 이상의 청크로 클라이언트로 스트리밍합니다.

서버는 Transfer-Encoding: chunked 헤더를 반환하여 브라우저가 스트리밍 HTML을 기대하도록 합니다.

이는 브라우저가 문서의 여러 청크를 수신하면서 렌더링할 준비를 하게 합니다.

개발자 도구의 네트워크 탭을 열고 새로 고침을 트리거한 후 문서 요청을 클릭하면 실제로 이 헤더를 볼 수 있습니다.

![](https://files.smashing.media/articles/forensics-react-server-components/5-streaming-header.jpeg)

터미널에서 curl 명령어를 사용하여 Next.js가 청크를 전송하는 방식을 디버깅할 수도 있습니다:

```
curl -D - --raw localhost:3000 > chunked-response.txt
```

![](https://files.smashing.media/articles/forensics-react-server-components/6-chunked-response.jpeg)

아마 패턴을 보셨을 겁니다.

각 청크에 대해 서버는 청크의 내용을 전송하기 전에 청크의 크기를 응답합니다.

출력을 보면 서버가 전체 페이지를 16개의 서로 다른 청크로 스트리밍했다는 것을 알 수 있습니다.

마지막에는 크기가 0인 청크를 보내 스트림의 끝을 알립니다.

첫 번째 청크는 `<!DOCTYPE html>` 선언으로 시작합니다.

끝에서 두 번째 청크는 닫는 `</body>`와 `</html>` 태그를 포함합니다.

따라서 서버가 전체 문서를 위에서 아래로 스트리밍한 후 서스펜드된 컴포넌트를 기다리기 위해 일시 중지하고, 마지막에 본문과 HTML을 닫는 태그를 전송한 후 스트리밍을 멈추는 것을 볼 수 있습니다.

서버가 문서 스트리밍을 완전히 끝내지 않았더라도, 브라우저의 내결함성 기능 덕분에 닫는 `</body>`와 `</html>` 태그를 기다리지 않고도 현재 가지고 있는 내용을 그리거나 실행할 수 있습니다.

## 컴포넌트 서스펜딩

렌더링 라이프사이클에서 배운 바에 따르면, 페이지가 방문될 때 Next.js는 해당 페이지의 RSC 컴포넌트를 매칭하고 React에게 서브 트리를 HTML로 렌더링하라고 요청합니다.

React가 서스펜드된 컴포넌트(즉, 비동기 함수 컴포넌트)를 만나면 `<Suspense>` 컴포넌트(또는 Next.js 경로의 경우 loading.js 파일)에서 대체 값을 가져와 이를 대신 렌더링하고, 다른 컴포넌트 로딩을 계속합니다.

이와 동시에 RSC는 백그라운드에서 비동기 컴포넌트를 호출하고, 이는 로딩이 완료되면 나중에 스트리밍됩니다.

이 시점에서 Next.js는 컴포넌트 자체(정적 HTML로 렌더링된) 또는 대체 값(서스펜드된 경우)을 포함한 정적 HTML의 전체 페이지를 반환합니다.

그런 다음 이 정적 HTML과 RSC 페이로드를 하나 이상의 청크로 나누어 브라우저로 스트리밍합니다.

![](https://files.smashing.media/articles/forensics-react-server-components/7-fallbacks-suspended-components.jpeg)

서스펜드된 컴포넌트의 로딩이 완료되면, React는 다른 중첩된 `<Suspense>`경계를 찾으면서 재귀적으로 HTML을 생성하고, 해당 RSC 페이로드를 생성한 후, Next.js가 HTML과 RSC 페이로드를 새로운 청크로 브라우저에 스트리밍하도록 합니다.

브라우저가 새로운 청크를 수신하면, 필요한 HTML과 RSC 페이로드를 확보하여 DOM에서 대체 요소를 새로 스트리밍된 HTML로 교체할 준비를 합니다.

이런 방식으로 계속 진행됩니다.

![](https://files.smashing.media/articles/forensics-react-server-components/8-suspended-components-html.jpeg)

위 그림 7과 8에서, 대체 요소가 B:0, B:1 등의 고유 ID를 가지고 있는 반면 실제 컴포넌트는 S:0, S:1 등의 유사한 형태의 ID를 가지고 있음을 주목하세요.

서스펜드된 컴포넌트의 HTML을 포함하는 첫 번째 청크와 함께, 서버는 $RC 함수(예: React 소스 코드의 completeBoundary)도 전송합니다.

이 함수는 DOM에서 B:0 대체 요소를 찾아 서버에서 받은 S:0 템플릿으로 교체하는 방법을 알고 있습니다.

이 "교체" 함수는 브라우저에 도착한 컴포넌트 내용을 볼 수 있게 해줍니다.

결국 전체 페이지는 청크 단위로 완전히 로드됩니다.

## - 레이지 로딩(Lazy Loading) 컴포넌트

서버 컴포넌트가 지연 로드된 클라이언트 컴포넌트를 포함하고 있는 경우, Next.js는 지연 로드된 컴포넌트의 코드를 가져오고 로드하는 방법에 대한 지시사항을 포함하는 RSC 페이로드 청크도 전송합니다.

이는 페이지 로드가 자바스크립트에 의해 지연되지 않기 때문에 성능이 크게 향상됩니다. 해당 자바스크립트는 그 세션 동안 로드되지 않을 수도 있습니다.

![](https://files.smashing.media/articles/forensics-react-server-components/9-fetching-lazy-loaded-scripts.jpeg)

제가 이 글을 쓰는 시점에서, Next.js의 서버 컴포넌트에서 클라이언트 컴포넌트를 지연 로드하는 동적 방법은 기대한 대로 작동하지 않습니다.

클라이언트 컴포넌트를 효과적으로 지연 로드하려면, 클라이언트 컴포넌트를 실제로 지연 로드하는 동적 방법을 사용하는 ["래퍼" 클라이언트 컴포넌트](https://github.com/nikolovlazar/rsc-forensics/blob/main/src/components/addToCartWrapper.js) 안에 넣어야 합니다.

이 래퍼는 클라이언트 컴포넌트의 자바스크립트와 CSS 파일을 필요한 시점에 가져오고 로드하는 스크립트로 변환됩니다.

## TL;DR

여러 요소들이 다양한 시간에 동시에 움직이고 있다는 것은 이해하기 어려울 수 있습니다.

하지만 핵심은, 페이지 방문 시 Next.js가 가능한 많은 HTML을 렌더링하고, 지연된 컴포넌트에 대한 대체값을 사용하여 브라우저로 전송한다는 것입니다.

그 동안 Next.js는 지연된 비동기 컴포넌트를 트리거하여, 이를 HTML 형식으로 변환하고, 브라우저로 스트리밍되는 RSC 페이로드에 포함시킵니다.

이와 함께 $RC 스크립트도 전송되어 어떻게 요소들을 교체할지 알고 있습니다.

## 페이지 로드의 타임라인

이제 RSC가 어떻게 작동하는지, Next.js가 이를 어떻게 렌더링하는지, 그리고 모든 구성 요소들이 어떻게 맞물려 돌아가는지에 대해 충분히 이해했을 것입니다.

이번 섹션에서는 브라우저에서 RSC 페이지를 방문할 때 실제로 어떤 일이 일어나는지 자세히 살펴보겠습니다.

## 초기 로드

앞서 TL;DR 섹션에서 언급했듯이, 페이지를 방문할 때 Next.js는 초기 HTML을 렌더링하고 지연된 컴포넌트를 제외한 상태로 브라우저에 첫 번째 스트리밍 청크의 일부로 전송합니다.

페이지 로드 중 일어나는 모든 과정을 확인하기 위해, Chrome DevTools의 “Performance” 탭을 열고 “reload” 버튼을 클릭하여 페이지를 다시 로드하고 프로파일을 캡처해 보겠습니다.

아래는 그 과정의 모습입니다.

![](https://files.smashing.media/articles/forensics-react-server-components/10-first-chunks-being-streamed.jpeg)

처음 부분을 자세히 살펴보면, 첫 번째 "Parse HTML" 구간을 볼 수 있습니다.

이는 서버가 문서의 첫 번째 청크를 브라우저로 스트리밍하는 것입니다.

브라우저는 페이지의 기본 틀과 폰트, CSS 파일, 자바스크립트 등의 리소스 링크를 포함하는 초기 HTML을 막 수신한 상태입니다.

이때 브라우저는 스크립트를 호출하기 시작합니다.

![](https://files.smashing.media/articles/forensics-react-server-components/11-first-frames.jpeg)

어느 정도 시간이 지나면, 페이지의 첫 번째 프레임들이 나타나기 시작하고 초기 자바스크립트 스크립트가 로드되면서 하이드레이션이 이루어집니다.

프레임을 자세히 보면, 페이지의 전체 틀은 렌더링되어 있으며, 지연된 서버 컴포넌트가 있는 곳에는 "로딩" 컴포넌트가 사용되고 있음을 알 수 있습니다.

이것은 약 800ms 즈음에 발생하며, 브라우저가 첫 번째 HTML을 받은 시점은 100ms입니다.

이 700ms 동안 브라우저는 서버로부터 지속적으로 청크를 수신하고 있습니다.

여기서 주의할 점은, 이는 Next.js 데모 앱이 로컬에서 개발 모드로 실행되고 있다는 점입니다.

따라서 실제 운영 모드로 실행될 때보다는 속도가 느릴 것입니다.

## 지연 된(Suspended) 컴포넌트

몇 초가 지나면 페이지 로드 타임라인에서 또 다른 "Parse HTML" 구간을 볼 수 있습니다.

이번 구간은 지연된 서버 컴포넌트가 로딩을 완료하고 브라우저로 스트리밍되고 있음을 나타냅니다.

![](https://files.smashing.media/articles/forensics-react-server-components/12-suspended-component.jpeg)

우리는 또한 지연 로드된 클라이언트 컴포넌트가 동시에 발견되며, 이 컴포넌트가 필요한 CSS와 자바스크립트 파일을 가져와야 한다는 것을 알 수 있습니다.

이 파일들은 초기 번들에 포함되지 않았습니다.

왜냐하면 이 컴포넌트는 나중에 필요하기 때문이며, 코드는 별도의 파일로 분할됩니다.

이러한 코드 분할 방식은 초기 페이지 로드 성능을 확실히 향상시킵니다.

또한 클라이언트 컴포넌트의 코드가 필요한 경우에만 전송되도록 보장합니다.

서버 컴포넌트(클라이언트 컴포넌트의 부모 컴포넌트 역할)가 오류를 발생시키면, 클라이언트 컴포넌트는 로드되지 않습니다.

해당 컴포넌트가 로드될지 여부를 알기 전에 모든 코드를 로드하는 것은 의미가 없습니다.

위 사진은`DOMContentLoaded` 이벤트가 페이지 로드 타임라인의 끝에서 보고되는 것을 보여줍니다.

그리고 그 직전에 `로컬호스트` HTTP 요청이 끝나는 것을 볼 수 있습니다

이는 서버가 마지막으로 크기가 0인 청크를 전송하여 클라이언트에게 데이터 전송이 완료되었음을 알리고, 스트리밍 통신을 종료할 수 있음을 의미합니다.

## - 최종적인 결과

주요 로컬호스트 HTTP 요청은 약 5초가 걸렸지만, 스트리밍 덕분에 페이지 콘텐츠는 그보다 훨씬 일찍 로드되기 시작했습니다.

만약 이것이 전통적인 서버 사이드 렌더링(SSR) 설정이었다면, 우리는 그 5초 동안 아무 것도 로드되지 않은 빈 화면을 보게 되었을 것입니다.

반면, 전통적인 클라이언트 사이드 렌더링(CSR) 설정이었다면, 훨씬 더 많은 자바스크립트를 전송하고 브라우저와 네트워크에 큰 부담을 주었을 것입니다.

하지만 이 방식 덕분에 앱은 5초 동안 완전히 인터랙티브하게 작동했습니다.

초기 메인 번들의 일부로 로드된 클라이언트 컴포넌트와 상호작용하고 페이지 간 탐색도 가능했습니다.

이는 사용자 경험 측면에서 매우 큰 이점입니다.

## - 결론

RSC(React Server Components)는 React 생태계에서 중요한 진화를 나타냅니다.

이들은 서버 사이드 렌더링(SSR)과 클라이언트 사이드 렌더링(CSR)의 강점을 활용하면서, HTML 스트리밍을 통해 콘텐츠 전달 속도를 높입니다.

이러한 접근 방식은 CSR에서 겪는 SEO와 로딩 시간 문제를 해결할 뿐만 아니라, SSR의 서버 부하를 줄여 성능을 향상시킵니다.

제가 이전에 공유한 RSC 앱을 Next.js 페이지 라우터와 SSR을 사용하도록 리팩토링했습니다.

RSC의 개선 사항은 다음과 같이 상당히 눈에 띕니다:

![](https://files.smashing.media/articles/forensics-react-server-components/13-ssr-vs-rscs.jpeg)

이 두 개의 Sentry 보고서를 보면, 스트리밍을 통해 실제 요청이 완료되기 전에 페이지가 리소스를 로드하기 시작할 수 있음을 알 수 있습니다.

이는 웹 바이탈(Web Vitals) 지표를 크게 향상시키며, 두 보고서를 비교했을 때 이를 확인할 수 있습니다.

결론: **RSCs에 의존하는 아키텍처를 통해 사용자는 더 빠르고 반응적인 인터페이스를 즐길 수 있습니다.**

RSC 아키텍처는 서버 컴포넌트와 클라이언트 컴포넌트라는 두 가지 새로운 컴포넌트 유형을 도입합니다.

이러한 분리는 React와 Next.js와 같은 프레임워크가 콘텐츠 전달을 간소화하면서도 상호작용성을 유지하는 데 도움을 줍니다.

그러나 이 설정은 상태 관리, 인증, 컴포넌트 아키텍처와 같은 영역에서 새로운 과제를 도입합니다.

이러한 도전 과제를 탐구하는 것도 또 다른 블로그 포스트의 좋은 주제가 될 것입니다!

이러한 도전 과제에도 불구하고, RSC의 장점은 그것의 도입에 대한 강력한 사례를 제시합니다.

RSC의 성장과 함께 이러한 과제를 해결하는 방법에 대한 가이드가 분명히 등장할 것이지만, 제 의견으로는 RSC가 이미 현대 웹 개발의 렌더링 관행의 미래를 차지할 것이라 생각합니다.
