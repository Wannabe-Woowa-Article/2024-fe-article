## 🔗 [When do useEffect() callbacks get run? Before paint or after paint?](https://jser.dev/2023-08-09-effects-run-paint/)

### 🗓️ 번역 날짜: 2024.07.11

### 🧚 번역한 크루: 버건디(전태헌)

---

요약: 대부분의 경우 useEffect() 콜백은 페인트 후에 실행되지만 항상 그런 것은 아닙니다.

[리액트 공식문서](https://react.dev/reference/react/useEffect)에서 아래와 같이 설명하고 있습니다. (2023/08/09)

> 사용자의 상호작용(예: 클릭)으로 인해 발생한 Effect가 아니라면, React는 Effect를 실행하기 전에 브라우저가 업데이트된 화면을 먼저 페인트하게 합니다. 만약 Effect가 툴팁 위치 지정과 같은 시각적 작업을 수행하고, 지연이 눈에 띄게 보인다면(예: 깜빡거림), useEffect를 useLayoutEffect로 대체하십시오.

안타깝게도 공식 내용은 정확하지 않습니다. 상호작용으로 인해 발생하지 않은 Effect라도 React는 Effect를 실행한 후에 페인트할 수 있습니다.

저는 ["React에서 useEffect()는 내부적으로 어떻게 작동하나요?"](https://jser.dev/2023-07-08-how-does-useeffect-work)에서 이를 설명했지만, 브라우저 페인트와 비교한 타이밍의 세부 사항은 다루지 않았습니다. 오늘 몇 가지 데모를 통해 이를 알아보겠습니다.

## 1. 데모 1 - useEffect의 콜백은 페인트 전에 실행 된다.

아래의 코드 스니펫의 결과는 어떻게 될까요?

```tsx
import React, { useState, useEffect } from "react";
export default function App() {
  const [state] = useState(0);
  console.log(1);
  useEffect(() => {
    console.log(2);
  }, [state]);
  Promise.resolve().then(() => console.log(3));
  setTimeout(() => console.log(4), 0);
  return <div>open console to see the logs</div>;
}
const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

결과가 꽤 놀랍습니다!

만약 useEffect()가 비동기라면, 그 콜백은 Promise의 then 콜백 이후에 실행되어야 합니다. 따라서 결과는 1 3 4 2가 되어야 합니다.

하지만 실제로는 1 2 3 4로 로그가 찍히고 있어, 이는 useEffect의 콜백이 페인트 후가 아니라 그 전에 실행된다는 것을 의미합니다.

> 2024-03-19 업데이트: [다른 결과가 보인다는 피드백](https://github.com/JSerZANP/jser.dev-comment/issues/5#issuecomment-1870810085)이 있었습니다. 1 2 3 4의 결과가 나오는 이유는 [섹션 3](https://jser.dev/2023-08-09-effects-run-paint/#3-react-scheduler-tries-to-process-multiple-taskssync-before-yielding)에서 설명되어 있으므로, 다른 결과는 브라우저와 기기 사양의 차이로 인해 발생할 수 있습니다.

## 데모 2 - useEffect의 콜백은 페인트 후에 실행 된다.

위의 예와는 조금 다른 코드 스니펫을 실행 시켜보겠습니다.

이 스니펫은 while 루프가 추가 되었어요.

```tsx
import React, { useState, useEffect } from "react";
export default function App() {
  const [state] = useState(0);
  console.log(1);
  let start = Date.now();
  while (Date.now() - start < 50) {}
  useEffect(() => {
    console.log(2);
  }, [state]);
  Promise.resolve().then(() => console.log(3));
  setTimeout(() => console.log(4), 0);
  return <div>open console to see the logs</div>;
}
const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

현재의 결과는 `1 3 2 4`가 되었습니다.

왜 이 2개의 데모는 다른 결과를 불러왔을까요? 한번 시도해보죠.

## 3. React 스케줄러는 양보하기 전에 여러 작업(동기)을 처리하려고 합니다.

[리액트 스케줄러가 내부적으로 어떻게 작동](https://jser.dev/react/2022/03/16/how-react-scheduler-works/)하는지 간단히 언급했지만, 더 자세히 살펴보겠습니다.
렌더링이 완료된 후 커밋 단계에서는 `useEffect()` 콜백이 실행되도록 스케줄됩니다.

```tsx
function commitRootImpl(
  root: FiberRoot,
  recoverableErrors: null | Array<CapturedValue<mixed>>,
  transitions: Array<Transition> | null,
  renderPriorityLevel: EventPriority,
) {
  ...
  if (
    (finishedWork.subtreeFlags & PassiveMask) !== NoFlags ||
    (finishedWork.flags & PassiveMask) !== NoFlags
  ) {
    if (!rootDoesHavePassiveEffects) {
      rootDoesHavePassiveEffects = true;
      pendingPassiveEffectsRemainingLanes = remainingLanes;
      // workInProgressTransitions가 덮어써질 수 있으므로,
      // 처리될 때까지 pendingPassiveTransitions에 저장하고자 합니다.
      // 만약 setTimeout으로 커밋을 지연시키면
      // 이전 렌더와 커밋 사이에 workInProgressTransitions가 변경될 수 있으므로,
      // 이를 인수로 commitRoot에 전달해야 합니다.
      pendingPassiveTransitions = transitions;
      scheduleCallback(NormalSchedulerPriority, () => {
        // 대부분의 경우 비동기입니다
        flushPassiveEffects();
        // 이 렌더링은 수동 효과를 트리거했습니다: 수동 효과가 실행된 후에
        // 루트 캐시 풀을 해제하여 트리의 노드(HostRoot, Cache 경계 등)가
        // 참조할 수 있는 캐시 풀을 해제하는 것을 방지합니다.
        return null;
      });
    }
  }
  ...
}
```

이것이 `flushPassiveEffects()`가 트리거되는 주요 위치라는 점에 주목하십시오. 다른 위치에서도 트리거될 수 있습니다.

```ts
function unstable_scheduleCallback(
  priorityLevel: PriorityLevel,
  callback: Callback,
  options?: { delay: number }
): Task {
  ...
  var expirationTime = startTime + timeout;
  var newTask: Task = {
    id: taskIdCounter++,
    callback,
    priorityLevel,
    startTime,
    expirationTime,
    sortIndex: -1,
  };
  // 새로운 작업이 생성됨
  if (startTime > currentTime) {
    ...
  } else {
    newTask.sortIndex = expirationTime;
    push(taskQueue, newTask);
    // 작업이 푸시되었지만 스케줄링되지 않음

    // 호스트 콜백을 스케줄링할 필요가 있다면 스케줄링합니다.
    // 이미 작업을 수행 중이라면, 다음 번 양보할 때까지 기다립니다.
    if (!isHostCallbackScheduled && !isPerformingWork) {
      isHostCallbackScheduled = true;
      requestHostCallback();
      // requestHostCallback()가 스케줄링하여 작업 처리를 시작합니다.
    }
    // 따라서 스케줄러가 이미 작업을 처리 중이라면 여기서는 스케줄링되지 않습니다.
  }
  return newTask;
}

```

React 스케줄러는 여러 작업을 한 번에 처리하려고 합니다. `commitRoot()`는 이전 렌더링 작업 내부에서 호출됩니다. 이는 `useEffect()`를 위한 `scheduleCallback()`이 호출될 때 `isPerformingWork`가 true임을 의미하며, 따라서 스케줄러는 작업을 큐에 푸시하고 스케줄링하지 않고 함께 처리되기를 희망합니다.

하지만 물론 스케줄러는 각 작업의 시간 비용을 실제로 처리하기 전까지는 알 수 없으므로, 각 작업을 실행하기 전에 시간이 남아 있는지 확인하여 다시 양보가 필요한지 확인하고 나머지 작업의 처리를 스케줄링합니다.

```ts
function workLoop(hasTimeRemaining, initialTime) {
  let currentTime = initialTime;
  advanceTimers(currentTime);
  currentTask = peek(taskQueue);
  while (
    currentTask !== null &&
    !(enableSchedulerDebugging && isSchedulerPaused)
  ) {
    if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost())
    ) {
      // 시간을 너무 많이 사용했으니, 브라우저에 페인트할 시간을 줍시다!
      break;
      // 여기서 break 이후, workLoop의 또 다른 라운드가 스케줄됩니다.
    }
    const callback = currentTask.callback;
    ...
  }
}
```

> 만약 리액트 스케줄러에 대해서 더 알고 싶다면, [리액트 스케줄러의 내부 구조](https://jser.dev/react/2022/03/16/how-react-scheduler-works/)글을 추천합니다.

이제 첫 번째와 두 번째 데모를 이해할 수 있게 되었습니다.

1. 두 데모 모두에서 `useEffect()`로 인해 `scheduleCallback()`에서 새로운 작업이 생성됩니다. 그러나 호출될 때 우리는 여전히 이전 렌더링 작업 내부에 있으며, 새로운 작업은 단지 푸시될 뿐 비동기적으로 스케줄링되지 않습니다.

2. 스케줄러는 작업을 동기적으로 처리하려고 합니다. 데모 1의 경우, 전체 렌더링이 너무 많은 시간을 소모하지 않으므로 동기적으로 실행됩니다. 하지만 데모 2의 경우, 렌더링이 너무 많은 시간을 소모하므로 스케줄러는 메인 스레드에 양보하고 비동기로 스케줄링합니다.

이제 사용자 상호작용으로 인해 발생하지 않은 효과라도 페인트 전 또는 페인트 후에 실행되는지 확신할 수 없다는 것을 알 수 있습니다.

좋습니다, 이제 다른 경우들도 살펴보겠습니다.

# 4. 데모 - 사용자 상호작용으로 인해 발생하는 useEffect()는 페인트 전에 실행된다.

```tsx
import React, { useState, useEffect } from "react";
export default function App() {
  const [state, setState] = useState(0);
  console.log(1);
  let start = Date.now();
  while (Date.now() - start < 50) {}
  useEffect(() => {
    console.log(2);
  }, [state]);
  Promise.resolve().then(() => console.log(3));
  setTimeout(() => console.log(4), 0);
  return (
    <div>
      click{" "}
      <button onClick={() => setState((state) => state + 1)}>rerender</button>{" "}
      {/*           ------------------------*/}
      and open console to see the logs{" "}
    </div>
  );
}
```

결과는 1 3 4 2 1 2 3 4입니다. 따라서 클릭 이벤트로 인해 useEffect() 콜백이 페인트 전에 동기적으로 실행된다는 것을 알 수 있습니다.

문제는 데모 2에 따르면 React 스케줄러가 비동기적으로 실행을 스케줄링해야 한다는 것입니다. 이벤트 핸들러에서 무슨 일이 일어나는 걸까요?

답은, **동일하게 작동하지만 SyncLane에서 수동 효과를 동기적으로 실행하는 재렌더링에 추가 단계가 하나 더 있다는 것입니다. 그리고 사용자 상호작용에 의해 발생하는 재렌더링은 SyncLane입니다.**

참고로, 초기 마운트 렌더링 레인은 SyncLane이 아니라 DefaultLane입니다. Lanes에 대한 자세한 내용은 [리액트에서 Lanes는 무엇인가?](https://jser.dev/react/2022/03/26/lanes-in-react/)를 참고하세요.

```tsx
function commitRootImpl(
  root: FiberRoot,
  recoverableErrors: null | Array<CapturedValue<mixed>>,
  transitions: Array<Transition> | null,
  renderPriorityLevel: EventPriority,
) {
  ...
  // 처리 대기 중인 수동 효과가 있는 경우, 이를 처리하기 위한 콜백을 스케줄합니다.
  // 가능한 한 빨리 이 작업을 수행하여 커밋 단계에서 스케줄될 수 있는 다른 모든 것보다 먼저 큐에 넣습니다. (#16714 참조)
  // TODO: 수동 효과 콜백을 스케줄하는 다른 모든 위치 삭제
  // 이들은 중복됩니다.
  if (
    (finishedWork.subtreeFlags & PassiveMask) !== NoFlags ||
    (finishedWork.flags & PassiveMask) !== NoFlags
  ) {
    if (!rootDoesHavePassiveEffects) {
      rootDoesHavePassiveEffects = true;
      pendingPassiveEffectsRemainingLanes = remainingLanes;
      // workInProgressTransitions가 덮어써질 수 있으므로,
      // 처리될 때까지 pendingPassiveTransitions에 저장하고자 합니다.
      // 만약 setTimeout으로 커밋을 지연시키면
      // 이전 렌더와 커밋 사이에 workInProgressTransitions가 변경될 수 있으므로,
      // 이를 인수로 commitRoot에 전달해야 합니다.
      pendingPassiveTransitions = transitions;
      scheduleCallback(NormalSchedulerPriority, () => {
        flushPassiveEffects();
        // 이 렌더링은 수동 효과를 트리거했습니다: 수동 효과가 실행된 후에
        // 루트 캐시 풀을 해제하여 트리의 노드(HostRoot, Cache 경계 등)가
        // 참조할 수 있는 캐시 풀을 해제하는 것을 방지합니다.
        return null;
      });
      // 이 부분은 이미 다루어졌고, 사용자 상호작용에서도 실행됩니다!
    }
  }
  ...
  // `commitRoot`를 종료하기 전에 항상 이 작업을 호출하여
  // 이 루트에 대한 추가 작업이 스케줄되도록 합니다.
  ensureRootIsScheduled(root, now());
  ...
  // 수동 효과가 개별 렌더링의 결과인 경우, 현재 작업이 끝날 때
  // 동기적으로 이를 플러시하여 결과가 즉시 관찰될 수 있도록 합니다.
  // 그렇지 않으면 순서에 의존하지 않으며 외부 시스템에서 관찰할 필요가 없다고 가정하여,
  // 페인트 후까지 기다릴 수 있습니다.
  // TODO: 콜백을 이전에 스케줄링하지 않도록 최적화할 수 있습니다.
  // 현재 여러 위치에서 콜백을 스케줄링하고 있으므로,
  // 이들이 통합될 때까지 기다리겠습니다.
  if (
    includesSomeLane(pendingPassiveEffectsLanes, SyncLane) &&
    root.tag !== LegacyRoot
  ) {
    flushPassiveEffects();
    // flushPassiveEffects()가 다시 동기적으로 호출됩니다.
  }
  // 스케줄된 flushPassiveEffects()가 실행될 때, 효과는 비어 있습니다.
  // 레이아웃 작업이 스케줄된 경우, 지금 이를 플러시합니다.
  flushSyncCallbacks();
  return null;
}
```

이는 사용자 상호작용 후 최신 UI를 사용자에게 보여주는 것이 더 중요하다고 React가 생각하기 때문에 의도적으로 이렇게 한 것입니다.

## 데모 4 - useEffect()가 useLayoutEffect()에 의해 끌어올려져 페인트 전에 실행될 수 있다.

그렇다면 레이아웃 효과에서 재렌더링이 발생하면 어떻게 될까요?

```tsx
import React, { useState, useEffect, useLayoutEffect } from "react";

export default function App() {
  const [state, setState] = useState(0);
  console.log(1);
  let start = Date.now();
  while (Date.now() - start < 50) {}

  useEffect(() => {
    console.log(2);
  }, [state]);

  useLayoutEffect(() => {
    setState((state) => state + 1);
  }, []);

  Promise.resolve().then(() => console.log(3));

  setTimeout(() => console.log(4), 0);

  return <div>open console to see the logs </div>;
}
```

답은 `1 2 1 2 3 3 4 4`입니다. 이건 `useState`에 대한 설명이 필요합니다.

> useState에 대한 자세한 설명은 [useState의 내부에서 어떻게 동작하는가?](https://jser.dev/2023-06-19-how-does-usestate-work/)에 관한 글에서 확인할 수 있습니다.

우선, 커밋 단계에서 트리거된 업데이트는 DiscreteEventPriority로 처리되며, 이는 사용자 상호작용과 동일하게 SyncLane으로 처리된다는 것을 의미합니다.

```ts
function commitRoot(
  root: FiberRoot,
  recoverableErrors: null | Array<CapturedValue<mixed>>,
  transitions: Array<Transition> | null
) {
  // TODO: 더 이상 의미가 없습니다. 우리는 이미 변이와 레이아웃 단계를 감쌉니다.
  // 제거할 수 있어야 합니다.
  const previousUpdateLanePriority = getCurrentUpdatePriority();
  const prevTransition = ReactCurrentBatchConfig.transition;
  try {
    ReactCurrentBatchConfig.transition = null;
    setCurrentUpdatePriority(DiscreteEventPriority);
    // 커밋 단계에서 업데이트를 위한 SyncLane!

    commitRootImpl(
      root,
      recoverableErrors,
      transitions,
      previousUpdateLanePriority
    );
  } finally {
    ReactCurrentBatchConfig.transition = prevTransition;
    setCurrentUpdatePriority(previousUpdateLanePriority);
  }
  return null;
}
```

`setState()`가 호출되면, 작업이 필요한 Fiber 노드를 표시한 다음 `ensureRootIsScheduled()`를 통해 재렌더링을 스케줄링합니다.

```ts
function ensureRootIsScheduled(root: FiberRoot, currentTime: number) {
  ...
  // 작업할 다음 레인과 그 우선순위를 결정합니다.
  const nextLanes = getNextLanes(
    root,
    root === workInProgressRoot ? workInProgressRootRenderLanes : NoLanes,
  );
  ...
  // 콜백의 우선순위를 나타내기 위해 가장 높은 우선순위 레인을 사용합니다.
  const newCallbackPriority = getHighestPriorityLane(nextLanes);
  ...
  // 새로운 콜백을 스케줄링합니다.
  let newCallbackNode;
  if (newCallbackPriority === SyncLane) {
    // SyncLane!

    // 특수한 경우: Sync React 콜백은 특별한 내부 큐에 스케줄링됩니다.
    if (root.tag === LegacyRoot) {
      scheduleLegacySyncCallback(performSyncWorkOnRoot.bind(null, root));
    } else {
      scheduleSyncCallback(performSyncWorkOnRoot.bind(null, root));
    }
    ...
  }
  root.callbackPriority = newCallbackPriority;
  root.callbackNode = newCallbackNode;
}

```

`performSyncWorkOnRoot()`에서는 `useEffect()` 콜백을 실행합니다!

```ts
export function performSyncWorkOnRoot(root: FiberRoot): null {
  if (enableProfilerTimer && enableProfilerNestedUpdatePhase) {
    syncNestedUpdateFlag();
  }
  if ((executionContext & (RenderContext | CommitContext)) !== NoContext) {
    throw new Error("Should not already be working.");
  }
  flushPassiveEffects();
}
```

`scheduleSyncCallback()`은 콜백을 `syncQueue`에 푸시합니다.

```ts
export function scheduleSyncCallback(callback: SchedulerCallback) {
  // 이 콜백을 내부 큐에 푸시합니다. 다음 틱에서 이를 플러시하거나,
  // `flushSyncCallbackQueue`가 호출되면 더 빨리 플러시합니다.
  if (syncQueue === null) {
    syncQueue = [callback];
  } else {
    // 기존 큐에 푸시합니다. 큐를 생성할 때 이미 콜백을 스케줄링했기 때문에
    // 콜백을 스케줄링할 필요가 없습니다.
    syncQueue.push(callback);
  }
}
```

`commitRoot()` 내부에서, `syncQueue`는 동기적으로 플러시됩니다.

```ts
function commitRootImpl(
  root: FiberRoot,
  recoverableErrors: null | Array<CapturedValue<mixed>>,
  transitions: Array<Transition> | null,
  renderPriorityLevel: EventPriority,
) {
  ...
  // 수동 효과가 개별 렌더링의 결과인 경우, 현재 작업이 끝날 때
  // 동기적으로 이를 플러시하여 결과가 즉시 관찰될 수 있도록 합니다.
  // 그렇지 않으면 순서에 의존하지 않으며 외부 시스템에서 관찰할 필요가 없다고 가정하여,
  // 페인트 후까지 기다릴 수 있습니다.
  // TODO: 콜백을 이전에 스케줄링하지 않도록 최적화할 수 있습니다.
  // 현재 여러 위치에서 콜백을 스케줄링하고 있으므로,
  // 이들이 통합될 때까지 기다리겠습니다.
  if (
    includesSomeLane(pendingPassiveEffectsLanes, SyncLane) &&
    root.tag !== LegacyRoot
  ) {
    flushPassiveEffects();
  }
  ...
  // 레이아웃 작업이 스케줄된 경우, 지금 이를 플러시합니다.
  flushSyncCallbacks();
  // 따라서 레이아웃 효과로 인한 재렌더링은 여기서부터 동기적으로 시작됩니다!
  // 또한 SyncLane에서도 수행됩니다!
  // Concurrent 모드가 사용되지 않으므로, 나중에 commitRoot()가 다시 동기적으로 호출됩니다.
  return null;
}

```

이제 데모 4의 로그를 이해할 수 있게 되었습니다.

1. 커밋 시작 → 1
2. `flushPassiveEffects()` 스케줄링 → 비동기
3. 레이아웃 효과 실행, 재렌더링 트리거, `performSyncWorkOnRoot()`를 동기적으로 스케줄링
4. `performSyncWorkOnRoot()`에서 `flushPassiveEffects()` 호출 → 2
5. 재렌더링 완료, 다시 커밋 시작 → 1
6. `flushPassiveEffects()` 스케줄링 → 비동기
7. SyncLane에서 `flushPassiveEffects()` 호출됨 (데모 3에서 설명됨) → 2
8. 지금까지 모두 동기적, 따라서 콜백이 실행되고 그 후 setTimeout 실행 → `3 3 4 4`
9. 스케줄된 `flushPassiveEffects()` 실행 → 이미 처리됨.

## 6. 요약

`useEffect()` 콜백이 페인트 전에 실행되는 몇 가지 경우를 다루었으며, 실제로 더 많은 경우가 있습니다. 하지만 React.dev의 설명이 정확하지 않다는 것을 이미 증명했습니다 (의심을 가지세요!).

`useEffect()` 콜백 실행 시점을 다음과 같이 설명할 수 있습니다.

**`useEffect()` 콜백은 DOM 변형이 완료된 후에 실행됩니다. 대부분의 경우, 페인트 후에 비동기적으로 실행되지만, 최신 UI를 보여주는 것이 더 중요한 경우(예: 사용자 상호작용에 의해 재렌더링이 발생하거나 레이아웃 효과 아래에서 스케줄된 경우) 또는 React가 내부적으로 그럴 시간을 가지고 있을 때는 페인트 전에 동기적으로 실행될 수 있습니다.**

이 내용을 수정하기 위해 React.dev에 풀 리퀘스트를 만들었으며, 병합되기를 바랍니다.
