# [React: "mount" vs "render"?](https://reacttraining.com/blog/mount-vs-render)

### 🗓️ 번역 날짜: 2024.08.19

### 🧚 번역한 크루: 마스터위(명재위)

---

## React: "mount" vs "render" ?

Brad Westfall
Updated On Jan 31, 2020  
Keyword : **React**

새로운 것을 배우는 것은 종종 많은 새로운 용어와 함께 옵니다. 그렇다면 React에서 "mounting"과 "rendering"의 차이점은 무엇일까요?

### tldr;

- "Rendering"은 함수형 컴포넌트가 호출되거나 클래스 기반의 render 메서드가 호출되어 DOM을 생성하는 일련의 명령을 반환하는 모든 순간을 의미합니다.
- "Mounting"은 React가 컴포넌트를 처음 "렌더링"하고, 이러한 명령을 사용하여 초기 DOM을 실제로 구축하는 과정을 의미합니다.

"재렌더링(re-render)"은 React가 이미 마운트된 컴포넌트에서 새로운 명령 세트를 얻기 위해 함수형 컴포넌트를 다시 호출하는 것입니다.

함수형 컴포넌트를 가르치면서 "마운팅(mounting)"과 "렌더링(rendering)"에 대한 질문을 클래스 기반 컴포넌트를 가르칠 때보다 더 자주 받기 시작했습니다. 왜 그런지 이해한 것 같습니다. 먼저 클래스 기반 컴포넌트에서 "마운팅"과 "렌더링"의 차이를 설명하고, 그 후 동일한 개념을 함수형 컴포넌트와 비교해 보겠습니다.

React가 처음으로 만나는 컴포넌트가 `App`이라고 가정해보겠습니다. 이 컴포넌트를 `ReactDOM.render`에 전달합니다.

```tsx
class App extends React.Component {
  state = {
    showUser: false,
  };

  render() {
    return (
      <div>
        {this.state.showUser && <User name='Brad' />}
        <button onClick={() => this.setState({ showUser: true })}>
          Show User
        </button>
        <button onClick={() => this.setState({ showUser: false })}>
          Hide User
        </button>
      </div>
    );
  }
}

ReactDOM.render(<App />, document.getElementById('root'));
```

React는 내부적으로 `App`의 인스턴스를 생성하고, 초기 DOM을 구성하기 위한 첫 번째 명령 세트를 얻기 위해 `render` 메서드를 호출합니다. 클래스 기반 컴포넌트에서 render 메서드가 호출될 때마다 이를 "렌더링"이라고 합니다. 첫 번째 렌더링에서는 컴포넌트가 "마운팅"된다고 하며, React가 DOM에 명령을 적용하고 이전에 존재하지 않았던 DOM을 생성하는 것입니다.

사용자가 `Show User` 버튼을 클릭합니다. 상태가 변경될 때마다 React는 컴포넌트를 "재렌더링"합니다. 이는 상태 변경에 따라 새로운 명령을 얻기 위해 render 메서드를 다시 호출하는 것을 의미합니다. 위의 컴포넌트의 경우, 이 재렌더링에서 `showUser` 값이 `true`가 됩니다. 이 값이 `true`가 되면, 컴포넌트에서 반환된 명령은 React에게 DOM 업데이트가 필요함을 알려줍니다. 마운팅 단계는 첫 번째 렌더링에서만 발생하므로 두 번째 렌더링은 마운팅이 아니며, DOM에서 어떤 부분이 업데이트되어야 하는지를 파악해야 합니다. React는 내부적으로 이전 명령과 새로운 명령을 비교하여 DOM에서 실제로 업데이트가 필요한 부분을 확인하는데, 이를 "[조정(reconciliation)](https://legacy.reactjs.org/docs/reconciliation.html)"이라고 합니다.

중요한 점은 첫 번째 "렌더링"이 React가 DOM에 항목을 "마운팅"하는 것으로 이어지며, 재렌더링은 DOM을 업데이트하지만 마운팅은 한 번만 발생한다는 것입니다.

이제 위의 같은 코드를 다시 써봅시다. 대신 함수 컴포넌트와 훅과 함께 씁니다:

```tsx
function App() {
  const [showUser, setShowUser] = useState(false);

  return (
    <div>
      {showUser && <User name='Brad' />}
      <button onClick={() => setShowUser(true)}>Show User</button>
      <button onClick={() => setShowUser(false)}>Hide User</button>
    </div>
  );
}
```

함수형 컴포넌트도 렌더와 마운트 과정을 가지고 있습니다. React가 함수형 컴포넌트를 사용할 때, 함수가 호출(렌더링)되고 반환된 명령이 마운트에 사용됩니다. 그래서 함수형 컴포넌트는 전체 함수가 클래스 컴포넌트의 render 메서드와 유사하다고 할 수 있습니다. 코드에서 "render"라는 단어를 명시적으로 보지 않더라도 말입니다.

함수형 컴포넌트가 이제 상태를 가질 수 있기 때문에, 상태가 변경될 때마다 컴포넌트는 이전과 마찬가지로 재렌더링이 필요합니다.

```tsx
// 리액트가 우리의 컴포넌트를 render하기 시작하고 이것이 첫번째 render이기 때문에, 이것이 DOM에 컴포넌트를 "mounts'입니다.
App();

// 상태가 바뀌고 리액트는 이제 새로운 상태가 놓은 컴포넌트 재렌더링이 필요합니다.
App();

// 상태가 다시 바뀌고 리액트의 재렌더링이 다시 일어납니다.
App();
```

초기 렌더링은 마운팅을 유도하며, 재렌더링은 단순히 함수를 다시 호출하는 과정입니다.

이제 두 컴포넌트, `User`와 `App`을 예로 들어보겠습니다.

React는 처음에 App을 렌더링하지만 User가 반환되지 않기 때문에 User는 React에게 알려지지 않습니다. 이 시점에서 App은 마운트되지만 User는 마운트되지 않습니다.

그 후 showUser가 true로 설정되면 `App`이 재렌더링되고, `User`를 포함한 명령이 반환되어 `User`가 처음 렌더링되고 마운트됩니다.

사용자가 `Hide User` 버튼을 클릭하면 상태가 변경되어 `App`이 다시 재렌더링됩니다. 새로운 명령과 이전 렌더링에서 반환된 명령이 비교되고, React는 `User`가 더 이상 `App`에서 반환되지 않음을 감지하여 `User`를 DOM에서 "언마운트"합니다.

컴포넌트 아키텍처에서 부모 노드는 자식 노드를 인식하지만 자식 노드는 부모 노드를 인식하지 않는 경우가 많습니다. React는 부모 컴포넌트가 재렌더링될 때마다 자식 노드도 재렌더링하는 것이 자연스러운 특성입니다.

각 컴포넌트가 자체적인 마운팅 및 렌더링 "스케줄"을 가지더라도, 자식 컴포넌트는 부모가 마운트된 경우에만 렌더링 및 마운트될 수 있습니다. 부모 컴포넌트가 언마운트되면, 그 자식들도 언마운트됩니다.
