## 🔗 [Stop mocking fetch](https://kentcdodds.com/blog/stop-mocking-fetch)

### 🗓️ 번역 날짜: 2024.05.26

### 🧚 번역한 크루: 러기(박정우)

---

## “fetch” 모킹은 그만!

```jsx
// __tests__/checkout.js
import * as React from "react";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { client } from "~/utils/api-client";

jest.mock("~/utils/api-client");

test('clicking "confirm" submits payment', async () => {
  const shoppingCart = buildShoppingCart();
  render(<Checkout shoppingCart={shoppingCart} />);

  client.mockResolvedValueOnce(() => ({ success: true }));

  userEvent.click(screen.getByRole("button", { name: /confirm/i }));

  expect(client).toHaveBeenCalledWith("checkout", { data: shoppingCart });
  expect(client).toHaveBeenCalledTimes(1);
  expect(await screen.findByText(/success/i)).toBeInTheDocument();
});
```

이 텍스트는 당신이 **‘Checkout’**과 **‘/checkout’** 엔드포인트의 실제 API와 요구 사항을 알지 못한다면 정말로 답할 수 없다는 것을 전제로 하고 있습니다. 이 점을 양해해 주십시오.

이 경우 문제점 중 하나는, 클라이언트를 모킹하고 있기 때문에 클라이언트가 제대로 사용되고 있는지 어떻게 알 수 없다는 것입니다.

클라이언트가 window.fetch를 제대로 호출하는지 단위 테스트를 할 수 있지만, 클라이언트의 API가 최근에 데이터 대신 본문을 받도록 변경되지 않았다는 것을 어떻게 알 수 있을까요?

