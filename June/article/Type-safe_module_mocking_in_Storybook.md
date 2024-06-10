## 🔗 [Type-safe module mocking in Storybook](https://storybook.js.org/blog/type-safe-module-mocking/?utm_source=newsletter.reactdigest.net&utm_medium=newsletter&utm_campaign=sneaky-react-memory-leaks)

### 🗓️ 번역 날짜: 2024.06.03

### 🧚 번역한 크루: 러기(박정우)

---

## 스토리북에서의 Type-safe 모듈 모킹

UI를 분리하여 개발하고 테스트하는 데 있어 일관성은 매우 중요합니다.

이상적으로, 스토리북의 스토리는 누가 언제 보든, 백엔드가 작동하고 있든 아니든 항상 동일한 UI를 렌더링해야 합니다. 스토리에 주어진 입력은 항상 동일한 출력 결과를 가져야 합니다.

이는 UI의 입력이 컴포넌트에 전달되는 props만 있을 때는 단순합니다. 컴포넌트가 컨텍스트 제공자로부터 데이터를 의존하는 경우, 이를 모킹하기 위해 스토리를 Decorator로 감싸서 모킹할 수 있습니다. 네트워크에서 가져온 입력이 있는 UI의 경우, 네트워크 요청을 결정론적으로 모의하는 매우 인기 있는 Mock Service Worker 애드온이 있습니다.

그렇다면 컴포넌트가 브라우저 API와 같은 다른 출처에 의존하는 경우는 어떻게 될까요? 예를 들어 사용자의 테마 선호도, 로컬 스토리지의 데이터, 또는 쿠키를 읽는 경우, 혹은 컴포넌트가 현재 날짜나 시간에 따라 다르게 동작하는 경우, 아니면 컴포넌트가 Next.js의 next/router와 같은 메타 프레임워크 API를 사용하는 경우는 어떨까요?

이러한 유형의 입력을 모킹하는 것은 역사적으로 스토리북에서 어려웠습니다. 그리고 바로 오늘 우리가 모듈 모킹을 통해 해결하고 있는 문제입니다! 우리의 접근 방식은 간단하고, 타입 안전하며, 표준 기반입니다. 그것은 불투명하거나 독점적인 모듈 API보다 명확성과 디버깅의 명확성을 선호합니다.
_그리고 우리는 좋은 회사에 있습니다: Epic Stack의 창조자 Kent C. Dodd는 절대적인 수입과 React 서버 컴포넌트 아키텍트 Seb Markbåge가 스토리북 모킹에 직접적인 영감을 주었다고 추천합니다._

<aside>
💡 참고: 이 작업을 통해 우리는 Node 전용 코드를 모킹하고 브라우저에서 스토리북을 사용하여 React 서버 컴포넌트(RSCs)를 테스트할 수 있습니다. 이에 대한 자세한 내용은 향후 블로그 포스트에서 공유할 예정입니다. 계속 주목해 주세요!

</aside>

## 모듈 모킹이 무엇인가요?

모듈 모킹은 컴포넌트가 직접적이거나 간접적으로 가져오는 모듈을 일관되고 독립적인 대체물로 교체하는 기술입니다. 유닛 테스트에서는 이 기술이 코드를 재현 가능한 상태로 테스트하는 데 도움을 줄 수 있습니다. 스토리북에서는 이를 사용하여 데이터를 흥미로운 방식으로 검색하는 컴포넌트를 렌더링하고 테스트할 수 있습니다.

예를 들어, 사용자가 표시할 정보를 선택할 수 있고 그 설정을 브라우저의 로컬 스토리지에 저장하는 사용자 설정 가능한 대시보드 컴포넌트를 고려해 보세요.

