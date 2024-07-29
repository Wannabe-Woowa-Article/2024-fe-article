## 🔗 [How does useEffect() work internally in React?](https://jser.dev/2023-07-08-how-does-useeffect-work/)

### 🗓️ 번역 날짜: 2024.07.17

### 🧚 번역한 크루: 버건디(전태헌)

---

> 리액트 18.2 버전을 기반으로 하고 있습니다. 새로운 버전에서는 실행이 달라질 수 있습니다.

useEffect()는 useState() 다음으로 React에서 가장 많이 사용되는 훅일 것입니다.

매우 강력하지만 때때로 혼란스러울 수 있습니다. 내부적으로 어떻게 작동하는지 알아봅시다.

```tsx
useEffect(() => {
  // ...
}, [deps]);
```

## 1. 초기 마운트 시의 `useEffect()`

`useEffect()`는 초기 마운트 시에 `mountEffect()`를 사용합니다.

```tsx
function mountEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null
): void {
  return mountEffectImpl(
    PassiveEffect | PassiveStaticEffect, // 이 플래그는 Layout Effects와의 차이를 구분하기 위해 중요합니다.
    HookPassive,
    create,
    deps
  );
}

function mountEffectImpl(fiberFlags, hookFlags, create, deps): void {
  const hook = mountWorkInProgressHook(); // 새로운 hook을 만듭니다.

  const nextDeps = deps === undefined ? null : deps;
  currentlyRenderingFiber.flags |= fiberFlags;
  hook.memoizedState = pushEffect(
    // pushEffect()는 Effect 객체를 생성한 후, 해당 객체를 hook에 설정합니다.

    HookHasEffect | hookFlags, // 이 플래그는 초기 마운트 시 이 효과를 실행해야 함을 의미합니다.
    create,
    undefined,
    nextDeps
  );
}
```

```tsx
function pushEffect(tag, create, destroy, deps) {
  const effect: Effect = {
    tag, // tag는 이 효과가 실행될 필요가 있는지 여부를 표시하는 데 사용되므로 중요합니다.

    create, // 우리가 전달하는 콜백 함수입니다.

    destroy, // 콜백 함수에서 반환되는 정리 함수입니다.

    deps, // 우리가 전달하는 deps 배열입니다.

    // Circular
    next: (null: any), // 하나의 컴포넌트에 여러 효과가 있을 수 있으므로, 이를 연결합니다.
  };

  let componentUpdateQueue: null | FunctionComponentUpdateQueue = (currentlyRenderingFiber.updateQueue: any);
  if (componentUpdateQueue === null) {
    componentUpdateQueue = createFunctionComponentUpdateQueue();
    currentlyRenderingFiber.updateQueue = (componentUpdateQueue: any);
    // 효과는 fiber의 updateQueue에 저장됩니다.

    // 이는 hook의 memoizedState와 다르다는 점에 유의하세요.

    componentUpdateQueue.lastEffect = effect.next = effect;
  } else {
    const lastEffect = componentUpdateQueue.lastEffect;
    if (lastEffect === null) {
      componentUpdateQueue.lastEffect = effect.next = effect;
    } else {
      const firstEffect = lastEffect.next;
      lastEffect.next = effect;
      effect.next = firstEffect;
      componentUpdateQueue.lastEffect = effect;
    }
  }
  return effect;
}
```

우리는 초기 마운트 시 useEffect()가 필요한 플래그와 함께 Effect 객체를 생성하는 것을 볼 수 있습니다. 이들은 다른 시점에 처리될 것입니다.

## 2. 리렌더링시에 `useEffect`

