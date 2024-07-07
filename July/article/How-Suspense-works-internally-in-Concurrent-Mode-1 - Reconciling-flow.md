## 🔗 [How Suspense works internally in Concurrent Mode 1 - Reconciling flow](https://jser.dev/react/2022/04/02/suspense-in-concurrent-mode-1-reconciling)

### 🗓️ 번역 날짜: 2024.07.01

### 🧚 번역한 크루: 버건디(전태헌)

---

저는 한 번 Suspense가 어떻게 작동하는지 알아보려고 했습니다.

제 [유튜브 비디오](https://www.youtube.com/watch?v=4Ippewm6AXk)에서 확인할 수 있지만, 굉장히 간단하며 React 18의 최신 논리를 반영하지 않았습니다.

이제 **동시성 모드에서 Suspense가 어떻게 작동하는지** 더 깊이 살펴보려고 합니다. 이것은 매우 복잡하게 밝혀져서 여러 에피소드에 걸쳐 다음과 같은 단계로 진행하려고 합니다.

1. 재조정(reconciling) - Suspense가 어떻게 재조정하는가

2. 오프스크린 컴포넌트(Offscreen component) - Suspense 컴포넌트에서 사용되는 내부 컴포넌트

3. Suspense 컨텍스트(Suspense context) - ??

4. 핑 & 재시도(Ping & Retry) - 프라미스가 해결된 후 깔끔하게 다시 렌더링되도록 하기

이번 에피소드는 첫 번째 주제인 재조정(reconciling)에 관한 것입니다.

## - Suspense Demo

[간단한 Suspense 데모](https://jser.dev/demos/react/suspense/basic)를 살펴보세요.

![](https://jser.dev/static/basic-suspense.gif)

코드는 매우 간단하며, 데이터가 준비되지 않았을 때 Promise를 던지는 기본적인 구현입니다.

```tsx
const getResource = (data, delay = 1000) => ({
  _data: null,
  _promise: null,
  status: "pending",
  get data() {
    if (this.status === "ready") {
      return this._data;
    } else {
      if (this._promise == null) {
        this._promise = new Promise((resolve) => {
          setTimeout(() => {
            this._data = data;
            this.status = "ready";
            resolve();
          }, delay);
        });
      }
      throw this._promise;
    }
  },
});
function App() {
  const [resource, setResource] = React.useState(null);
  return (
    <div className="app">
      <button
        onClick={() => {
          setResource(getResource("JSer"));
        }}
      >
        start
      </button>
      <React.Suspense fallback={<p>loading...</p>}>
        <Child resource={resource} />
      </React.Suspense>
    </div>
  );
}
```

기대했던대로 Fallback은 리소스가 로딩 될때 보여집니다.

## 어떻게 Suspense 컴포넌트가 자기 자신을 렌더링 하는지 살펴봅시다.

`beginWork()`에서 우리는 해당 소스 코드를 찾아볼수 있습나다. [코드](https://github.com/facebook/react/blob/b8cfda15e1232554487c7285fb464f22705a23ce/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L3925)

```ts
case SuspenseComponent:
  return updateSuspenseComponent(current, workInProgress, renderLanes);
```

초기 렌더링과 업데이트 모두 `updateSuspenseComponent`에서 이루어지며, 이는 매우 방대한 [코드 소스](https://github.com/facebook/react/blob/b8cfda15e1232554487c7285fb464f22705a23ce/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L2343)입니다. 이를 나누어서 설명해 보겠습니다.

```ts
function updateSuspenseComponent(current, workInProgress, renderLanes) {
  const nextProps = workInProgress.pendingProps;
  let suspenseContext: SuspenseContext = suspenseStackCursor.current;
  let showFallback = false;
  const didSuspend = (workInProgress.flags & DidCapture) !== NoFlags;
  if (
    didSuspend ||
    shouldRemainOnFallback(suspenseContext, current, workInProgress, renderLanes)
  ) {
    // Something in this boundary's subtree already suspended. Switch to
    // rendering the fallback children.
    showFallback = true;
    workInProgress.flags &= ~DidCapture;
  }
```

먼저, `SuspenseContext`가 무엇인지에 대해 알아볼 것입니다.

이는 다음 에피소드에서 다룰 내용이므로 지금은 넘어가겠습니다.

`showFallback`는 비교적 간단합니다.

이는 폴백을 표시할지 여부를 결정하는 변수로, 기본값은 false입니다.

`showFallback`이 `didSuspend`에 의존한다는 것을 알 수 있습니다. didSuspend는 다시 `DidCapture`에 의존하는데, 이는 중요한 플래그이므로 기억해 두세요.

`shouldRemainOnFallback()`은 SuspenseContext와 관련된 것으로, 이는 다른 에피소드에서 다룰 예정입니다.

`DidCapture`가 제거된다는 점에 주목하세요. 이는 미래의 리렌더링 시 올바른 내용을 가져오려고 시도할 것이며, 이는 다시 말해 프라미스가 다시 던져질 수 있음을 의미합니다. (이 [데모를 시도](https://jser.dev/demos/react/suspense/rethrow)해 보세요)

## initial mount

```ts
if (current === null) {
  const nextPrimaryChildren = nextProps.children;
  const nextFallbackChildren = nextProps.fallback;
  if (showFallback) {
    const fallbackFragment = mountSuspenseFallbackChildren(
      workInProgress,
      nextPrimaryChildren,
      nextFallbackChildren,
      renderLanes
    );
    const primaryChildFragment: Fiber = (workInProgress.child: any);
    primaryChildFragment.memoizedState =
      mountSuspenseOffscreenState(renderLanes);
    workInProgress.memoizedState = SUSPENDED_MARKER;
    return fallbackFragment;
  } else {
    return mountSuspensePrimaryChildren(
      workInProgress,
      nextPrimaryChildren,
      renderLanes
    );
  }
}
```

`current === null`은 초기 렌더링을 의미합니다.

`mountSuspenseFallbackChildren()`는 기본 자식 콘텐츠와 폴백(fallback) 모두를 마운트하지만, 반환되는 것은 Fallback입니다.

`memoizedState`도 초기화됩니다. 이는 이 Suspense가 폴백을 렌더링하고 있다는 표시입니다.

Fallback을 렌더링하지 않는 경우, `mountSuspensePrimaryChildren()`가 자식들을 마운트합니다.

`mountSuspenseFallbackChildren()`와 `mountSuspensePrimaryChildren()`에 대해서는 이번 에피소드에서 나중에 다시 다루겠습니다.

## - 업데이트

그리고 업데이트의 경우, 실제로 로직은 비슷합니다.

현재 상태와 될 상태에 따라 네 가지 분기로 나뉘게 되며, 이를 자세히 다룰 것입니다.

```ts
} else {
   // 이건 업데이트입니다.
  // 현재의 fiber에 SuspenseState가 있다면, 이는 이미 폴백(fallback)을 표시하고 있다는 것을 의미합니다.
  const prevState: null | SuspenseState = current.memoizedState;
  if (prevState !== null) {
    // current tree는 이미 fallback을 보여주고 있습니다.
    if (showFallback) {
      // prev: fallback, now: fallback
      ...
    } else {
      // prev: fallback, now: content
      ...
    }
  } else {
    if (showFallback) {
     // prev: content, now: callback
     ...
    } else {
      // prev: content, now: content
      ...
    }
  }
}
```

prev: fallback, now: fallback

```ts
const nextFallbackChildren = nextProps.fallback;
const nextPrimaryChildren = nextProps.children;
const fallbackChildFragment = updateSuspenseFallbackChildren(
  current,
  workInProgress,
  nextPrimaryChildren,
  nextFallbackChildren,
  renderLanes
);
const primaryChildFragment: Fiber = (workInProgress.child: any);
const prevOffscreenState: OffscreenState | null = (current.child: any)
  .memoizedState;
primaryChildFragment.memoizedState =
  prevOffscreenState === null
    ? mountSuspenseOffscreenState(renderLanes)
    : updateSuspenseOffscreenState(prevOffscreenState, renderLanes);
primaryChildFragment.childLanes = getRemainingWorkInPrimaryTree(
  current,
  renderLanes
);
workInProgress.memoizedState = SUSPENDED_MARKER;
return fallbackChildFragment;

```

두 경우 모두 폴백(fallback)을 렌더링하지만, 폴백 자체는 변경될 수 있습니다. `updateSuspenseFallbackChildren()`는 재조정을 수행합니다.

OffscreenState 부분은 약간 혼란스러울 수 있는데, 이는 Suspense Cache와 관련이 있습니다. 이 부분은 앞으로의 에피소드에서 다루겠습니다.

prev: fallback, now: content

```ts
const nextPrimaryChildren = nextProps.children;
const primaryChildFragment = updateSuspensePrimaryChildren(
  current,
  workInProgress,
  nextPrimaryChildren,
  renderLanes
);
workInProgress.memoizedState = null;
return primaryChildFragment;
```

이것은 간단합니다. 자식 부분을 재조정하면 됩니다.

**이전: 콘텐츠, 현재: 폴백**

코드는 `이전: 폴백, 현재: 콘텐츠`와 유사하게 건너뜁니다.

**이전: 콘텐츠, 현재: 콘텐츠**

코드는 `이전: 폴백, 현재: 콘텐츠`와 유사합니다.

## Suspense 내의 Wrapper들

Suspense 컴포넌트는 단순한 컴포넌트가 아니며, 자식들을 Offscreen 컴포넌트와 같은 것으로 감싸서 멋진 효과를 얻습니다.

Offscreen에 대해 잠깐 살펴보고, 자세한 내용은 앞으로의 에피소드에서 다루겠습니다.

**mountSuspenseFallbackChildren()**

이제 `mountSuspenseFallbackChildren()`에서 실제로 어떤 일이 발생하는지 살펴보겠습니다.

```ts
function mountSuspenseFallbackChildren(
  workInProgress,
  primaryChildren,
  fallbackChildren,
  renderLanes
) {
  const mode = workInProgress.mode;
  const progressedPrimaryFragment: Fiber | null = workInProgress.child;
  const primaryChildProps: OffscreenProps = {
    mode: "hidden",
    children: primaryChildren,
  };
  let primaryChildFragment;
  let fallbackChildFragment;
  primaryChildFragment = mountWorkInProgressOffscreenFiber(
    primaryChildProps,
    mode,
    NoLanes
  );
  fallbackChildFragment = createFiberFromFragment(
    fallbackChildren,
    mode,
    renderLanes,
    null
  );
  primaryChildFragment.return = workInProgress;
  fallbackChildFragment.return = workInProgress;
  primaryChildFragment.sibling = fallbackChildFragment;
  workInProgress.child = primaryChildFragment;
  return fallbackChildFragment;
}
```

1. primary child는 Offscreen Fiber로 감싸고, mode는 `hidden`으로 설정됩니다.

2. fallback은 Fragment로 감싸집니다.

3. primary child와 fallback 모두 자식으로 배치됩니다.

왜 fallback을 Fragment로 감쌀까요?

fallback은 `ReactNodeList`의 일종이기 때문에 숫자나 문자열일 수 있고, 일반적으로 문자열은 특별한 처리가 필요합니다.

따라서 Fragment로 감싸는 것이 더 쉽게 처리할 수 있을 것으로 보입니다.

```ts
export type ReactNode =
  | React$Element<any>
  | ReactPortal
  | ReactText
  | ReactFragment
  | ReactProvider<any>
  | ReactConsumer<any>;
export type ReactEmpty = null | void | boolean;
export type ReactFragment = ReactEmpty | Iterable<React$Node>;
export type ReactNodeList = ReactEmpty | React$Node;
export type ReactText = string | number;
```

여기 Suspense의 fiber 구조를 담은 다이어그램이 있습니다.

![](https://jser.dev/static/suspense-fiber-structure-hidden.png)

`mountWorkInProgressOffscreenFiber`의 특별한 점은 무엇일까요 ?

```ts
function mountWorkInProgressOffscreenFiber(
  offscreenProps: OffscreenProps,
  mode: TypeOfMode,
  renderLanes: Lanes
) {
  // `createFiberFromOffscreen` 함수의 props 인수는 `any` 타입으로 되어 있으므로,
  // 이를 제한하기 위해 이 래퍼 함수를 사용합니다.
  return createFiberFromOffscreen(offscreenProps, mode, NoLanes, null);
}
export function createFiberFromOffscreen(
  pendingProps: OffscreenProps,
  mode: TypeOfMode,
  lanes: Lanes,
  key: null | string
) {
  const fiber = createFiber(OffscreenComponent, pendingProps, key, mode);
  fiber.elementType = REACT_OFFSCREEN_TYPE;
  fiber.lanes = lanes;
  const primaryChildInstance: OffscreenInstance = {};
  fiber.stateNode = primaryChildInstance;
  return fiber;
}
```

특별한 점은 없어보이지만, `hidden`이나 `visible` 값을 가질 수 있는 `mode` 프로퍼티가 존재합니다.

**mountSuspensePrimaryChildren()**

```ts
function mountSuspensePrimaryChildren(
  workInProgress,
  primaryChildren,
  renderLanes
) {
  const mode = workInProgress.mode;
  const primaryChildProps: OffscreenProps = {
    mode: "visible",
    children: primaryChildren,
  };
  const primaryChildFragment = mountWorkInProgressOffscreenFiber(
    primaryChildProps,
    mode,
    renderLanes
  );
  primaryChildFragment.return = workInProgress;
  workInProgress.child = primaryChildFragment;
  return primaryChildFragment;
}
```

이번에도 Offscreen fiber를 사용하지만, 이번에는 fallback 없이 모드를 "visible"로 설정합니다.

![](https://jser.dev/static/suspense-fiber-structure-visible.png)

참고로, `workInProgress`도 `mode`를 가지고 있지만, 다른 유형인 `TypeOfMode`입니다.

```ts
export type TypeOfMode = number;
export const NoMode = /*                         */ 0b000000;
// TODO: ConcurrentMode를 제거하고 루트 태그에서 읽어오기
export const ConcurrentMode = /*                 */ 0b000001;
export const ProfileMode = /*                    */ 0b000010;
export const DebugTracingMode = /*               */ 0b000100;
export const StrictLegacyMode = /*               */ 0b001000;
export const StrictEffectsMode = /*              */ 0b010000;
export const ConcurrentUpdatesByDefaultMode = /* */ 0b100000;
```

왜 우리가 primary children을 fiber tree에 유지하는지 궁금할 수 있습니다.

왜 그냥 제거하지 않을까요? 좋은 질문입니다. 간단히 말하면, 이는 fiber의 상태를 유지하기 위함입니다.

fallback에서 다시 전환될 때 모든 것이 새로워지기를 원하지 않기 때문입니다.

자세한 내용은 Offscreen의 다음 에피소드에서 다룰 예정입니다.

이제 Promise가 어떻게 작용하는지 알아봅시다.

## 어떻게 Suspense 내부에서 Promise 가 감지 되고 업데이트가 유발 되는 걸까요 ?

우리는 이미 suspense가 promise가 발생했을 때 반응한다는 것을 알고 있습니다. 이는 에러 처리의 일부이므로, 먼저 handleError로 가보겠습니다. [출처](https://github.com/facebook/react/blob/5a1e558df21bd3cafbaea01cc418fa69d14a8cab/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L1553)

```ts
function handleError(root, thrownValue): void {
  do {
    let erroredWork = workInProgress;
    try {
      // 렌더링 단계 동안 설정된 모듈 수준 상태를 리셋합니다.
      resetContextDependencies();
      resetHooksAfterThrow();
      // TODO: 별도의 문제를 조사하는 동안 이 누락된 줄을 발견하고 추가했습니다.
      // string refs를 사용하여 회귀 테스트를 작성하세요.
      ReactCurrentOwner.current = null;
      throwException(
        root,
        erroredWork.return,
        erroredWork,
        thrownValue,
        workInProgressRootRenderLanes
      );
      completeUnitOfWork(erroredWork);
    } catch (yetAnotherThrownValue) {
      // 반환 경로에서 무언가가 다시 오류를 발생시켰습니다.
      thrownValue = yetAnotherThrownValue;
      if (workInProgress === erroredWork && erroredWork !== null) {
      // 이 경계(boundary)에서 이미 오류가 발생한 경우, 오류를 처리하는 데 문제가 있었습니다.
      // 오류를 다음 경계로 전파합니다.
      }

        erroredWork = erroredWork.return;
        workInProgress = erroredWork;
      } else {
        erroredWork = workInProgress;
      }
      continue;
    }
      // 정상적인 작업 루프로 돌아갑니다.
    return;
  } while (true);
}
```

중요한 부분은 이 두 함수의 호출입니다.

1. throwException
2. completeUnitOfWork

## - throwException

[소스 코드](https://github.com/facebook/react/blob/b8cfda15e1232554487c7285fb464f22705a23ce/packages/react-reconciler/src/ReactFiberThrow.new.js#L430)

이것은 매우 큰 코드 조각이므로, 나누어서 설명해 보겠습니다. 먼저, throw한 fiber가 Incomplete로 표시됩니다.

```ts
// 이 source Fiber는 해결 되지 않았습니다.
sourceFiber.flags |= Incomplete;
```

```ts
if (
  value !== null &&
  typeof value === 'object' &&
  typeof value.then === 'function'
) {
  // 이것은 깨울 수 있는(wakeable) 상태입니다. 해당 컴포넌트가 일시 중단되었습니다.
  const wakeable: Wakeable = (value: any);
  ...
} else {
  // 일반적인 에러
}
```

우리는 `wakeable`을 단순히 throw된 Promise로 생각할 수 있습니다.

Promise가 아니라면, 이는 단순히 Error Boundary에서 처리해야 할 일반적인 오류입니다.

([ErrorBoundary에 대한 내 영상](https://www.youtube.com/watch?v=0TnuJKLjMyg)을 참고하세요).

이제 Suspense 부분에 집중해봅시다.

```ts
// 가장 가까운 Suspense가 시간 초과된 뷰를 다시 렌더링하도록 스케줄합니다.
const suspenseBoundary = getNearestSuspenseBoundaryToCapture(returnFiber);
```

먼저 가장 가까운 Suspense를 찾습니다. 여기서 이를 Suspense Boundary라고 부르며, 이는 Error Boundary와 매우 유사합니다.

`getNearestSuspenseBoundaryToCapture`는 간단하므로 생략할 것입니다.

이는 단순히 `return`을 통해 조상 fiber 노드를 재귀적으로 추적합니다. [출처](https://github.com/facebook/react/blob/b8cfda15e1232554487c7285fb464f22705a23ce/packages/react-reconciler/src/ReactFiberThrow.new.js#L277)

```ts
if (suspenseBoundary !== null) {
  suspenseBoundary.flags &= ~ForceClientRender;
  markSuspenseBoundaryShouldCapture(
    suspenseBoundary,
    returnFiber,
    sourceFiber,
    root,
    rootRenderLanes
  );
  // concurrent 모드에서만 ping 리스너를 연결합니다. Legacy Suspense는 항상 fallback을 동기적으로 커밋하기 때문에 ping이 없습니다.

  if (suspenseBoundary.mode & ConcurrentMode) {
    attachPingListener(root, wakeable, rootRenderLanes);
  }
  attachRetryListener(suspenseBoundary, root, wakeable, rootRenderLanes);
  return;
}
```

Suspense Boundary를 찾은 후, 여기서 3가지 작업을 수행합니다:

1. markSuspenseBoundaryShouldCapture()
2. attachPingListener()
3. attachRetryListener()

명백히, `markSuspenseBoundaryShouldCapture()`는 Suspense가 fallbacks를 렌더링하도록 하기 위한 것이고, 다른 두 가지는 어떤 방식으로든 Promise에 콜백을 연결하는 것입니다. 이들이 완료되면 콘텐츠를 렌더링해야 하기 때문입니다.

2번과 3번은 Ping & Retry의 다음 에피소드에서 자세히 설명될 것입니다.

### 만약 우리가 Suspense를 찾지 못하면 어떻게 될까요

코드를 계속 보면, SyncLane이 아닌 경우, Suspense Boundary가 없어도 괜찮다는 것을 알 수 있습니다.

```ts
else {
  // 경계(boundary)가 발견되지 않았습니다. sync 업데이트가 아닌 경우, 괜찮습니다.
  // 우리는 일시 중단하고 더 많은 데이터가 도착하기를 기다릴 수 있습니다.
  if (!includesSyncLane(rootRenderLanes)) {
    // 이것은 sync 업데이트가 아닙니다. 일시 중단합니다. Suspense 경계를 활성화하지 않기 때문에,
    // fallback을 렌더링하기 위해 두 번째 패스를 수행하지 않고 루트까지 모두 되감깁니다.
    // (이는 refresh 전환이 작동해야 하는 방식이기도 합니다. 어차피 fallback을 커밋하지 않을 것이기 때문입니다.)
    //
    // 이 경우는 초기 하이드레이션에도 적용됩니다.
    attachPingListener(root, wakeable, rootRenderLanes);
    renderDidSuspendDelayIfPossible();
    return;
  }
  // 이것은 sync/개별 업데이트입니다. 우리는 이 경우를 오류처럼 취급합니다.
  // 개별 렌더링은 외부 상태와의 일관성을 유지하기 위해 동기적으로 완전한 트리를 생성해야 합니다.
  const uncaughtSuspenseError = new Error(
    "동기 입력에 응답하는 동안 컴포넌트가 일시 중단되었습니다. " +
      "이로 인해 UI가 로딩 표시기로 대체됩니다. 이를 수정하려면, " +
      "일시 중단되는 업데이트를 startTransition으로 감싸야 합니다."
  );
  // 전환 외부에 있는 경우, 일반적인 오류 경로로 넘어갑니다.
  // 오류는 가장 가까운 Suspense 경계에서 포착될 것입니다.
  value = uncaughtSuspenseError;
}
```

간단히 말하면, suspense가 사용자 액션에 의해 발생한 경우, Suspense 경계가 필요합니다.

사용자 액션이 아니거나 전환 중인 경우, `attachPingListener()`와 `renderDidSuspendDelayIfPossible()`가 복구를 시도합니다.

다음은 [Suspense 경계 없이 전환을 사용하는 데모](https://jser.dev/demos/react/suspense/transition)입니다. 이 데모에서 Suspense 경계가 없어도 여전히 작동하는 것을 볼 수 있습니다.

**markSuspenseBoundaryShouldCapture()**

[소스 코드](https://github.com/facebook/react/blob/b8cfda15e1232554487c7285fb464f22705a23ce/packages/react-reconciler/src/ReactFiberThrow.new.js#L297)

`markSuspenseBoundaryShouldCapture()` 함수는 Concurrent Mode에서 사용되는 로직에만 집중하면 됩니다. Legacy Suspense는 Concurrent Mode 이전에 사용된 것이므로, 이를 무시하고 Concurrent Mode에만 초점을 맞추세요.

```
suspenseBoundary.flags |= ShouldCapture;
```

여기서 `ShouldCapture`가 설정되는데, 이것이 `DidCapture`로 변환되는 단계가 있을 것입니다. 그 부분은 나중에 다룰 테니 잠시 기다려 주세요.

```ts
sourceFiber.flags |= ForceUpdateForLegacySuspense;
// 이 fiber가 완료되지 않았음에도 불구하고 커밋할 것입니다.
// 하지만 어떤 라이프사이클 메서드나 콜백도 호출해서는 안 됩니다. 모든 라이프사이클 효과 태그를 제거합니다.
sourceFiber.flags &= ~(LifecycleEffectMask | Incomplete);
```

source fiber는 이미 Incomplete로 표시했지만, 여기서는 그 플래그를 제거합니다.

```ts
export const LifecycleEffectMask =
  Passive | Update | Callback | Ref | Snapshot | StoreConsistency;
```

LifecycleEffectMask는 모든 부수 효과를 포함하므로, 이는 실제로 완료되지 않았지만 마치 완료된 것처럼 처리한다는 의미입니다.

```ts
// source fiber가 완료되지 않았습니다. 아직 처리해야 할 작업이 남아 있음을 나타내기 위해 Sync 우선순위로 표시합니다.
sourceFiber.lanes = mergeLanes(sourceFiber.lanes, SyncLane);
```

이는 Suspense가 렌더링될 때 DidCapture를 제거하는 것과 관련이 있습니다. 다시 렌더링할 때, 오류가 발생한 컴포넌트가 다시 렌더링되도록 하기 위해 `lanes`가 설정되어 [bail-out](https://jser.dev/react/2022/01/07/how-does-bailout-work)을 피합니다.

그런 다음 `completeUnitOfWork(erroredWork)`로 이동합니다.

## completeUnitOfWork

throwException()이 완료된 후, `completeUnitOfWork()`가 호출됩니다. [source](https://github.com/facebook/react/blob/b8cfda15e1232554487c7285fb464f22705a23ce/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L1858)

Suspense에서는 작업이 Incomplete 상태이므로, 우리는 Incomplete 브랜치만 살펴보겠습니다.

```ts
function completeUnitOfWork(unitOfWork: Fiber): void {
  // 현재 작업 단위를 완료한 후 다음 형제 작업으로 이동합니다.
  // 형제 작업이 없으면 부모 fiber로 돌아갑니다.
  let completedWork = unitOfWork;
  do {
    // 현재의 플러시된 상태는 alternate입니다.
    // 이상적으로는 이 상태에 의존하지 않아야 하지만, 여기서 의존하면 진행 중인 작업에 추가 필드가 필요하지 않습니다.
    const current = completedWork.alternate;
    const returnFiber = completedWork.return;

    // 작업이 완료되었는지 아니면 예외가 발생했는지 확인합니다.
    if ((completedWork.flags & Incomplete) === NoFlags) {
      // 작업이 정상적으로 완료된 경우의 처리입니다.
      // 이 부분은 생략합니다.
    } else {
      // 이 fiber는 예외가 발생하여 완료되지 않았습니다.
      // complete 단계에 들어가지 않고 스택에서 값을 pop 합니다.
      // 만약 이것이 boundary라면, 가능한 값을 캡처합니다.
      const next = unwindWork(current, completedWork, subtreeRenderLanes);

      // 이 fiber가 완료되지 않았기 때문에, lanes를 초기화하지 않습니다.
      if (next !== null) {
        // 이 작업을 완료하는 과정에서 새로운 작업이 생성된 경우, 그 작업을 다음으로 수행합니다.
        // 다시 여기에 돌아올 것입니다.
        // 재시작하고 있으므로, 호스트 효과가 아닌 것은 effect 태그에서 제거합니다.
        next.flags &= HostEffectMask;
        workInProgress = next;
        return;
      }

      if (returnFiber !== null) {
        // 부모 fiber를 미완료 상태로 표시하고, 서브트리 플래그를 초기화합니다.
        returnFiber.flags |= Incomplete;
        returnFiber.subtreeFlags = NoFlags;
        returnFiber.deletions = null;
      } else {
        // 루트까지 모두 되돌아왔습니다.
        workInProgressRootExitStatus = RootDidNotComplete;
        workInProgress = null;
        return;
      }
    }

    const siblingFiber = completedWork.sibling;
    if (siblingFiber !== null) {
      // 이 returnFiber에 더 할 일이 있으면, 다음으로 수행합니다.
      workInProgress = siblingFiber;
      return;
    }

    // 그렇지 않으면, 부모로 돌아갑니다.
    completedWork = returnFiber;
    // 예외가 발생한 경우 다음 작업 항목을 업데이트합니다.
    workInProgress = completedWork;
  } while (completedWork !== null);

  // 루트에 도달했습니다.
  if (workInProgressRootExitStatus === RootInProgress) {
    workInProgressRootExitStatus = RootCompleted;
  }
}
```

completeUnitOfWork는 fiber 노드를 조정하는 과정에서 마지막 단계입니다.

[Traversal 알고리즘](https://jser.dev/react/2022/01/16/fiber-traversal-in-react)의 설명에 따라 이 함수는 작업 단위를 완료하고, 형제 노드나 부모 노드로 이동하면서 모든 작업을 처리합니다.

Incomplete 상태의 fiber 노드를 처리하는 코드를 좀 더 자세히 설명하겠습니다.

```ts
const next = unwindWork(current, completedWork, subtreeRenderLanes);

// 이 fiber가 완료되지 않았기 때문에, lanes를 초기화하지 않습니다.
if (next !== null) {
  // 이 작업을 완료하는 과정에서 새로운 작업이 생성된 경우, 그 작업을 다음으로 수행합니다.
  // 다시 여기에 돌아올 것입니다.
  // 재시작하고 있으므로, 호스트 효과가 아닌 것은 effect 태그에서 제거합니다.
  next.flags &= HostEffectMask;
  workInProgress = next;
  return;
}
```

unwindWork가 반환되면 일부 작업을 계속할 기회를 제공합니다.

또한 조상 노드를 재귀적으로 미완료 상태로 표시합니다.

이름에서 알 수 있듯이 `unwindWork`는 컨텍스트 등의 정리를 수행합니다.[소스코드](https://github.com/facebook/react/blob/b8cfda15e1232554487c7285fb464f22705a23ce/packages/react-reconciler/src/ReactFiberUnwindWork.new.js#L52)

```ts
case SuspenseComponent: {
  popSuspenseContext(workInProgress);
  const flags = workInProgress.flags;
  if (flags & ShouldCapture) {
    workInProgress.flags = (flags & ~ShouldCapture) | DidCapture;
    // 서스펜스 효과를 캡처했습니다. 경계를 다시 렌더링합니다.
    if (
      enableProfilerTimer &&
      (workInProgress.mode & ProfileMode) !== NoMode
    ) {
      transferActualDuration(workInProgress);
    }
    return workInProgress;
  }
  return null;
}
```

Suspense로 되돌아갈 때, 다음과 같은 일이 일어납니다:

1. 서스펜스 컨텍스트를 pop 합니다. 이는 이후 에피소드에서 다룰 것입니다.
2. ShouldCapture를 찾으면, 이를 DidCapture로 설정하고 자신을 반환합니다.

ShouldCapture는 complete 단계에서 DidCapture로 변환됩니다.

## 요약

오랜 여정을 거쳐, 다음은 요약입니다.

Suspense는 DidCapture 플래그를 사용:

1. `DidCapture` 플래그를 사용하여 fallback을 렌더링할지, 내용을 렌더링할지 결정합니다 (기본 자식들).

2. Suspense는 내용을 Offscreen 컴포넌트로 감쌉니다:
   이로 인해 fallback이 렌더링되더라도, 내용이 fiber 트리에서 제거되지 않아 상태가 유지됩니다.

3. 조정 중 Suspense의 결정:
   `DidCapture` 플래그를 기준으로 Offscreen을 건너뛸지 결정합니다.이는 "일부 fibers를 숨기는" 효과를 만듭니다.

4. Promise가 throw될 때:
   가장 가까운 Suspense 경계를 찾아 `ShouldCapture` 플래그를 설정하고, promise는 ping 및 retry 리스너와 함께 체인됩니다.
   오류가 발생한 경우, errored 컴포넌트에서 Suspense까지의 모든 fiber가 Incomplete로 완료됩니다.
   가장 가까운 Suspense를 완료하려고 할 때, `ShouldCapture`는 DidCapture로 표시되고 Suspense 자체를 반환합니다.
   작업 루프가 Suspense를 조정: 이번에는 fallback 브랜치를 렌더링하면서 계속 진행합니다.

5. Promise가 해결될 때:
   ping 및 retry 리스너가 다시 렌더링되도록 합니다. (자세한 내용은 이후 에피소드에서 다룹니다)
