# [4 React Tips to Instantly Improve Your Code](https://javascript.plainenglish.io/4-react-tips-to-instantly-improve-your-code-7456e028cfa3)

### 🗓️ 번역 날짜: 2024.05.31

### 🧚 번역한 크루: 마스터위(명재위)

---

Pavel Pogosov   
Updated On Feb 2, 2023   
Keyword : **React**   

React에 대한 탄탄한 지식은 프론트엔드 개발자에게 가장 가치 있는 기술 중 하나입니다. 많은 기업들이 지속적으로 React 개발자를 찾고 있으며, 그들에게 더 많은 보수를 지불하려고 합니다. 그렇기 때문에 개발자로서 지속적으로 성장하는 것은 매우 보람 있는 일입니다.

여러분의 여정에 도움이 되기 위해, 제가 더 나은 React 코드를 작성하는 데 도움이 되었던 네 가지 팁을 공유하고자 합니다. 새로운 것을 배우고 유용하게 사용하시길 바랍니다. 그럼 바로 시작해 봅시다!

### 목차

- 핸들러에서 함수 반환하기
- 책임 분리
- 조건문 대신 객체 맵 사용하기
- React lifecycle의 외부에 독립 변수 두기

### 핸들러에서 함수 반환하기

함수형 프로그래밍에 익숙하다면 제가 무슨 말을 하는지 아실 겁니다. 우리는 이를 "[커링](https://en.wikipedia.org/wiki/Currying)"이라고 부를 수 있습니다. 본질적으로, 일부 함수 매개변수를 미리 설정하는 것입니다.

아래 보일러플레이트 코드에서 명시적인 문제가 있습니다. 이 기술이 도움이 될 것입니다!

```tsx
export default function App() {
  const [user, setUser] = useState({
    name: "",
    surname: "",
    address: "",
  });

  // 첫 핸들러
  const handleNameChange = (e) => {
    setUser((prev) => ({
      ...prev,
      name: e.target.value,
    }));
  };

  // 두번째 핸들러!
  const handleSurnameChange = (e) => {
    setUser((prev) => ({
      ...prev,
      surname: e.target.value,
    }));
  };

  // 세번째 핸들러!!!
  const handleAddressChange = (e) => {
    setUser((prev) => ({
      ...prev,
      address: e.target.value,
    }));
  };

  // 만약 하나의 input이 더 필요하다면 어떨까요? 이를 위한 또 다른 핸들러를 만들어야할까요?

  return (
    <>
      <input value={user.name} onChange={handleNameChange} />
      <input value={user.surname} onChange={handleSurnameChange} />
      <input value={user.address} onChange={handleAddressChange} />
    </>
  );
}
```

**Solution**

```tsx
export default function App() {
  const [user, setUser] = useState({
    name: "",
    surname: "",
    address: "",
  });

  const handleInputChange = (field) => {
    return (e) => {
      setUser((prev) => ({
        ...prev,
        [field]: e.target.value,
      }));
    };
  };

  return (
    <>
      <input value={user.name} onChange={handleInputChange("name")} />
      <input value={user.surname} onChange={handleInputChange("surname")} />
      <input value={user.address} onChange={handleInputChange("address")} />

      {JSON.stringify(user)}
    </>
  );
}
```

https://medium.com/@pashkapag/5-react-usestate-mistakes-that-will-get-you-fired-b342289debfe

### 책임 분리

"God" 컴포넌트를 만드는 것은 개발자들이 흔히 저지르는 실수입니다. "God" 컴포넌트라고 불리는 이유는 이해하고 유지 보수하기 어려운 많은 코드 라인을 포함하고 있기 때문입니다. 독립적인 하위 모듈 세트로 컴포넌트를 나누는 것을 강력히 권장합니다.

일반적인 구조는 다음과 같습니다:

- **UI 모듈**: 시각적 표현만 담당합니다.
- **로직/모델 모듈**: 비즈니스 로직만 포함합니다. 예를 들어, 커스텀 훅이 여기에 해당합니다.
- **Lib 모듈**: 컴포넌트에 필요한 모든 유틸리티를 포함합니다.

이 개념을 설명하기 위한 작은 데모 예제를 보여드리겠습니다.

```tsx
export function ListComponent() {
  // Our local state
  const [list, setList] = useState([]);

  // 서버로부터 데이터를 로드하기위한 핸들러
  const fetchList = async () => {
    try {
      const resp = await fetch("https://www.url.com/list");
      const data = await resp.json();

      setList(data);
    } catch {
      showAlert({ text: "Something went wrong!" });
    }
  };

  // 마운트될 때만 fetch하길 원하기에
  useEffect(() => {
    fetchList();
  }, []);

  // 아이템을 지우는 역할의 핸들러
  const handleDeleteItem = (id) => {
    return () => {
      try {
        fetch(`https://www.url.com/list/${id}`, {
          method: "DELETE",
        });
        setList((prev) => prev.filter((x) => x.id !== id));
      } catch {
        showAlert({ text: "Something went wrong!" });
      }
    };
  };

  // 여기서 데이터 아이템을 render한다
  return (
    <div className="list-component">
      {list.map(({ id, name }) => (
        <div key={id} className="list-component__item>">
          {/* We want to trim long name with ellipsis */}
          {name.slice(0, 30) + (name.length > 30 ? "..." : "")}

          <div onClick={handleDeleteItem(id)} className="list-component__icon">
            <DeleteIcon />
          </div>
        </div>
      ))}
    </div>
  );
}
```

우리는 모델과 UI 모듈에서 사용할 유틸리티를 작성하는 것부터 시작해야 합니다.

```tsx
export async function getList(onSuccess) {
  try {
    const resp = await fetch("https://www.url.com/list");
    const data = await resp.json();

    onSuccess(data);
  } catch {
    showAlert({ text: "Something went wrong!" });
  }
}

