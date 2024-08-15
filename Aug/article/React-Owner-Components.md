## 🔗 [React Owner Components](https://reacttraining.com/blog/react-owner-components?utm_source=newsletter.reactdigest.net&utm_medium=newsletter&utm_campaign=dissecting-partial-pre-rendering&_bhlid=390ab20d8fe806e7a1b16d9b9afd50d1db6fbdcb)

### 🗓️ 번역 날짜: 2024.08.14

### 🧚 번역한 크루: 러기(박정우)

---

## React Owner Components

# React Owner Components

Owners vs Parents & Server Components와 Client Components에 왜 중요한가

React에서 "owner"라는 용어는 계층 구조에서 컴포넌트를 설명하는 데 사용될 수 있으며, "parent(부모)"라는 용어와 유사합니다. 이 용어는 새로운 React 개발자들이 거의 사용하지 않는 다소 잊혀진 용어이지만, React의 초기 시절에는 더 자주 사용되었으며, 특히 클라이언트 컴포넌트와 서버 컴포넌트(RSC)를 이해할 때 여전히 유용합니다.

다음은 이를 가장 잘 설명하는 예제입니다:

```jsx
function UserProfile() {
  const [user, setUser] = useState({});
  return (
    <div>
      <span></span>
    </div>
  );
}
```

이 예제에서 div는 부모이고 span은 자식이라고 할 수 있죠? JSX는 HTML이나 XML에서 사용되는 것과 동일한 부모-자식 용어를 사용합니다. 이제 컴포넌트를 사용하여 다시 작성해보겠습니다:

```jsx
function UserProfile() {
  const [user, setUser] = useState({});
  return (
    <ProfileLayout>
      <Avatar />
    </ProfileLayout>
  );
}
```

우리는 계층 구조에 대해 크게 바꾸지 않았습니다. ProfileLayout은 부모이고 Avatar는 자식입니다. "owner(소유자)" 컴포넌트는 UserProfile입니다.

소유자 컴포넌트는 다른 컴포넌트의 JSX를 "소유"하며, 해당 컴포넌트에 props를 전달할 수 있습니다.

UserProfile이 원한다면 user 상태를 ProfileLayout과 Avatar에 props로 전달할 수 있다는 점에 주목하세요. 이것이 UserProfile을 소유자로 만드는 이유입니다. UserProfile은 이 다른 컴포넌트들의 JSX를 생성하고, 그들에게 props를 전달할 수 있기 때문입니다.

계층 구조상 UserProfile과 ProfileLayout은 Avatar "위"에 있지만, 이들이 동일하지 않기 때문에 별도의 용어를 사용합니다. 부모 컴포넌트는 자식에게 props를 전달할 수 없습니다. 위의 예에서 ProfileLayout은 Avatar의 부모이며, 이를 children으로 받습니다. 따라서 props를 전달할 수 없으며, 따라서 그 관계는 소유자-자식이 아닌 부모-자식 관계입니다:

```jsx
function ProfileLayout({ children }) {
  // 우리는 Avatar를 children props로 받지만, props를 전달하지 않습니다.
}
```

부모는 자식을 다시 렌더링하지 않습니다.
React가 렌더링의 연쇄를 생성할 수 있다는 사실을 아마 알고 계실 겁니다. 소유자 컴포넌트가 다시 렌더링되면, 소유한 모든 컴포넌트를 다시 렌더링합니다. 이는 부모 컴포넌트에 해당하지 않습니다. 우리의 예에서 UserProfile이 어떤 이유로든 다시 렌더링되면, ProfileLayout과 Avatar 모두를 다시 렌더링합니다. 왜냐하면 UserProfile이 그들을 소유하고 있기 때문입니다.

```jsx
function UserProfile() {
  const [user, setUser] = useState({});
  return (
    <ProfileLayout>
      <Avatar />
    </ProfileLayout>
  );
}
```

그러나 다른 상황에서는 ProfileLayout에 상태가 있고, 그것이 다시 렌더링된다고 해도, 이는 Avatar의 다시 렌더링을 생성하지 않습니다. 비록 Avatar가 계층 구조상 ProfileLayout "아래"에 있더라도 말이죠:

```jsx
function ProfileLayout({ children }) {
  const [someState, setSomeState] = useState();
  return <div>{children}</div>;
}
```

React 서버 컴포넌트
"서버 컴포넌트"는 React의 새로운 유형의 컴포넌트입니다. 이 용어는 혼란을 줄 수 있는데, 우리가 항상 SSR(서버 측 렌더링) 측면에서 컴포넌트를 서버에서 실행할 수 있었기 때문입니다. 그렇다면 우리가 수년간 작성해온 다른 종류의 컴포넌트들은 무엇이라고 부를까요? 우리는 그것들을 "클라이언트 컴포넌트"라고 부르기 시작했습니다. 왜냐하면 그것들은 서버에서 실행할 수 있지만 클라이언트에서도 실행할 수 있기 때문입니다. 반면에, 서버 컴포넌트는 클라이언트에서 실행될 수 없으며, 오직 서버에서만 실행됩니다.

