## 🔗 [React State Management — using Zustand](https://medium.com/globant/react-state-management-b0c81e0cbbf3)

### 🗓️ 번역 날짜: 2024.08.19

### 🧚 번역한 크루: 소하(최소연)

---

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*y_e4OOiSszPMpyjUD9_BOw.png"/>

당신은 상태(state)와 상태 관리(state management)가 React 애플리케이션의 중요한 부분이라고 생각하시나요? 상태 관리를 어려워해본 적이 있거나 간단한 상태 관리 라이브러리를 찾고 있었다면, 이 글이 당신을 위한 것입니다. 이 글에서는 간단한 외부 라이브러리인 Zustand를 사용하여 React 상태를 어떻게 다룰 수 있는지 보여줄 것입니다.

정말로! 상태와 상태 관리는 항상 React 애플리케이션의 중요한 측면이었습니다. React는 상태가 변경될 때만 다시 렌더링하기 때문입니다. 상태는 컴포넌트에 대한 데이터나 정보를 포함할 수 있습니다. 애플리케이션이 성장함에 따라 상태와 컴포넌트 간의 데이터 흐름을 관리하는 것이 매우 중요해집니다.

### React 상태 관리

React 애플리케이션의 상태를 관리하는 방법에는 여러 가지가 있습니다.

1. **React 네이티브 상태 관리.** useState, useReducer, useRef, useContext와 같은 훅(hooks)과 사용자 정의 훅(custom hooks)은 네이티브 상태 관리를 지원합니다.
2. **간접 상태 관리자.** React Router와 React Query와 같은 외부 라이브러리들이 있습니다. 이들은 주로 상태 관리를 위해 사용되지 않지만, 네이티브 훅과 결합하면 상태를 잘 관리할 수 있습니다.
3. **직접 상태 관리자.** 상태 관리를 위해서만 사용되는 서드파티 라이브러리도 있습니다. Redux, Zustand, Jotai, Valtio가 이 범주에 속합니다.

### Zustand란 무엇인가요?

Zustand는 상태 관리 라이브러리입니다. 작고 빠르며 확장성이 있습니다. 이는 간단한 시스템으로, 보일러플레이트 코드가 거의 없습니다. Redux 및 유사한 라이브러리보다 적은 코드로 React 상태를 관리할 수 있습니다. Zustand는 provider에 의존하지 않기 때문에, React 로직을 덜 작성해도 되며, 우리가 종종 잊기 쉬운 부분들을 줄일 수 있습니다. 이는 간소화된 flux 원칙을 기반으로 작동하며, 주로 Hook을 사용합니다.

왜 Zustand를 선택해야 할까요?

- Zustand는 context보다 빠릅니다. 특정 상태를 선택할 수 있는 옵션을 제공합니다.
- 상태 병합이 기본적으로 지원됩니다. 객체 상태 `{x:1, y:2}`의 단일 속성을 업데이트한다고 가정해봅시다. `{y:3}`으로 직접 설정할 수 있습니다. Zustand는 데이터를 자동으로 병합해 줍니다. 기존 상태를 분배하고 `{…state, y:3}`처럼 속성을 업데이트할 필요가 없습니다.
- Zustand는 기본적으로 확장 가능합니다. 따라서 다양한 미들웨어 유형을 사용할 수 있습니다.
- Zustand는 특정 방식에 얽매이지 않습니다. 권장되는 접근 방식이 있더라도, 그것을 반드시 채택할 필요는 없습니다.

### Zustand vs. Redux

아시다시피, Redux는 React와 함께 작동하는 고전적인 상태 관리 라이브러리입니다. 이는 React 상태 관리를 위한 가장 인기 있는 라이브러리로 간주됩니다. 따라서 이 두 라이브러리의 아키텍처를 비교하는 것이 이 글의 적합한 요소가 됩니다. 아래의 아키텍처를 살펴보면서 Redux가 어떻게 작동하는지 확인해보세요.

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*GUUEnrvJJTkKSx-XXdvW_w.png"/>