export async function deleteListItem(id, onSuccess) {
  try {
    fetch(`https://www.url.com/list/${id}`, {
      method: "DELETE",
    });
    onSuccess();
  } catch {
    showAlert({ text: "Something went wrong!" });
  }
}

export function trimName(name) {
  return name.slice(0, 30) + (name.lenght > 30 ? "..." : "");
}
```

이제 비즈니스 로직을 구현해야 합니다. 이를 위해 필요한 것들을 반환하는 커스텀 훅을 작성하면 됩니다.

```tsx
export function useList() {
  const [list, setList] = useState([]);

  const handleDeleteItem = useCallback((id) => {
    return () => {
      deleteListItem(id, () => {
        setList((prev) => prev.filter((x) => x.id !== id));
      });
    };
  }, []);

  useEffect(() => {
    getList(setList);
  }, []);

  return useMemo(
    () => ({
      list,
      handleDeleteItem,
    }),
    [list, handleDeleteItem]
  );
}
```

마지막 단계는 UI 모듈을 작성한 후 모든 것을 함께 결합하는 것입니다.

```tsx
export function ListComponentItem({ name, onDelete }) {
  return (
    <div className="list-component__item>">
      {trimName(name)}

      <div onClick={onDelete} className="list-component__icon">
        <DeleteIcon />
      </div>
    </div>
  );
}

export function ListComponent() {
  const { list, handleDeleteItem } = useList();

  return (
    <div className="list-component">
      {list.map(({ id, name }) => (
        <ListComponentItem
          key={id}
          name={name}
          onDelete={handleDeleteItem(id)}
        />
      ))}
    </div>
  );
}
```

### 조건문 대신 객체 맵 사용하기

변수에 따라 다양한 요소를 표시해야 할 경우, 이 팁을 구현할 수 있습니다. 이 간단한 전략을 사용하면 컴포넌트를 더 선언적으로 만들고 코드 이해를 단순화할 수 있습니다. 게다가 기능을 확장하는 것도 더 쉬워집니다.

**Problem**

```tsx
function Account({ type }) {
  let Component = UsualAccount;

  if (type === "vip") {
    Component = VipAccount;
  }

  if (type === "moderator") {
    Component = ModeratorAccount;
  }

  if (type === "admin") {
    Component = AdminAccount;
  }

  return (
    <div className="account">
      <Component />
      <AccountStatistics />
    </div>
  );
}
```

**Solution**

```tsx
const ACCOUNTS_MAP = {
  vip: VipAccount,
  usual: UsualAccount,
  admin: AdminAccount,
  moderator: ModeratorAccount,
};

function Account({ type }) {
  const Component = ACCOUNTS_MAP[type];

  return (
    <div className="account">
      <Component />
      <AccountStatistics />
    </div>
  );
}
```

[링크](http://javascript.plainenglish.io/frontend-architectures-classic-approach-no-architecture-d3c839e46403?source=post_page-----7456e028cfa3--------------------------------)

### React lifecycle의 외부에 독립 변수 두기

아이디어는 React 컴포넌트 생명주기 메서드를 필요로 하지 않는 로직을 컴포넌트 자체에서 분리하는 것입니다. 이렇게 하면 종속성을 더 명확하게 만들어 코드의 명확성을 향상시킵니다. 따라서 컴포넌트를 읽고 이해하는 것이 훨씬 쉬워집니다.

**Problem**

```tsx
function useItemsList() {
  const defaultItems = [1, 2, 3, 4, 5];
  const [items, setItems] = useState(defaultItems);

  const toggleArrayItem = (arr, val) => {
    return arr.includes(val) ? arr.filter((el) => el !== val) : [...arr, val];
  };

  const handleToggleItem = (num) => {
    return () => {
      setItems(toggleArrayItem(items, num));
    };
  };

  return {
    items,
    handleToggleItem,
  };
}
```

**Solution**

```tsx
const DEFAULT_ITEMS = [1, 2, 3, 4, 5];

const toggleArrayItem = (arr, val) => {
  return arr.includes(val) ? arr.filter((el) => el !== val) : [...arr, val];
};

function useItemsList() {
  const [items, setItems] = useState(DEFAULT_ITEMS);

  const handleToggleItem = (num) => {
    return () => {
      setItems(toggleArrayItem(items, num));
    };
  };

  return {
    items,
    handleToggleItem,
  };
}
```

**Thanks for reading!**