```ts
function updateEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null
): void {
  return updateEffectImpl(PassiveEffect, HookPassive, create, deps);
}
function updateEffectImpl(fiberFlags, hookFlags, create, deps): void {
  const hook = updateWorkInProgressHook();
  // 현재 hook 가져오기

  const nextDeps = deps === undefined ? null : deps;
  let destroy = undefined;
  if (currentHook !== null) {
    const prevEffect = currentHook.memoizedState;
    // Effect hook의 memoizedState는 Effect 객체입니다.

    destroy = prevEffect.destroy;
    if (nextDeps !== null) {
      const prevDeps = prevEffect.deps;
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        hook.memoizedState = pushEffect(hookFlags, create, destroy, nextDeps);
        return;
      }
      // deps가 변경되지 않으면, Effect 객체를 재생성하는 것 외에는 아무 것도 하지 않습니다.

      // 여기서 Effect 객체를 재생성해야 하는 이유는 updateQueue를 단순히 재생성할 필요가 있기 때문일 수 있습니다.

      // 그리고 업데이트된 create()를 가져와야 합니다.

      // 여기서는 이전 destroy()를 사용하고 있다는 점에 주목하세요.
    }
  }
  currentlyRenderingFiber.flags |= fiberFlags;
  hook.memoizedState = pushEffect(
    HookHasEffect | hookFlags,
    // deps가 변경되면, HookHasEffect는 이 효과를 실행해야 함을 표시합니다.

    create,
    destroy,
    nextDeps
  );
}
```

우리는 deps 배열이 어떻게 작동하는지 알 수 있습니다. 다시 렌더링할 때, deps가 변경되지 않는 경우를 제외하고는 Effect 객체를 다시 생성합니다. deps가 변경되면, 생성된 Effect는 실행되도록 표시되며, 이전의 정리 함수가 함께 사용됩니다.

## 3. Effect가 언제 그리고 어떻게 실행되고 정리되는가?

위에서 우리는 `useEffect()`가 단순히 fiber 노드에 추가적인 데이터 구조를 생성한다는 것을 알았습니다. 이제 이러한 Effect 객체들이 어떻게 처리되는지 알아보아야 합니다.

### 3.1 `commitRoot()`에서 이펙트 내부 콜백 함수의 실행이 트리거됨

두 개의 fiber 트리를 비교하여(diffing) 얻은 결과를 바탕으로, 우리는 커밋 단계에서 호스트 DOM에 변경 사항을 반영합니다. 이펙트 내부 콜백 함수의 실행을 시작하는 코드를 쉽게 찾을 수 있습니다.

```ts
function commitRootImpl(
  root: FiberRoot,
  recoverableErrors: null | Array<CapturedValue<mixed>>,
  transitions: Array<Transition> | null,
  renderPriorityLevel: EventPriority,
) {
  // 수동 효과가 대기 중인 경우, 이를 처리하기 위한 콜백을 예약합니다.
  // 가능한 빨리 이 작업을 수행하여, 커밋 단계에서 예약될 수 있는 다른 작업보다 먼저 대기열에 넣습니다. (참조: #16714)
  // TODO: 수동 효과 콜백을 예약하는 다른 모든 위치를 삭제하십시오.
  // 이것들은 중복입니다.
  if (
    (finishedWork.subtreeFlags & PassiveMask) !== NoFlags ||
    (finishedWork.flags & PassiveMask) !== NoFlags
  ) {
    if (!rootDoesHavePassiveEffects) {
      rootDoesHavePassiveEffects = true;
      pendingPassiveEffectsRemainingLanes = remainingLanes;
      // workInProgressTransitions는 덮어쓰여질 수 있으므로,
      // 이를 처리할 때까지 pendingPassiveTransitions에 저장하려고 합니다.
      // 이것을 commitRoot에 인수로 전달해야 합니다.
      // 왜냐하면 workInProgressTransitions는 이전 렌더와 커밋 사이에 변경될 수 있기 때문입니다
      // setTimeout으로 커밋을 제한하면
      pendingPassiveTransitions = transitions;
      scheduleCallback(NormalSchedulerPriority, () => {
        flushPassiveEffects();
        // 이 렌더링이 수동 효과를 트리거했습니다: 수동 효과가 발생한 후에 루트 캐시 풀을 해제합니다.
        // 트리 내의 노드(HostRoot, Cache 경계 등)가 참조할 수 있는 캐시 풀을 해제하지 않도록 하기 위함입니다.
        return null;
      });
      // 여기서 useEffect에 의해 생성된 수동 효과를 플러시합니다.

      // 이는 즉시가 아닌 다음 틱에서 플러싱을 예약합니다.

      // 자세한 내용은 React 스케줄러 작동 방식을 참조하십시오.
    }
  }
  ...
}

```

## 3.2 flushPassiveEffects()