먼저, 앞서 설명된 아키텍처에서 볼 수 있듯이 프론트엔드 사용자 인터페이스가 있습니다. action creators는 각 사용자 요청에 대해 올바른 액션이 트리거되도록 보장합니다. 액션은 애플리케이션에서 어떤 일이 발생했는지를 설명하는 이벤트라고 생각할 수 있습니다. 버튼 클릭이나 검색 수행 같은 것이 이에 해당할 수 있습니다. dispatchers는 이러한 액션을 store로 보내는 데 도움을 줍니다. 이후 reducers는 상태를 어떻게 처리할지 결정합니다. reducer 함수는 현재 상태와 액션 객체를 받아 상태를 변경합니다. 필요에 따라 새로운 상태를 반환하며, 업데이트된 상태 수정 사항이 UI를 렌더링하게 됩니다.

이제 흥미로운 부분이 다가옵니다! 아래에 제공된 아키텍처를 통해 Zustand가 어떻게 작동하는지 살펴보세요. 이 단순화된 다이어그램에 흥미를 느낄 수도 있습니다.

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*K5SdovTtjwHe545H6Q2D_A.png"/>

여기에서도 UI 컴포넌트가 있습니다. 변경 요청이 들어오면 그것은 store로 라우팅됩니다. store는 상태가 어떻게 변경되어야 하는지를 결정합니다. store가 새로운 상태를 반환하면, UI는 업데이트된 변경 사항과 함께 렌더링됩니다. 여기에서는 action creator, dispatcher, reducer가 보이지 않습니다. 대신, Zustand는 상태 변경을 구독할 수 있는 기능을 제공합니다. 이를 통해 UI가 데이터와 동기화 상태를 유지하는 데 도움이 됩니다.

> 더 크고 복잡한 툴킷인 Redux의 복잡성을 피하면서도 간단하고 가벼운 상태 관리 솔루션을 찾고 있는 개발자들에게 Zustand는 완벽한 선택입니다.

이론은 여기서 끝입니다. 이제 실습을 시작해보세요!

### ReactJs에서 Zustand 사용 방법

Zustand를 사용하여 상태를 관리하는 방법을 보여주기 위해 React 프로젝트를 만들어 봅시다. 도서 발행 및 반납을 추적하는 Library Store 애플리케이션을 예로 들어보겠습니다. 단계는 아래와 같습니다.

#### 1. React 애플리케이션 생성

아래 명령을 사용하여 React 애플리케이션을 생성합니다.

```bash
npx create-react-app project_name
```

#### 2. Zustand 의존성 설치

프로젝트 디렉토리로 이동하여 React 상태 관리를 위해 Zustand 의존성을 설치합니다.

```bash
npm i zustand
```

#### 3. store 생성

bookStore.js 파일을 생성하기 위해 아래 코드를 참고하여 도서 store를 만듭니다.

```javascript
import { create } from 'zustand';

const bookStore = (set, get) => ({
	books: [],
	noOfAvailable: 0,
	noOfIssued: 0,
	addBook: (book) => {
		set((state) => ({
			books: [...state.books, { ...book, status: 'available' }],
			noOfAvailable: state.noOfAvailable + 1,
		}));
	},
	issueBook: (id) => {
		const books = get().books;
		const updatedBooks = books?.map((book) => {
			if (book.id === id) {
				return {
					...book,
					status: 'issued',
				};
			} else {
				return book;
			}
		});
		set((state) => ({
			books: updatedBooks,
			noOfAvailable: state.noOfAvailable - 1,
			noOfIssued: state.noOfIssued + 1,
		}));
	},
	returnBook: (id) => {
		const books = get().books;
		const updatedBooks = books?.map((book) => {
			if (book.id === id) {
				return {
					...book,
					status: 'available',
				};
			} else {
				return book;
			}
		});
		set((state) => ({
			books: updatedBooks,
			noOfAvailable: state.noOfAvailable + 1,
			noOfIssued: state.noOfIssued - 1,
		}));
	},
	reset: () => {
		set({
			books: [],
			noOfAvailable: 0,
			noOfIssued: 0,
		});
	},
});

const useBookStore = create(bookStore);

export default useBookStore;
```

