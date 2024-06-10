# [When do I use functions in a Hooks Dependency Array?](https://reacttraining.com/blog/when-to-use-functions-in-hooks-dependency-array)

### 🗓️ 번역 날짜: 2024.06.03

### 🧚 번역한 크루: 마스터위(명재위)

---

Brad Westfall   
Updated On Sep 30, 2019   
Keyword : **React** **Hooks**   

### 간단히 말하자면...

**linter가 당신에게 그렇게 하라고 할때마다 해야합니다.**
[그리고 이것은 꽤 자주 일어납니다.](https://legacy.reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)
하지만 왜 linter는 때때로 그렇게 하라고 하고, 항상 그렇지는 않을까요?

---

Hooks는 React에 함수형 구성을 도입하기 위해 설계되었지만, 제대로 작동하기 위해 따라야 할 몇 가지 규칙이 있습니다. React에는 특정 작업을 잘못할 때(예: 훅의 순서를 조건부로 변경하는 경우) 알려주는 몇 가지 내장 린팅 규칙이 있습니다. [또한 별도로 설치해야 하는 추가 규칙도 있습니다.](https://www.npmjs.com/package/eslint-plugin-react-hooks) 이러한 추가 규칙은 의존성 배열에 무엇을 추가해야 하는지 알려주는 [exhaustive-deps](https://github.com/facebook/react/issues/14920) 규칙과 같이 버그를 조기에 잡는 데 도움이 됩니다.

의존성 배열 개념을 사용하는 훅은 여러 가지가 있지만, 아마도 많은 사람들이 처음 배우는 훅 중 하나인 useEffect를 배우면서 의존성 배열에 대해 알게 될 것입니다:

```jsx
function ComposeMessage() {
  const [message, setMessage] = useState();

  // 메시지가 변경될 때마다 제목을 변경하는 것은 side-effects입니다.
  // 그래서 `useEffect`가 필요합니다.
  useEffect(() => {
    document.title = message;
  }, [message]);

  return (
    <div>
      <input type="text" onChange={(e) => setMessage(e.target.value)} />
    </div>
  );
}
```

우리의 effect는 message 상태에 "의존"합니다. 따라서 그 상태가 변경되면 effect를 다시 실행해야 합니다.

이제 메시지가 변경될 때 메시지를 로컬 저장소에 저장해 보겠습니다. 이렇게 하면 메시지가 저장되지 않았을 때 신속하게 초안으로 복구할 수 있습니다. 또한 수신자의 사용자 ID를 나타내는 uid prop도 사용하겠습니다.

```jsx
import { saveToLocalStorage, getFromLocalStorage } from "./saveToLocalStorage";

function ComposeMessage({ uid }) {
  const [message, setMessage] = useState(getFromLocalStorage(uid) || "");

  useEffect(() => {
    saveToLocalStorage(uid, message);
  }, [uid, message]); // 우리의 effect는 이제 더 많은 것에 의존하고 있습니다.

  return (
    <input
      type="text"
      value={message}
      onChange={(e) => setMessage(e.target.value)}
    />
  );
}
```

linter는 이제 uid가 side effect의 일부이기 때문에 의존성 배열에 uid를 추가하라고 요청할 것입니다.

> 그러나 'uid'는 상태가 아닌 props 입니다 !

자, 괜찮습니다. 비록 그것이 우리의 컴포넌트 상태는 아니지만, 다른 곳(예: 부모 컴포넌트)의 상태이므로 동일한 개념입니다.

그렇다면 함수 `saveToLocalStorage`는 어떻게 할까요? effect에서 이를 사용하고 있으니 의존성 배열에 추가해야 할까요?

이 경우에는 아닙니다. 왜 그런지 논의하기 전에, `saveToLocalStorage`가 prop으로 전달되는 아래의 리팩터링된 코드와 비교해 봅시다.

```jsx
function ComposeMessage({ uid, defaultMessage, saveToLocalStorage }) {
  const [message, setMessage] = useState(defaultMessage || "");

  useEffect(() => {
    saveToLocalStorage(uid, message);
  }, [uid, message, saveToLocalStorage]); // Now it goes here

  return (
    <input
      type="text"
      value={message}
      onChange={(e) => setMessage(e.target.value)}
    />
  );
}
```

이제 linter가 saveToLocalStorage를 의존성 배열에 추가하라고 요청합니다. 차이점은 무엇일까요?

궁극적으로, React는 effects 내의 상태가 변경될 경우 effect를 "re-run" 해야 합니다. 이전에 `saveToLocalStorage`가 import 되었을 때, linter는 그 함수가 컴포넌트 상태를 "close over"하여 변경되었을 때 effect를 다시 실행할 필요가 없다는 것을 알고 있습니다. 그러나 `saveToLocalStorage`가 prop일 때, 린터는 부모 컴포넌트가 `ComposeMessage`를 어떻게 구현할지에 대해 충분히 알지 못합니다. 다시 말해, linter는 전체 앱을 탐색하여 `ComposeMessage`가 사용되는 모든 위치와 부모가 prop을 전달하는 방식을 알지 못합니다. 그리고 설령 그렇게 한다 하더라도, linter는 당신이 미래에 그것을 어떻게 사용할지 의도를 알지 못합니다. 이러한 불확실성 때문에, 린터는 이제 `saveToLocalStorage`를 의존성 배열에 추가하라고 요청합니다.

다음은 부모 컴포넌트가 구현될 수 있는 한 가지 예입니다:

```jsx
import { saveToLocalStorage, getFromLocalStorage } from "./saveToLocalStorage";

function UserProfile({ uid }) {
  return (
    <ComposeMessage
      uid={uid}
      defaultMessage={getFromLocalStorage(uid)}
      saveToLocalStorage={saveToLocalStorage}
    />
  );
}
```

비록 `saveToLocalStorage`가 여전히 import에 대한 참조일 뿐이지만, 자식 `ComposeMessage`는 prop이 의존성 배열에 추가되어야 한다고 말하고 있습니다. 다시 말하지만, 이제 중요한 차이점은 **확실성**입니다. 이전에는 React가 `saveToLocalStorage`가 어떤 상태도 close over하지 않는다는 것을 알고 있었습니다. 부모를 리팩터링한다면 어떻게 될까요?

부모 컴포넌트에 애플리케이션 세부 정보를 유지하고 `ComposeMessage`는 저장해야 할 시점만 보고하고 `uid`와 같은 것에 대해 알 필요가 없다고 가정해 봅시다. 이 경우 코드가 다음과 같이 보일 수 있습니다:

```jsx
// UserProfile.js
import ComposeMessage from "./ComposeMessage";
import { saveToLocalStorage, getFromLocalStorage } from "./saveToLocalStorage";

function UserProfile({ uid }) {
  return (
    <ComposeMessage
      defaultMessage={getFromLocalStorage(uid)}
      saveToLocalStorage={(message) => saveToLocalStorage(uid, message)}
    />
  );
}

// ComposeMessage.js
function ComposeMessage({ defaultMessage, saveToLocalStorage }) {
  const [message, setMessage] = useState(defaultMessage || "");

  useEffect(() => {
    saveToLocalStorage(message);
  }, [message, saveToLocalStorage]);

  return (
    <input
      type="text"
      value={message}
      onChange={(e) => setMessage(e.target.value)}
    />
  );
}
```

저장하기 위해 실제 함수로 전달되는 `saveToLocalStorage`는 import된 `saveToLocalStorage`를 래핑하는 화살표 함수입니다. 이 리팩터링을 통해 전달된 함수는 `uid`를 close over하게 되므로, 이제 변경될 가능성이 있기 때문에 `ComposeMessage`에서 `saveToLocalStorage` prop을 의존성 배열에 포함하는 것이 중요합니다. React는 이 함수가 필요할지 여부를 알 수 없으므로 항상 포함시키도록 했습니다.

다른 고려 사항:

부모 컴포넌트가 어떤 이유로든 다시 렌더링되면(예: 상태 변경 또는 새로운 props), 화살표 함수는 다시 렌더링될 때마다 새로 생성됩니다. 이는 부모가 다시 렌더링될 때마다 `saveToLocalStorage` prop의 함수 아이덴티티가 변경된다는 것을 의미합니다. `ComposeMessage`에서는 `saveToLocalStorage`를 의존성 배열에 포함시켜 버그를 방지해야 하지만, 부모가 다시 렌더링될 때마다 그 아이덴티티가 변경되므로 불필요하게 로컬 저장소에 저장하게 됩니다. 이를 [시연하는 예제](https://codesandbox.io/s/funny-taussig-5tont)를 보세요.

로컬 저장소에 너무 자주 저장하는 것은 괜찮을 수 있지만, side effect가 네트워크 요청인 경우에는 이를 피하고 싶을 것입니다. 따라서 함수의 아이덴티티가 변하지 않도록 유지할 필요가 있습니다.

### `useCallback` 사용하기

`uid`와 함수의 아이덴티티를 동기화하기 위해 부모 컴포넌트의 함수를 다음과 같이 구현할 수 있습니다:

```jsx
import ComposeMessage from "./ComposeMessage";
import { saveToLocalStorage, getFromLocalStorage } from "./saveToLocalStorage";

function UserProfile({ uid }) {
  const save = useCallback(
    (message) => {
      saveToLocalStorage(uid, message);
    },
    [uid]
  );

  return (
    <ComposeMessage
      defaultMessage={getFromLocalStorage(uid)}
      saveToLocalStorage={save}
    />
  );
}
```

`useCallback` 훅은 이와 같은 목적을 위해 함수의 메모이제이션 버전을 생성합니다. 이는 또 다른 의존성 배열 개념을 가진 훅입니다. 이 경우 `save` 함수는 의존성 배열 내의 무언가가 변경되지 않는 한, `UserProfil`e이 몇 번이나 다시 렌더링되더라도 동일한 아이덴티티를 유지합니다. 유일한 예외는 의존성 배열의 무언가가 변경될 때 새로운 아이덴티티가 생성되는 것입니다.

시나리오를 통해 설명해 보겠습니다:

- 부모 컴포넌트 `UserProfile`이 `ComposeMessage`에 함수 prop을 전달합니다.
- `ComposeMessage`에서 효과(effect)를 다시 실행시키는 두 가지 조건은 다음과 같습니다:

  - 메시지가 변경될 때 (우리가 원하는 것)
  - 또는 `saveToLocalStorage` prop이 변경될 때 (이 점을 염두에 두세요)

- 부모 함수는 리렌더링을 경험할 수 있으며 이는 `ComposeMessage`를 다시 렌더링하게 합니다. 이 경우 useEffect의 의존성 배열이 다시 평가됩니다. 부모가 다시 렌더링되지만 `uid`가 변경되지 않는 경우, `saveToLocalStorage` 함수가 변경되지 않도록 하여 effect가 실행되지 않도록 해야 합니다. `useCallback`이 이를 해결해줍니다.
- 부모의 `uid`가 변경되면, `useCallback`은 `save`를 위한 새로운 아이덴티티를 생성하고, 따라서 `uid`가 변경될 때 이후의 effect가 다시 실행됩니다.

## 요약

따라서 함수가 의존성 배열에 포함되어야 하는 경우는 언제일까요? 상태를 잠재적으로 close over 할 수 있을 때입니다.

이 예제가 이를 완벽하게 요약해 줍니다:

```jsx
const MyComponent = () => {
  // 이 함수는 이 순간에 state를 close over하지 않습니다.
  function logData() {
    console.log("logData");
  }

  useEffect(() => {
    logData();
  }, []); // `logData`는 의존성 배열 안으로 들어가지 않아도 됩니다.

  // ...
};
```

그렇다면 어떤 props를 `console.log` 해봅시다.

```jsx
const MyComponent = ({ data }) => {
  // 이 함수는 이제 상태를 close over 합니다.(props는 어떤 것의 상태임을 기억하세요)
  function logData() {
    console.log(data);
  }

  useEffect(() => {
    logData();
  }, [logData]); // 이제 이것을 여기에 추가하세요

  // ...
};
```

이제 `logData`가 의존성 배열에 포함되었으므로, 새로운 문제는 `MyComponent`가 리렌더링될 때마다 이 함수가 변경된다는 점입니다. 따라서 `useCallback`을 사용해야 합니다:

```jsx
const MyComponent = ({ data }) => {
  const logData = useCallback(() => {
    console.log(data);
  }, [data]);

  useEffect(() => {
    logData();
  }, [logData]); // 이제 이것을 여기에 추가하세요

  // ...
};
```

또는 이렇게 할 수 있습니다.

```jsx
const MyComponent = ({ data }) => {
  useEffect(() => {
    function logData() {
      console.log(data);
    }
    logData();
  }, [data]); // 이제 오직 `data`만 이것에 필요합니다.

  // ...
};
```

`logData`는 상태를 close over하지만, 그것 자체가 effect의 일부이므로 배열에는 `data`만 필요합니다.
