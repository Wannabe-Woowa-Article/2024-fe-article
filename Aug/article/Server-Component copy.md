## 🔗 [Storybook 8.2](https://storybook.js.org/blog/storybook-8-2/)

### 🗓️ 번역 날짜: 2024.08.19

### 🧚 번역한 크루: 러기(박정우)

---

## Storybook 8.2

Storybook은 세계적인 디자인 시스템(Carbon, Polaris, Fluent, Lightning 등) 뒤에서 활약하는 컴포넌트 개발 도구입니다. 그래서 흔히 재사용 가능한 작은 단위의 컴포넌트를 만드는 것과 관련이 있다고 생각됩니다.

하지만 UI 컴포넌트는 다양한 크기와 형태로 존재합니다.

Faire, Yoobic, Monday.com의 팀들은 Storybook을 쇼핑 카트, 대시보드, 채팅 위젯, 달력, 전체 페이지 등과 같은 "제품 UI"를 구축하는 데 주로 사용합니다. 이들은 복잡하고 상태를 가지는 컴포넌트로, 높은 품질의 테스트를 통해 개발팀은 안정성을 확보하고 반복적으로 개선할 수 있습니다.

그래서 우리는 Storybook에서 테스트 기능을 더욱 강화하고 있습니다. 그리고 오늘, 우리는 타협 없는 컴포넌트 테스트를 향한 여정의 다음 단계인 Storybook 8.2를 발표하게 되어 매우 기쁩니다.

### Storybook 8.2 주요 기능:

- 🪝 인기 있는 다른 테스트 도구들과 동등한 수준의 새로운 테스트 훅
- 🧳 다른 테스트 및 문서화 도구에서도 재사용 가능한 휴대 가능한 스토리
- 📦 종속성 충돌을 줄이기 위한 패키지 통합
- 🛼 새 사용자가 쉽게 적응할 수 있도록 온보딩 과정 간소화
- ✨ 더 나은 문서 제공을 위한 새롭게 개선된 웹사이트
- 💯 수백 가지의 추가 개선사항

### 새로운 테스트 훅

Jest, Vitest, Playwright, Cypress와 같은 최신 테스트 도구들은 모두 비슷한 테스트 구조를 사용합니다. 문법에는 약간의 차이가 있지만, 기본적인 테스트 훅은 모두 비슷합니다.`before/afterAll`, `before/afterEach`, `describe`, `test`.

Storybook은 자체 스토리 문법인 Component Story Format(CSF)을 사용합니다. CSF는 컴포넌트가 다양한 상태에서 어떻게 작동하는지 보여주는 가장 좋은 방법입니다. Storybook 6.4에서는 상호작용과 검증을 위해 play 기능을 도입했습니다.

```jsx
// ToolbarMenu.stories.js
import { fn, expect } from "@storybook/test";
import { ToolbarMenu } from "./ToolbarMenu";

export default { component: ToolbarMenu };
export const Disabled = {
  args: { disabled: true, onSelected: fn() },
  play: async ({ canvas, args }) => {
    await userEvent.click(canvas.getByRole("button"));
    expect(args.onSelected).not.toHaveBeenCalled();
  },
};
```

이 play 기능을 통해 Storybook에서도 많은 Jasmine 스타일의 테스트를 작성할 수 있었습니다. 하지만 모든 테스트를 다 지원하는 것은 아니었고, 다른 테스트 도구에서 전환해 온 개발자들에게는 CSF 문법이 익숙하지 않을 수 있었습니다.

이제 Storybook 8.2에서는 `before` 훅(클린업 함수를 반환하여 `after` 훅으로 사용할 수 있음)과 React, Vue 3, Svelte용 `play`의 선택적 `mount` 인수를 도입했습니다. 이를 통해 이제는 ‘Arrange’, ‘Act’, ‘Assert’를 하나의 play 함수 내에서 할 수 있어, Jasmine 스타일 도구와 동일한 흐름으로 컴포넌트 테스트를 작성할 수 있습니다.

