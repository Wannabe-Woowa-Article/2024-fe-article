# [Best Practices for Writing Tests with React Testing Library](https://claritydev.net/blog/improving-react-testing-library-tests)

Updated On March 12, 2024
Keyword : **React**, **React-Testing-Library**, **Testing**

[React 테스트 라이브러리](https://testing-library.com/docs/react-testing-library/intro/)는 React 컴포넌트 테스트를 위한 사실상의 표준이 되었습니다. 사용자 관점에서의 테스트에 집중하고, 테스트에서는 구현 세부 사항을 감추는 것이 성공의 주요 이유 중 일부입니다.

올바르게 작성된 테스트는 regressions와 버그가 있는 코드를 방지하는 데 도움이 될 뿐만 아니라, React Testing Library의 경우 [컴포넌트의 접근성](https://claritydev.net/blog/creating-accessible-form-components-with-react)과 전반적인 사용자 경험을 향상시킵니다. React 컴포넌트로 작업할 때는 적절한 테스트 기법을 사용하여 React 테스트 라이브러리에서 흔히 발생하는 실수를 피하는 것이 중요합니다.

이 글에서는 특정 쿼리를 사용하여 테스트 커버리지를 개선하는 방법, `*ByRole` 쿼리의 중요성, 사용자 상호작용 시뮬레이션을 위해 `fireEvent`보다 `userEvent` 메서드를 사용하는 방법, 비동기 테스트를 위해 `findBy*` 쿼리와 `waitForElementToBeRemoved`를 사용하는 방법과 같은 주제와 함께 React 테스트 라이브러리를 사용할 때 가장 흔히 저지르는 실수 몇 가지를 다룰 것입니다. 이 글이 끝나면 더 나은 React 테스트 라이브러리 테스트를 작성하고, 일반적인 실수를 피하고, React 애플리케이션의 전반적인 품질을 개선할 수 있는 지식을 갖추게 될 것입니다.

### Default to `*ByRole` queries

React 테스트 라이브러리의 강력한 장점 중 하나는 올바른 쿼리를 사용하면 컴포넌트가 예상대로 작동할 뿐만 아니라 [접근성](https://claritydev.net/blog/creating-accessible-form-components-with-react)도 보장할 수 있다는 것입니다. 그렇다면 어떤 쿼리가 가장 좋은지 어떻게 알아낼 수 있을까요? 규칙은 매우 간단합니다. 기본적으로 `*ByRole` 쿼리를 사용하세요. 이 쿼리는 많은 요소, 심지어 [복잡한 select components](https://claritydev.net/blog/testing-select-components-react-testing-library)에서도 작동합니다.

대부분의 규칙과 마찬가지로 모든 HTML 요소에 기본 역할이 있는 것은 아니므로 예외가 있습니다. HTML 요소의 기본 역할 목록은 [w3.org](https://www.w3.org/TR/html-aria/#docconformance)에서 확인할 수 있습니다.

### Testing form components

[이전의 튜토리얼 글](https://claritydev.net/blog/typescript-typing-form-events-in-react)에서 변형한 다음 컴포넌트를 보겠습니다.

```jsx
export const Form = ({ saveData }) => {
  const [state, setState] = useState({
    name: "",
    email: "",
    password: "",
    confirmPassword: "",
    conditionsAccepted: false,
  });

  const onFieldChange = (event) => {
    let value = event.target.value;
    if (event.target.type === "checkbox") {
      value = event.target.checked;
    }

    setState({ ...state, [event.target.id]: value });
  };

  const onSubmit = (event) => {
    event.preventDefault();
    saveData(state);
  };

  return (
    <form className="form" onSubmit={onSubmit}>
      <div className="field">
        <label>Name</label>
        <input
          id="name"
          onChange={onFieldChange}
          placeholder="Enter your name"
        />
      </div>
      <div className="field">
        <label>Email</label>
        <input
          type="email"
          id="email"
          onChange={onFieldChange}
          placeholder="Enter your email address"
        />
      </div>
      <div className="field">
        <label>Password</label>
        <input
          type="password"
          id="password"
          onChange={onFieldChange}
          placeholder="Password should be at least 8 characters"
        />
      </div>
      <div className="field">
        <label>Confirm password</label>
        <input
          type="password"
          id="confirmPassword"
          onChange={onFieldChange}
          placeholder="Enter the password once more"
        />
      </div>
      <div className="field checkbox">
        <input type="checkbox" id="conditions" onChange={onFieldChange} />
        <label>I agree to the terms and conditions</label>
      </div>
      <button type="submit">Sign up</button>
    </form>
  );
};
```

form elements를 통해 데이터 입력을 시뮬레이션하고 양식을 제출한 다음 `saveData` prop가 입력한 데이터를 수신했는지 확인하여 이를 테스트할 수 있습니다. 이를 3단계로 나눌 수 있습니다:

1. 테스트할 필드에 텍스트를 입력하거나 확인란을 클릭합니다.
2. `Sign up` 버튼을 클릭합니다.
3. 입력한 데이터로 `saveData`가 호출되었는지 확인합니다.

이 workflow는 사용자가 form과 상호 작용하는 방식을 매우 유사하게 반영합니다(저장된 데이터를 동일한 방식으로 검사하지는 않을 수 있음).

### Querying by placeholder text

첫 번째 입력 필드에 이름을 입력하는 것으로 시작하겠습니다. name placeholder임을 적었으므로, 해당 Inputd을 이를 사용하여 querying 하면 어떨까요?

```jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import "@testing-library/jest-dom/extend-expect";

const defaultData = {
  conditionsAccepted: false,
  confirmPassword: "",
  email: "",
  name: "",
  password: "",
};

describe("Form", () => {
  it("should submit correct form data", async () => {
    const user = userEvent.setup();
    const mockSave = jest.fn();
    render(<Form saveData={mockSave} />);

    await user.type(screen.getByPlaceholderText("Enter your name"), "Test");
    await user.click(screen.getByText("Sign up"));

    expect(mockSave).toHaveBeenLastCalledWith({ ...defaultData, name: "Test" });
  });
});
```

이 방법도 효과가 있지만 그보다 더 나은 방법이 있습니다.
첫째, 이 접근 방식은 placeholder text를 label로 사용하는 관행을 조장할 수 있으며, 이는 placeholder의 원래 용도가 아니면서 [W3C WAI](https://www.w3.org/WAI/tutorials/forms/instructions/#placeholder-text)에서 권장하지 않습니다. 둘째, 접근성 문제를 염두에 두고 테스트하지 않습니다.

### Querying specific components by role

대신 쿼리를 `getByRole`로 대체해 보겠습니다. 문서에 나와 있듯이 `text box`의 역할에 따라 `text` 유형의 입력을 일치시킬 수 있습니다. 하지만 양식에 여러 개의 text box가 있으므로 이보다 더 구체적이어야 합니다. 다행히도 쿼리는 옵션 객체인 두 번째 매개 변수를 허용하므로 `name` 속성을 사용하여 일치하는 항목의 범위를 좁힐 수 있습니다.

[RTL 공식문서](https://testing-library.com/docs/queries/byrole/)를 보면 여기서 `name`이 입력의 이름 속성이 아니라 [accessible `name`](https://www.tpgi.com/what-is-an-accessible-name/)을 가리킨다는 것을 알 수 있습니다. 따라서 입력의 경우 accessible name은 레이블의 텍스트 콘텐츠인 경우가 많습니다. 우리 양식에서는 `name input`에 Name label이 있으므로 이를 사용하겠습니다.

```jsx
user.type(screen.getByRole("textbox", { name: "Name" }), "Test");
```

...
