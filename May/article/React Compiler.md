## 🔗 [React Compiler](https://react.dev/learn/react-compiler)

### 🗓️ 번역 날짜: 2024.06.01

### 🧚 번역한 크루: 버건디(전태헌)

---

# 리액트 컴파일러

This page will give you an introduction to the new experimental React Compiler and how to try it out successfully.

이 페이지는 새롭지만 아직은 실험적인 리액트 컴파일러에 대해 어떻게 시도해볼수 있을지 설명 합니다.

> 이 문서는 아직 작업 중입니다. 더 많은 문서는 [React Compiler Working Group](https://github.com/reactwg/react-compiler/discussions) 저장소에서 확인할 수 있으며, 문서가 더 안정화되면 이 문서로 업스트림될 예정입니다.

## 이런걸 배우시게 될거에요

1. 컴파일러를 어떻게 시작하는지
2. 컴파일러와 eslint 플러그인을 설치하는 법
3. 문제 해결

### 주의 사항

React Compiler는 커뮤니티로부터 초기 피드백을 받기 위해 오픈 소스화된 새로운 실험적 컴파일러입니다. 아직 완벽하지 않으며 프로덕션 환경에서는 사용하기에 준비되지 않았습니다.

React Compiler는 React 19 RC를 필요로 합니다.

만약 React 19로 업그레이드할 수 없다면, [Working Group](<(https://github.com/reactwg/react-compiler/discussions/6)>)에서 설명된 캐시 함수의 사용자 공간 구현을 시도할 수 있습니다.

React Compiler는 커뮤니티로부터 초기 피드백을 받기 위해 오픈 소스화된 새로운 실험적 컴파일러입니다.

이는 빌드 타임 전용 도구로, React 앱을 자동으로 최적화합니다. 이 컴파일러는 순수 JavaScript와 함께 작동하며, [React의 규칙](https://react.dev/reference/rules)을 이해하므로 코드를 다시 작성할 필요가 없습니다.

컴파일러는 또한 컴파일러의 분석 결과를 에디터 내에서 바로 확인할 수 있게 해주는 [eslint 플러그인](https://react.dev/learn/react-compiler#installing-eslint-plugin-react-compiler)을 포함하고 있습니다.

이 플러그인은 컴파일러와 독립적으로 실행되며, 앱에서 컴파일러를 사용하지 않더라도 사용할 수 있습니다.

모든 React 개발자들이 이 eslint 플러그인을 사용하여 코드베이스의 품질을 향상시키는 것을 권장합니다.

그러나 이는 권장되지 않으며 가능할 때 React 19로 업그레이드하는 것이 좋습니다.

## 컴파일러는 무엇을 하나요?

애플리케이션을 최적화하기 위해 React Compiler는 자동으로 코드를 메모이제이션(memoization)합니다.

오늘날 `useMemo`, `useCallback`, `React.memo`와 같은 API를 통해 메모이제이션을 접해보셨을 것입니다.

이러한 API를 사용하면 입력 값이 변경되지 않았을 때 애플리케이션의 특정 부분을 다시 계산할 필요가 없다고 React에 알릴 수 있어 업데이트 작업을 줄일 수 있습니다.

하지만 강력한 기능임에도 불구하고 메모이제이션을 적용하는 것을 잊거나 잘못 적용하기 쉽습니다.

이로 인해 React가 의미 있는 변경이 없는 UI 부분을 확인해야 하므로 비효율적인 업데이트가 발생할 수 있습니다.

React Compiler는 JavaScript와 React의 규칙에 대한 지식을 활용하여 컴포넌트와 훅 내의 값이나 값 그룹을 자동으로 메모이제이션합니다.

규칙 위반이 감지되면 해당 컴포넌트나 훅만 건너뛰고 안전하게 다른 코드를 계속 컴파일합니다.

이미 코드베이스가 잘 메모이제이션되어 있다면 컴파일러를 사용해도 큰 성능 향상을 기대하지 않을 수 있습니다.

하지만 실질적으로는 성능 문제를 일으키는 정확한 종속성을 메모이제이션하는 것이 손으로 하기에는 까다롭습니다.

## - 리액트 컴파일러는 무슨 종류의 메모이제이션을 하나요 ?

React 컴파일러의 초기 버전은 업데이트 성능(기존 컴포넌트의 다시 렌더링)을 개선하는 데 중점을 두고 있습니다.

그래서 다음 두 가지 사용 사례에 집중하고 있습니다:

1. **계단식으로 발생하는 다시 렌더링 건너뛰기**

`<Parent />` 컴포넌트를 다시 렌더링할 때, 실제로는 `<Parent />`만 변경되었지만 해당 컴포넌트 트리의 많은 컴포넌트가 다시 렌더링되는 상황을 방지합니다.

2. **React 외부에서 발생하는 비용이 많이 드는 계산 건너뛰기**

예를 들어, 컴포넌트나 훅 내부에서 `expensivelyProcessAReallyLargeArrayOfObjects()`와 같은 대용량 배열 객체를 처리하는 비용이 많이 드는 함수를 호출하는 경우를 방지합니다.

### 리렌더링 최적화

React는 현재 상태(구체적으로는 props, state, 그리고 context)에 따라 UI를 함수로 표현할 수 있게 해줍니다.

현재 구현 방식에서는 컴포넌트의 상태가 변경될 때 React는 해당 컴포넌트와 그 자식 컴포넌트를 모두 다시 렌더링합니다.

단, useMemo(), useCallback(), 또는 React.memo()와 같은 수동 메모이제이션을 적용한 경우는 예외입니다.

예를 들어, 다음 예제에서 `<FriendList>`의 상태가 변경될 때마다 `<MessageButton>`이 다시 렌더링됩니다.

```tsx
function FriendList({ friends }) {
  const onlineCount = useFriendOnlineCount();
  if (friends.length === 0) {
    return <NoFriends />;
  }
  return (
    <div>
      <span>{onlineCount} online</span>
      {friends.map((friend) => (
        <FriendListCard key={friend.id} friend={friend} />
      ))}
      <MessageButton />
    </div>
  );
}
```

[리액트 컴파일러 예제 사이트에서 확인해보세요](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAMygOzgFwJYSYAEAYjHgpgCYAyeYOAFMEWuZVWEQL4CURwADrEicQgyKEANnkwIAwtEw4iAXiJQwCMhWoB5TDLmKsTXgG5hRInjRFGbXZwB0UygHMcACzWr1ABn4hEWsYBBxYYgAeADkIHQ4uAHoAPksRbisiMIiYYkYs6yiqPAA3FMLrIiiwAAcAQ0wU4GlZBSUcbklDNqikusaKkKrgR0TnAFt62sYHdmp+VRT7SqrqhOo6Bnl6mCoiAGsEAE9VUfmqZzwqLrHqM7ubolTVol5eTOGigFkEMDB6u4EAAhKA4HCEZ5DNZ9ErlLIWYTcEDcIA)

React 컴파일러는 수동 메모이제이션과 동등한 작업을 자동으로 적용하여 상태 변경 시 앱의 관련 부분만 다시 렌더링되도록 보장합니다.

이를 때로는`세밀한 반응성(fine-grained reactivity)`이라고 합니다.

위의 예에서 React 컴파일러는 friends가 변경되더라도 `<FriendListCard />`의 반환 값을 재사용할 수 있음을 판단하고, 이 JSX를 다시 생성하거나 count가 변경될 때 `<MessageButton>`을 다시 렌더링하지 않아도 됩니다.

### 비용이 많이 드는 계산도 메모이제이션됩니다.

컴파일러는 렌더링 중에 사용되는 비용이 많이 드는 계산에 대해서도 자동으로 메모이제이션할 수 있습니다.

```tsx
// 이것은 컴포넌트나 훅이 아니기 때문에 리액트 컴파일러가 메모이제이션 하지 못합니다.
function expensivelyProcessAReallyLargeArrayOfObjects() {
  /* ... */
}

// 이것은 컴포넌트이기 때문에 리액트 컴파일러가 메모이제이션 합니다.
function TableContainer({ items }) {
  // 이 함수는 메모이제이션 됩니다.
  const data = expensivelyProcessAReallyLargeArrayOfObjects(items);
  // ...
}
```

[리액트 컴파일러 예제 사이트에서 확인해보세요](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAejQAgFTYHIQAuumAtgqRAJYBeCAJpgEYCemASggIZyGYDCEUgAcqAGwQwANJjBUAdokyEAFlTCZ1meUUxdMcIcIjyE8vhBiYVECAGsAOvIBmURYSonMCAB7CzcgBuCGIsAAowEIhgYACCnFxioQAyXDAA5gixMDBcLADyzvlMAFYIvGAAFACUmMCYaNiYAHStOFgAvk5OGJgAshTUdIysHNy8AkbikrIKSqpaWvqGIiZmhE6u7p7ymAAqXEwSguZcCpKV9VSEFBodtcBOmAYmYHz0XIT6ALzefgFUYKhCJRBAxeLcJIsVIZLI5PKFYplCqVa63aoAbm6u0wMAQhFguwAPPRAQA+YAfL4dIloUmBMlODogDpAA)

그러나 `expensivelyProcessAReallyLargeArrayOfObjects`가 정말로 비용이 많이 드는 함수라면, React 외부에서 자체적으로 메모이제이션을 구현하는 것을 고려해야 합니다. 그 이유는:

- React 컴파일러는 React 컴포넌트와 훅만 메모이제이션하며, 모든 함수를 메모이제이션하지 않습니다.

- React 컴파일러의 메모이제이션은 여러 컴포넌트나 훅에서 공유되지 않습니다.

따라서 `expensivelyProcessAReallyLargeArrayOfObjects`가 여러 다른 컴포넌트에서 사용된다면, 동일한 항목이 전달되더라도 그 비용이 많이 드는 계산이 반복적으로 실행될 것입니다.

코드가 더 복잡해지기 전에 먼저 [프로파일링](https://react.dev/reference/react/useMemo#how-to-tell-if-a-calculation-is-expensive)을 통해 실제로 비용이 많이 드는지 확인하는 것을 권장합니다.

## 컴파일러는 무엇을 가정할까요 ?

React 컴파일러는 다음과 같은 사항을 가정합니다:

1. 유효하고 의미론적인 JavaScript 코드:

2. 코드가 유효하고 의미론적인 JavaScript로 작성되어 있어야 합니다.
   널 가능/옵션 값 및 속성의 정의 여부를 테스트:

널 가능(nullable)하거나 옵션(optional)인 값과 속성을 접근하기 전에 정의되어 있는지 테스트합니다.

예를 들어, 타입스크립트를 사용할 경우 [strictNullChecks](https://www.typescriptlang.org/tsconfig/#strictNullChecks)를 활성화하여 if `(object.nullableProperty) { object.nullableProperty.foo }` 또는 옵셔널 체이닝을 사용하여 `object.nullableProperty?.foo`와 같이 작성해야 합니다.

3. [React의 규칙](https://react.dev/reference/rules)을 따릅니다.

React 컴파일러는 React의 규칙을 정적으로 검증할 수 있으며, 오류가 감지되면 안전하게 컴파일을 건너뜁니다.

오류를 확인하려면 [eslint-plugin-react-compiler](https://www.npmjs.com/package/eslint-plugin-react-compiler)를 설치하는 것을 권장합니다.

## 컴파일러를 시도해봐야 할까요?

컴파일러는 아직 실험적인 단계에 있으며, 많은 부분이 미완성 상태임을 유의하세요.

Meta와 같은 회사에서 프로덕션에 사용되었지만, 컴파일러를 여러분의 앱 프로덕션에 도입할지는 코드베이스의 상태와 [React의 규칙](https://react.dev/reference/rules)을 얼마나 잘 준수했는지에 따라 달라집니다.

**지금 당장 컴파일러를 사용해야 할 필요는 없습니다.**

**안정된 릴리스가 될 때까지 기다렸다가 도입해도 괜찮습니다.**

하지만, 앱에서 작은 실험을 통해 컴파일러를 시도해보고 피드백을 제공해 주시면 컴파일러를 더욱 개선하는 데 도움이 됩니다.

## 시작하기

이 문서들 외에도, React 컴파일러에 대한 추가 정보와 논의를 위해 [React Compiler Working Group](https://github.com/reactwg/react-compiler)을 확인하는 것을 추천드립니다.

## 호환성 확인

컴파일러를 설치하기 전에, 먼저 코드베이스가 호환되는지 확인할 수 있습니다:

```
npx react-compiler-healthcheck@latest
```

이 스크립트는 다음을 수행합니다:

- 최적화에 성공할 수 있는 컴포넌트의 수를 확인합니다: 숫자가 클수록 좋습니다.

- `<StrictMode>` 사용 여부를 확인합니다: 이를 활성화하고 따르는 경우 [React의 규칙](https://react.dev/reference/rules)을 따를 가능성이 높아집니다.

- 호환되지 않는 라이브러리 사용 여부를 확인합니다: 컴파일러와 호환되지 않는 것으로 알려진 라이브러리들을 체크합니다.

```
Successfully compiled 8 out of 9 components.
StrictMode usage not found.
Found no usage of incompatible libraries.
```

## eslint-plugin-react-compiler 설치

React 컴파일러는 eslint 플러그인도 지원합니다.

이 eslint 플러그인은 컴파일러와 독립적으로 사용할 수 있으므로, 컴파일러를 사용하지 않더라도 eslint 플러그인을 사용할 수 있습니다.

```
npm install eslint-plugin-react-compiler
```

그리고, eslint config에 다음을 추가하세요.

```ts
module.exports = {
  plugins: ["eslint-plugin-react-compiler"],
  rules: {
    "react-compiler/react-compiler": "error",
  },
};
```

eslint 플러그인은 에디터에서 React의 규칙 위반 사항을 표시합니다.

이 경우, 컴파일러는 해당 컴포넌트나 훅의 최적화를 건너뛴다는 의미입니다.

이는 전혀 문제가 없으며, 컴파일러는 이를 복구하고 코드베이스의 다른 컴포넌트를 계속해서 최적화할 수 있습니다.

**모든 eslint 위반 사항을 즉시 수정할 필요는 없습니다.**

최적화되는 컴포넌트와 훅의 수를 늘리기 위해 자신의 속도에 맞춰 이를 해결할 수 있지만, 컴파일러를 사용하기 전에 모든 것을 수정해야 하는 것은 아닙니다.

## 컴파일러를 코드베이스에 도입하기

### 기존 프로젝트

컴파일러는 [React의 규칙](https://react.dev/reference/rules)을 따르는 함수형 컴포넌트와 훅을 컴파일하도록 설계되었습니다.

또한, 해당 규칙을 위반하는 코드를 처리할 때는 해당 컴포넌트나 훅을 건너뛰도록 되어 있습니다.

그러나 JavaScript의 유연한 특성 때문에, 컴파일러가 모든 위반을 잡아내지는 못하며, 잘못된 긍정(false negative)으로 인해 규칙을 위반하는 컴포넌트나 훅을 컴파일할 수 있습니다.

이는 정의되지 않은 동작을 초래할 수 있습니다.

이러한 이유로, 기존 프로젝트에서 컴파일러를 성공적으로 도입하기 위해서는 제품 코드의 작은 디렉토리에서 먼저 실행해보는 것을 권장합니다.

컴파일러를 특정 디렉토리 집합에서만 실행하도록 설정하여 이를 수행할 수 있습니다:

```ts
const ReactCompilerConfig = {
  sources: (filename) => {
    return filename.indexOf("src/path/to/dir") !== -1;
  },
};
```

드문 경우이지만, `compilationMode: "annotation"` 옵션을 사용하여 컴파일러를 "옵트인" 모드로 실행하도록 설정할 수도 있습니다.

이 옵션을 사용하면 `"use memo"` 지시어가 주석된 컴포넌트와 훅만 컴파일러가 컴파일합니다.

주석 모드는 초기 도입자를 돕기 위한 임시적인 것이며, "use memo" 지시어가 장기적으로 사용되기를 의도한 것은 아닙니다.

```ts
const ReactCompilerConfig = {
  compilationMode: "annotation",
};

// src/app.jsx
export default function App() {
  "use memo";
  // ...
}
```

컴파일러 도입에 대한 확신이 생기면, 다른 디렉토리로 적용 범위를 확장하여 점차 전체 앱으로 범위를 넓힐 수 있습니다.

### 새로운 프로젝트

새 프로젝트를 시작하는 경우, 컴파일러를 전체 코드베이스에 활성화할 수 있으며, 이는 기본 동작입니다.

### 사용법

### Babel

```
npm install babel-plugin-react-compiler
```

컴파일러는 빌드 파이프라인에서 실행할 수 있는 Babel 플러그인을 포함하고 있습니다.

설치 후, Babel 설정에 추가하십시오. 컴파일러가 파이프라인에서 **가장 먼저** 실행되는 것이 중요합니다:

```ts
// babel.config.js
const ReactCompilerConfig = {
  /* ... */
};

module.exports = function () {
  return {
    plugins: [
      ["babel-plugin-react-compiler", ReactCompilerConfig], // 가장 먼저 실행 되어야 합니다!
      // ...
    ],
  };
};
```

`babel-plugin-react-compiler`는 다른 Babel 플러그인들보다 먼저 실행되어야 합니다.

컴파일러는 정확한 분석을 위해 입력 소스 정보를 필요로 하기 때문입니다.

## Vite

만약 Vite를 사용하신다면, 설정 파일에 플러그인을 넣으실 수 있습니다.

```ts
// vite.config.js
const ReactCompilerConfig = {
  /* ... */
};

export default defineConfig(() => {
  return {
    plugins: [
      react({
        babel: {
          plugins: [["babel-plugin-react-compiler", ReactCompilerConfig]],
        },
      }),
    ],
    // ...
  };
});
```

## Next.js

Next.js는 React 컴파일러를 활성화하기 위한 실험적 설정을 제공합니다. 이 설정은 `babel-plugin-react-compiler`가 자동으로 설정되도록 합니다.

- Next.js canary를 설치하세요. 이는 React 19 Release Candidate를 사용합니다.

- `babel-plugin-react-compiler`를 설치하세요.

```
npm install next@canary babel-plugin-react-compiler
```

그 후에, `next.config.js`에서 실험적인 옵션을 설정해주세요.

```ts
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    reactCompiler: true,
  },
};

module.exports = nextConfig;
```

실험적 옵션을 사용하면 다음에서 React 컴파일러에 대한 지원이 보장됩니다:

- 앱 라우터(App Router)

- 페이지 라우터(Pages Router)

- 웹팩(Webpack) (기본 설정)

- 터보팩(Turbopack) (옵션으로 --turbo를 통해 활성화)

## - Remix

`vite-plugin-babel`을 설치하시고, 해당 플러그인을 추가하세요.

```
npm install vite-plugin-babel
```

```ts
// vite.config.js
import babel from "vite-plugin-babel";

const ReactCompilerConfig = {
  /* ... */
};

export default defineConfig({
  plugins: [
    remix({
      /* ... */
    }),
    babel({
      filter: /\.[jt]sx?$/,
      babelConfig: {
        presets: ["@babel/preset-typescript"], // if you use TypeScript
        plugins: [["babel-plugin-react-compiler", ReactCompilerConfig]],
      },
    }),
  ],
});
```

## Webpack

이 처럼 본인만의 React Compiler를 위한 로더를 만들 수 있습니다.

```ts
const ReactCompilerConfig = { /* ... */ };
const BabelPluginReactCompiler = require('babel-plugin-react-compiler');

function reactCompilerLoader(sourceCode, sourceMap) {
  // ...
  const result = transformSync(sourceCode, {
    // ...
    plugins: [
      [BabelPluginReactCompiler, ReactCompilerConfig],
    ],
  // ...
  });

  if (result === null) {
    this.callback(
      Error(
        `Failed to transform "${options.filename}"`
      )
    );
    return;
  }

  this.callback(
    null,
    result.code
    result.map === null ? undefined : result.map
  );
}

module.exports = reactCompilerLoader;
```

## Expo

Expo는 Metro를 통해 Babel을 사용하므로, 설치 지침은 [Babel 사용 섹션](https://react.dev/learn/react-compiler#usage-with-babel)을 참조하세요.

## Metro (React Native)

React Native는 Metro를 통해 Babel을 사용하므로, 설치 지침은 [Babel 사용 섹션](https://react.dev/learn/react-compiler#usage-with-babel)을 참조하세요.

## 문제 해결

이슈를 보고하려면 먼저 [React Compiler Playground](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAejQAgFTYHIQAuumAtgqRAJYBeCAJpgEYCemASggIZyGYDCEUgAcqAGwQwANJjBUAdokyEAFlTCZ1meUUxdMcIcIjyE8vhBiYVECAGsAOvIBmURYSonMCAB7CzcgBuCGIsAAowEIhgYACCnFxioQAyXDAA5gixMDBcLADyzvlMAFYIvGAAFACUmMCYaNiYAHStOFgAvk5OGJgAshTUdIysHNy8AkbikrIKSqpaWvqGIiZmhE6u7p7ymAAqXEwSguZcCpKV9VSEFBodtcBOmAYmYHz0XIT6ALzefgFUYKhCJRBAxeLcJIsVIZLI5PKFYplCqVa63aoAbm6u0wMAQhFguwAPPRAQA+YAfL4dIloUmBMlODogDpAA)에서 최소 재현 예제를 만들어 버그 보고서에 포함하세요.

이슈는 [facebook/react 저장소](https://github.com/facebook/react/issues)에 등록할 수 있습니다.

React Compiler Working Group에 가입하여 피드백을 제공할 수도 있습니다.

[가입에 대한 자세한 내용은 README를 참조](https://github.com/reactwg/react-compiler)하세요.

## (0 , \_c) is not a function 오류

이 오류는 React 19 RC 이상을 사용하지 않을 때 발생합니다.

이를 해결하려면 앱을 [React 19 RC로 업그레이드](https://react.dev/blog/2024/04/25/react-19-upgrade-guide)하세요.

React 19로 업그레이드할 수 없는 경우, [Working Group](https://github.com/reactwg/react-compiler/discussions/6)에서 설명한 캐시 함수의 사용자 공간 구현을 시도할 수 있습니다.

그러나 이는 권장되지 않으며 가능한 한 빨리 React 19로 업그레이드해야 합니다.

## 내 컴포넌트가 최적화되었는지 어떻게 알 수 있나요?

[React Devtools(v5.0+)](https://react.dev/learn/react-developer-tools)는 React 컴파일러에 대한 내장 지원을 제공하며, 컴파일러에 의해 최적화된 컴포넌트 옆에 "Memo ✨" 배지를 표시합니다.

## 컴파일 후 작동하지 않는 경우

eslint-plugin-react-compiler를 설치한 경우, 컴파일러는 에디터에서 React 규칙 위반 사항을 표시합니다.

이 경우, 컴파일러는 해당 컴포넌트나 훅의 최적화를 건너뛰었다는 의미입니다.

이는 전혀 문제가 없으며, 컴파일러는 이를 복구하고 코드베이스의 다른 컴포넌트를 계속해서 최적화할 수 있습니다.

모든 eslint 위반 사항을 즉시 수정할 필요는 없습니다.

최적화되는 컴포넌트와 훅의 수를 늘리기 위해 자신의 속도에 맞춰 이를 해결할 수 있습니다.

그러나 JavaScript의 유연하고 동적인 특성 때문에 모든 경우를 포괄적으로 감지하는 것은 불가능합니다.

이러한 경우에 무한 루프와 같은 버그나 정의되지 않은 동작이 발생할 수 있습니다.

컴파일 후 앱이 제대로 작동하지 않고 eslint 오류가 표시되지 않는다면, 컴파일러가 코드를 잘못 컴파일했을 수 있습니다.

이를 확인하려면 관련이 있을 수 있는 컴포넌트나 훅을 `"use no memo"` 지시어를 통해 강제로 옵트아웃하여 문제를 해결해보세요.

```ts
function SuspiciousComponent() {
  "use no memo"; // 이 컴포넌트를 React 컴파일러에 의해 컴파일되지 않도록 제외합니다.
  // ...
}
```

## 알아두어야 할 점

> "use no memo"

`"use no memo"`는 React 컴파일러에 의해 컴파일되지 않도록 컴포넌트와 훅을 배제할 수 있는 임시 탈출구입니다.

이 지시어는 "use client"와 같은 방식으로 장기적으로 사용되는 것이 아닙니다.

이 지시어는 꼭 필요한 경우가 아니라면 사용하는 것을 권장하지 않습니다.

일단 컴포넌트나 훅을 배제하면, 지시어를 제거하기 전까지는 영구적으로 배제된 상태로 유지됩니다.

즉, 코드를 수정하더라도 지시어를 제거하지 않으면 컴파일러는 여전히 컴파일을 건너뛰게 됩니다.

오류가 사라지면, 옵트아웃 지시어를 제거했을 때 문제가 다시 발생하는지 확인하십시오.

그런 다음 [React Compiler Playground](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAejQAgFTYHIQAuumAtgqRAJYBeCAJpgEYCemASggIZyGYDCEUgAcqAGwQwANJjBUAdokyEAFlTCZ1meUUxdMcIcIjyE8vhBiYVECAGsAOvIBmURYSonMCAB7CzcgBuCGIsAAowEIhgYACCnFxioQAyXDAA5gixMDBcLADyzvlMAFYIvGAAFACUmMCYaNiYAHStOFgAvk5OGJgAshTUdIysHNy8AkbikrIKSqpaWvqGIiZmhE6u7p7ymAAqXEwSguZcCpKV9VSEFBodtcBOmAYmYHz0XIT6ALzefgFUYKhCJRBAxeLcJIsVIZLI5PKFYplCqVa63aoAbm6u0wMAQhFguwAPPRAQA+YAfL4dIloUmBMlODogDpAA)를 사용하여 버그 보고서를 작성해 주세요.

문제를 작은 재현 예제로 줄이거나, 오픈 소스 코드인 경우 전체 소스를 붙여넣어도 됩니다.

이를 통해 문제를 식별하고 해결할 수 있도록 도와드리겠습니다.

## 다른 이슈들

[이 곳](https://github.com/reactwg/react-compiler/discussions/7)에 접속해주세요.
