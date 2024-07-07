## 🔗 [4 useState Mistakes You Should Avoid in React🚫](https://medium.com/gitconnected/4-usestate-mistakes-you-should-avoid-in-react-0d9d676869e2)

### 🗓️ 번역 날짜: 2024.06.10

### 🧚 번역한 크루: 소하(최소연)

---

## React에서 피해야 할 4가지 `useState` 실수🚫

#### 소개

리액트(React.js)는 컴포넌트 내 상태 관리에 대한 독특한 접근 방식으로 현대 웹 개발의 초석이 되었습니다. 일반적인 훅 중 하나인 useState는 기본적이지만 종종 잘못 사용됩니다. 효율적이고 버그 없는 애플리케이션을 만들고자 하는 초보자와 경험이 풍부한 개발자 모두에게 이러한 일반적인 실수를 이해하고 피하는 것은 매우 중요합니다.

이 블로그에서는 리액트에서 useState를 사용할 때 피해야 할 네 가지 중요한 실수를 자세히 살펴보겠습니다. 함께 리액트 기술을 향상시켜 봅시다!

본격적으로 살펴보기 전에 [제 개인 웹사이트](https://programwithjayanth.com/?source=post_page-----0d9d676869e2--------------------------------)에서 웹 개발에 대한 보다 심층적인 기사를 살펴보세요:

#### 실수 1: 이전 상태를 고려하지 않음 😨

React의 useState 훅을 사용할 때 가장 최근의 상태를 업데이트할 때 이전 상태를 고려하지 않는 것은 흔한 실수입니다. 이러한 실수는 특히 빠른 또는 여러 상태 업데이트를 처리할 때 예상치 못한 동작을 초래할 수 있습니다.

#### ❌ 문제 이해

React에서 카운터를 만든다고 가정해 봅시다. 버튼을 클릭할 때마다 카운트를 증가시키는 것이 목표입니다. 간단한 접근 방법은 현재 상태 값에 1을 더하는 것일 수 있습니다. 그러나 이것은 문제가 될 수 있습니다.

```jsx
import React, { useState } from 'react';

const CounterComponent = () => {
	const [counter, setCounter] = useState(0);

	const incrementCounter = () => {
		setCounter(counter + 1); // 항상 예상대로 작동하지 않을 수 있습니다.
	};

	return (
		<div>
			<p>Counter: {counter}</p>
			<button onClick={incrementCounter}>Increment</button>
		</div>
	);
};

export default CounterComponent;
```

위의 코드에서 incrementCounter는 현재 값을 기반으로 카운터를 업데이트합니다. 이것은 간단해 보이지만 문제를 일으킬 수 있습니다. React는 여러 setCounter 호출을 함께 묶거나 다른 상태 업데이트가 간섭하여 카운터가 매번 올바르게 업데이트되지 않을 수 있습니다.

#### ✅ 수정:

이러한 문제를 피하기 위해 setCounter 메서드의 함수형을 사용하세요. 이 버전은 React가 가장 최근의 상태 값으로 호출하는 함수를 인수로 받습니다. 이렇게 하면 항상 최신 상태 값으로 작업할 수 있습니다.

```jsx
import React, { useState } from 'react';

const CounterComponent = () => {
	const [counter, setCounter] = useState(0);

	const incrementCounter = () => {
		setCounter((prevCounter) => prevCounter + 1); // 가장 최근의 상태에 따라 올바르게 업데이트됩니다.
	};

	return (
		<div>
			<p>Counter: {counter}</p>
			<button onClick={incrementCounter}>Increment</button>
		</div>
	);
};

export default CounterComponent;
```

수정된 코드에서는 incrementCounter가 상태를 업데이트하는 데 함수를 사용합니다. 이 함수는 가장 최근의 상태(prevCounter)를 받아 업데이트된 상태를 반환합니다. 이 접근 방식은 특히 업데이트가 빠르게 일어나거나 연속적으로 여러 번 발생할 때 훨씬 더 신뢰성이 높습니다.

React JS 실시간 교육에 관심이 있다면 자세한 내용은 저에게 문의해 주세요.

#### 실수 2: 상태 불변성 무시 🧊

#### ❌ 문제 이해

React에서는 상태를 불변으로 취급해야 합니다. 흔한 실수는 객체와 배열과 같은 복잡한 데이터 구조로 상태를 직접 수정하는 것입니다.

상태 객체에 대한 잘못된 접근 방식을 고려해 봅시다:

```jsx
import React, { useState } from 'react';

const ProfileComponent = () => {
	const [profile, setProfile] = useState({ name: 'John', age: 30 });

	const updateAge = () => {
		profile.age = 31; // 직접 상태 수정
		setProfile(profile);
	};

	return (
		<div>
			<p>Name: {profile.name}</p>
			<p>Age: {profile.age}</p>
			<button onClick={updateAge}>Update Age</button>
		</div>
	);
};

export default ProfileComponent;
```

이 코드는 profile 객체를 잘못 수정합니다. 이러한 수정은 재렌더링을 트리거하지 않으며 예측할 수 없는 동작을 초래합니다.

#### ✅ 수정:

상태를 업데이트할 때는 항상 불변성을 유지하기 위해 새 객체 또는 배열을 생성하세요. 이를 위해 스프레드 연산자를 사용하세요.

```jsx
import React, { useState } from 'react';

const ProfileComponent = () => {
	const [profile, setProfile] = useState({ name: 'John', age: 30 });

	const updateAge = () => {
		setProfile({ ...profile, age: 31 }); // 올바르게 상태 업데이트
	};

	return (
		<div>
			<p>Name: {profile.name}</p>
			<p>Age: {profile.age}</p>
			<button onClick={updateAge}>Update Age</button>
		</div>
	);
};

export default ProfileComponent;
```

수정된 코드에서 updateAge는 상태 불변성을 유지하면서 업데이트된 나이로 새 profile 객체를 생성하기 위해 스프레드 연산자를 사용합니다.

#### 실수 3: 비동기 업데이트 이해 부족 ⏳

#### ❌ 문제 이해

useState를 통한 React의 상태 업데이트는 비동기적입니다. 특히 여러 상태 업데이트가 빠른 연속으로 이루어질 때, 이는 종종 혼란을 초래합니다. 개발자는 setState 호출 직후 상태가 즉시 변경될 것으로 예상할 수 있지만, 실제로는 React가 성능상의 이유로 이러한 업데이트를 일괄 처리합니다.

이러한 오해가 문제를 일으킬 수 있는 일반적인 시나리오를 살펴봅시다.

```jsx
import React, { useState } from 'react';

const AsyncCounterComponent = () => {
	const [count, setCount] = useState(0);

	const incrementCount = () => {
		setCount(count + 1);
		setCount(count + 1);
		// 개발자는 count가 두 번 증가할 것을 예상
	};

	return (
		<div>
			<p>Count: {count}</p>
			<button onClick={incrementCount}>Increment Count</button>
		</div>
	);
};

export default AsyncCounterComponent;
```

이 예제에서 개발자는 count를 두 번 증가시키려고 합니다. 그러나 상태 업데이트의 비동기적 특성으로 인해 두 setCount 호출은 모두 동일한 초기 상태를 기반으로 하므로 count가 한 번만 증가합니다.

#### ✅ 수정:

비동기 업데이트를 올바르게 처리하려면 setCount의 함수형 업데이트 형식을 사용하세요. 이렇게 하면 각 업데이트가 가장 최근의 상태를 기반으로 수행됩니다.

```jsx
import React, { useState } from 'react';

const AsyncCounterComponent = () => {
	const [count, setCount] = useState(0);

	const incrementCount = () => {
		setCount((prevCount) => prevCount + 1);
		setCount((prevCount) => prevCount + 1);
		// 이제 각 업데이트는 가장 최근 상태에 올바르게 의존
	};
	// 선택사항: useEffect를 사용하여 업데이트 된 상태 확인
	useEffect(() => {
		console.log(count); // 2
	}, [count]);

	return (
		<div>
			<p>Count: {count}</p>
			<button onClick={incrementCount}>Increment Count</button>
		</div>
	);
};

export default AsyncCounterComponent;
```

위의 코드에서 각 setCount 호출은 상태의 가장 최근 값을 사용하여 정확하고 순차적인 업데이트를 보장합니다. 특히 여러 상태 업데이트가 연속적으로 빠르게 발생할 때 현재 상태에 의존하는 작업에는 이 접근 방식이 매우 중요합니다.

#### 실수 4: 파생 데이터에 대한 상태 오용 📊

#### ❌ 문제 이해

기존 상태 또는 props에서 파생 될 수 있는 데이터에 상태를 사용하는 것은 빈번한 오류입니다. 이 중복 상태는 복잡하고 오류가 발생하기 쉬운 코드를 초래할 수 있습니다.

예를 들어:

```jsx
import React, { useState } from 'react';

const GreetingComponent = ({ name }) => {
	const [greeting, setGreeting] = useState(`Hello, ${name}`);

	return <div>{greeting}</div>;
};

export default GreetingComponent;
```

여기서 greeting 상태는 name에서 직접 파생 될 수 있으므로 불필요합니다.

#### ✅ 수정:

상태를 사용하는 대신, 기존 상태 또는 props에서 데이터를 직접 파생시키세요.

```jsx
import React from 'react';

const GreetingComponent = ({ name }) => {
	const greeting = `Hello, ${name}`; // props에서 직접 파생

	return <div>{greeting}</div>;
};

export default GreetingComponent;
```

수정된 코드에서는 greeting은 name prop에서 직접 계산되므로 컴포넌트가 단순화되고 불필요한 상태 관리를 피할 수 있습니다.

#### 결론 🚀

React에서 useState 훅을 효과적으로 사용하는 것은 신뢰성 있고 효율적인 애플리케이션을 구축하는 데 매우 중요합니다. 이전 상태 무시, 상태 불변성 관리 오류, 비동기 업데이트 간과, 파생 데이터에 대한 중복 상태 방지와 같은 일반적인 실수를 이해하고 피함으로써 보다 매끄럽고 예측 가능한 컴포넌트 동작을 보장할 수 있습니다. 이러한 인사이트를 염두에 두고 React 개발 여정을 향상시키고 보다 견고한 애플리케이션을 만들어보세요.

이 글이 마음에 드셨나요? 웹 개발에 대한 심층적인 토론과 인사이트를 보려면 제 개인 블로그인 <u>Program With Jayanth</u> 를 방문해주세요.

Happy Coding!!