TypeScript를 사용하므로 [일부 버그 카테고리는 제거되었습니다!](https://kentcdodds.com/blog/eliminate-an-entire-category-of-bugs-with-a-few-simple-tools) 하지만 비즈니스 로직 버그가 발생할 수 있으므로 클라이언트를 모킹하는 데 주의가 필요합니다. 물론 E2E 테스트를 통해 확신을 얻을 수 있지만, 더 빠른 피드백 루프를 가지고 있는 이 하위 레벨에서 클라이언트에 직접 호출하여 확신을 얻는 것이 더 나을 수도 있습니다. 만약 그렇게 어렵지 않다면요!

하지만 실제로 ‘**fetch’** 요청을 하고 싶지는 않겠죠? 그럼 ‘**window.fetch**’를 모킹해 봅시다.

```jsx
// __tests__/checkout.js
import * as React from "react";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

beforeAll(() => jest.spyOn(window, "fetch"));
// jest의 resetMocks가 "true"로 설정되어 있다고 가정합니다.
// 정리에 대해 걱정할 필요가 없습니다.
// 또한 `whatwg-fetch`와 같은 불러오기 폴리필을 로드했다고 가정합니다.

test('clicking "confirm" submits payment', async () => {
  const shoppingCart = buildShoppingCart();
  render(<Checkout shoppingCart={shoppingCart} />);

  window.fetch.mockResolvedValueOnce({
    ok: true,
    json: async () => ({ success: true }),
  });

  userEvent.click(screen.getByRole("button", { name: /confirm/i }));

  expect(window.fetch).toHaveBeenCalledWith(
    "/checkout",
    expect.objectContaining({
      method: "POST",
      body: JSON.stringify(shoppingCart),
    })
  );
  expect(window.fetch).toHaveBeenCalledTimes(1);
  expect(await screen.findByText(/success/i)).toBeInTheDocument();
});
```

이렇게 하면 실제로 요청이 이루어지고 있음을 좀 더 확신할 수 있지만, 이 테스트에서 부족한 것은 '**Content-Type**'이 '**application/json**'인지 확인하는 단언이 없다는 것입니다. 그것 없이 어떻게 서버가 당신이 만든 요청을 인식할 것이라고 확신할 수 있나요? 그리고 올바른 인증 정보가 전송되고 있는지 어떻게 보장할 수 있나요?

"**우리는 클라이언트 단위 테스트에서 이미 그것을 검증했어요, Kent. 더 무엇을 원하나요? 저는 모든 곳에 단언을 복사/붙여넣기 하고 싶지 않아요!**" 분명히 당신의 입장을 이해합니다. 하지만 모든 테스트에서 그 확신을 얻으면서도 모든 곳에서 추가적인 작업을 피할 수 있는 방법이 있다면 어떨까요? 계속 읽어보세요.

저를 정말 괴롭히는 것 중 하나는 fetch와 같은 것을 모킹할 때, 당신은 테스트 곳곳에서 전체 백엔드를 재구현하게 된다는 것입니다. 종종 여러 테스트에서 이런 일이 발생합니다. 특히 "**이 테스트에서는 정상적인 백엔드 응답을 가정합니다**"라고 할 때, 그것을 곳곳에서 모킹해야만 하는 것은 정말 짜증납니다. 그런 경우에는 정말로 단지 설정 소음이 당신과 실제로 테스트하려는 것 사이에 끼어들게 됩니다.

결국 불가피하게 다음과 같은 시나리오 중 하나가 발생합니다:

1. **클라이언트를 모킹합니다** (첫 번째 테스트에서처럼) 그리고 몇 가지 E2E 테스트에 의존하여 적어도 가장 중요한 부분이 클라이언트를 올바르게 사용하고 있음을 약간 확신할 수 있습니다. 이는 백엔드를 건드리는 것들을 테스트할 때마다 백엔드를 재구현하는 결과를 초래합니다. 종종 작업이 중복됩니다.
2. **window.fetch를 모킹합니다** (두 번째 테스트에서처럼). 이 방법은 조금 나은 편이지만, #1과 같은 문제를 일부 겪습니다.
3. 모든 것을 작은 함수들로 나누고 모두를 독립적으로 단위 테스트합니다 (자체적으로 나쁘지 않은 일) 그리고 통합 테스트를 하지 않습니다 (좋은 일은 아닙니다).

결국 우리는 더 낮은 확신과 더 느린 피드백 루프, 많은 중복된 코드 또는 이들의 조합에 직면하게 됩니다.

오랫동안 나에게 잘 작동했던 한 가지 방법은 fetch를 한 함수에서 모킹하는 것인데, 이는 기본적으로 내가 테스트한 백엔드의 모든 부분을 재구현하는 것입니다. 나는 이 방식을 PayPal에서 사용했고 매우 잘 작동했습니다.

이를 다음과 같이 생각할 수 있습니다

```jsx
// add this to your setupFilesAfterEnv config in jest so it's imported for every test file
import * as users from "./users";

async function mockFetch(url, config) {
  switch (url) {
    case "/login": {
      const user = await users.login(JSON.parse(config.body));
      return {
        ok: true,
        status: 200,
        json: async () => ({ user }),
      };
    }
    case "/checkout": {
      const isAuthorized = user.authorize(config.headers.Authorization);
      if (!isAuthorized) {
        return Promise.reject({
          ok: false,
          status: 401,
          json: async () => ({ message: "Not authorized" }),
        });
      }
      const shoppingCart = JSON.parse(config.body);
      // do whatever other things you need to do with this shopping cart
      return {
        ok: true,
        status: 200,
        json: async () => ({ success: true }),
      };
    }
    default: {
      throw new Error(`Unhandled request: ${url}`);
    }
  }
}

beforeAll(() => jest.spyOn(window, "fetch"));
beforeEach(() => window.fetch.mockImplementation(mockFetch));
```

```jsx
// __tests__/checkout.js
import * as React from "react";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

test('clicking "confirm" submits payment', async () => {
  const shoppingCart = buildShoppingCart();
  render(<Checkout shoppingCart={shoppingCart} />);

  userEvent.click(screen.getByRole("button", { name: /confirm/i }));

  expect(await screen.findByText(/success/i)).toBeInTheDocument();
});
```

내 성공 경로 테스트는 특별히 할 것이 없습니다. 실패 사례에 대한 fetch 모킹을 추가할 수도 있지만, 이 방식에 대해 매우 만족했습니다.

이 방식의 장점은 내 확신이 증가하고 대부분의 경우에 작성해야 할 테스트 코드가 더욱 줄어든다는 것입니다.

## 그러던 중에 msw를 발견했습니다.

[msw](https://github.com/mswjs/msw)는 "Mock Service Worker"의 약자입니다. 서비스 워커는 Node에서 작동하지 않고, 브라우저 기능입니다. 하지만 msw는 테스트 목적으로 Node에서도 지원합니다.

기본 아이디어는 다음과 같습니다: 모든 요청을 가로채서 실제 서버처럼 처리할 수 있는 모의 서버를 생성합니다. 제 개인적인 구현에서 이는 json 파일을 사용하여 "데이터베이스"를 "씨딩"하거나 faker나 test-data-bot과 같은 것을 사용하는 "빌더"를 만드는 것을 의미합니다. 그런 다음 서버 핸들러(Express API와 유사)를 만들고 그 모의 데이터베이스와 상호 작용합니다. 이 방법은 내 테스트를 빠르고 쉽게 작성할 수 있게 해줍니다(일단 설정을 마치면).

이전에 nock과 같은 것을 사용하여 이러한 작업을 해본 적이 있을 수 있습니다. 그러나 msw에 대해 멋진 점 (그리고 나중에 쓸 수도 있는 것)은 개발 중에 브라우저에서도 동일한 "서버 핸들러"를 사용할 수 있다는 것입니다. 이는 몇 가지 큰 이점을 가지고 있습니다:

- 엔드포인트가 준비되지 않았을 때
- 엔드포인트가 고장 났을 때
- 인터넷 연결이 느리거나 존재하지 않을 때

[Mirage](https://miragejs.com/)에 대해 들어보셨을 수 있는데, 많은 부분에서 비슷한 일을 합니다. 그러나 (현재로서는) mirage는 클라이언트에서 서비스 워커를 사용하지 않으며, msw를 설치했는지 여부에 관계없이 네트워크 탭이 동일하게 작동한다는 점이 마음에 듭니다. 그 차이점에 대해 더 알아보세요.

```jsx
// server-handlers.js
// this is put into here so I can share these same handlers between my tests
// as well as my development in the browser. Pretty sweet!
import { rest } from "msw"; // msw supports graphql too!
import * as users from "./users";

const handlers = [
  rest.get("/login", async (req, res, ctx) => {
    const user = await users.login(JSON.parse(req.body));
    return res(ctx.json({ user }));
  }),
  rest.post("/checkout", async (req, res, ctx) => {
    const user = await users.login(JSON.parse(req.body));
    const isAuthorized = user.authorize(req.headers.Authorization);
    if (!isAuthorized) {
      return res(ctx.status(401), ctx.json({ message: "Not authorized" }));
    }
    const shoppingCart = JSON.parse(req.body);
    // do whatever other things you need to do with this shopping cart
    return res(ctx.json({ success: true }));
  }),
];

export { handlers };
```

```jsx
// test/server.js
import { rest } from "msw";
import { setupServer } from "msw/node";
import { handlers } from "./server-handlers";

const server = setupServer(...handlers);
export { server, rest };
```

(→ 24년 5월 기준 `import {rest} from 'msw'`대신 `import {http} from 'msw'` 사용)

```jsx
// test/setup-env.js
// add this to your setupFilesAfterEnv config in jest so it's imported for every test file
import { server } from "./server.js";

beforeAll(() => server.listen());
// if you need to add a handler after calling setupServer for some specific test
// this will remove that handler for the rest of them
// (which is important for test isolation):
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

테스트 코드는 다음과 같습니다.

```jsx
// __tests__/checkout.js
import * as React from "react";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

test('clicking "confirm" submits payment', async () => {
  const shoppingCart = buildShoppingCart();
  render(<Checkout shoppingCart={shoppingCart} />);

  userEvent.click(screen.getByRole("button", { name: /confirm/i }));

  expect(await screen.findByText(/success/i)).toBeInTheDocument();
});
```

fetch 모킹보다 이 방법이 더 만족스러운 이유는 다음과 같습니다.

1. 가져오기 응답 속성과 헤더의 구현 세부 사항에 대해 걱정할 필요가 없습니다.
2. **‘fetch’**를 호출하는 방법에 문제가 있으면 서버 핸들러가 호출되지 않고 내 테스트가 (정확하게) 실패하여, 결함 있는 코드를 배포하는 것을 방지할 수 있습니다.
3. 이와 동일한 서버 핸들러를 개발 중에도 재사용할 수 있습니다!

## 코로케이션과 오류/엣지 케이스 테스트에 관하여

이 접근 방식에 대한 하나의 합리적인 우려는 모든 서버 핸들러를 한 곳에 모아두고, 그 서버 핸들러에 의존하는 테스트들이 전혀 다른 파일에 위치하게 되어 코로케이션의 이점을 잃어버린다는 것입니다.

우선, 중요하고 테스트에 특유한 요소들만 코로케이션하는 것이 좋습니다. 모든 설정을 각 테스트마다 중복해서 가질 필요는 없습니다. 유일한 부분만 필요합니다. 그래서 "성공 경로"에 해당하는 것들은 일반적으로 설정 파일에 포함시켜 테스트 자체에서는 제거하는 것이 더 낫습니다. 그렇지 않으면 너무 많은 잡음이 생겨 실제로 무엇이 테스트되고 있는지 분리하기 어렵습니다.

그렇다면 엣지 케이스와 오류에 대해서는 어떨까요? 이 경우, MSW는 테스트 중에 추가적인 서버 핸들러를 런타임에 추가할 수 있는 기능을 제공하며, 서버를 원래의 핸들러로 재설정하여 (런타임 핸들러를 효과적으로 제거하여) 테스트 격리를 유지합니다. 예를 들어 다음과 같습니다:

```jsx
// __tests__/checkout.js
import * as React from "react";
import { server, rest } from "test/server";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

// happy path test, no special server stuff
test('clicking "confirm" submits payment', async () => {
  const shoppingCart = buildShoppingCart();
  render(<Checkout shoppingCart={shoppingCart} />);

  userEvent.click(screen.getByRole("button", { name: /confirm/i }));

  expect(await screen.findByText(/success/i)).toBeInTheDocument();
});

// edge/error case, special server stuff
// note that the afterEach(() => server.resetHandlers()) we have in our
// setup file will ensure that the special handler is removed for other tests
test("shows server error if the request fails", async () => {
  const testErrorMessage = "THIS IS A TEST FAILURE";
  server.use(
    rest.post("/checkout", async (req, res, ctx) => {
      return res(ctx.status(500), ctx.json({ message: testErrorMessage }));
    })
  );
  const shoppingCart = buildShoppingCart();
  render(<Checkout shoppingCart={shoppingCart} />);

  userEvent.click(screen.getByRole("button", { name: /confirm/i }));

  expect(await screen.findByRole("alert")).toHaveTextContent(testErrorMessage);
});
```

따라서 코로케이션이 필요한 곳에는 코로케이션을, 추상화가 필요한 곳에는 추상화를 사용할 수 있습니다.

## 결론

**‘msw’**와 관련하여 할 일이 분명히 더 많지만, 일단 여기서 마무리하겠습니다. **‘msw’**를 실제로 보고 싶다면, 제가 진행하는 4부작 워크숍 "Build React Apps" (EpicReact.Dev에 포함)에서 사용되며, GitHub에서 모든 자료를 찾아볼 수 있습니다.

이 테스트 방법의 정말 멋진 측면 중 하나는 구현 세부 사항으로부터 멀리 떨어져 있기 때문에, 상당한 리팩토링을 수행할 수 있으며, 테스트가 사용자 경험을 깨뜨리지 않았다는 확신을 줄 수 있다는 것입니다. 바로 이것이 테스트가 존재하는 이유입니다! 이런 일이 일어날 때 정말 좋습니다:

Kent C. Dodds

@kentcdodds

최근에 내 앱에서 인증 방식을 완전히 변경했는데, 테스트 유틸리티에 약간의 수정만 필요했고, 모든 테스트(단위, 통합, E2E)가 통과하여 사용자 경험이 변경에 영향을 받지 않았다는 확신을 주었습니다. 바로 이것이 테스트가 존재하는 이유죠!

Dillon

@d11erh
오늘은 React 컴포넌트를 리팩토링하는 데 하루를 보냈습니다. react-testing-library로 잘 작성된 테스트(감사합니다 @kentcdodds)는 큰 확신을 주었고 미묘한 오류를 잡는 데 도움이 되었습니다.

결론: 좋은 단위 테스트는 정말 중요합니다!

---
