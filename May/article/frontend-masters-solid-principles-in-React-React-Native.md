# [프론트엔드 마스터: React / React Native에서의 SOLID 원칙](https://blog.stackademic.com/react-native-masters-solid-principles-in-react-react-native-a1b8df8d261d)

### 🗓️ 번역 날짜: 2024.05.27

### 🧚 번역한 크루: 렛서(김다은)

---

> 소프트웨어 개발자라면 면접에서 "**솔리드 원칙에 대해 들어본 적이 있나요?**"라는 질문을 받을 수 있습니다. 이미 이런 질문을 받은 적이 있을 수도 있습니다. 여러분은 분명 SOLID 원칙에 대해 들어보았고 사용한 적도 있을테지만, 지금 당장 예시를 들어 설명할 수는 없을 것입니다. 이제 앞으로의 면접에서 잊지 않도록 SOLID가 무엇인지, 그리고 이 원칙을 React/React Native 프로젝트에서 어떻게 적용할 수 있는지 예시를 통해 알아봅시다.😎
>
> **시작하기 전에: 글이 길어질 테니 지금 커피를 드세요 :) ☕️**

![](https://miro.medium.com/v2/resize:fit:400/format:webp/1*Z3dOY0ZYYwdAF8InAKKX7A.gif)

[SOLID](https://en.wikipedia.org/wiki/SOLID)는 소프트웨어 설계에서 중요한 5가지 규칙입니다. 이러한 규칙은 코드를 **이해하기 쉽고**,**유연하며**, **유지 관리하기 쉽게** 만드는 데 도움이 됩니다.

![SOLID](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Kt_kIt2U-AuDbXK578LTDg.png)

<br/>

### 1. 단일 책임 원칙 (Single Responsibility Principle, SRP)

> 클래스는 명확하게 정의된 하나의 목적을 위해 사용되어야 하며, 잦은 변경의 필요성을 줄여야 합니다.

애플리케이션의 각 컴포넌트가 구체적이고 잘 정의된 목적을 가지고 있는지 확인함으로써 React/React Native 프로젝트에서 단일 책임 원칙(SRP)을 지킬 수 있습니다.
예를 들어, 컴포넌트는 *특정 부분을 표시*하거나, *사용자 입력을 처리*하거나, *데이터를 가져오기 위한 API 호출*을 담당할 수 있습니다.
컴포넌트를 잘 정의된 단일 작업으로 제한하면 코드베이스의 명확성과 유지보수성을 향상시킬 수 있습니다.

다음은 React 애플리케이션에서 단일 책임 원칙(SRP)을 구현하기 위한 몇 가지 지침입니다:

1. **컴포넌트를 작게 만들고 한 가지 일만 하세요**: 컴포넌트를 작고 집중적으로 유지하여 각 컴포넌트에 명확하게 정의된 단일 책임을 할당하세요.
2. **서로 다른 작업을 혼합하지 마세요**: 관련 없는 작업을 하나의 컴포넌트에 묶지 마세요. 예를 들어 폼 표시를 담당하는 컴포넌트가 목록 데이터를 가져오는 API 호출도 처리해서는 안 됩니다.
3. **컴포지션 사용**: 작은 컴포넌트를 결합하여 재사용 가능한 UI 컴포넌트를 만드세요. 이렇게 하면 개발자는 복잡한 UI를 더 작고 관리하기 쉬운 부분으로 나누고 애플리케이션의 다른 부분에서 쉽게 재사용할 수 있습니다.
4. **프로퍼티와 상태를 현명하게 처리하세요**: 프로퍼티는 데이터와 동작을 하위 컴포넌트에 전달하는 메신저와 같습니다. 반면에 상태는 컴포넌트의 고유한 정보를 보관하는 개인 메모장과 같습니다. 한 가지에 국한되지 않는 정보에는 상태를 사용하세요.

이 개념을 설명하기 위해 **안티 패턴** 예시를 가져왔습니다. 아래의 코드 스니펫을 보세요. 이 코드의 역할은 목록 항목을 가져와서 표시하는 것입니다.

<br/>

```jsx
import React, { useEffect, useState } from 'react';
import { View, Text, Image } from 'react-native';
import axios from 'axios';

const ListItems = () => {
  const [listItems, setListItems] = useState([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    axios
      .get('https://.../listitems/')
      .then((res) => {
        setListItems(res.data);
      })
      .catch((e) => {
        errorToast(e.message);
      })
      .finally(() => {
        setIsLoading(false);
      });
  }, []);

  if (isLoading) {
    return <Text>Loading...</Text>;
  }

  return (
    <View>
      {listItems.map((item) => {
        return (
          <View>
            <Image src={{ uri: item.img }} />
            <Text>{item.name}</Text>
            <Text>{item.description}</Text>
          </View>
        );
      })}
    </View>
  );
};

export default ListItems;
```

코드를 훑어 보면, 꽤나 잘 쓰여지고 목적에 맞게 잘 작성된 것 같습니다. 하지만 컴포넌트 정의를 자세히 살펴보면, 몇 가지 '**문제**'가 있다는 걸 알아차릴 수 있습니다.
ListItems 컴포넌트는 여러 역할을 하고 있습니다:

1. State 관리
2. List 항목을 fetch하고 처리하는 과정
3. 컴포넌트 렌더링

처음에는 일반적인 구성 요소처럼 보였지만, 단일 책임 원칙(SRP)에 따르면 이 구성 요소는 예상보다 더 많은 책임을 처리하고 있습니다.

어떻게 개선할 수 있을까요?

일반적으로 컴포넌트 내부에 `useEffect`가 있는 경우, 해당 액션을 처리하는 커스텀 훅을 생성하여 `useEffect`를 컴포넌트 외부에 유지할 수 있습니다.

이 경우 사용자 정의 훅을 사용하여 새 파일을 만듭니다. 이 훅은 목록 데이터를 가져오고 상태를 관리하는 로직을 담당합니다.

<br/>

```js
import React, { useEffect, useState } from 'react';
import axios from 'axios';

const useFetchListItems = () => {
  const [listItems, setListItems] = useState([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    axios
      .get('https://.../listitems/')
      .then((res) => {
        setListItems(res.data);
      })
      .catch((e) => {
        errorToast(e.message);
      })
      .finally(() => {
        setIsLoading(false);
      });
  }, []);

  return { listItems, isLoading };
};
```

<br/>

State 관리와 List 항목을 fetcing하는 작업을 분리함으로서, 우리의 컴포넌트는 확실히 단순해지고 이해하기 쉬워졌습니다.
`ListItems` 컴포넌트는 이제 정보를 화면에 보여주기만 하면 돼, 유지 보수성과 확장성이 향상되었습니다.

리팩터링된 SRP 컴포넌트는 다음과 같습니다:

<br/>

```jsx
import { useFetchListItems } from 'hooks/useFetchListItems';

const ListItems = () => {
  const { listItems, isLoading } = useFetchListItems();

  if (isLoading) {
    return <Text>Loading...</Text>;
  }

  return (
    <View>
      {listItems.map((item) => {
        return (
          <View>
            <Image src={{ uri: item.img }} />
            <Text>{item.name}</Text>
            <Text>{item.description}</Text>
          </View>
        );
      })}
    </View>
  );
};

export default ListItems;
```

<br/>

하지만 새 훅을 자세히 살펴보면 몇 가지 기능을 수행한다는 것을 알 수 있습니다. 상태 관리와 리스트 항목 가져오기를 모두 처리합니다. 따라서 한 가지 작업만 수행한다는 단일 책임 원칙을 따르지 않습니다.

이것을 개선하기 위해, 우리는 리스트 항목을 가져오는 로직을 분리할 수 있습니다. 간단히 말하면, `api.js`라는 새 파일을 만들어 리스트 항목을 fetching을 하는 책임을 이동시킵니다.

<br/>

```js
import axios from 'axios';
import errorToast from './errorToast';

const fetchListItems = () => {
  return axios
    .get('https://.../listitems/')
    .catch((e) => {
      errorToast(e.message);
    })
    .then((res) => res.data);
};
```

<br/>

🎉 리팩토링된 새로운 커스텀 훅은 다음과 같습니다:

<br/>

```js
import { useEffect, useState } from 'react';
import { fethListItems } from './api';

const useFetchListItems = () => {
  const [listItems, setListItems] = useState([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    fetchListItems()
      .then((listItems) => setListItems(listItems))
      .finally(() => setIsLoading(false));
  }, []);

  return { listItems, isLoading };
};
```

<br/>

이 예제에서는 각 파일이 한 가지 일을 하도록 하여 구조를 조금 더 복잡하게 만들었지만 단일 책임 원칙에 부합하도록 했습니다.
하지만 SRP를 엄격하게 준수하는 것이 항상 최선은 아니라는 점을 명심하세요. 때로는 코드를 지나치게 복잡하게 만드는 것보다 약간 복잡하게 만드는 것도 괜찮을 때가 있습니다.

SRP를 엄격하게 따를 필요가 없는 상황도 있습니다:

1. **Form components**: 폼은 데이터 확인, 상태 관리, 정보 업데이트 등 다양한 작업을 수행합니다.다른 도구나 라이브러리를 사용하는 경우 특히 이러한 작업을 분할하면 작업이 지저분해질 수 있습니다.
2. **Table components**: 테이블도 데이터 표시나 사용자 상호작용 관리 등 복잡한 작업을 처리합니다. 이를 별도의 작업으로 나누면 코드가 더 복잡해질 수 있습니다.

<br/>

---

### 2. 개방-폐쇄 원칙(Open-closed principle, OCP)

> 소프트웨어 엔티티(클래스, 모듈, 기능 등)은 _**확장엔 열려 있되 수정에는 닫혀 있어야 한다**_.

- **확장엔 열려 있다**: 이미 작동하는 방식을 변경하지 않고도 컴포넌트에 새로운 동작, 기능 및 특징을 추가할 수 있어야 함을 의미합니다.
- **수정엔 닫혀 있다**: React 컴포넌트를 생성하고 구현한 이후엔 불가피한 경우가 아니라면 소스 코드를 직접 조작하지 않아야 합니다.

기본적인 React Native 컴포넌트를 함께 살펴 봅시다.

<br/>

```jsx
import React from 'react';

interface IListItem {
  title: string;
  image: string;
  isAuth: boolean;
  onClickMember: () => void;
  onClickGuest: () => void;
}

const ListItem: React.FC<ILıstItem> = ({ title, image, isAuth, onClickMember, onClickGuest }: IListItem) => {
  const handleMember = () => {
    // Some logic
    onClickMember();
  };

  const handleGuest = () => {
    // Some logic
    onClickGuest();
  };
  return (
    <View>
      <Image source={{uri:image}} />
      <Text>{title}</Text>
      {
      isAuth ?
      <Button onClick={handleMember}>Add to cart +</Button>
      :
      <Button onClick={handleGuest}>Show Modal</button>
      }
    </View>
  );
};
```

보시다시피 위의 코드는 인증 상태에 따라 다른 기능을 렌더링하여 원칙을 위반합니다.

다른 로직으로 다른 버튼을 렌더링하려면 이 코드 블록을 수정하는 것이 좋습니다:

<br/>

```jsx
interface IButtonHandler {
  handle(): void;
}

export const GuestButtonHandler: React.FC<{ onClickGuest?: () => void }> = ({
  onClickGuest,
}) => {
  const handle = () => {
    // Some logic for guests
    onClickGuest();
  };

  return <Button onClick={handle}>Show Modal</Button>;
};

export const MemberButtonHandler: React.FC<{ onClickMember?: () => void }> = ({
  onClickMember,
}) => {
  const handle = () => {
    // Some logic for members
    onClickMember();
  };

  return <Button onClick={handle}>Add to cart +</Button>;
};
```

```jsx
import React from 'react';

interface IListItem {
  title: string;
  image: string;
}

export const ListItem: React.FC<IListItem> = ({ title, image, children }) => {
  return (
    <View>
      <Image source={{ uri: image }} />
      <Text>{title}</Text>
      {children}
    </View>
  );
};

export default ListItem;
```

<br/>

마지막으로 불필요한 코드를 제거하고 `children`이라는 새로운 프로퍼티를 만들어 다른 컴포넌트가 자식 컴포넌트를 전달함으로써 컴포넌트를 확장할 수 있도록 합니다. :

<br/>

```jsx
import { ListItem } from '../index.tsx';
import { GuestButtonHandler, MemberButtonHandler } from '../handlers';

const App = () => {
  return (
    <ListItem title={item.title} image={item.image}>
      isAuth ? <MemberButtonHandler /> : <GuestButtonHandler />
    </ListItem>
  );
};

export default App;
```

<br/>

🎉 이제 ListItem 컴포넌트는 확장엔 열려 있고 수정엔 닫혀 있습니다.
이 방법은 여러 가지 버전을 보여주기 위해 많은 프로퍼티가 필요하지 않은 분리된 컴포넌트를 이용하기 때문에 더욱 효과적입니다.
우리는 그냥 필요한 곳에 알맞은 부분을 보여주면 됩니다.
또한, 컴포넌트가 내부적으로 많은 일을 수행하면 단일 책임 원칙을 위반할 수 있습니다.
따라서 이 방법은 코드를 체계적이고 명확하게 유지하는 데 도움이 된다.

<br/>

---

### 3. 리스코프 치환 원칙 (Liskov Subsitution Principle, LSP)

> _슈퍼 클래스의 객체는 서브 클래스의 객체로 대체될 수 있어야 합니다_.

즉, 특정 클래스의 서브 클래스(자식 클래스)는 어떠한 기능도 손상시키지 않고 슈퍼 클래스(부모 클래스)를 대체할 수 있어야 합니다.

예시:

`RacingCar`가 `Car`의 서브 클래스인 경우, 아무 문제 없이 `Car`의 인스턴스를 `RacingCar`로 대체할 수 있어야 합니다. 즉, `RacingCar`은 `Car` 클래스의 모든 기대를 충족해야 합니다.

<br/>

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*XXWO3JVZmJf6V2prvAO0Sg.png)

<br/>

리액트에서 리스코프 치환 원칙(LSP)는 부모 컴포넌트가 하는 일을 자식 컴포넌트가 대체 가능해야 한다는 것입니다. 컴포넌트가 다른 컴포넌트를 사용하더라도 이전처럼 계속 작동해야 합니다.

코드를 통해 살펴보겠습니다:

<br/>

```jsx
const SuccessButton = () => {
  return <Text>Success</Text>;
};
```

<br/>

우리가 생성한 `SuccessButton` 컴포넌트의 기능은 `Text`에 의해 대체될 수 없습니다. 대신, 우리는 button을 이렇게 작성해야 합니다:

```jsx
const SuccessButton = () => {
  return (
    <TouchableOpacity style={styles.button} onPress={onPress}>
      <Text>Success</Text>
    </TouchableOpacity>
  );
};
```

<br/>

이 코드는 한결 낫지만 충분하진 않습니다. 우리는 버튼 자체의 모든 기능을 상속해야 합니다.:

<br/>

```jsx
const SuccessButton = () => {
  return (
    <TouchableOpacity style={styles.button} onPress={onPress} {…props}>
      <Text>Success</Text>
    </TouchableOpacity>
  );
};
```

<br/>

🎉 이제 우린 버튼의 모든 속성을 상속받았고 새 버튼에 속성을 전달합니다. 또한 프로그램의 동작을 변경하지 않고 리스코프 대체 원칙에 따라 버튼의 인스턴스 대신 성공 버튼의 인스턴스를 계속 사용할 수 있습니다.

<br/>

### 4. 인터페이스 분리 원칙 (Interface Segregation Principle, ISP)

> _“어떤 코드도 사용하지 않는 메서드에 의존하도록 강요해서는 안 된다”_. React 애플리케이션의 경우 “컴포넌트는 사용하지 않는 프로퍼티에 의존해서는 안 된다”로 바꾸어 표현하겠습니다.

<br/>

![](https://miro.medium.com/v2/resize:fit:960/format:webp/1*r3WsuDZfzcXrtq0n9NLL-A.gif)

<br/>

더 잘 이해하기 위해 예시를 살펴보겠습니다:

<br/>

```jsx
const ListItem = ({ item }) => {
  return (
    <View>
      <Image source={{ uri: item.image }} />
      <Text>{item.title}</Text>
      <Text>{item.description}</Text>
    </View>
  );
};
```

<br/>

이미지, 제목, 설명이라는 데이터가 필요한 ListItem 컴포넌트가 있습니다. ListItem을 프로퍼티로 제공하면 컴포넌트에 실제로 필요한 것보다 더 많은 데이터를 제공하게 됩니다.

이러한 문제를 해결하기 위해 컴포넌트가 필요한 프로퍼티만 받는 것이 필요합니다.

<br/>

```jsx
const ListItem = ({ image, title, description }) => {
  return (
    <View>
      <Image source={{ uri: image }} />
      <Text>{title}</Text>
      <Text>{description}</Text>
    </View>
  );
};
```

<br/>

---

### 5. 의존성 역전 원리(Dependency Inversion Principle, DIP)

> 1. _상위 모듈은 하위 모듈에서 아무것도 가져오지 않아야 합니다. 둘 다 추상화(예: 인터페이스)에 의존해야 합니다._
>
> 2. _추상화는 세부 사항에 의존해서는 안 됩니다. 세부 사항(구체적인 구현)은 추상화에 의존해야 합니다._

React의 맥락에서 이 원칙은 상위 컴포넌트가 하위 컴포넌트에 직접적으로 의존하지 않으며, 대신 공통 추상화(common abstraction)에 의존하도록 합니다.
이 경우 '컴포넌트'란 React 컴포넌트, 함수, 모듈, 클래스형 컴포넌트 또는 서드파티 라이브러리 등 애플리케이션의 모든 부분을 가리킵니다. 예시를 통해 살펴봅시다.:

<br/>

```jsx
const CreateListItemForm = () => {
  const handleCreateListItemForm = async (e) => {
    try {
      const formData = new FormData(e.currentTarget);
      await axios.post('https://myapi.com/listItems', formData);
    } catch (err) {
      console.error(err.message);
    }
  };

  return (
    <form onSubmit={handleCreateListItemForm}>
      <input name="title" />
      <input name="description" />
      <input name="image" />
    </form>
  );
};
```

위의 컴포넌트는 폼을 렌더링하고 제출된 데이터를 API로 전송해 리스트 아이템 생성을 처리합니다.

이 시나리오를 고려해 보세요. 동일한 UI로 리스트 아이템을 편집하는 로직(이 예제에서는 API 엔드포인트)만 다른 form이 있습니다. 현재 form은 엔드 포인트를 수정할 수 없으므로 재사용할 수 없습니다. 따라서, 우리는 특정 하위 모듈에 종속되지 않는 컴포넌트를 만들어야 합니다.

<br/>

```jsx
const ListItemForm = ({ onSubmit }) => {
  return (
    <form onSubmit={onSubmit}>
      <input name="title" />
      <input name="description" />
      <input name="image" />
    </form>
  );
};
```

<br/>

양식에서 종속성을 제거했으므로 이제 프로퍼티를 통해 필요한 로직을 제공할 수 있습니다.

<br/>

```jsx
const CreateListItemForm = () => {
  const handleCreateListItem = async (e) => {
    try {
      const formData = new FormData(e.currentTarget);
      await axios.post('https://myapi.com/listItems', formData);
    } catch (err) {
      console.error(err.message);
    }
  };
  return <ListItemForm onSubmit={handleCreateListItem} />;
};
```

```jsx
const EditListItemForm = () => {
  const handleEditListItem = async (e) => {
    try {
      // Editing logic
    } catch (err) {
      console.error(err.message);
    }
  };
  return <ListItemForm onSubmit={handleEditListItem} />;
};
```

DIP를 적용하고 간소화하면 의도치 않게 다른 컴포넌트에 영향을 미칠 염려 없이 각 컴포넌트를 개별적으로 테스트할 수 있습니다.

<br/>

---

<br/>

> 지금까지 모든 원칙을 예시를 들어 설명하려 노력했습니다. 기억하기 어려운 이 원칙들을 잘 익히셨기를 바랍니다. 이번에는 외울 필요도 없으시길 바랍니다. 이 글이 유익하거나 도움이 되었다면 커피 한 잔을 사서 제 작업을 응원해 주세요. 여러분의 후원은 이와 같은 콘텐츠를 더 많이 만드는 데 도움이 됩니다. 가상 커피를 사려면 [여기를 클릭](https://buymeacoffee.com/ismailharmanda)하세요 ☕️. Happy hackings! 🚀

<br/>