```jsx
// ToolbarMenu.stories.jsx
import { fn, expect } from "@storybook/test";
import { ToolbarMenu } from "./ToolbarMenu";

export default {
  component: ToolbarMenu,
};

export const Disabled = {
  args: { disabled: true, onSelected: fn() },
  play: async ({ mount, args }) => {
    // Arrange
    const items = await loadItems(10);
    const canvas = await mount(<ToolbarMenu items={items} />);

    // Act
    await userEvent.click(canvas.getByRole("button"));

    // Assert
    expect(canvas.getAllByRole("button").length).toBe(items.length);
    expect(args.onSelected).not.toHaveBeenCalled();
  },
};
```

이번 변화는 기존의 스토리와 호환되므로 모든 기존 스토리가 이전과 동일하게 작동합니다. 그리고 암시적 마운트를 사용하는 스토리 작성 권장 방식도 그대로 유지됩니다. 하지만 더 강력한 기능이 필요할 때, 이제 새로운 테스트 훅이 준비되어 있습니다.

“처음으로 @storybookjs의 플레이 테스트를 사용했어요. 정말 놀라워요! 이렇게 빠르게 테스트를 작성한 적이 없어요!!” — @brechtilliet

문서를 확인하시고, 곧 나올 전체 기능 발표와 더 많은 후크 예제를 기대해 주세요!

### 재사용 가능한 스토리 (Portabel Stories)

Storybook은 컴포넌트를 반복적으로 개발하고, 문서화하고, 테스트하는 데 최고의 도구입니다. 하지만 JS 생태계에는 훌륭한 도구들이 많고, 여러분이 원하는 도구에서 스토리를 활용할 수 있기를 바랍니다.

8.1에서는 Playwright 컴포넌트 테스트를 위한 "Portable Story"를 제한적으로 제공했었습니다. 이제 8.2에서는 React 및 Vue 3용 휴대 가능한 스토리를 완성하고, 실험적으로 Svelte 지원도 추가했습니다. 이를 통해 스토리를 프레임워크에 맞는 네이티브 컴포넌트로 변환하여, 필요한 테스트 훅을 포함한 추가 코드와 함께 사용할 수 있습니다:

```jsx
// Button.test.js
import { test, expect } from "vitest";
import { screen } from "@testing-library/react";
import { composeStories } from "@storybook/react";

// 모든 스토리와 컴포넌트 주석을 스토리 파일에서 가져오기
import * as stories from "./Button.stories";

// 반환된 컴포넌트는 스토리와 1:1로 매핑되며,
// 스토리, 메타, 프로젝트 수준의 모든 주석을 포함합니다.
const { Primary, Secondary } = composeStories(stories);

test("기본 인수로 기본 버튼 렌더링", async () => {
  /**
   * 스토리 생명주기를 통해 실행됩니다:
   * 1. 로더
   * 2. beforeEach
   * 3. 데코레이터 등 주석과 함께 렌더링
   * 4. play 함수 (mount 사용 시 포함)
   * 5. cleanup (beforeEach, hooks에서)
   */
  await Primary.play();
});
```

이 API 외에도, Vitest 및 Jest에서 스토리를 재사용하는 방법에 대한 추가 문서를 준비했습니다. 문서를 확인해보세요, 그리고 곧 전체 기능 발표가 있을 예정입니다.

### 통합된 종속성 관리

State of JS 설문조사에서 20,000명 이상의 개발자가 응답한 결과, JS 생태계의 흐름을 파악할 수 있었습니다. 2022년에는 약간 하락세였지만, Storybook 개선을 위한 우리의 노력은 2023년에 빛을 발했습니다. 이번 설문조사에서 가장 큰 불만은 종속성 관리(과도하게 큰 설치 크기, 긴 설치 시간, 버전 충돌 등)었습니다.

8.2에서는 이러한 문제를 해결하기 시작했습니다. 18개의 패키지를 하나의 핵심 패키지(storybook)와 빌더/렌더러/애드온을 위한 위성 패키지로 통합했습니다. 이는 Vite의 패키지 구조를 따르는 비파괴적 변화입니다. 또한 Storybook의 많은 종속성을 번들링하여 버전 충돌을 방지했습니다.

