## 🔗 [How to handle multiple modals in a React application](https://dev.to/zettadam/how-to-handle-multiple-modals-in-a-react-application-2pei)

### 🗓️ 번역 날짜: 2024.06.29

### 🧚 번역한 크루: 러기(박정우)

---

## 리액트 어플리케이션에서 여러개의 모달들을 관리하는 방법

참고: 전체 예제 애플리케이션은 여기에서 확인할 수 있습니다: https://stackblitz.com/edit/react-modals

React 애플리케이션에서 모달을 관리하는 방법은 하나만 있는 것이 아니지만, 몇몇 방식이 다른 방식보다 나을 수 있습니다. 이 글에서는 Redux 스토어 같은 글로벌 스토어를 사용하여 모달을 관리하는 것보다 더 간단한 방법을 소개하고자 합니다. 이 예제에서는 컴포넌트 상태와 이벤트 버블링을 사용할 것이며, 이는 React의 Portals도 언급되어 있습니다.

모달은 보통 라우터에 의해 관리되는 별도의 화면과 비슷한 부분이 있습니다.

## AppShell

예를 들어 다음 src/AppShell.jsx 예제와 같이, 두 종류의 컴포넌트를 중앙 컴포넌트에서 서로 가깝게 렌더링하는 것이 합리적일 수 있습니다.

```tsx
import React, { useState } from 'react'
import { BrowserRouter, NavLink, Route, Switch } from 'react-router-dom'

import ScreenOne from './components/screen-one/ScreenOne'
import ScreenTwo from './components/screen-two/ScreenTwo'
import ScreenThree from './components/screen-three/ScreenThree'

import ModalOne from './components/common/modal-one/ModalOne'
import ModalTwo from './components/common/modal-two/ModalTwo'
import ModalThree from './components/common/modal-three/ModalThree'

import './app-shell.css'

const AppShell = () => {
  const [modalOpen, setModal] = useState(false)

  const openModal = event => {
    event.preventDefault()
    const { target: { dataset: { modal }}} = event
    if (modal) setModal(modal)
  }

  const closeModal = () => {
    setModal('')
  }

  return (
    <BrowserRouter>
      <div className="app--shell" onClick={openModal}>

        {/* Application header and navigation */}
        <header className="app--header">
          <h1>React Modal Windows</h1>
          <nav className="app--nav">
            <NavLink to="/screen-one">Screen One</NavLink>
            <NavLink to="/screen-two">Screen Two</NavLink>
            <NavLink to="/screen-three">Screen Three</NavLink>
          </nav>
        </header>

        {/* Application screens */}
        <Switch>
          <Route path="/screen-three">
            <ScreenThree />
          </Route>
          <Route path="/screen-two">
            <ScreenTwo />
          </Route>
          <Route path="/screen-one">
            <ScreenOne />
          </Route>
          <Route exact path="/">
            <ScreenOne />
          </Route>
        </Switch>

        {/* Modals */}
        <ModalOne
          closeFn={closeModal}
          open={modalOpen === 'modal-one'} />

        <ModalTwo
          closeFn={closeModal}
          open={modalOpen === 'modal-two'} />

        <ModalThree
          closeFn={closeModal}
          open={modalOpen === 'modal-three'} />

        {/* Application footer */}
        <footer className="app--footer">
          <p className="copyright">&copy; 2021 Some Company</p>
        </footer>

      </div>
    </BrowserRouter>
```

## 단일 책임 컴포넌트로 리팩터링 하기

만약 여러분의 애플리케이션이 많은 화면 혹은 많은 모달을 포함하고 있다면, 예를 들어 ScreenSwitchboard.jsx와 ModalManager.jsx와 같이 라우트와 모달을 별도의 컴포넌트로 추출함으로써 AppShell.jsx 컴포넌트를 좀 더 깔끔하게 만들 수 있습니다.

```tsx
import React, { useState } from "react";
import { BrowserRouter } from "react-router-dom";

import AppHeader from "./AppHeader";
import AppFooter from "./AppFooter";

import ScreenSwitchboard from "./ScreenSwitchboard";
import ModalManager from "./ModalManager";

import "./app-shell.css";

const AppShell = () => {
  const [modalOpen, setModal] = useState(false);

  const openModal = (event) => {
    event.preventDefault();
    const {
      target: {
        dataset: { modal },
      },
    } = event;
    if (modal) setModal(modal);
  };

  const closeModal = () => {
    setModal("");
  };

  return (
    <BrowserRouter>
      <div className="app--shell" onClick={openModal}>
        <AppHeader />
        <ScreenSwitchboard />
        <ModalManager closeFn={closeModal} modal={modalOpen} />
        <AppFooter />
      </div>
    </BrowserRouter>
  );
};

export default AppShell;
```

## 이벤트 버블링을 사용하여 특정 모달 열기

