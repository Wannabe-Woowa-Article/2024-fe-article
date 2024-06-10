## 🔗 [API Layer & Fetch Functions](https://profy.dev/article/react-architecture-api-layer-and-fetch-functions)

### 🗓️ 번역 날짜: 2024.06.10

### 🧚 번역한 크루: 러기(박정우)

---

## 번역 제목

# API계층 & Fetch 함수

리액트의 의견이 없는 특성은 양날의 검과 같습니다:

- 한편으로는 선택의 자유가 있습니다.
- 다른 한편으로 많은 프로젝트들이 맞춤형이고 종종 지저분한 구조로 끝나곤 합니다.

이 글은 소프트웨어 아키텍처와 리액트 앱에 관한 연재의 두 번째 부분으로, 우리는 나쁜 관행이 많은 코드 베이스를 단계별로 리팩토링해 나갈 것입니다.

[이전에는 애플리케이션 내의 모든 요청 사이에서 API 기본 URL과 같은 공통 구성을 공유하기 위해 API 클라이언트를 추출했습니다.](https://profy.dev/article/react-architecture-api-client)

이번 글에서는 API 관련 코드를 UI 컴포넌트에서 분리하는 데 초점을 맞추고 싶습니다.

![alt text](https://ik.imagekit.io/87wct6jq4ql/tr:w-1280/https://media.graphassets.com/RUDy4wgkRquKJoBUs6Pn)

## 나쁜 코드의 예시: API와 UI 코드가 섞여 있음

나쁜 코드 예제를 살펴보겠습니다. 여기 이전 글의 첫 번째 리팩토링 단계의 결과가 있습니다:

두 API 엔드포인트에서 데이터를 가져와서 데이터를 렌더링하는 컴포넌트입니다.

```tsx
import { useEffect, useState } from "react";
import { useParams } from "react-router";
import { apiClient } from "@/api/client";
import { LoadingSpinner } from "@/components/loading";
import { ShoutList } from "@/components/shout-list";
import { UserResponse, UserShoutsResponse } from "@/types";
import { UserInfo } from "./user-info";

export function UserProfile() {
  const { handle } = useParams<{ handle: string }>();

  const [user, setUser] = useState<UserResponse>();
  const [userShouts, setUserShouts] = useState<UserShoutsResponse>();
  const [hasError, setHasError] = useState(false);

  useEffect(() => {
    apiClient
      .get<UserResponse>(`/user/${handle}`)
      .then((response) => setUser(response.data))
      .catch(() => setHasError(true));

    apiClient
      .get<UserShoutsResponse>(`/user/${handle}/shouts`)
      .then((response) => setUserShouts(response.data))
      .catch(() => setHasError(true));
  }, [handle]);

  if (hasError) {
    return <div>An error occurred</div>;
  }

  if (!user || !userShouts) {
    return <LoadingSpinner />;
  }

  return (
    <div className="max-w-2xl w-full mx-auto flex flex-col p-6 gap-6">
      <UserInfo user={user.data} />
      <ShoutList users={[user.data]} shouts={userShouts.data} images={userShouts.included} />
    </div>
  );
}
```

## 이게 왜 나쁜 코드인가요?

간단합니다. API 요청과 UI 코드가 석여있습니다. API코드와 UI코드가 여기저기 조금씩 존재합니다.

![alt text](https://ik.imagekit.io/87wct6jq4ql/tr:w-1280/https://media.graphassets.com/Il7QrSIMRFuFq782XMOg)

사실상, UI는 데이터를 어떻게 가져오는지는 신경쓰지 않아야 합니다.

- GET, POST, 또는 PATCH 요청이 보내지는지 신경 쓰지 않아야 합니다.
- 엔드포인트의 정확한 경로가 무엇인지 신경 쓰지 않아야 합니다.
- 요청 매개변수가 API에 어떻게 전달되는지 신경 쓰지 않아야 합니다.
- 심지어 REST API에 연결되는지 웹소켓에 연결되는지조차 정말로 신경 쓰지 않아야 합니다.

이 모든 것은 UI의 관점에서 구현 세부사항이어야 합니다.

## 해결책: API 연결 함수를 추출한다.

API에 연결하는 함수를 별도의 장소로 추출하면 API 관련 코드와 UI 코드의 결합을 크게 줄일 수 있습니다.

이 함수들은 다음과 같은 구현 세부사항을 숨깁니다:

- 요청 방식
- 엔드포인트 경로
- 데이터 유형

분명한 분리를 위해 이러한 함수들을 전역 API 폴더에 위치시킬 것입니다.

```tsx
// src/api/user.ts

import { User, UserShoutsResponse } from "@/types";
import { apiClient } from "./client";

async function getUser(handle: string) {
  const response = await apiClient.get<{ data: User }>(`/user/${handle}`);
  return response.data;
}

async function getUserShouts(handle: string) {
  const response = await apiClient.get<UserShoutsResponse>(`/user/${handle}/shouts`);
  return response.data;
}

export default { getUser, getUserShouts };
```

이제 컴포넌트 안에서 fetch 함수를 사용합니다.

```tsx
import UserApi from "@/api/user";

...

export function UserProfile() {
  const { handle } = useParams<{ handle: string }>();

  const [user, setUser] = useState<User>();
  const [userShouts, setUserShouts] = useState<UserShoutsResponse>();
  const [hasError, setHasError] = useState(false);

  useEffect(() => {
    if (!handle) {
      return;
    }

    UserApi.getUser(handle)
      .then((response) => setUser(response.data))
      .catch(() => setHasError(true));

    UserApi.getUserShouts(handle)
      .then((response) => setUserShouts(response))
      .catch(() => setHasError(true));
  }, [handle]);

  ...
```

## UI코드와 API코드를 분리하는것이 왜 더 좋은 코드죠?

맞습니다. 코드의 변화가 크지 않습니다. 우리는 기본적으로 약간의 코드만 옮겼을 뿐입니다.

결과물이 처음에만 조금 깔끔하게 보일 수 있습니다.

```tsx
// before
apiClient.get<UserResponse>(`/user/${handle}`);

// after
UserApi.getUser(handle);
```

그러나 사실상, 우리는 UI코드와 API 관련 함수를 효과적으로 분리하기 시작했습니다.

반복해서 말하는 것이 지겹지 않습니다: 이제 컴포넌트는 다음과 같은 많은 API 관련 세부사항을 알지 못합니다.

- 요청 방식 GET
- 데이터 유형 UserResponse의 정의
- 엔드포인트 경로 /user/some-handle
- 또는 핸들이 URL 매개변수로 API에 전달된다는 사실

대신, 타입이 지정된 결과를 프로미스로 감싼 간단한 함수 `UserApi.getUser(handle)`를 호출합니다.

또한, 이러한 가져오기 함수들을 여러 컴포넌트에서 재사용하거나 클라이언트 측 또는 서버 측 렌더링과 같은 다른 렌더링 접근 방식에 사용할 수 있습니다.

## 다음 리팩터링 단계

네, 우리는 API로 부터 UI코드를 분리하는데 첫 번째 큰 진전을 이루었습니다. API계층을 도입하고, UI컴포넌트에 구현된 많은 세부사항을 제거하였습니다.

그러나 여전히 API 계층과 우리 컴포넌트 사이에 상당한 결합이 있습니다. 예를 들어, 우리는 컴포넌트 내에서 응답 데이터를 변환합니다:

![alt text](https://ik.imagekit.io/87wct6jq4ql/tr:w-1280/https://media.graphassets.com/lJjVG80BRVi817s4vOwF)

이게 인상적인 예시가 아닐 수 있습니다. 같은 코드의 다른 예시를 보겠습니다.

![alt text](https://ik.imagekit.io/87wct6jq4ql/tr:w-1280/https://media.graphassets.com/6FUzRWpTSbmufwlBQQy5)

여기에서 우리는 피드의 응답에 사용자와 이미지를 포함하는 필드가 있음을 볼 수 있습니다.

예쁘지는 않지만, 때때로 우리는 이러한 API를 다루어야 합니다.

그런데 왜 컴포넌트가 이것을 알아야 할까요?

어쨌든, 이 문제는 다음 아티클에서 다루겠습니다.