우리는 또한 Ecosystem Performance(e18e) 프로젝트의 리더인 James Garbutt와 협력하여 유틸리티 라이브러리를 더 작고 빠르며 현대적인 버전으로 업그레이드했습니다(안녕 doctrine! 👋).

이 작업은 아직 초기 단계이지만, 현재까지 설치 크기와 시간을 약 20% 줄였습니다. 새로운 통합 구조는 향후 최적화를 위한 발판이 되어, 8.3 및 그 이후에는 더 큰 개선을 기대할 수 있습니다.

### 간소화된 온보딩

또 다른 중요한 부분은 온보딩 경험입니다. 매주 수천 명의 개발자가 Storybook을 처음 사용해보지만, 설치부터 성공적으로 사용하는 데 이르기까지는 여러 난관이 있습니다. Storybook 8.2에서는 온보딩 경험을 새롭게 개선하고, React에서 Vue 3 및 Angular로 확장하여 사용자를 올바른 방향으로 안내할 수 있도록 했습니다.

이 경험은 8.x 버전에서 더 확장되어, 프로젝트에서 Storybook을 더 쉽게 설정할 수 있게 할 것입니다.

### 새롭게 개선된 문서 사이트

Storybook은 새롭게 개편된 문서 사이트를 제공합니다! 이 사이트에는 다음과 같은 내용이 포함됩니다:

- ✨ Storybook의 다양한 기능을 잘 보여주는 새로운 반응형 애니메이션과 함께한 새로운 홈페이지
- 📚 Next.js, SvelteKit 등 각 프레임워크별 전용 랜딩 페이지가 있는 새로운 문서 홈
- 🌙 시스템 설정에 따라 자동으로 전환되는 문서의 라이트/다크 모드
- 🧱 더 빠른 개선을 위한 추가 작업을 가능하게 하는 기초 작업

### 수백 가지의 추가 개선사항

위의 기능 외에도, 모든 Storybook 릴리스에는 모든 수준에서 수백 가지의 개선사항과 버그 수정이 포함되어 있습니다. 몇 가지 하이라이트:

- ✅ Webpack5/Vite: 소스 맵 수정 (#27171, @valentinpalkovic에게 감사)
- ✅ Angular: 소스 미리보기의 형식 구성 허용 (#28305, @64BitAsura에게 감사)
- ✅ CLI: init에 --no-dev 옵션 추가 (#26918, @fastfrwrd에게 감사)
- ✅ CLI: 새 프로젝트에 @storybook/addon-svelte-csf 포함 (#27070, @benmccann에게 감사)
- ✅ Core: watchStorySpecifiers로 인한 시작 지연 수정 (#27016, @heyimalex에게 감사)
- ✅ Vue3: 새로운 하이드레이션 불일치 컴파일 시간 플래그 활성화 (#27192, @Cherry에게 감사)

### 지금 바로 사용해보세요!

Storybook 8.2는 오늘부터 사용 가능합니다. 새 프로젝트에서 시도해보세요

```
npx storybook@latest init
```

또는 기존 프로젝트를 업그레이드하세요

```
npx storybook@latest upgrade
```

만약 7.x 버전에서 업그레이드하는 경우, 도움이 될 가이드를 제공하고 있습니다. 이전 버전에서 마이그레이션하는 가이드도 있습니다.

### 다음 단계

우리는 8.3을 위해 여러 가지를 준비하고 있습니다:

- 번개처럼 빠른 컴포넌트 테스트를 위한 Vitest와의 통합
- Next.js 프레임워크에 Vite 지원 추가로 Vitest 호환성과 더 나은 개발 경험 제공
- 상호작용형 사이드바 필터링을 위한 UI가 포함된 스토리 태그
- 설치 크기를 더욱 줄이기(8.3-alpha를 시도해보세요, 40% 추가 감소!)
- 특정 뷰포트, 테마, 로케일에 맞춘 스토리 작성의 새로운 방식