우리는 #app--shell 요소에서 버블링된 클릭 이벤트를 캡처합니다. 특정 모달을 열도록 트리거하는 이벤트 핸들러 openModal은 우리 애플리케이션의 일부 요소(버튼, 링크 등)에 설정할 수 있는 data-modal 속성을 찾습니다.

아래는 클릭했을 때 모달을 열도록 트리거하는 버튼이 있는 화면 컴포넌트의 예시입니다.

```tsx
import React from "react";

const ScreenOne = ({}) => {
  return (
    <main className="app--screen screen--one">
      <h2>Screen One</h2>

      <div style={{ display: "flex", columnGap: "1rem" }}>
        <button type="button" data-modal="modal-one">
          Open Modal One
        </button>
        <button type="button" data-modal="modal-two">
          Open Modal Two
        </button>
        <button type="button" data-modal="modal-three">
          Open Modal Three
        </button>
      </div>
    </main>
  );
};

export default ScreenOne;
```

보시다시피, 우리는 애플리케이션의 계층 구조 props 내리기를 통해 함수나 값을 전달하고 있지 않습니다. 대신, data-modal 속성과 이벤트 버블링에 의존하여 특정 모달을 여는 것을 처리합니다.

## ModalManager

우리의 <ModalManager /> 컴포넌트는 두 가지 props를 요구합니다: 어떤 모달이 열려야 하는지를 설명하는 modal prop으로서의 상태 값과, 열려 있는 모달을 닫도록 애플리케이션을 효과적으로 지시하는 closeFn prop입니다.

참고: 모달은 간단한 콘텐츠를 포함할 수도 있고, 폼 처리와 같은 더 복잡한 경우를 다룰 수도 있습니다. 우리는 그들의 닫힘을 처리하기 위해 클릭 이벤트 버블링에 의존하고 싶지 않습니다. 여기에서 prop을 사용하는 것이 더 간단하고 유연합니다.

다음은 <ModalManager /> 컴포넌트입니다.

```tsx
import React from "react";

import ModalOne from "./components/common/modal-one/ModalOne";
import ModalTwo from "./components/common/modal-two/ModalTwo";
import ModalThree from "./components/common/modal-three/ModalThree";

const ModalManager = ({ closeFn = () => null, modal = "" }) => (
  <>
    <ModalOne closeFn={closeFn} open={modal === "modal-one"} />

    <ModalTwo closeFn={closeFn} open={modal === "modal-two"} />

    <ModalThree closeFn={closeFn} open={modal === "modal-three"} />
  </>
);

export default ModalManager;
```

## React Portal을 사용하여 모달 렌더링하기

한 번에 하나의 Modal만 표시하는 것이 가장 일반적인 패턴이므로, 자식 요소를 React 포털로 렌더링할 래퍼 컴포넌트를 만드는 것이 타당하다고 생각합니다.

다음은 src/components/common/modal/Modal.jsx 컴포넌트의 코드입니다.

```tsx
import React, { useEffect } from "react";
import ReactDOM from "react-dom";

const modalRootEl = document.getElementById("modal-root");

const Modal = ({ children, open = false }) => {
  if (!open) return null;

  return ReactDOM.createPortal(children, modalRootEl);
};

export default Modal;
```

우리는 #modal-root 요소가 document 어딘가에, 특히 애플리케이션이 마운트된 #app-root 요소의 형제 요소로서 존재하기를 기대합니다.

예를 들어, index.html의 <body />는 다음과 같이 보일 수 있습니다:

```html
<body>
  <div id="app-root"></div>
  <div id="modal-root"></div>
</body>
```

마지막으로, 특정 모달 컴포넌트의 예시입니다.

```tsx
import React from "react";

import Modal from "../modal/Modal";

const ModalOne = ({ closeFn = () => null, open = false }) => {
  return (
    <Modal open={open}>
      <div className="modal--mask">
        <div className="modal-window">
          <header className="modal--header">
            <h1>Modal One</h1>
          </header>
          <div className="modal--body">
            <p>Modal One content will be rendered here.</p>
          </div>
          <footer className="modal--footer">
            <button type="button" onClick={closeFn}>
              Close
            </button>
          </footer>
        </div>
      </div>
    </Modal>
  );
};

export default ModalOne;
```

이 글에서는 구체적인 예시를 들면서 상대적으로 짧고 간단하게 만들고자 모든 내용을 다루지 않았습니다. 스타일링, 접근성 그리고 아마도 다른 요소들을 고려해야 할 것입니다.

이 글의 맨 위에 게시된 링크에서 이 소스 코드를 찾을 수 있습니다.

댓글로 이 글에 대해 어떻게 생각하는지, 혹은 여러분의 애플리케이션에서 모달을 어떻게 관리하고 있는지 알려주세요.
