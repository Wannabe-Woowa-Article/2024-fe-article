## 🔗 [Next.js 15 RC](https://nextjs.org/blog/next-15-rc#incremental-adoption-of-partial-prerendering-experimental)

### 🗓️ 번역 날짜: 2024.07.30

### 🧚 번역한 크루: 러기(박정우)

---

## Next.js 15 RC

Next.js 15 Release Candidate (RC)가 이제 사용할 수 있습니다. 이 초기 버전을 통해 다가오는 안정화 릴리스 전에 최신 기능을 테스트할 수 있습니다.

- React: React 19 RC 지원, React 컴파일러(실험적), 하이드레이션 오류 개선
- 캐싱: fetch 요청, GET 라우트 핸들러, 클라이언트 탐색이 기본적으로 캐시되지 않음
- 부분 프리렌더링(실험): 점진적 채택을 위한 새로운 레이아웃 및 페이지 구성 옵션
- next/after(실험): 응답이 스트리밍된 후 코드를 실행하는 새로운 API
- create-next-app: 로컬 개발에서 Turbopack을 활성화하는 새 플래그와 업데이트된 디자인
- 외부 패키지 번들링(안정): App 및 Pages 라우터를 위한 새로운 구성 옵션

오늘 Next.js 15 RC를 시도해보세요:

```bash

npm install next@rc react@rc react-dom@rc
```

참고: Next.js 15 RC 문서는 Next.js 15 GA가 발표될 때까지 rc.nextjs.org/docs에서 볼 수 있습니다.

## React 19 RC

Next.js 앱 라우터는 프레임워크용 React `canary channel`을 기반으로 구축되어, 개발자들이 React 19 출시 전에 이러한 새로운 React API를 사용하고 피드백을 제공할 수 있도록 합니다.

Next.js 15 RC는 이제 React 19 RC를 지원하며, 이는 Actions와 같은 클라이언트 및 서버의 새로운 기능을 포함합니다.

자세한 내용을 알아보려면 Next.js 15 업그레이드 가이드, React 19 업그레이드 가이드, 그리고 `React Conf Keynote`를 참조하세요.

참고: 일부 타사 라이브러리는 아직 React 19와 호환되지 않을 수 있습니다.

## React 컴파일러(실험)

React 컴파일러는 Meta의 React 팀이 만든 새로운 실험적 컴파일러입니다. 이 컴파일러는 JavaScript의 의미와 React의 규칙을 깊이 이해하여 코드를 자동으로 최적화할 수 있습니다. 이 컴파일러는 useMemo와 useCallback과 같은 API를 통해 개발자가 해야 할 수동 메모화를 줄여 코드가 더 간단하고 유지 관리가 쉬워지며 오류가 적어지게 합니다.

Next.js 15에서는 React 컴파일러에 대한 지원을 추가했습니다.

babel-plugin-react-compiler를 설치하세요:

터미널

```bash
npm install babel-plugin-react-compiler
```

그런 다음 next.config.js에 experimental.reactCompiler 옵션을 추가하세요:

next.config.ts

```tsx
const nextConfig = {
  experimental: {
    reactCompiler: true,
  },
};

module.exports = nextConfig;
```

선택적으로, 컴파일러를 "opt-in" 모드로 실행하도록 구성할 수 있습니다:

next.config.ts

```tsx
typescript코드 복사
const nextConfig = {
  experimental: {
    reactCompiler: {
      compilationMode: 'annotation',
    },
  },
};

module.exports = nextConfig;

```

참고: React 컴파일러는 현재 Babel 플러그인을 통해 Next.js에서만 사용할 수 있으며, 이는 빌드 시간을 느리게 할 수 있습니다.

React 컴파일러와 사용 가능한 Next.js 구성 옵션에 대해 자세히 알아보세요.

## Hydration 오류 개선

Next.js 14.1에서는 오류 메시지와 하이드레이션 오류가 개선되었습니다. Next.js 15는 여기에 더해 향상된 하이드레이션 오류 뷰를 추가합니다. 이제 하이드레이션 오류는 오류의 소스 코드와 문제 해결에 대한 제안을 표시합니다.

예를 들어, 다음은 Next.js 14.1의 이전 하이드레이션 오류 메시지입니다:

하이드레이션 오류 메시지 in Next.js 14.1