Zustand store는 hook입니다. 그래서 `useBookStore`가 컴포넌트 이름으로 사용됩니다. `create`는 store를 생성하는 데 사용되는 메서드입니다. store는 각 컴포넌트가 공유하는 유일한 진리의 원천입니다. `set` 함수는 변수나 객체의 상태를 수정하는 데 사용되고, `get` 함수는 액션 내에서 상태를 접근하는 데 사용됩니다.

이 예제에서 라이브러리 store의 상태 객체는 세 개의 필드를 포함하고 있습니다. `books`는 책의 ID, 이름, 저자와 같은 세부 정보가 포함된 배열을 포함하며, `noOfAvailable`에는 라이브러리 내의 총 도서 수가 저장됩니다. `noOfIssued`는 사용자에게 발행된 도서의 총 수를 저장합니다. 라이브러리 store는 네 가지 메서드를 제공합니다. `addBook` 함수는 책의 배열에 새로운 책을 추가하고, 현재 이용 가능한 책의 수를 늘리며, 새로 추가된 모든 책의 상태를 "available"로 설정합니다. `issueBook` 함수는 사용자가 도서를 발행할 수 있게 해주며, 관련된 도서는 "issued" 상태로 변경됩니다. 발행 수가 증가하고 이용 가능한 책의 수가 감소합니다. `returnBook` 함수는 발행된 도서를 라이브러리에 반환하는 데 사용되며, 반환된 도서의 상태가 "available"로 변경되고 발행된 도서 수가 감소하며 이용 가능한 도서 수가 증가합니다. 마지막으로, `reset` 메서드는 모든 상태 필드를 초기화합니다.

#### 4. 컴포넌트를 store와 연결

먼저 `App.js` 파일을 생성해 봅시다. 아래 코드를 참고하세요.

```javascript
//App.js

import { useEffect } from 'react';
import BookForm from './components/BookForm';
import BookList from './components/BookList';
import useBookStore from './bookStore';
import './App.css';

function App() {
	const reset = useBookStore((state) => state.reset);

	useEffect(() => {
		reset();
	}, [reset]);

	return (
		<div className="App">
			<h2>My Library Store</h2>
			<BookForm />
			<BookList />
		</div>
	);
}

export default App;
```

`App.js` 파일에는 `BookForm`과 `BookList`라는 두 개의 컴포넌트가 있습니다. `App` 컴포넌트가 mount될 때마다 상태 데이터를 제거하기 위해 `reset` 함수를 사용합니다.

이제 `BookForm.js` 컴포넌트를 생성합니다. 아래 코드를 참고하세요.

```javascript
//BookForm.js

import { useState } from 'react';
import useBookStore from '../bookStore';

function BookForm() {
	const addBook = useBookStore((state) => state.addBook);
	const [bookDetails, setBookDetails] = useState({});

	const handleOnChange = (event) => {
		const { name, value } = event.target;
		setBookDetails({ ...bookDetails, [name]: value });
	};

	const handleAddBook = () => {
		if (!Object.keys(bookDetails).length) return alert('Please enter book details!');
		addBook(bookDetails);
	};

	return (
		<div className="input-div">
			<div className="input-grp">
				<label>Book ID</label>
				<input type="text" name="id" size={50} onChange={handleOnChange} />
			</div>
			<div className="input-grp">
				<label>Book Name</label>
				<input type="text" name="name" size={50} onChange={handleOnChange} />
			</div>
			<div className="input-grp">
				<label>Author</label>
				<input type="text" name="author" size={50} onChange={handleOnChange} />
			</div>
			<button onClick={handleAddBook} className="add-btn">
				Add
			</button>
		</div>
	);
}

export default BookForm;
```

