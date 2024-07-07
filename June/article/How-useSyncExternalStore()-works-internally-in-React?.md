## 🔗 [How useSyncExternalStore() works internally in React?](https://jser.dev/2023-08-02-usesyncexternalstore/)

### 🗓️ 번역 날짜: 2024.06.10

### 🧚 번역한 크루: 버건디(전태헌)

---

# 리액트에서 useSyncExternalStore는 내부적으로 어떻게 동작하는가 ?

`useSyncExternalStore()`는 외부 저장소에 구독할 수 있게 해주는 React 훅입니다.

저는 이 훅을 사용해본 적이 없지만, 로컬 스토리지와 같은 자체 상태를 가진 웹 API와 상호작용하는 것이 일반적이기 때문에 꽤 유용할 것 같습니다.

## 1. 왜 useSyncExternalStore는 필요한가 ?

아래는 React.dev에서 가져온 [데모 코드](https://react.dev/reference/react/useSyncExternalStore)입니다.

이 코드는 브라우저의 현재 온라인 상태를 보여줍니다.

네트워크를 끄면 상태가 그에 따라 업데이트되는 것을 볼 수 있습니다.

일부 코드는 수정되었습니다.

## - useOnlineStatus.js

```jsx
import { useEffect, useCallback, useState } from "react";

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  const update = useCallback(() => {
    setIsOnline(navigator.onLine);
  }, []);

  useEffect(() => {
    window.addEventListener("online", update);
    window.addEventListener("offline", update);
    return () => {
      window.removeEventListener("online", update);
      window.removeEventListener("offline", update);
    };
  }, [update]);

  return isOnline;
}
```

## - App.js

```jsx
import { useOnlineStatus } from "./useOnlineStatus.js";

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? "✅ Online" : "❌ Disconnected"}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log("✅ Progress saved");
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? "Save progress" : "Reconnecting..."}
    </button>
  );
}

export default function App() {
  return (
    <>
      <p>Turn on & off your network to see the status changing</p>
      <StatusBar />
    </>
  );
}
```

저만의 `useOnlineStatus`를 아래와 같이 만들었습니다.

```jsx
import { useEffect, useCallback, useState } from "react";
export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  const update = useCallback(() => {
    setIsOnline(navigator.onLine);
  }, []);
  useEffect(() => {
    window.addEventListener("online", update);
    window.addEventListener("offline", update);
    return () => {
      window.removeEventListener("online", update);
      window.removeEventListener("offline", update);
    };
  }, [update]);
  return isOnline;
}
```

코드는 괜찮아 보이지만 실제로는 견고하지 않습니다.

핵심 문제는 외부 저장소가 언제든지 업데이트될 수 있다는 점입니다.

따라서 `useState()`와 `useEffect()` 사이에 변경될 수 있으며, 이 경우 이벤트 핸들러가 나중에 등록되므로 변경 사항을 감지할 수 없습니다.

좀 더 일찍 실행되는 `useLayoutEffect()`나 `useInsertionEffect()`로 전환해도 문제는 해결되지 않습니다.

외부 저장소에 대해 아무것도 알 수 없기 때문입니다.

이 문제를 해결하려면 **이벤트 핸들러가 외부 저장소를 처음 가져올 때보다 늦지 않게 등록되도록 하거나**, 이벤트 핸들러가 등록된 후 한 번 더 확인해야 합니다.

아래와 같이 할 수 있습니다.

```ts
import { useEffect, useCallback, useState } from "react";
export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  const update = useCallback(() => {
    setIsOnline(navigator.onLine);
  }, []);
  useEffect(() => {
    window.addEventListener("online", update);
    window.addEventListener("offline", update);
    update();
    // 외부 저장소가 변경될 수 있으므로 한 번 더 확인하는 방법은 다음과 같습니다.

    return () => {
      window.removeEventListener("online", update);
      window.removeEventListener("offline", update);
    };
  }, [update]);
  return isOnline;
}
```

중복된 코드를 지움으로써 조금 더 낫도록 수정해보겠습니다.

```ts
import { useEffect, useCallback, useState } from "react";
function getIsOnLine() {
  return navigator.onLine;
}
function subscribe(callback) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);
  callback();
  // 이는 최신 데이터를 검색하는 데 중요합니다.

  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}
export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(getIsOnLine());
  const update = useCallback(() => {
    setIsOnline(getIsOnLine());
  }, []);
  useEffect(() => {
    return subscribe(update);
  }, [update]);
  return isOnline;
}
```

우리는 이것을 `useSyncExternalStore()`처럼 더 비슷하게 만들 수 있습니다.

```tsx
import { useEffect, useCallback, useState } from "react";

function getIsOnLine() {
  return navigator.onLine;
}

function subscribe(callback) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);
  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}
function useSyncExternalStore(subscribe, getSnapshot) {
  const [data, setData] = useState(getSnapshot());
  const update = useCallback(() => {
    setData(getSnapshot());
  }, []);
  useEffect(() => {
    update();
    return subscribe(update);
  }, [update]);
  return data;
}

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getIsOnLine);
  return isOnline;
}
```

## - App.js

```tsx
import { useOnlineStatus } from "./useOnlineStatus.js";

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? "✅ Online" : "❌ Disconnected"}</h1>;
}

export default function App() {
  return (
    <>
      <p>Turn on & off your network to see the status changing</p>
      <StatusBar />
    </>
  );
}
```

우리가 외부 저장소를 동기화할 때 어려운 부분은 외부의 모든 업데이트가 감지되도록 하는 것입니다.

우리의 해결책은 **이벤트 리스너가 등록된 후 업데이트가 발생하도록 보장하는 것**입니다.

하지만 여전히 우리의 구현에는 문제가 있습니다. 바로 '찢김(tear)' 현상입니다.

## - 찢김(Tearing) 문제

리액트 팀이 [찢김(tearing)](https://github.com/reactwg/react-18/discussions/69) 현상에 대해 훌륭하게 설명해주었는데, 여기서는 간단히 요약해보겠습니다.

리액트 팀에서 [useTransition()이 내부적으로 어떻게 작동하는지 설명](https://jser.dev/2023-05-19-how-does-usetransition-work/)할 때 언급했듯이, 동시 모드(concurrent mode)에서는 렌더링 단계가 작업의 우선순위에 따라 중단되고 재개될 수 있습니다.

그래서 리액트가 React 트리를 렌더링하는 동안, 이 과정이 중단되고 비동기 스케줄링으로 여러 번 시도될 가능성이 있습니다.

이 스케줄링이 동기적이지 않기 때문에, [React 스케줄러가 어떻게 작동](https://jser.dev/react/2022/03/16/how-react-scheduler-works)하는지 설명할 때 언급했듯이, 기본적으로 최소 지연이 없는 더 나은 `setTimeout()`이라고 생각할 수 있습니다.

따라서 렌더링 도중 외부 저장소가 변경되면 UI의 다른 부분에서 서로 다른 데이터가 커밋되는 것을 볼 수 있습니다.

이것이 찢김(tear) 현상이 발생하는 이유이며, 아래는 그 예시입니다.

## - App.js

```tsx
import { useEffect, startTransition, useCallback, useState } from "react";

let data = 1;
function getData() {
  return data;
}

setTimeout(() => (data = 2), 100);

function Cell() {
  let start = Date.now();
  while (Date.now() - start < 50) {
    // 동시성 모드에서 메인 스레드에 작업을 양보합니다.
    // 50밀리초 동안 실행 되어 메인 스레드를 차단하고, 메인 스레드가
    // 이 시긴 동안 다른 작업을 처리할 수 없게 만듭니다.
  }
  const data = getData();
  return <div style={{ padding: "1rem", border: "1px solid red" }}>{data}</div>;
}

export default function App() {
  const [showCells, setShowCells] = useState(false);

  useEffect(() => {
    startTransition(() => setShowCells(true));
  }, []);
  return (
    <>
      <p>
        Example of tearing. <br />
        below are multiple cells rendering same external data which changes during
        rendering.
      </p>
      {showCells ? (
        <div style={{ display: "flex", gap: "1rem" }}>
          <Cell />
          <Cell />
          <Cell />
          <Cell />
        </div>
      ) : (
        <p>preparing..</p>
      )}
    </>
  );
}
```

찢김(tear) 현상은 동시 모드(concurrent mode)에서 렌더링이 중단될 가능성 때문에 발생합니다.

(위 코드에서 `startTransition()`을 제거하면 찢김 현상이 재현되지 않습니다.)

React는 내부 Fiber 아키텍처에 우리가 개입하는 것을 허용하지 않기 때문에, 이 문제를 해결할 방법이 없습니다.

그래서 React는 우리에게 `useSyncExternalStore()`를 제공합니다.

아래는 위 예제에서 약간 수정한 후 `useSyncExternalStore()`를 사용한 데모입니다.

## - App.js

```ts
import { useEffect, useSyncExternalStore, startTransition, useCallback, useState } from 'react';

let data = 1
function getData() {
  return data
}

setTimeout(() => data = 2, 100)

function Cell() {
  let start = Date.now()
  while (Date.now() - start < 50) {
    // 동시성 모드에서 메인 스레드에 작업을 양보합니다.
    // 50밀리초 동안 실행 되어 메인 스레드를 차단하고, 메인 스레드가
    // 이 시긴 동안 다른 작업을 처리할 수 없게 만듭니다.
  }
  const data = useSyncExternalStore(() => {return () => {}},getData);
  return <div style={{padding: '1rem', border: '1px solid red'}}>{data}</div>
}

export default function App() {
  const [showCells, setShowCells] = useState(false)

  useEffect(() => {
    startTransition(() => setShowCells(true))
  }, [])
  return (
    <>
      <p>Example of tearing. <br/>below are multiple cells rendering same external data which changes during rendering.</p>
      {showCells ? <div style={{display: 'flex', gap: '1rem'}}>
        <Cell/><Cell/><Cell/><Cell/>
      </div> : <p>preparing..</p>}
    </>
  );
}
```

찢김(tearing) 현상이 고쳐진걸 볼 수 있습니다.

## 2. 어떻게 useSyncExternalStore()가 리액트 내부에서 동작하나요 ?

요약하자면, `useSyncExternalStore()`는 우리에게 두 가지를 제공합니다.

1. 외부 저장소의 모든 변경 사항이 감지되도록 합니다.

2. 동시 모드(concurrent mode)에서도 동일한 데이터가 UI에서 동일한 저장소로 렌더링되도록 합니다.

이제 React 내부 구조를 살펴보면서 이것이 어떻게 구현되는지 알아보겠습니다.

### 2.1 초기 마운트 시에 mountSyncExternalStore()

```ts
function mountSyncExternalStore<T>(
  subscribe: (() => void) => () => void,
  getSnapshot: () => T,
  getServerSnapshot?: () => T,
): T {
  const fiber = currentlyRenderingFiber;
  const hook = mountWorkInProgressHook();
 // 새로운 훅을 만듭니다.

  let nextSnapshot;
  const isHydrating = getIsHydrating();
  if (isHydrating) {
    ...
  } else {
    nextSnapshot = getSnapshot();
retrieve the data once, just like we did in initializer of useState()

// 블로킹 레인을 렌더링하지 않는 한, 일관성 검사를 스케줄링합니다.
// 커밋하기 직전에 트리를 탐색하여
// 어떤 저장소가 변경되었는지 확인할 것입니다.
//
// 서버 렌더링된 콘텐츠를 수분 공급(hydrating)하고 있는 경우에는 이것을 수행하지 않습니다.
// 콘텐츠가 오래되었을 경우, 어차피 이미 보이고 있기 때문입니다.
// 대신, 우리는 이를 수동 효과(passive effect)에서 수정할 것입니다.

    const root: FiberRoot | null = getWorkInProgressRoot();
    if (!includesBlockingLane(root, renderLanes)) {
      pushStoreConsistencyCheck(fiber, getSnapshot, nextSnapshot);
    }

// 이 부분이 중요합니다.
// 커밋하기 직전에 외부 데이터가 변경되었는지 확인하는 검사를 스케줄링합니다.
// 이는 비차단 레인(non-blocking lane)에만 적용되며, 기본적으로 이는 동시 모드(concurrent mode)에서 작동함을 의미합니다.
// 이는 앞서 언급한 찢김(tear) 현상을 해결하려고 시도하는 것입니다.

  }

// 매 렌더링 시마다 저장소에서 현재 스냅샷을 읽어옵니다.
// 이것은 React의 일반적인 규칙을 깨지만, 저장소 업데이트는 항상 동기적이기 때문에 작동합니다.
  hook.memoizedState = nextSnapshot;
// 이 hoo.memoizedState는 데이터를 유지하는 역할을 합니다.
// 우리의 구현에서 useState()와 동일한 기능을 합니다.

  const inst: StoreInstance<T> = {
    value: nextSnapshot,
    getSnapshot,
  };
  hook.queue = inst;

//이것은 우리가 "React에서 useState()가 내부적으로 어떻게 작동하는가"에서 언급한 업데이트 큐와 유사해 보이지만, 실제로는 필드 이름만 빌려온 것입니다.

  //저장소에 구독하기 위한 효과를 스케줄링합니다.
  mountEffect(subscribeToStore.bind(null, fiber, inst, subscribe), [subscribe]);
Great, mountEffect() is the internal of useEffect(), so

it is similar to what we do, scheduling a (passive) effect to init subscription

// 변경 가능한 인스턴스 필드를 업데이트하기 위한 효과를 스케줄링합니다.
// subscribe, getSnapshot 또는 값이 변경될 때마다 이를 업데이트할 것입니다.
// 정리(clean-up) 함수가 없고, 종속성을 올바르게 추적하기 때문에
// 추가 상태를 저장하지 않고도 직접 pushEffect를 호출할 수 있습니다.
// 같은 이유로, 정적 플래그를 설정할 필요도 없습니다.
// TODO: 커밋 전 일관성 검사를 추가하면 이를 수동 효과(passive phase)로 이동할 수 있습니다.
// 다음 주석을 참조하세요.
  fiber.flags |= PassiveEffect;

  // pushEffect : React에서 useEffect()가 내부적으로 어떻게 작동하는지 설명한 것과 유사합니다.
  // 이것은 효과를 Fiber에 푸시합니다.
  pushEffect(


    HookHasEffect | HookPassive,
// HookPassive : 수동 효과(passive effect)는 useEffect()에서 한 것과 유사하다는 것을 의미합니다.

    updateStoreInstance.bind(null, fiber, inst, nextSnapshot, getSnapshot),
// updateStoreInstance :
// 초기 마운트 시, nextSnapshot이 동일하기 때문에 검사의 이점을 취합니다. 이론적으로 314번 줄의 getSnapshot()과 359번 줄의 mountEffect() 사이에 작은 창이 있기 때문입니다. 찢김(tear) 문제를 해결하기 위한 일관성 검사와는 달리, 최신 데이터가 업데이트되었는지 확인하기 위해 다시 렌더링합니다. 이는 우리의 구현에서 update()를 다시 호출하는 목적과 유사합니다.

    undefined,
    null,
  );
  return nextSnapshot;
}
function subscribeToStore(fiber, inst, subscribe) {
  const handleStoreChange = () => {
// 저장소가 변경되었습니다.
// 마지막으로 저장소에서 읽은 이후 스냅샷이 변경되었는지 확인합니다.
    if (checkIfSnapshotChanged(inst)) {
      // 리렌더링을 강제합니다.
      forceStoreRerender(fiber);
    }
  };
// 저장소에 구독하고 정리(clean-up) 함수를 반환합니다.
  return subscribe(handleStoreChange);
}
function updateStoreInstance<T>(
  fiber: Fiber,
  inst: StoreInstance<T>,
  nextSnapshot: T,
  getSnapshot: () => T,
) {
// 이것들은 수동 단계에서 업데이트됩니다.
  inst.value = nextSnapshot;
  inst.getSnapshot = getSnapshot;

// 렌더와 커밋 사이에 무언가가 변경되었을 수 있습니다.
// 이것은 수동 효과가 실행되기 전에 발생한 이벤트에서 발생했을 수도 있고,
// 레이아웃 효과에서 발생했을 수도 있습니다.
// 그런 경우에는 이전 스냅샷과 getSnapshot 값을 사용하여 빠져나갔을 것입니다.
// 한 번 더 확인해야 합니다.

  if (checkIfSnapshotChanged(inst)) {
   // 리렌더링을 다시 강제합니다.
    forceStoreRerender(fiber);
  }
}
function checkIfSnapshotChanged(inst) {
  const latestGetSnapshot = inst.getSnapshot;
  const prevValue = inst.value;
  try {
    const nextValue = latestGetSnapshot();
    return !is(prevValue, nextValue);
 // subscribeToStore()와 updateStoreInstance() 둘 다에서,
// 이 검사는 외부 데이터가 변경될 경우 리렌더링이 스케줄링되도록 보장합니다.

  } catch (error) {
    return true;
  }
}
function forceStoreRerender(fiber) {
  const root = enqueueConcurrentRenderForLane(fiber, SyncLane);
  if (root !== null) {
    scheduleUpdateOnFiber(root, fiber, SyncLane, NoTimestamp);

// SyncLane : 여기서 리렌더링을 위해 SyncLane을 강제합니다. React가 초기 마운트를 어떻게 수행하는지 설명했듯이, SyncLane은 블로킹 레인입니다. 따라서 이 새로운 렌더링은 동시 모드에서 실행되지 않습니다. 찢김(tear) 현상은 동시 모드에서만 발생하기 때문에, 이는 다음 렌더링에서 찢김 현상이 발생하지 않도록 방지합니다.

  }
}
```

코드는 다소 복잡하지만, 아이디어는 간단합니다.

중요하게 보아야할 점은 일관성 검사가 어떻게 작동하는지입니다.

계속해서 살펴보겠습니다.

### 2.2 리렌더링 안에서 updateSyncExternalStore()가 어떻게 동작하는가

```ts

function updateSyncExternalStore<T>(
  subscribe: (() => void) => () => void,
  getSnapshot: () => T,
  getServerSnapshot?: () => T,
): T {
  const fiber = currentlyRenderingFiber;
  const hook = updateWorkInProgressHook();
// 매 렌더링마다 저장소에서 현재 스냅샷을 읽어옵니다.
// 이는 React의 일반 규칙을 깨뜨리지만, 저장소 업데이트가 항상 동기적으로 이루어지기 때문에 작동합니다.
  const nextSnapshot = getSnapshot();

  const prevSnapshot = hook.memoizedState;
  const snapshotChanged = !is(prevSnapshot, nextSnapshot);
  if (snapshotChanged) {
    hook.memoizedState = nextSnapshot;
 //  따라서 새로운 상태가 발견되면, 데이터를 즉시 업데이트합니다.

    markWorkInProgressReceivedUpdate();
  }
  const inst = hook.queue;
  updateEffect(subscribeToStore.bind(null, fiber, inst, subscribe), [
    subscribe,
  ]);
// 이는 종속성(deps)에 따라 업데이트와 정리를 수행하기 위해 useEffect()를 따릅니다.

// getSnapshot 또는 subscribe가 변경될 때마다,
// 커밋 단계에서 중첩된 변경이 있었는지 확인해야 합니다.
// 동시 모드에서는 이러한 일이 자주 발생할 수 있지만,
// 심지어 동기 모드에서도 이전 효과가 저장소를 변경했을 수 있습니다.
  if (
    inst.getSnapshot !== getSnapshot ||
    snapshotChanged ||
// subscribe 함수가 변경되었는지 확인합니다.
// 위에서 구독 효과를 스케줄링했는지 확인함으로써 메모리를 절약할 수 있습니다.
    (workInProgressHook !== null &&
      workInProgressHook.memoizedState.tag & HookHasEffect)
  ) {
    fiber.flags |= PassiveEffect;
    pushEffect(
      HookHasEffect | HookPassive,
      updateStoreInstance.bind(null, fiber, inst, nextSnapshot, getSnapshot),
      undefined,
      null,
    );
//  getSnapshot()이 변경되면, 의존성 배열이 변경될 때 useEffect()를 업데이트하는 것처럼 효과를 업데이트해야 합니다.

// 블로킹 레인을 렌더링하지 않는 한, 일관성 검사를 스케줄링합니다.
// 커밋 직전에 트리를 탐색하여 저장소가 변경되었는지 확인할 것입니다.
    const root: FiberRoot | null = getWorkInProgressRoot();
    if (root === null) {
      throw new Error(
        'Expected a work-in-progress root. This is a bug in React. Please file an issue.',
      );
    }
    if (!includesBlockingLane(root, renderLanes)) {
      pushStoreConsistencyCheck(fiber, getSnapshot, nextSnapshot);
    }
// 여기서 일관성 검사가 다시 스케줄링됩니다.
// 솔직히 말해, 왜 여기에 배치되었는지 잘 이해가 되지 않습니다...

// 스냅샷 변화와 getSnapshot() 변화가 없더라도 외부 저장소가 우리에게 알리지 않고 변경될 가능성은 여전히 있습니다(예를 들어, 쓰로틀된 이벤트를 생각해보세요). 따라서 이 조건에서 검사를 여기에 두는 것은 버그처럼 보입니다.

// 2023-08-11 업데이트:

// 이것은 버그가 아니었습니다. 여기서는 주로 subscribe()와 getSnapshot()의 변경에 대해 일관성 검사를 스케줄링합니다. PR을 확인해보세요. (https://github.com/facebook/react/pull/27180)

  }
  return nextSnapshot;
}
```

코드는 일관성 검사를 스케줄링하는 마지막 부분을 제외하고는 합리적으로 보입니다.

특정 조건 하에서 예상대로 작동하지 않는다는 것을 보여주는 데모를 작성했습니다.

## - App.js

```ts

import {
  useEffect,
  useSyncExternalStore,
  startTransition,
  useState,
  useLayoutEffect,
  useMemo,
} from 'react';

let data = 1;
let callbacks = [];

function getSnapShot() {
  return data;
}

setTimeout(() => (data = 2), 800);

function subscribe(callback) {
  callbacks.push(callback);
  return () => {
    callbacks = callbacks.filter((item) => item != callback);
  };
}

function Cell() {
  let start = Date.now();
  while (Date.now() - start < 50) {
    // force yielding to main thread in concurrent mode
  }
  const data = useSyncExternalStore(subscribe, getSnapShot);

  return <div style={{ padding: '1rem', border: '1px solid red' }}>{data}</div>;
}

export default function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    startTransition(() => {
      setCount((count) => count + 1);
    });
  }, []);

  return (
    <div>
      <p>
        Example of tearing. <br />
        below are multiple cells rendering same external data which changes
        during rendering.
      </p>
      <div style={{ display: 'flex', gap: '1rem' }}>
        <Cell />
        <Cell />
        <Cell />
        <Cell />
        <Cell />
        <Cell />
      </div>
    </div>
  );
}
```

여기서 useSyncExternalStore()를 사용했음에도 불구하고 찢김(tear) 현상이 다시 발생하는 것을 볼 수 있습니다.

이는 버그일 가능성이 있으며, 이를 수정하기 위해 [풀 리퀘스트](https://github.com/facebook/react/pull/27180)를 작성했습니다.

> 만약 찢김(tear) 현상이 보이지 않는다면, 코드에서 타임아웃을 더 작은 값으로 변경해보세요.

> 2023-08-11 업데이트: 일관성 검사가 하는 역할을 오해한 것으로 드러났습니다. useSyncExternalStore()는 모든 외부 저장소 데이터 변경이 반드시 발생하도록 요구합니다. 그래서 subscribe() 내부의 forceStoreRerender()가 외부 데이터 변경으로 인한 찢김 현상을 실제로 방지하는 것입니다. 여기서 일관성 검사는 subscribe() 또는 getSnapshot()의 변경으로 인한 찢김을 방지합니다.

### 2.3 동시성 모드에서 일관성 확인

어떻게 일관성 체크가 이루어지는지 한번 찾아봅시다.

```ts
function pushStoreConsistencyCheck<T>(
  fiber: Fiber,
  getSnapshot: () => T,
  renderedSnapshot: T,
) {
  fiber.flags |= StoreConsistency;
  const check: StoreConsistencyCheck<T> = {
    getSnapshot,
    value: renderedSnapshot,
  };
  let componentUpdateQueue: null | FunctionComponentUpdateQueue = (currentlyRenderingFiber.updateQueue: any);
  if (componentUpdateQueue === null) {
    componentUpdateQueue = createFunctionComponentUpdateQueue();
    currentlyRenderingFiber.updateQueue = (componentUpdateQueue: any);

// currentlyRenderingFiber.updateQueue :  useEffect를 상기해보면, updateQueue는 효과들을 저장하기 위해 lastEffect도 보유하고 있습니다. 그리고 renderWithHooks()에서 이것이 지워지므로, 컴포넌트가 렌더링될 때마다 새롭게 시작됩니다.

    componentUpdateQueue.stores = [check];
// componentUpdateQueue.stores :  외부 저장소 검사를 위한 별도의 필드입니다.

  } else {
    const stores = componentUpdateQueue.stores;
    if (stores === null) {
      componentUpdateQueue.stores = [check];
    } else {
      stores.push(check);
    }
  }
}
```

이제 일관성 검사가 언제 그리고 어떻게 트리거되는지 볼 차례입니다.

이는 실제로 커밋 직전에, 렌더 단계의 끝에서 실행됩니다.

```ts
// 이 함수는 모든 동시 작업의 진입점입니다. 즉, Scheduler를 통해
// 진행되는 모든 작업은 여기로 들어옵니다.
export function performConcurrentWorkOnRoot(
  root: FiberRoot,
  didTimeout: boolean,
): RenderTaskFn | null {
  ...
  // 루트에 저장된 필드를 사용하여 다음 작업할 레인을 결정합니다.
  // TODO: 호출자에서 이미 계산한 값입니다. 인수로 전달하도록 수정해야 합니다.
  let lanes = getNextLanes(
    root,
    root === workInProgressRoot ? workInProgressRootRenderLanes : NoLanes,
  );
  if (lanes === NoLanes) {
    // 방어적 코딩. 이 경우는 발생하지 않아야 합니다.
    return null;
  }
  // 특정 경우에는 시간 분할을 비활성화합니다: 작업이 너무 오래 CPU에 묶여 있었을 경우("만료된" 작업으로, 기아를 방지하기 위함) 또는 동기 업데이트 기본 모드일 경우.
  // TODO: Scheduler 버그를 조사 중이라, 방어적으로 `didTimeout`을 체크합니다.
  // Scheduler의 버그가 수정되면 이를 제거할 수 있습니다. 우리는 만료를 자체적으로 추적합니다.
  const shouldTimeSlice =
    !includesBlockingLane(root, lanes) &&
    !includesExpiredLane(root, lanes) &&
    (disableSchedulerTimeoutInWorkLoop || !didTimeout);
  let exitStatus = shouldTimeSlice
    ? renderRootConcurrent(root, lanes)
    : renderRootSync(root, lanes);

  if (exitStatus !== RootInProgress) {
    if (exitStatus === RootErrored) {
      // 오류가 발생했을 경우, 한 번 더 렌더링을 시도합니다.
      // 동시 데이터 변조를 막기 위해 동기적으로 렌더링하고, 모든 보류 중인 업데이트가 포함되도록 합니다.
      // 두 번째 시도 후에도 실패하면 결과 트리를 커밋합니다.
      const originallyAttemptedLanes = lanes;
      const errorRetryLanes = getLanesToRetrySynchronouslyOnError(
        root,
        originallyAttemptedLanes,
      );
      if (errorRetryLanes !== NoLanes) {
        lanes = errorRetryLanes;
        exitStatus = recoverFromConcurrentError(
          root,
          originallyAttemptedLanes,
          errorRetryLanes,
        );
      }
    }
    if (exitStatus === RootFatalErrored) {
      const fatalError = workInProgressRootFatalError;
      prepareFreshStack(root, NoLanes);
      markRootSuspended(root, lanes);
      ensureRootIsScheduled(root);
      throw fatalError;
    }
    if (exitStatus === RootDidNotComplete) {
      // 트리를 완성하지 못하고 렌더링이 종료되었습니다. 이는 일관된 트리를 생성하거나 커밋하지 않고
      // 현재 렌더링을 종료해야 하는 특수한 경우에 발생합니다.
      markRootSuspended(root, lanes);
    } else {
      // 렌더링이 완료되었습니다.
      // 이 렌더링이 동시 이벤트에 양보했을 가능성이 있는지 확인하고, 그런 경우
      // 새로 렌더링된 스토어가 일관된지 확인합니다.
      // TODO: 렌더링이 충분히 빠르거나 만료된 경우 주 스레드에 양보하지 않았을 수도 있습니다. 그런 경우 일관성 검사를 생략할 수 있습니다.
      const renderWasConcurrent = !includesBlockingLane(root, lanes);
      const finishedWork: Fiber = (root.current.alternate: any);
      if (
        renderWasConcurrent &&
        !isRenderConsistentWithExternalStores(finishedWork)
        // 여기서 일관성 체크가 일어납니다.
      ) {
        // 스토어가 교차 이벤트에서 변조되었습니다. 동시 변조를 막기 위해 동기적으로 다시 렌더링합니다.
        exitStatus = renderRootSync(root, lanes);
        // exitStatus = renderRootSync(root, lanes) : 동기 모드에서 다시 렌더링이 시작됩니다.
        // 커밋이 발생하기 전에, 사용자들은 UI에서 찢어짐(화면 깜빡임 등)을 보지 않게 됩니다.

        // 오류가 발생했는지 다시 확인해야 합니다.
        if (exitStatus === RootErrored) {
          const originallyAttemptedLanes = lanes;
          const errorRetryLanes = getLanesToRetrySynchronouslyOnError(
            root,
            originallyAttemptedLanes,
          );
          if (errorRetryLanes !== NoLanes) {
            lanes = errorRetryLanes;
            exitStatus = recoverFromConcurrentError(
              root,
              originallyAttemptedLanes,
              errorRetryLanes,
            );
            // 동시 이벤트에 양보하지 않았으므로 이제 트리가 일관되다고 가정합니다.
          }
        }
        if (exitStatus === RootFatalErrored) {
          const fatalError = workInProgressRootFatalError;
          prepareFreshStack(root, NoLanes);
          markRootSuspended(root, lanes);
          ensureRootIsScheduled(root);
          throw fatalError;
        }
        // FIXME: RootDidNotComplete를 다시 확인해야 합니다. 현재 구조가 이상적이지 않습니다.
      }
      // 이제 트리가 일관된 상태입니다. 다음 단계는 이를 커밋하거나,
      // 무언가가 대기 중이라면 타임아웃 후 커밋을 기다리는 것입니다.
      root.finishedWork = finishedWork;
      root.finishedLanes = lanes;
      finishConcurrentRender(root, exitStatus,
      finishedWork, lanes);
      // commitRoot()는 여기서 실행됩니다.
    }
  }
  ensureRootIsScheduled(root);
  return getContinuationForRoot(root, originalCallbackNode);
}
```

```ts
function isRenderConsistentWithExternalStores(finishedWork: Fiber): boolean {
  // 렌더된 트리에서 외부 스토어 읽기를 검색하고, 동시 이벤트에서 스토어가 변조되었는지 확인합니다.
  // 재귀 대신 반복 루프를 사용하여 일찍 종료할 수 있도록 합니다.
  let node: Fiber = finishedWork;
  while (true) {
    if (node.flags & StoreConsistency) {
      const updateQueue: FunctionComponentUpdateQueue | null =
        (node.updateQueue: any);
      if (updateQueue !== null) {
        const checks = updateQueue.stores;
        if (checks !== null) {
          for (let i = 0; i < checks.length; i++) {
            const check = checks[i];
            const getSnapshot = check.getSnapshot;
            const renderedValue = check.value;
            try {
              if (!is(getSnapshot(), renderedValue)) {
                // 불일치하는 스토어를 발견했습니다.
                return false;
              }
            } catch (error) {
              // `getSnapshot`이 오류를 발생시키면 `false`를 반환합니다.
              // 이로 인해 다시 렌더링이 예약되고, 렌더링 중에 오류가 다시 발생합니다.
              return false;
            }
          }
        }
      }
    }
    const child = node.child;
    if (node.subtreeFlags & StoreConsistency && child !== null) {
      child.return = node;
      node = child;
      continue;
    }
    if (node === finishedWork) {
      return true;
    }
    while (node.sibling === null) {
      if (node.return === null || node.return === finishedWork) {
        return true;
      }
      node = node.return;
    }
    node.sibling.return = node.return;
    node = node.sibling;
  }
  // Flow는 이것이 도달할 수 없는 코드라는 것을 알지 못하지만, eslint는 알 수 있습니다.
  // eslint-disable-next-line no-unreachable
  return true;
}

```

### 3. 요약

이제 `useSyncExternalStore()`가 내부적으로 어떻게 작동하는지 이해했으므로, 주로 두 가지 문제를 해결하는 것을 알 수 있습니다.

1. **동시 모드에서의 찢어짐(Tearing):** `useSyncExternalStore()`는 렌더링이 완료된 후 커밋이 시작되기 전에 일관성 검사를 예약하여 이 문제를 해결합니다. 동기 모드에서 다시 렌더링이 강제되어 불일치한 데이터가 UI에 표시되지 않게 합니다.

2. **감지되지 않은 외부 스토어 변경:** `useSyncExternalStore()`는 수동 효과를 통해 변경 사항이 있는지 확인하고, 변경 사항이 있으면 동기 모드에서 다시 렌더링을 예약합니다. 이는 커밋 후에 발생하므로 사용자는 UI 깜빡임을 볼 수 있습니다.