![alt text](https://storybookblog.ghost.io/content/images/size/w1600/2024/05/Dashboard.png)

이것은 사용자의 설정을 로컬 스토리지에 읽고 쓰는 설정 데이터 접근 계층으로 구현되며, UI를 담당하는 디스플레이 컴포넌트인 대시보드 예시입니다:

```tsx
// lib/settings.ts
export const getDashboardLayout = () => {
  const layout = window.localStorage.getItem("dashboard.layout");
  return layout ? parseLayout(layout) : [];
};
```

```tsx
// components/Dashboard.tsx
import { getDashboardLayout } from "../lib/settings.ts";

export const Dashboard = (props) => {
  const layout = getDashboardLayout();
  // logic to display layout
};
```

대시보드 컴포넌트를 테스트하기 위해, 우리는 핵심 상태를 다루는 다양한 레이아웃의 예제 세트를 만들고자 합니다. 간단하게 하기 위해서, 일반성을 잃지 않는 선에서, 우리는 레이아웃을 읽는 부분에만 집중할 것입니다.

이 글에서는 모듈 모킹을 설명하고, 우리가 어떻게 그것을 달성하는지, 그리고 우리의 접근법이 다른 구현과 비교했을 때 어떤 이점이 있는지를 설명하는 실행 예제로 이를 사용할 것입니다.

## 기존 접근 방식: 독점 API

Jest 및 Vitest와 같은 인기 있는 유닛 테스트 도구들은 모듈 모킹을 위한 유연한 메커니즘을 제공합니다. 이들은 인접한 mocks 디렉토리에서 모킹 파일을 자동으로 찾는 예시입니다:

```tsx
// lib/__mocks__/settings.ts
export const getDashboardLayout = () => [
  /* dummy data here */
];
```

또는, 테스트 파일 내에서 모킹를 선언하기 위해 명령형 API를 제공합니다:

```tsx
// components/Dashboard.test.ts
import { vi, fn } from 'vitest';
import { getDashboardLayout } from '../lib/settings.ts';

vi.mock('../lib/settings.ts', () => ({
  getDashboardLayout: fn(() => ([ /* dummy data here */])),
});
```

이 API는 간단해 보이지만, 실제로는 가져오기를 모킹으로 대체하기 위해 복잡하고 다소 마법 같은 파일 변환을 유발합니다. 결과적으로 코드의 작은 변경이 모킹을 혼란스럽게 만드는 방식으로 깨뜨릴 수 있습니다. 예를 들어, 다음과 같은 변형은 실패합니다:

```tsx
// components/Dashboard.test.ts
import { vi, fn } from 'vitest';
import { getDashboardLayout } from '../lib/settings.ts';

const dummyLayout = [ /* dummy data here */];
vi.mock('../lib/settings.ts', () => ({
  getDashboardLayout: fn(() => dummyLayout), // FAIL!!!
});
```

하지만 우리의 목표는 이 뛰어난 도구들을 비판하는 것이 아닙니다. 오히려, 우리는 새롭고 표준 기반의 접근 방식을 사용하여 더 나은 모킹을 어떻게 할 수 있는지 탐구하고자 합니다.

## 우리의 접근 방식: Subpath Imports

스토리북에서의 모듈 모킹은 서브패스 임포트 표준을 활용하며, 이는 `package.json의 imports` 필드를 통해 설정 가능합니다 — 모든 JS 프로젝트의 심장부입니다 — 프로젝트 전체에 걸쳐 모의를 가져오는 파이프라인 역할을 합니다.

이 접근 방식의 하나의 슈퍼파워는 `package.json exports`와 마찬가지로, `package.json imports`도 조건부로 만들 수 있다는 것입니다. 즉, 런타임 환경에 따라 가져오는 경로를 다르게 할 수 있습니다. 이는 스토리북에서는 모의된 모듈을 가져오고, 다른 곳에서는 실제 모듈을 가져오도록 package.json을 맞춤 설정할 수 있다는 의미입니다!

서브패스 임포트는 처음에 Node.js에서 도입되었지만, 이제는 JS 생태계 전반에서 지원되고 있으며, TypeScript(버전 5.4부터), Webpack, Vite, Jest, Vitest 등에서도 지원됩니다.

위의 예제를 계속해서, 다음은 `./lib/settings.ts`에서 모듈을 모킹하는 방법입니다:

```tsx
{
  "imports": {
    "#lib/settings": {
      "storybook": "./lib/settings.mock.ts",
      "default": "./lib/settings.ts"
    },
    "#*": [ // fallback for non-mocked absolute imports
      "./*",
      "./*.ts",
      "./*.tsx"
    ]
  }
}
```

여기서 우리는 모듈 리졸버에게 모든 `#lib/settings`에서의 가져오기가 스토리북에서는 `../lib/settings.mock.ts`로, 애플리케이션에서는 `../lib/settings.ts`로 해석되도록 지시하고 있습니다.

이는 또한 Node.js 사양에 따라 #-기호로 시작하는 절대 경로에서 가져오도록 컴포넌트를 수정해야 하며, 이는 경로나 패키지 가져오기와 관련된 모호성이 없도록 보장합니다.

```tsx
// Dashboard.test.ts

- import { getDashboardLayout } from '../lib/settings';
+ import { getDashboardLayout } from '#lib/settings';
```

이것은 다소 번거로워 보일 수 있지만, 런타임에 따라 모듈이 달라질 수 있다는 것을 파일을 읽는 개발자들에게 명확하게 전달하는 장점이 있습니다. 실제로, 모킹을 위해 훌륭한 모든 이유들로 인해, 일반적으로 절대 가져오기에 대해 이 표준을 권장합니다(아래 참조).

## 스토리별 모킹

Subpath Import를 사용하여, 우리는 `settings.ts` 파일 전체를 표준 기반 접근 방식을 사용하는 새 모듈로 교체할 수 있습니다. 그러나 각 테스트(또는 우리의 경우, 스토리북 스토리)마다 그 구현을 달리하고 싶다면 `settings.mock.ts`를 어떻게 구조화해야 할까요?

다음은 어떤 모듈이든 모킹하기 위한 보일러플레이트 구조입니다. 코드를 완전히 제어할 수 있기 때문에, 특별한 상황에 맞게 수정할 수 있습니다(예를 들어, 브라우저에서 실행되지 않도록 노드 코드를 제거하거나 그 반대의 경우).

```tsx
// lib/settings.mock.ts
import { fn } from "@storybook/test";
import * as actual from "./settings"; // 👈 Import the actual implementation

// 👇 Re-export the actual implementation.
// This catch-all ensures that the exports of the mock file always contains
// all the exports of the original. It is up to the user to override
// individual exports below as appropriate.
export * from "./settings";

// 👇 Export a mock function whose default implementation is the actual implementation.
// With a useful mockName, it displays nicely in Storybook's Actions addon
// for debugging.
export const getDashboardLayout = fn(actual.getDashboardLayout).mockName("settings::getDashboardLayout");
```

이 모의 파일은 이제 #lib/settings이 가져와질 때마다 스토리북에서 사용됩니다. 실제 구현을 감싸는 것 외에는 아직 많은 것을 하지 않지만 — 그것이 중요한 부분입니다.

이제 스토리북 스토리에서 사용해 봅시다:

```tsx
// components/Dashboard.stories.ts

import type { Meta, StoryObj } from "@storybook/react";
import { expect } from "@storybook/test";

// 👇 You can use subpaths as an absolute import convention even
// for non-conditional paths
import { Dashboard } from "#components/Dashboard";

// 👇 Import the mock file explicitly, as that will make
// TypeScript understand that these exports are the mock functions
import { getDashboardLayout } from "#lib/settings.mock";

const meta = {
  component: Dashboard,
} satisfies Meta<typeof Dashboard>;
export default meta;

type Story = StoryObj<typeof meta>;

export const Empty: Story = {
  beforeEach: () => {
    // 👇 Mock return an empty layout
    getDashboardLayout.mockReturnValue([]);
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    // 👇 Expect the UI to prompt when the dashboard is empty
    await expect(canvas).toHaveTextContent("Configure your dashboard");
    // 👇 Assert directly on the mock function that it was called as expected
    expect(getDashboardLayout).toHaveBeenCalled();
  },
};

export const Row: Story = {
  beforeEach: () => {
    // 👇 Mock return a different, story-specific layout
    getDashboardLayout.mockReturnValue([
      /* hard-coded "row" layout data */
    ]);
  },
};
```

스토리북에서 'fn' 모의 함수를 사용하는 것은 다음을 의미합니다:

1. 스토리북의 새로운 beforeEach 훅을 사용하여 각 스토리에 대한 동작을 수정할 수 있습니다.
2. 함수가 호출될 때마다 Actions 패널이 이를 기록합니다.
3. play 함수에서 호출을 확인할 수 있습니다.

<aside>
💡 텍스트 이상을 확인해야 합니다. Chromatic의 Visual Tests 애드온을 사용하면 컴포넌트가 실제로 어떻게 보이는지 빠르게 테스트하여 다양한 브라우저와 뷰포트에서 UI 버그를 잡을 수 있습니다.
우리 접근 방식의 장점
이제 우리는 스토리북에서 package.json의 Subpath Imports 표준을 기반으로 한 모듈 모킹의 종단 간 예제를 보았습니다. Jest와 Vitest가 취한 독점적 접근 방식과 비교할 때, 이 접근 방식은 명시적이고 타입 안전하며 표준 기반이라는 장점이 있습니다.

</aside>

### 명시적

모킹 프레임워크의 일부 마법과 같은 기능은 모킹이 어떻게 그리고 언제 적용되는지 이해하기 어렵게 만듭니다. 예를 들어, vi.mock 호출 내에서 외부 정의 변수를 참조하는 것이 유효한 자바스크립트임에도 불구하고 모킹 오류를 일으키는 것을 보았습니다.

반면에 모든 모킹을 package.json에서 명시적으로 정의함으로써, 우리의 솔루션은 다양한 환경에서 모듈이 어떻게 해결되는지 이해하는 명확하고 예측 가능한 방법을 제공합니다. 이러한 투명성은 디버깅을 단순화하고 테스트를 더 예측 가능하게 만듭니다.

### Type-Safe

모킹 프레임워크는 개발자가 익숙해져야 할 관례, 문법 스타일, 특정 API를 도입합니다. 또한, 이러한 솔루션들은 종종 타입 검사를 지원하지 않습니다.

기존의 package.json을 사용함으로써 우리의 솔루션은 최소한의 설정이 필요합니다. 또한, TypeScript가 이제 package.json의 서브패스 임포트를 자동완성으로 지원함에 따라(2024년 3월 TypeScript 5.4 버전부터) 자연스럽게 TypeScript와 통합됩니다.

### 표준 기반

가장 중요한 것은 스토리북의 접근 방식이 100% 표준 기반이기 때문에, 어떤 도구 체인이나 환경에서도 모킹을 사용할 수 있다는 것입니다.

이는 표준을 배우고 그 지식을 어디에서나 재사용할 수 있게 해주므로 유용합니다. 예를 들어, vi.mock 사용법은 Jest의 모킹과 비슷하지만 동일하지는 않습니다.

또한 여러 도구를 함께 사용할 수 있습니다. 예를 들어, 사용자가 컴포넌트에 대한 스토리를 작성한 다음, 우리의 Portable Stories 기능을 사용하여 그 스토리를 다른 테스트 도구에서 재사용하는 것이 일반적입니다.

가장 중요한 것은 스토리북의 접근 방식이 100% 표준 기반이기 때문에, 어떤 도구 체인이나 환경에서도 모킹을 사용할 수 있다는 것입니다.

이는 표준을 배우고 그 지식을 어디에서나 재사용할 수 있게 해주므로 유용합니다. 예를 들어, vi.mock 사용법은 Jest의 모킹과 비슷하지만 동일하지는 않습니다.

또한 여러 도구를 함께 사용할 수 있습니다. 예를 들어, 사용자가 컴포넌트에 대한 스토리를 작성한 다음, 우리의 Portable Stories 기능을 사용하여 그 스토리를 다른 테스트 도구에서 재사용하는 것이 일반적입니다.

또한, 여러 환경에서 모킹을 사용할 수 있습니다. 예를 들어, 스토리북의 모킹은 Node 표준의 일부이기 때문에 Node에서 "for free"로 작동하지만, 그 표준이 Webpack과 Vite에 의해 구현되었기 때문에 브라우저에서도 잘 작동한다고 가정할 경우 사용할 수 있습니다.

마지막으로, 우리는 ESM 표준에 맞춰져 있기 때문에, 미래의 JS 변경과 호환될 수 있습니다. 우리는 플랫폼에 베팅하고 있습니다. 우리는 이것이 모듈 모킹의 미래이며 모든 테스트 도구가 이를 채택해야 한다고 믿습니다.

### 오늘 바로 시도해보세요

모듈 모킹은 스토리북 8.1에서 사용할 수 있습니다. 새 프로젝트에서 시도해보세요:

```bash
npx storybook@latest init
```

또는 기존 프로젝트를 업그레이드하세요:

```bash
npx storybook@latest upgrade
```

모듈 모킹에 대해 자세히 알아보려면 스토리북 문서를 참조해 주세요. 여기에는 더 많은 예제와 전체 API가 포함되어 있습니다. 우리는 모듈 모킹 접근 방식을 사용하여 테스트한 Next.js React 서버 컴포넌트(RSC) 앱의 전체 데모를 만들었습니다. 이에 대해 더 자세히 설명할 예정이며, 곧 블로그 포스트를 통해 문서화할 계획입니다.

다음 단계
스토리북의 모듈 모킹은 기능 완성 단계이며 사용 준비가 되어 있습니다. 우리는 다음과 같은 개선 사항을 고려하고 있습니다:

- 주어진 모듈에 대해 모의 보일러플레이트를 자동으로 생성하는 CLI 유틸리티
- UI에서 모의 데이터를 시각화하고 편집할 수 있는 지원

모듈 모킹 외에도, 우리는 여러 테스트 개선 작업을 진행 중입니다. 예를 들어, 브라우저에서 React 서버 컴포넌트를 유닛 테스트할 수 있는 새로운 방법을 개발했습니다. 또한 스토리북의 테스트를 Jest/Vitest의 Jasmine에서 영감을 받은 구조에 훨씬 더 가깝게 가져오는 작업을 진행하고 있습니다.