`BookForm` 컴포넌트는 ID, 이름, 저자와 같은 책의 세부 정보를 입력할 수 있는 폼 필드를 가지고 있습니다. 또한, 이 책의 세부 정보를 입력하기 위해 도서 store의 `addBook` 메서드를 사용하는 "Add" 버튼도 있습니다. 샘플 책 세부 정보와 함께 UI가 아래에 표시됩니다.

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*q0b70FPKH7oc3BLGk5CMMw.png"/>

다음 코드로 BookList.js 컴포넌트를 생성하세요.

```javascript
//BookList.js

import { Fragment } from 'react';
import useBookStore from '../bookStore';

function BookList() {
	const { books, noOfAvailable, noOfIssued, issueBook, returnBook } = useBookStore((state) => ({
		books: state.books,
		noOfAvailable: state.noOfAvailable,
		noOfIssued: state.noOfIssued,
		issueBook: state.issueBook,
		returnBook: state.returnBook,
	}));

	return (
		<ul className="book-list">
			{!!books?.length && (
				<span className="books-count">
					<h4>Available: {noOfAvailable}</h4>
					<h4>Issued: {noOfIssued}</h4>
				</span>
			)}
			{books?.map((book) => {
				return (
					<Fragment key={book.id}>
						<li className="list-item">
							<span className="list-item-book">
								<span>{book.id}</span>
								<span>{book.name}</span>
								<span>{book.author}</span>
							</span>
							<div className="btn-grp">
								<button
									onClick={() => issueBook(book.id)}
									className={`issue-btn ${book.status === 'issued' ? 'disabled' : ''}`}
									disabled={book.status === 'issued'}
								>
									{' '}
									Issue{' '}
								</button>
								<button
									onClick={() => returnBook(book.id)}
									className={`return-btn ${book.status === 'available' ? 'disabled' : ''}`}
									disabled={book.status === 'available'}
								>
									{' '}
									Return{' '}
								</button>
							</div>
						</li>
					</Fragment>
				);
			})}
		</ul>
	);
}

export default BookList;
```

**BookList** 컴포넌트는 도서관에 새로 추가된 모든 책을 표시합니다. 또한 이용 가능 도서와 대출된 도서의 수를 보여줍니다. 목록에 있는 각 책 기록에는 **Issue**(대출) 및 **Return**(반납) 버튼이 두 개 있습니다. UI는 다음과 같이 보입니다.

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*mzzOO6phLadV07DhRW4j-w.png"/>

위 UI에는 두 권의 책이 있으며, 각 책 기록에 대해 **Issue** 버튼이 활성화되어 있습니다. **Return** 버튼은 비활성화되어 있으며, 책이 대출되었을 때만 활성화됩니다.

**Issue** 버튼을 클릭하면 store의 **issueBook** 메서드가 호출됩니다. 해당 책 ID가 전달되고, 해당 책의 상태가 '대출됨(issued)'으로 설정됩니다. 그런 다음, 관련된 **Issue** 버튼이 비활성화되고, **Return** 버튼이 활성화됩니다. 사용 가능한 책의 수가 줄어들고, 대출된 책의 수가 증가하는 것을 볼 수 있습니다. 아래 스크린샷을 참조하세요.

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*XmKsjPTJCcBU-cMeHrj8OQ.png"/>

**Return** 버튼을 클릭하면 store의 **returnBook** 메서드가 호출됩니다. 해당 책 ID가 전달되고, 해당 책의 상태가 '이용 가능(available)'으로 다시 설정됩니다. 관련된 **Return** 버튼이 비활성화되고, **Issue** 버튼이 다시 활성화됩니다. 사용 가능한 책의 수가 증가하고, 대출된 책의 수가 감소하는 것을 볼 수 있습니다. 위 스크린샷을 참조하세요.