NextJS와 함께 서버 컴포넌트를 사용하는 경우, 컴포넌트 트리가 기본적으로 서버 컴포넌트임을 알 수 있으며, 하위 트리의 일부에 대해 클라이언트 경계를 지정할 때까지 그렇습니다. 컴포넌트에서 `use client` 지시어를 지정하면, 이제 트리 아래의 모든 컴포넌트가 클라이언트 컴포넌트가 됩니다. Lydia Hallie는 이러한 개념을 애니메이션으로 훌륭하게 설명합니다:

React에서 "owner(소유자) vs parent(부모)"를 이해하는 것이 중요한 이유는 서버 컴포넌트와 클라이언트 컴포넌트를 함께 사용할 때 새로운 규칙을 이해하는 데 도움이 되기 때문입니다.

서버 컴포넌트는 클라이언트로 다시 수화(rehydrate)되지 않습니다. 서버 컴포넌트의 코드가 클라이언트로 전송되지 않으며, 서버 컴포넌트의 결과물(HTML 및 기타 메타 데이터)이 클라이언트에 고정된 상태로 존재할 뿐입니다. 따라서 서버 컴포넌트는 상태나 JavaScript 이벤트를 가질 수 없고, "다시 렌더링"이라는 개념도 존재하지 않습니다. 다시 말해, 컴포넌트 코드 자체가 클라이언트로 전송되지 않기 때문입니다. 이 때문에 서버 컴포넌트는 클라이언트 컴포넌트에 의해 소유될 수 없습니다. 클라이언트 컴포넌트가 클라이언트에서 다시 렌더링된다면, 소유한 서버 컴포넌트를 "다시 렌더링"할 방법이 없기 때문입니다.

예를 들어, ProfileLayout이 클라이언트 컴포넌트이고 Avatar가 서버 컴포넌트인 경우를 생각해봅시다. 이 상황에서 ProfileLayout이 클라이언트에서 다시 렌더링되면 문제가 발생할 수 있다는 점이 보이시나요? ProfileLayout이 Avatar의 소유자이기 때문에, 개념적으로는 Avatar를 다시 렌더링하게 됩니다. 하지만 서버 컴포넌트는 다시 렌더링될 수 없으므로, 이는 허용되지 않습니다:

```jsx
// 클라이언트 컴포넌트
function ProfileLayout() {
  const [state, setState] = useState();
  return (
    <div>
      <Avatar />
    </div>
  );
}

// 서버 컴포넌트
function Avatar() {}
```

만약 클라이언트 컴포넌트 아래에서 (DOM 관점에서) 서버 컴포넌트를 렌더링해야 한다면, 이를 재배치하여 누가 무엇을 소유하는지를 조정할 수 있습니다. 즉, 서버 컴포넌트인 UserProfile이 Avatar를 소유하게 하고, 이를 클라이언트 컴포넌트에 children으로 전달하는 것입니다:

```jsx
// 서버 컴포넌트
function UserProfile() {
  const user = getUser();
  return (
    <ProfileLayout>
      <Avatar user={user} />
    </ProfileLayout>
  );
}

// 클라이언트 컴포넌트
function ProfileLayout({ children }) {
  const [state, setState] = useState();
  return <div>{children}</div>;
}

// 서버 컴포넌트
function Avatar({ user }) {}
```

다른 말로 하면, 서버 컴포넌트만이 다른 서버 컴포넌트에 props를 전달할 수 있습니다. 위의 예에서처럼 서버 컴포넌트가 서버 컴포넌트에 props를 전달할 때, 이는 HTML이 생성되기 전에 서버에서 이루어집니다. 그런 다음 ProfileLayout이 클라이언트에서 수화(rehydrate)되면 상태를 가지게 되고 다시 렌더링될 수 있습니다.

### 이 용어가 비공식적인가요?

React 공식 문서에서는 "owner"라는 용어를 자주 사용하지 않는다는 것을 알 수 있습니다. 가끔 "owner"가 사용될 수 있는 자리에 "parent"가 사용되기도 합니다. 이 용어가 공식적으로 더 수용되었으면 좋겠다고 생각하는데, 왜냐하면 소유자와 부모를 구분하는 것이 React에서 무슨 일이 일어나고 있는지를 더 잘 설명하는 데 도움이 되기 때문입니다. 제 워크샵에서 "owner"라는 개념을 설명하면, 더 복잡한 React 개념을 이해하는 데 도움이 된다는 것을 느낍니다.

Google에서 이 용어를 검색하면 이에 대한 오래된 참고 자료를 많이 볼 수 있습니다.

[Google에서 "owner component react" 검색](https://www.google.com/search?q=owner+component+react)

가끔 "owner"라는 용어가 사용되는 것을 보게 될 텐데, 이제 이 용어가 설명하는 내용을 더 잘 이해할 수 있기를 바랍니다.
