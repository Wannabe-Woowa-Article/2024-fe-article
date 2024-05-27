# [프론트엔드 마스터: React / React Native에서의 SOLID 원칙](https://blog.stackademic.com/react-native-masters-solid-principles-in-react-react-native-a1b8df8d261d)

### 🗓️ 번역 날짜: 2024.05.27

### 🧚 번역한 크루: 렛서(김다은)

---

> 소프트웨어 개발자라면 면접에서 "**솔리드 원칙에 대해 들어본 적이 있나요?**"라는 질문을 받을 수 있습니다. 이미 이런 질문을 받은 적이 있을 수도 있습니다. 여러분은 분명 SOLID 원칙에 대해 들어보았고 사용한 적도 있을테지만, 지금 당장 예시를 들어 설명할 수는 없을 것입니다. 이제  앞으로의 면접에서 잊지 않도록 SOLID가 무엇인지, 그리고 이 원칙을 React/React Native 프로젝트에서 어떻게 적용할 수 있는지 예시를 통해 알아봅시다.😎
> 
> **시작하기 전에: 글이 길어질 테니 지금 커피를 드세요 :) ☕️**

![](https://miro.medium.com/v2/resize:fit:400/format:webp/1*Z3dOY0ZYYwdAF8InAKKX7A.gif)

[SOLID](https://en.wikipedia.org/wiki/SOLID)는 소프트웨어 설계에서 중요한 5가지 규칙입니다. 이러한 규칙은 코드를 **이해하기 쉽고**,**유연하며**, **유지 관리하기 쉽게** 만드는 데 도움이 됩니다. 

![SOLID](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Kt_kIt2U-AuDbXK578LTDg.png)

<br/>

### 1. 단일 책임 원칙 (SRP)
> 클래스는 명확하게 정의된 하나의 목적을 위해 사용되어야 하며, 잦은 변경의 필요성을 줄여야 합니다.

애플리케이션의 각 컴포넌트가 구체적이고 잘 정의된 목적을 가지고 있는지 확인함으로써 React/React Native 프로젝트에서 단일 책임 원칙(SRP)을 지킬 수 있습니다.
예를 들어, 컴포넌트는 _특정 부분을 표시_하거나, _사용자 입력을 처리_하거나, _데이터를 가져오기 위한 API 호출_을 담당할 수 있습니다. 
컴포넌트를 잘 정의된 단일 작업으로 제한하면 코드베이스의 명확성과 유지보수성을 향상시킬 수 있습니다.

다음은 React 애플리케이션에서 단일 책임 원칙(SRP)을 구현하기 위한 몇 가지 지침입니다:

1. **컴포넌트를 작게 만들고 한 가지 일만 하세요**: 컴포넌트를 작고 집중적으로 유지하여 각 컴포넌트에 명확하게 정의된 단일 책임을 할당하세요.
2. **서로 다른 작업을 혼합하지 마세요**: 관련 없는 작업을 하나의 컴포넌트에 묶지 마세요. 예를 들어 폼 표시를 담당하는 컴포넌트가 목록 데이터를 가져오는 API 호출도 처리해서는 안 됩니다.
3. **컴포지션 사용**: 작은 컴포넌트를 결합하여 재사용 가능한 UI 컴포넌트를 만드세요. 이렇게 하면 개발자는 복잡한 UI를 더 작고 관리하기 쉬운 부분으로 나누고 애플리케이션의 다른 부분에서 쉽게 재사용할 수 있습니다.
4. **프로퍼티와 상태를 현명하게 처리하세요**: 프로퍼티는 데이터와 동작을 하위 컴포넌트에 전달하는 메신저와 같습니다. 반면에 상태는 컴포넌트의 고유한 정보를 보관하는 개인 메모장과 같습니다. 한 가지에 국한되지 않는 정보에는 상태를 사용하세요.

이 개념을 설명하기 위해 **안티 패턴** 예시를 가져왔습니다. 아래의 코드 스니펫을 보세요. 이 코드의 역할은 목록 항목을 가져와서 표시하는 것입니다.

<br/>

```jsx
import React, { useEffect, useState } from "react";
import { View, Text, Image } from "react-native";
import axios from "axios";


const ListItems = () => {
 const [listItems, setListItems] = useState([]);
 const [isLoading, setIsLoading] = useState(true);


 useEffect(() => {
   axios.get("https://.../listitems/").then((res) => {
       setListItems(res.data)
   }).catch(e => {
       errorToast(e.message);
   }).finally(() => {
       setIsLoading(false);
   })
 }, [])


 if (isLoading){
   return <Text>Loading...</Text>
 }


 return (
   <View>
       {listItems.map(item => {
           return (
               <View>
                    <Image src={{uri:item.img}} />
                    <Text>{item.name}</Text>
                    <Text>{item.description}</Text>
               </View>
           )
       })}
   </View>
 );
};


export default ListItems;
```

코드를 훑어 보면, 꽤나 잘 쓰여지고 목적에 맞게 잘 작성된 것 같습니다. 하지만 컴포넌트 정의를 자세히 살펴보면, 몇 가지 **'문제'**가 있다는 걸 알아차릴 수 있습니다.
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
import React, { useEffect, useState } from "react";
import axios from "axios";


const useFetchListItems = () => {
   const [listItems, setListItems] = useState([]);
   const [isLoading, setIsLoading] = useState(true);

useEffect(() => {
   axios.get("https://.../listitems/").then((res) => {
       setListItems(res.data)
   }).catch(e => {
       errorToast(e.message);
   }).finally(() => {
       setIsLoading(false);
   })
 }, [])

   return { listItems, isLoading };
}
```

<br/>

State 관리와 List 항목을 fetcing하는 작업을 분리함으로서, 우리의 컴포넌트는 확실히 단순해지고 이해하기 쉬워졌습니다. 
`ListItems` 컴포넌트는 이제 정보를 화면에 보여주기만 하면 돼, 유지 보수성과 확장성이 향상되었습니다.

리팩터링된 SRP 컴포넌트는 다음과 같습니다:

<br/>

```jsx
import { useFetchListItems } from "hooks/useFetchListItems";


const ListItems = () => {
 const { listItems, isLoading } = useFetchListItems();


if (isLoading){
   return <Text>Loading...</Text>
 }

 return (
   <View>
       {listItems.map(item => {
           return (
               <View>
                    <Image src={{uri:item.img}} />
                    <Text>{item.name}</Text>
                    <Text>{item.description}</Text>
               </View>
           )
       })}
   </View>
 );
};


export default ListItems;
```

<br/>

하지만 새 훅을 자세히 살펴보면 몇 가지 기능을 수행한다는 것을 알 수 있습니다. 상태 관리와 리스트 항목 가져오기를 모두 처리합니다. 따라서  한 가지 작업만 수행한다는 단일 책임 원칙을 따르지 않습니다.

이것을 개선하기 위해, 우리는 리스트 항목을 가져오는 로직을 분리할 수 있습니다. 간단히 말하면, `api.js`라는 새 파일을 만들어 리스트 항목을 fetching을 하는 책임을 이동시킵니다.

<br/>

```js
import axios from "axios";
import errorToast from "./errorToast";


const fetchListItems = () => {
 return axios
   .get("https://.../listitems/")
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
import { useEffect, useState } from "react";
import { fethListItems } from "./api";


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