```ts
function flushPassiveEffectsImpl() {
  if (rootWithPendingPassiveEffects === null) {
    return false;
  }
  // 전환 플래그를 캐시하고 초기화합니다.
  const transitions = pendingPassiveTransitions;
  pendingPassiveTransitions = null;
  const root = rootWithPendingPassiveEffects;
  const lanes = pendingPassiveEffectsLanes;
  rootWithPendingPassiveEffects = null;
  // TODO: 이것은 때때로 rootWithPendingPassiveEffects와 동기화되지 않습니다.
  // 왜 그런지 찾아서 수정하십시오. 알려진 문제를 일으키지는 않지만(아마도 프로파일링에만 사용되기 때문에),
  // 리팩터링 위험이 있습니다.
  pendingPassiveEffectsLanes = NoLanes;
  const prevExecutionContext = executionContext;
  executionContext |= CommitContext;
  commitPassiveUnmountEffects(root.current);
  commitPassiveMountEffects(root, root.current, lanes, transitions);
  // 여기서 우리는 콜백이 실행되기 전에 효과 정리가 먼저 실행되는 것을 명확히 볼 수 있습니다.

  ...
}
```

## 3.3 commitPassiveUnmountEffects()

```tsx
export function commitPassiveUnmountEffects(finishedWork: Fiber): void {
  setCurrentDebugFiberInDEV(finishedWork);
  commitPassiveUnmountOnFiber(finishedWork);
  resetCurrentDebugFiberInDEV();
}
function commitPassiveUnmountOnFiber(finishedWork: Fiber): void {
  switch (finishedWork.tag) {
    case FunctionComponent:
    case ForwardRef:
    case SimpleMemoComponent: {
      recursivelyTraversePassiveUnmountEffects(finishedWork);
      // 여기서는 자식의 효과들이 먼저 정리됩니다.

      if (finishedWork.flags & Passive) {
        commitHookPassiveUnmountEffects(
          finishedWork,
          finishedWork.return,
          HookPassive | HookHasEffect,
          // 이 플래그는 의존성이 변경되지 않으면 콜백이 실행되지 않도록 보장합니다.
        );
      }
      break;
    }
    ...
  }
}
function commitHookPassiveUnmountEffects(
  finishedWork: Fiber,
  nearestMountedAncestor: null | Fiber,
  hookFlags: HookFlags,
) {
  if (shouldProfile(finishedWork)) {
    startPassiveEffectTimer();
    commitHookEffectListUnmount(
      hookFlags,
      finishedWork,
      nearestMountedAncestor,
    );
    recordPassiveEffectDuration(finishedWork);
  } else {
    commitHookEffectListUnmount(
      hookFlags,
      finishedWork,
      nearestMountedAncestor,
    );
  }
}
function commitHookEffectListUnmount(
  flags: HookFlags,
  finishedWork: Fiber,
  nearestMountedAncestor: Fiber | null,
) {
  const updateQueue: FunctionComponentUpdateQueue | null =
    (finishedWork.updateQueue: any);
  const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    do {
      if ((effect.tag & flags) === flags) {
        // 언마운트
        const inst = effect.inst;
        const destroy = inst.destroy;
        if (destroy !== undefined) {
          inst.destroy = undefined;
          safelyCallDestroy(finishedWork, nearestMountedAncestor, destroy);
        }
      }
      effect = effect.next;
    } while (effect !== firstEffect);
    // 여기서는 updateQueue의 모든 Effect들을 순회하고
    // 플래그에 맞는 것들만 필터링합니다.
  }
}
function safelyCallDestroy(
  current: Fiber,
  nearestMountedAncestor: Fiber | null,
  destroy: () => void,
) {
  try {
    destroy();
  } catch (error) {
    captureCommitPhaseError(current, nearestMountedAncestor, error);
  }
}
```

## 3.4 commitPassiveMountEffects()

`commitPassiveMountEffects` 또한 같은 방식으로 동작합니다.