> Zustand store는 훅(hook)이기 때문에 어디서나 사용할 수 있습니다. Redux나 Redux Toolkit과는 달리 context provider가 필요하지 않습니다. 상태를 선택하기만 하면, 상태가 변경될 때 컴포넌트가 다시 렌더링됩니다. 필요한 슬라이스만 얻기 위해 **useBookStore**에 선택기를 제공해야 합니다.

아직 끝나지 않았습니다! Zustand에는 큰 장점이 있습니다: **미들웨어(Middlewares)**입니다.

### Zustand 미들웨어

Zustand는 미들웨어와 함께 사용하여 애플리케이션에 더 많은 기능을 추가할 수 있습니다. 가장 널리 사용되는 Zustand 미들웨어는 아래에 나와 있습니다.

1. **Redux DevTools**

Redux DevTools는 애플리케이션에서 상태 변경을 디버그하는 데 사용됩니다. Zustand에서도 사용할 수 있습니다. Chrome 확장 프로그램으로 Redux DevTools를 설치했는지 확인하세요. 아래 코드를 사용하여 통합할 수 있습니다.

```javascript
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

const bookStore = (set, get) => ({
	books: [],
	noOfAvailable: 0,
	noOfIssued: 0,
	addBook: (book) => {
		set((state) => ({
			books: [...state.books, { ...book, status: 'available' }],
			noOfAvailable: state.noOfAvailable + 1,
		}));
	},
});

const useBookStore = create(devtools(bookStore));

export default useBookStore;
```

**devtools**를 사용하려면 **zustand/middleware**에서 가져와야 합니다. 그런 다음 **create** 메서드를 사용하여 store를 **devtools**로 래핑합니다.

웹 브라우저에서 Redux DevTools를 열고 상태를 검사합니다. 아래 스크린샷에 나와 있습니다.

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Tjau7gmPB8xsuMRfF08Ukw.png"/>

2. **Persist Middleware**

**Persist** 미들웨어를 사용하면 모든 종류의 클라이언트 저장소를 사용하여 상태를 지속시킬 수 있습니다. 애플리케이션을 새로고침해도 store의 데이터가 저장소에 남아 있습니다. 이 미들웨어를 추가하는 코드는 아래를 참조하세요.

```javascript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

const bookStore = (set, get) => ({
	books: [],
	noOfAvailable: 0,
	noOfIssued: 0,
	addBook: (book) => {
		set((state) => ({
			books: [...state.books, { ...book, status: 'available' }],
			noOfAvailable: state.noOfAvailable + 1,
		}));
	},
});

const useBookStore = create(
	persist(bookStore, {
		name: 'books',
		storage: createJSONStorage(() => sessionStorage),
	})
);

export default useBookStore;
```

**persist**를 **zustand/middleware**에서 가져옵니다. 그런 다음 **create** 메서드 내에서 store를 **persist**로 래핑합니다. 저장소의 항목은 고유한 이름을 가질 수 있습니다. 또한 저장소의 종류를 지정할 수 있습니다. 이 코드에서는 **sessionStorage**가 참조됩니다. 아무것도 지정하지 않으면 기본값으로 **localStorage**가 사용됩니다.

웹 브라우저를 열고 세션 스토리지에서 저장된 상태를 확인하세요. 스크린샷은 아래와 같습니다.

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*eVw9beZHm-3AgIJRF6XnGA.png"/>

### 요약

이 글에서 언급했듯이, 상태 관리는 React 애플리케이션에서 매우 중요합니다. 애플리케이션이 커질수록 상태를 관리할 강력한 방법을 선택해야 합니다. Zustand 라이브러리는 React 상태를 관리하는 데 이상적인 해결책입니다. Redux보다 훨씬 간단하고 보일러플레이트 코드가 적습니다. 또한, 애플리케이션에 적합한 상태 관리 라이브러리를 찾기 위해 다양한 옵션을 탐색하는 것도 좋습니다.

### 참고

1. https://docs.pmnd.rs/zustand/getting-started/introduction
2. https://www.youtube.com/watch?v=KCr-UNsM3vA
3. https://www.youtube.com/watch?v=fZPgBnL2x-Q
