## 🔗 [Wait for pending: A Suspense algorithm exploration](https://dev.to/alexandereardon/wait-for-pending-a-not-great-alternative-suspense-algorithm-1gdl)

### 🗓️ 번역 날짜: 2024.07.08

### 🧚 번역한 크루: 마스터위(명재위)

---

Alex Reardon  
Updated On Jun 25, 2024  
Keyword : **React**, **Javascript**

React 19의 `<Suspense>` 타이밍에 대한 [흥미로운 논의](https://github.com/facebook/react/issues/29898)가 진행 중입니다.

이 글에서는 `<Suspense>` 경계 내의 모든 대기 중인 promise가 해결된 후에 다시 렌더링을 시도하는 알고리즘이 어떻게 생길지 탐구합니다.

> 이는 처음에는 메모장에 적어본 아이디어에서 출발해 제안서로 발전했지만, 접근 방식에 문제가 있음을 발견했습니다. 그럼에도 불구하고 기록하고 공유하기로 했습니다. 이로 인해 더 나은 아이디어가 나올 수도 있습니다!

### TLDR

'Wait for pending'은 react@18과 react@19(alpha) `<Suspense>` 알고리즘의 흥미로운 변형입니다. 평면 비동기 트리(flat async tree)에서는 'wait for pending'이 빠른 렌더 함수 호출과 최소한의 재렌더링을 허용합니다. 중첩된 비동기 컴포넌트가 있는 트리에서는 자식 비동기 컴포넌트의 초기 렌더링이 react@18 알고리즘보다 느려질 수 있습니다.

### Background

**react@18의 `<Suspense>`**

- 가능한 모든 컴포넌트를 `<Suspense>` 경계 내에서 렌더링합니다.
- 어떤 promise가 resolve 될 때마다 리렌더링합니다.
- 더 이상 promise를 던지는 컴포넌트가 없을 때까지 계속합니다.

결과적으로 많은 재렌더링이 발생하지만, 'fetch in render' 호출의 병렬화를 가능하게 하고, 모든 컴포넌트가 최대한 빨리 렌더링되도록 합니다.

**react@19 (alpha)의 `<Suspense>`**

- 컴포넌트가 promise를 던질 때 tree 렌더링을 중지합니다.
- promise가 resolve될 때까지 기다립니다.
- tree를 리렌더링합니다.
- 더 이상 promise를 던지는 컴포넌트가 없을 때까지 계속합니다.

최소한의 fl렌더링이 발생하지만, 'fetch in render' 호출이 순차적으로 이루어집니다(폭포수 형태(waterfall)).

### 'Wait for pending' 알고리즘

`<Suspense>` 타이밍('Wait for pending')에 대한 아이디어:

- 형제 컴포넌트를 항상 렌더링합니다.
- 모든 현재 던져진 promise가 해결될 때까지 자식 컴포넌트를 리렌더링하지 않습니다.

어떻게 되는지 봅시다!

'Wait for pending'은 현재의 react@18 `<Suspense>`알고리즘과 비슷합니다. 단, 어떤 promise가 resolve 될 때마다 모든 자식을 렌더링하는 대신, 현재 던져진 모든 promise가 resolve될 때만 렌더링합니다.

- 형제 간 'fetch in render'가 병렬 fetch를 트리거할 수 있습니다.
- 여전히 pre-fetching에 좋은 story를 가지고 있습니다.
- 리렌더링으로 인한 낭비를 줄입니다.
- 비싼 컴포넌트 렌더링이 `throw`하는 형제들과 함께 발생해도 여전히 중복됩니다. 하지만 적어도 이러한 중복 렌더링이 줄어듭니다.
- 👎 중첩된 'fetch in render' 호출을 느리게 할 수 있습니다 (아래 참조).

대략의 알고리즘

1. 자식들을 렌더한다.
2. 던져진 promise가 없으면 완료 - 그렇지 않으면 3단계로 이동한다.
3. 모든 promise resolve 될 때까지 기다린다.
4. 1단계로 돌아간다.

### Example 1: Only siblings

```jsx
<Suspense fallback={'loading'}>
  <A />
  <B />
<Suspense>
```

해당 예시에서, `A`와 `B` 둘다 그들의 렌더에 데이터를 위한 `fetch`가 있다.

**react@18**

Render 1

- `A` 렌더되지만, promise 던짐
- `B` 렌더되지만, promise 던짐

Render 2

- `A`의 promise가 resolve됨
- `A`가 렌더링됨
- `B`가 렌더링되지만, promise 던짐

Render 3

- `B`의 promise가 resolve됨
- `A`가 렌더링됨
- `B`가 렌더링됨

**react@19 alpha timing**

Render 1

- `A` 렌더되지만, promise 던짐

Render 2

- `A`의 promise가 resolve됨
- `A`가 렌더링됨
- `B`가 렌더링되지만, promise 던짐

Render 3

- `B`의 promise가 resolve됨
- `A`가 렌더링됨
- `B`가 렌더링됨

😢 렌더링에서 데이터를 가져오면(promise를 던지면) 폭포수(waterfall)가 발생함
😊 잠재적으로 비용이 많이 드는 컴포넌트의 과도한 리렌더링을 방지함

**Wait for pending**

Render 1

- `A` 렌더되지만, promise 던짐
- `B` 렌더되지만, promise 던짐
- **`A`와 `B`가 resolve 되길 기다림**

Render 2

- `A`가 렌더링됨
- `B`가 렌더링됨

✅ 이 경우, 제안된 알고리즘은 훌륭한 결과를 가져옵니다!

### Example 2: children과 함께일 때

여기서 'wait for pending' 알고리즘이 부담을 느끼게 됩니다.

이제 `A`가 `ChildX`와 `ChildY`를 자식으로 렌더링하는데, 이들 또한 'fetch in render'를 수행합니다.

```jsx
<Suspense fallback={'loading'}>
  <A>
    <ChildX />
    <ChildY />
  </A>
  <B />
<Suspense>

```

**react@18**

Render 1

- `A` 렌더되지만, promise 던짐
- `B` 렌더되지만, promise 던짐

Render 2

- `A`의 promise가 resolve됨
- `A`가 렌더링됨
- `ChildX`가 렌더링되지만, promise 던짐
- `ChildY`가 렌더링되지만, promise 던짐
- `B`가 렌더링되지만, promise 던짐

Render 3

- `ChildX`의 promise가 resolve됨
- `A`가 렌더링됨
- `ChildX`가 렌더링됨
- `ChildY`가 렌더링되지만, promise 던짐
- `B`가 렌더링되지만, promise 던짐

Render 4

- `ChildY`의 promise가 resolve됨
- `A`가 렌더링됨
- `ChildX`가 렌더링됨
- `ChildY`가 렌더링됨
- `B`가 렌더링되지만, promise 던짐

Render 5

- `B`의 promise가 resolve됨
- `A`가 렌더링됨
- `ChildX`가 렌더링됨
- `ChildY`가 렌더링됨
- `B`가 렌더링됨

✅ ChildX와 ChildY가 가능한 빨리 렌더링됩니다.
😢 많은 중복 재렌더링이 발생합니다.

**react@19 alpha timing**

Render 1

- `A` 렌더되지만, promise 던짐

Render 2

- `A`의 promise가 resolve됨
- `A`가 렌더링됨
- `ChildX`가 렌더링되지만, `B`가 promise 던짐

Render 3

- `ChildX`의 promise가 resolve됨
- `A`가 렌더링됨
- `ChildX`가 렌더링됨
- `ChildY`가 렌더링되지만, `ChildY`가 promise 던짐

Render 4

- `ChildY`의 promise가 resolve됨
- `A`가 렌더링됨
- `ChildX`가 렌더링됨
- `ChildY`가 렌더링됨
- `B`가 렌더링되지만, `B`가 promise 던짐

Render 5

- `B`의 promise가 resolve됨
- `A`가 렌더링됨
- `ChildX`가 렌더링됨
- `ChildY`가 렌더링됨
- `B`가 렌더링됨

**제안한 알고리즘**

Render 1

- `A` 렌더되지만, promise 던짐
- `B` 렌더되지만, promise 던짐
- **`A`와 `B`가 resolve 되길 기다림**

Render 2

- `A`와 `B`의 promise가 resolve됨
- `A`가 렌더링됨
- `ChildX`가 렌더링되지만, promise 던짐
- `ChildY`가 렌더링되지만, promise 던짐
- `B`가 렌더링됨
- **`ChildX`와 `ChildY`가 resolve 되길 기다림**

Render 3

- `ChildX`와 `ChildY`의 promise가 resolve됨
- `A`가 렌더링됨
- `ChildX`가 렌더링됨
- `ChildY`가 렌더링됨
- `B`가 렌더링됨

✅ react@18 알고리즘보다 중복 렌더링이 훨씬 적습니다.
👎 ChildX와 ChildY는 B가 해결되기 전까지 'fetch in render' 호출을 시작할 수 없습니다. 이들은 부모의 가장 느린 형제가 해결될 때까지 기다려야 했습니다.
🤔 react@19 알고리즘보다 병렬 처리가 더 많이 되지만, react@18보다 모든 컴포넌트의 초기 렌더링 시작이 더 느립니다.

### 마무리 생각

'Wait for pending'은 흥미로운 접근 방식입니다.

flat async trees에서는 'wait for pending'이 렌더 함수 호출을 빠르게 하고 리렌더링을 최소화합니다. 그러나 중첩된 비동기 컴포넌트가 있는 트리에서는 자식 비동기 컴포넌트가 초기 렌더링을 시작하기 전에 부모 형제 컴포넌트가 렌더링을 완료해야 합니다. 중첩된 컴포넌트가 네트워크 호출과 같은 비용이 많이 드는 작업을 수행하는 경우, 가능한 한 빨리 초기 렌더링을 트리거하는 것이 이상적입니다(react@18 알고리즘). 'Wait for pending'은 react@19 접근 방식과 유사하지만, 각 레벨이 병렬 처리될 수 있습니다.

다른 `<Suspense>` 알고리즘이 어떻게 생길지 생각해보는 것은 흥미로웠습니다! 여기까지 읽어주셔서 감사합니다 😅.

Cheers