```tsx
export function commitPassiveMountEffects(
  root: FiberRoot,
  finishedWork: Fiber,
  committedLanes: Lanes,
  committedTransitions: Array<Transition> | null,
): void {
  setCurrentDebugFiberInDEV(finishedWork);
  commitPassiveMountOnFiber(
    root,
    finishedWork,
    committedLanes,
    committedTransitions,
  );
  resetCurrentDebugFiberInDEV();
}
function commitPassiveMountOnFiber(
  finishedRoot: FiberRoot,
  finishedWork: Fiber,
  committedLanes: Lanes,
  committedTransitions: Array<Transition> | null,
): void {
  // 이 함수를 업데이트할 때는, offscreen 트리가 숨김 -> 표시로 전환될 때 또는 숨겨진 트리 내부의 효과를 토글할 때 대부분 동일한 작업을 수행하는 reconnectPassiveEffects도 업데이트하십시오.
  const flags = finishedWork.flags;
  switch (finishedWork.tag) {
    case FunctionComponent:
    case ForwardRef:
    case SimpleMemoComponent: {
      recursivelyTraversePassiveMountEffects(
        // 여기서는 자식의 효과들이 먼저 실행됩니다.
        finishedRoot,
        finishedWork,
        committedLanes,
        committedTransitions,
      );
      if (flags & Passive) {
        commitHookPassiveMountEffects(
          finishedWork,
          HookPassive | HookHasEffect,
          // 이 플래그는 의존성이 변경되지 않으면 콜백이 실행되지 않도록 보장합니다.
        );
      }
      break;
    }
    ...
  }
}
function commitHookPassiveMountEffects(
  finishedWork: Fiber,
  hookFlags: HookFlags,
) {
  if (shouldProfile(finishedWork)) {
    startPassiveEffectTimer();
    try {
      commitHookEffectListMount(hookFlags, finishedWork);
    } catch (error) {
      captureCommitPhaseError(finishedWork, finishedWork.return, error);
    }
    recordPassiveEffectDuration(finishedWork);
  } else {
    try {
      commitHookEffectListMount(hookFlags, finishedWork);
    } catch (error) {
      captureCommitPhaseError(finishedWork, finishedWork.return, error);
    }
  }
}
function commitHookEffectListMount(flags: HookFlags, finishedWork: Fiber) {
  const updateQueue: FunctionComponentUpdateQueue | null =
    (finishedWork.updateQueue: any);
  const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    do {
      if ((effect.tag & flags) === flags) {
        // 마운트
        const create = effect.create;
        const inst = effect.inst;
        const destroy = create();
        // 콜백이 여기서 실행됩니다!
        inst.destroy = destroy;
      }
      effect = effect.next;
    } while (effect !== firstEffect);
    // 필요한 Effect들을 필터링하고 실행합니다.
  }
}
```

## 4. 요약

소스 코드를 살펴본 결과, `useEffect()`의 내부 구조는 매우 간단하다는 것을 알게 되었습니다.

1. `useEffect()`는 fiber에 저장되는 Effect 객체를 생성합니다.

- Effect는 실행이 필요한지를 나타내는 태그(tag)를 가지고 있습니다.
- Effect는 우리가 첫 번째 인수로 전달하는 create()를 가지고 있습니다.
- Effect는 create()에서의 정리 작업인 destroy()를 가지고 있으며, 이는 create()가 실행될 때만 설정됩니다.

2. `useEffect()`는 매번 새로운 Effect 객체를 생성하지만, 의존성 배열이 변경될 경우 다른 태그를 설정합니다.

3. 호스트 DOM에 업데이트를 커밋할 때, 다음 틱(tick)에서 모든 Effect를 태그에 따라 다시 실행하도록 작업이 예약됩니다.

- 자식 컴포넌트의 Effect가 먼저 처리됩니다.
- 정리 작업이 먼저 실행됩니다.

## 퀴즈

오늘 배운것을 토대로 한번 위 문제를 풀어보세요.

```tsx
function App() {
  const [count, setCount] = useState(1);

  console.log(1);

  useEffect(() => {
    console.log(2);
    return () => {
      console.log(3);
    };
  }, [count]);

  useEffect(() => {
    console.log(4);
    setCount((count) => count + 1);
  }, []);

  return <Child count={count} />;
}
function Child({ count }) {
  useEffect(() => {
    console.log(5);
    return () => {
      console.log(6);
    };
  }, [count]);

  return null;
}

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```