![이미지 1](https://nextjs.org/_next/image?url=%2Fstatic%2Fblog%2Fnext-15-rc%2Fhydration-error-before-dark.png&w=3840&q=75)
Next.js 15 RC에서는 이를 다음과 같이 개선했습니다:

![이미지 2](https://nextjs.org/_next/image?url=%2Fstatic%2Fblog%2Fnext-15-rc%2Fhydration-error-after-dark.png&w=3840&q=75)

하이드레이션 오류 메시지 improved in Next.js 15 RC

## 캐싱 업데이트

Next.js 앱 라우터는 의견이 반영된 캐싱 기본값으로 출시되었습니다. 이는 기본적으로 가장 성능이 좋은 옵션을 제공하며 필요한 경우 옵트아웃할 수 있는 기능을 제공합니다.

귀하의 피드백을 기반으로 우리는 캐싱 휴리스틱과 그것이 부분 프리렌더링(PPR) 및 fetch를 사용하는 타사 라이브러리와 어떻게 상호 작용할지를 재평가했습니다.

Next.js 15에서는 fetch 요청, GET 라우트 핸들러 및 클라이언트 라우터 캐시에 대한 캐싱 기본값을 캐시에서 기본적으로 제외하도록 변경합니다. 이전 동작을 유지하려면 계속해서 캐싱을 옵트인할 수 있습니다.

우리는 앞으로 몇 달 동안 Next.js에서 캐싱을 계속 개선할 것이며 Next.js 15 GA 발표에서 더 많은 세부 정보를 공유할 것입니다.

## fetch 요청은 더 이상 기본적으로 캐시되지 않음

Next.js는 Web fetch API 캐시 옵션을 사용하여 서버 사이드 fetch 요청이 프레임워크의 지속적인 HTTP 캐시와 어떻게 상호 작용할지를 구성합니다:

```jsx
fetch("https://...", { cache: "force-cache" | "no-store" });
```

- no-store: 매 요청마다 원격 서버에서 리소스를 가져오며 캐시를 업데이트하지 않음
- force-cache: 캐시에 리소스가 있으면 캐시에서, 그렇지 않으면 원격 서버에서 리소스를 가져오고 캐시를 업데이트

Next.js 14에서는 dynamic 함수 또는 dynamic 구성 옵션이 사용되지 않는 한 기본적으로 force-cache가 사용되었습니다.

Next.js 15에서는 캐시 옵션이 제공되지 않으면 기본적으로 no-store가 사용됩니다. 이는 fetch 요청이 기본적으로 캐시되지 않음을 의미합니다.

여전히 fetch 요청에 대해 캐싱을 옵트인할 수 있습니다:

- 단일 fetch 호출에서 캐시 옵션을 force-cache로 설정
- 단일 라우트에 대해 dynamic 라우트 구성 옵션을 'force-static'으로 설정
- 레이아웃 또는 페이지의 모든 fetch 요청을 기본적으로 force-cache를 사용하도록 fetchCache 라우트 구성 옵션을 'default-cache'로 설정하여 캐시 옵션을 명시적으로 지정하지 않은 경우 모두 무시
- GET 라우트 핸들러는 더 이상 기본적으로 캐시되지 않음

Next 14에서는 dynamic 함수 또는 dynamic 구성 옵션을 사용하지 않는 한 GET HTTP 메서드를 사용하는 라우트 핸들러가 기본적으로 캐시되었습니다. Next.js 15에서는 GET 함수가 기본적으로 캐시되지 않습니다.

여전히 static 라우트 구성 옵션을 사용하여 캐싱을 옵트인할 수 있습니다(ex. export dynamic = 'force-static').

sitemap.ts, opengraph-image.tsx, icon.tsx와 같은 특수 라우트 핸들러 및 기타 메타데이터 파일은 dynamic 함수 또는 dynamic 구성 옵션을 사용하지 않는 한 기본적으로 정적 상태로 유지됩니다.

## 클라이언트 라우터 캐시는 기본적으로 페이지 구성 요소를 캐시하지 않음

Next.js 14.2.0에서는 라우터 캐시의 사용자 정의 구성을 허용하기 위해 실험적 staleTimes 플래그를 도입했습니다.

Next.js 15에서는 이 플래그가 여전히 접근 가능하지만 페이지 세그먼트에 대한 staleTime을 0으로 설정하여 기본 동작을 변경하고 있습니다. 이는 앱을 탐색할 때 클라이언트가 탐색의 일부로 활성화되는 페이지 구성 요소에서 항상 최신 데이터를 반영함을 의미합니다. 그러나 중요한 동작은 여전히 변경되지 않습니다:

공유 레이아웃 데이터는 부분 렌더링을 계속 지원하기 위해 서버에서 다시 가져오지 않습니다.
이전/다음 탐색은 여전히 캐시에서 복원되어 브라우저가 스크롤 위치를 복원할 수 있도록 합니다.
Loading.js는 여전히 5분 동안 캐시됩니다(또는 staleTimes.static 구성의 값).
이전 클라이언트 라우터 캐시 동작을 옵트인하려면 다음 구성을 설정하세요:

```tsx
// next.config.ts
const nextConfig = {
  experimental: {
    staleTimes: {
      dynamic: 30,
    },
  },
};

module.exports = nextConfig;
```

### 부분 프리렌더링의 점진적 채택(실험적)

Next.js 14에서는 부분 프리렌더링(PPR)을 도입했습니다. 이는 같은 페이지에서 정적 렌더링과 동적 렌더링을 결합하는 최적화입니다.

Next.js는 현재 `cookies()`, `headers()` 및 캐시되지 않은 데이터 요청과 같은 동적 함수를 사용하지 않는 한 정적 렌더링을 기본값으로 합니다. 이러한 API는 전체 라우트를 동적 렌더링으로 옵트인합니다. PPR을 사용하면 동적 UI를 Suspense 경계에 래핑할 수 있습니다. 새로운 요청이 들어오면 Next.js는 정적 HTML 쉘을 즉시 제공한 다음 동일한 HTTP 요청에서 동적 부분을 렌더링하고 스트리밍합니다.

점진적 채택을 허용하기 위해 특정 레이아웃 및 페이지를 PPR에 옵트인할 수 있는 experimental_ppr 라우트 구성 옵션을 추가했습니다:

```jsx
import { Suspense } from "react"
import { StaticComponent, DynamicComponent } from "@/app/ui"

export const experimental_ppr = true

export default
```
