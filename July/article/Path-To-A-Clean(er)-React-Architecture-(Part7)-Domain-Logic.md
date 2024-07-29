## 🔗 [Path To A Clean(er) React Architecture (Part 7) - Domain Logic](https://profy.dev/article/react-architecture-domain-logic)

### 🗓️ 번역 날짜: 2024.07.17

### 🧚 번역한 크루: 마스터위(명재위)

---

Johannes Kettmann  
Updated On July 5, 2024  
Keyword : **React**, **Domain**, **Refactoring**, **Component**, **Structure**

리액트의 비의존적인 특성은 양날의 검입니다:

- 한편으로는 선택의 자유를 제공합니다.
- 다른 한편으로는 많은 프로젝트가 커스텀화되고 종종 혼란스러운 아키텍처로 끝나게 됩니다.

이 글은 소프트웨어 아키텍처와 리액트 앱에 대한 시리즈의 일곱 번째 부분으로, 많은 잘못된 관행이 있는 코드베이스를 단계적으로 리팩토링하는 과정을 다룹니다.

이전 글로는,

- [we created the initial API layer and extracted fetch functions](https://profy.dev/article/react-architecture-api-layer-and-fetch-functions)
- [added data transformations](https://profy.dev/article/react-architecture-api-layer-and-data-transformations)
- [separated domain entities and DTOs](https://profy.dev/article/react-architecture-domain-entities-and-dtos)
- [introduced infrastructure services using dependency injection and](https://profy.dev/article/react-architecture-infrastructure-services-and-dependency-injection)
- [separated business logic from the components.](https://profy.dev/article/react-architecture-business-logic-and-dependency-injection)

이것은 우리의 UI 코드를 서버로부터 분리하고, 비즈니스 로직을 UI 프레임워크와 독립적으로 만들며, 테스트 가능성을 높이는 데 도움이 되었습니다.

하지만 아직 끝나지 않았습니다.

이 글에서는 애플리케이션의 핵심인 도메인에 초점을 맞춥니다.

도메인 로직은 사용자 객체와 같은 도메인 모델에서 작동하는 코드입니다. 추상적으로 들릴 수 있지만, 실용적인 예제를 통해 그 의미를 살펴보겠습니다. 목표는 이 로직을 컴포넌트에서 분리하고, 리포지토리의 특정 위치로 이동시키며, 단위 테스트하는 것입니다.

유틸리티 파일이나 커스텀 훅이 아닌 다른 곳에 특정 로직을 배치할 수 있는 위치를 궁금해한 적이 있다면, 이 글이 흥미로울 것입니다.

[유튜브 링크](https://youtu.be/r2zuEqPFDDY)

## Table of Contents

- 문제가 있는 코드 예시: 컴포넌트 내부의 도메인 로직
- 문제: 혼합된 관심사와 낮은 테스트 가능성
- 해결책: 로직을 도메인 레이어로 추출
  - 1단계: 도메인 레이어에 함수 생성
  - 2단계: 컴포넌트 업데이트
  - 3단계: 도메인 로직 단위 테스트
- 또 다른 문제 있는 코드 예시
  - 문제: 도메인 로직과 비즈니스 로직의 혼합
  - 해결책: 도메인 레이어에 함수 생성
- 도메인 로직 추출의 장단점
  - 장점
  - 단점
- 다음 리팩토링 단계

## 문제가 있는 코드 예시: 컴포넌트 내부의 도메인 로직

문제가 있는 코드 예시를 살펴보겠습니다. 다음은 Shouts(일명 트윗 또는 게시물) 목록을 렌더링하는 컴포넌트입니다.

> 이 리팩토링 [전](https://github.com/jkettmann/react-architecture/tree/81378b16b13f0ff916498e276979a99353f67604)과 [후](https://github.com/jkettmann/react-architecture/tree/step-7-domain-logic)의 소스 코드를 확인할 수 있습니다. 또한 이 글의 모든 변경 사항에 대한 개요도 [여기](https://github.com/jkettmann/react-architecture/pull/14/files/81378b16b13f0ff916498e276979a99353f67604..65afcc8181d9971cebdbbea6b0d3737f8968e9de)에 있습니다.

```tsx
// src/components/shout-list/shout-list.tsx

import { Shout } from '@/components/shout';
import { Image } from '@/domain/media';
import { Shout as IShout } from '@/domain/shout';
import { User } from '@/domain/user';

interface ShoutListProps {
  shouts: IShout[];
  images: Image[];
  users: User[];
}

export function ShoutList({ shouts, users, images }: ShoutListProps) {
  return (
    <ul className='flex flex-col gap-4 items-center'>
      {shouts.map((shout) => {
        const author = users.find((u) => u.id === shout.authorId);
        const image = shout.imageId
          ? images.find((i) => i.id === shout.imageId)
          : undefined;

        return (
          <li key={shout.id} className='max-w-sm w-full'>
            <Shout shout={shout} author={author} image={image} />
          </li>
        );
      })}
    </ul>
  );
}
```

- 컴포넌트는 shouts 목록과 해당 shouts에 속하는 사용자 및 이미지 목록을 받습니다.
- shouts를 반복합니다.
- 각 shout에 대해 해당 작성자와 이미지를 가져옵니다.
- 마지막으로 각 shout에 대한 리스트 아이템 요소를 반환합니다.

## 문제: 혼합된 관심사와 낮은 테스트 가능성

이 컴포넌트의 문제는 Shout의 작성자와 이미지를 찾는 부분에 있습니다.

```tsx
const author = users.find((u) => u.id === shout.authorId);
const image = shout.imageId
  ? images.find((i) => i.id === shout.imageId)
  : undefined;
```

이는 사용자와 이미지 엔터티에 대한 작업입니다. 즉, 이 코드는 우리 애플리케이션에서 사용자와 이미지 조회가 어떻게 작동하는지를 정의합니다.

또한 Shout에 이미지가 없는 경우가 있을 수 있습니다. 그래서 삼항 연산자가 필요합니다. 제 관점에서 이는 가장 아름답거나 읽기 쉬운 코드가 아닙니다.

## 해결책: 로직을 도메인 레이어로 추출

[이전 글에서](https://profy.dev/article/react-architecture-domain-entities-and-dtos), 우리는 애플리케이션의 핵심 모델인 `User`나 `Image`를 TypeScript 인터페이스로 보유하는 도메인 레이어를 만들었습니다. 이 레이어는 도메인 엔터티에서 작동하는 로직을 배치하기에 훌륭한 장소임이 드러났습니다.

예를 보여드리겠습니다...

## 1단계: 도메인 레이어에 함수 생성

위의 예제 코드를 가지고 `getUserById`라는 새로운 도메인 함수를 만들 수 있습니다.

```tsx
// src/domain/user/user.ts

export interface User {
  id: string;
  handle: string;
  avatar: string;
  info?: string;
  blockedUserIds: string[];
  followerIds: string[];
}

export function getUserById(users?: User[], userId?: string) {
  if (!userId || !users) return;
  return users.find((u) => u.id === userId);
}
```

이 예제에서 두 매개변수를 선택 사항으로 만들어 함수의 유연성을 높였습니다. 이는 비동기 데이터나 `react-query` 같은 라이브러리를 사용할 때 특히 유용합니다.

이미지에 대해서도 동일한 작업을 할 수 있습니다.

```tsx
// src/domain/media/media.ts

export interface Image {
  id: string;
  url: string;
}

export function getImageById(images?: Image[], imageId?: string) {
  if (!imageId || !images) return;
  return images.find((i) => i.id === imageId);
}
```

## 2단계: 컴포넌트 업데이트

이제 컴포넌트에서의 장점이 명확해집니다:

```tsx
// src/components/shout-list/shout-list.tsx

import { Shout } from "@/components/shout";
import { Image, getImageById } from "@/domain/media"; // 추가
import { Shout as IShout } from "@/domain/shout";
import { User, getUserById } from "@/domain/user"; // 추가

...

export function ShoutList({ shouts, users, images }: ShoutListProps) {
  return (
    <ul className="flex flex-col gap-4 items-center">
      {shouts.map((shout) => (
        <li key={shout.id} className="max-w-sm w-full">
          <Shout
            shout={shout}
            author={getUserById(users, shout.authorId)} // 추가
            image={getImageById(images, shout.imageId)} // 추가
          />
        </li>
      ))}
    </ul>
  );
}
```

1. 코드는 더 읽기 쉬워졌습니다. 사용자의 생각에 `users.find(...)` 함수들을 ID 조회로 번역할 필요가 없기 때문입니다.
2. 삼항 연산자를 제거하여 가독성을 더욱 향상시켰습니다.
3. 로직을 테스트하기 훨씬 간단해졌습니다.

"테스트하기 간단해졌다"는 것은 무엇을 의미할까요?

## 3단계: 도메인 로직 단위 테스트

원래 코드에서는 `shouts`, `users`, 그리고 `images` props의 다양한 버전을 전달하여 로직을 테스트해야 했습니다. 다음은 원래 코드의 간단한 복습입니다:

```tsx
export function ShoutList({ shouts, users, images }: ShoutListProps) {
  return (
    <ul className='flex flex-col gap-4 items-center'>
      {shouts.map((shout) => {
        const author = users.find((u) => u.id === shout.authorId);
        const image = shout.imageId
          ? images.find((i) => i.id === shout.imageId)
          : undefined;

        return (
          <li key={shout.id} className='max-w-sm w-full'>
            <Shout shout={shout} author={author} image={image} />
          </li>
        );
      })}
    </ul>
  );
}
```

특히 `imageId`를 포함하는 Shout와 포함하지 않는 Shout에 대해 다른 테스트 케이스를 제공해야 합니다. 작성자가 없는 경우와 같은 다른 극단적인 경우도 테스트하고 싶을 수 있습니다. 그 외에도 React Testing Library와 같은 도구를 사용하여 컴포넌트를 테스트해야 하므로 추가적인 오버헤드가 발생합니다.

이에 비해 새로운 도메인 함수에 대한 단위 테스트 작성은 매우 간단합니다:

```tsx
// src/domain/media/media.test.ts

import { describe, expect, it } from 'vitest';

import { getImageById } from './media';

const mockImage = {
  id: '1',
  url: 'test',
};

describe('Media domain', () => {
  describe('getImageById', () => {
    it('should be able to get image by id', () => {
      const image = getImageById([mockImage], '1');
      expect(image).toEqual(mockImage);
    });

    it('should return undefined if image is not found', () => {
      const image = getImageById([{ ...mockImage, id: '2' }], '1');
      expect(image).toEqual(undefined);
    });

    it('should return undefined if provided images are not defined', () => {
      const image = getImageById(undefined, '1');
      expect(image).toEqual(undefined);
    });

    it('should return undefined if provided image id is not defined', () => {
      const image = getImageById([mockImage], undefined);
      expect(image).toEqual(undefined);
    });
  });
});
```

React Testing Library가 자주 요구하는 설정 코드 없이 모든 가능한 분기를 테스트할 수 있으며, 이러한 테스트는 매우 빠르게 실행됩니다.

이로 인해, 그렇지 않으면 필요할 (더 비싼) 통합 테스트 중 일부를 제거할 수 있는 기회를 제공합니다.

## 또 다른 문제 있는 코드 예시

이 접근 방식을 또 다른 예제로 확고히 해보겠습니다. [이전 글에서 우리는 컴포넌트에서 비즈니스 로직을 제거하는 방법](https://profy.dev/article/react-architecture-business-logic-and-dependency-injection)으로 use-case 함수를 노출하는 훅을 다루었습니다. 이 함수는

- 먼저 몇 줄의 유효성 검사 로직을 실행하고,
- 그 다음 일련의 서비스 호출을 수행합니다.

```tsx
// src/application/reply-to-shout/reply-to-shout.ts

import { useCallback } from "react";

import MediaService from "@/infrastructure/media";
import ShoutService from "@/infrastructure/shout";
import UserService from "@/infrastructure/user";

...

export async function replyToShout(
  { recipientHandle, shoutId, message, files }: ReplyToShoutInput,
  { getMe, getUser, saveImage, createReply, createShout }: typeof dependencies
) {
  const me = await getMe();
  if (me.numShoutsPastDay >= 5) {
    return { error: ErrorMessages.TooManyShouts };
  }

  const recipient = await getUser(recipientHandle);
  if (!recipient) {
    return { error: ErrorMessages.RecipientNotFound };
  }
  if (recipient.blockedUserIds.includes(me.id)) {
    return { error: ErrorMessages.AuthorBlockedByRecipient };
  }

  try {
    let image;
    if (files?.length) {
      image = await saveImage(files[0]);
    }

    const newShout = await createShout({
      message,
      imageId: image?.id,
    });

    await createReply({
      shoutId,
      replyId: newShout.id,
    });

    return { error: undefined };
  } catch {
    return { error: ErrorMessages.UnknownError };
  }
}

export function useReplyToShout() {
  return useCallback(
    (input: ReplyToShoutInput) => replyToShout(input, dependencies),
    []
  );
}
```

## 문제: 도메인 로직과 비즈니스 로직의 혼합

문제의 코드 라인은 데이터 유효성 검사에 있습니다:

```tsx
const me = await getMe();
if (me.numShoutsPastDay >= 5) {
  // 여기
  return { error: ErrorMessages.TooManyShouts };
}

const recipient = await getUser(recipientHandle);
if (!recipient) {
  return { error: ErrorMessages.RecipientNotFound };
}
if (recipient.blockedUserIds.includes(me.id)) {
  // 여기
  return { error: ErrorMessages.AuthorBlockedByRecipient };
}
```

이들은 다시 도메인 엔티티(`User`와 `Me`(현재 사용자))에 대한 작업입니다.

## 해결책: 도메인 레이어에 함수 생성

이 로직을 도메인 레이어로 이동할 수 있습니다:

```tsx
// src/domain/me/me.ts

import { User } from '@/domain/user';

export const MAX_NUM_SHOUTS_PER_DAY = 5;

export interface Me extends User {
  numShoutsPastDay: number;
}

export function hasExceededShoutLimit(me: Me) {
  return me.numShoutsPastDay >= MAX_NUM_SHOUTS_PER_DAY;
}
```

이제 도메인은 사용자가 샤우트 제한을 초과했는지 여부를 결정하는 역할을 합니다. 이를 통해 UI와 가까운 코드에서 다음과 같은 구현 세부 사항을 제거할 수 있습니다.

- 허용된 샤우트 수
- 이 임계값의 간격(이 경우 하루)

`blockedUserIds` 체크도 동일하게 할 수 있습니다:

```tsx
// src/domain/user/user.ts

export interface User {
  id: string;
  handle: string;
  avatar: string;
  info?: string;
  blockedUserIds: string[];
  followerIds: string[];
}

...

export function hasBlockedUser(user?: User, userId?: string) {
  if (!user || !userId) return false;
  return user.blockedUserIds.includes(userId);
}
```

이제 유스케이스 함수는 더 간단하게 읽히며, 도메인과 관련된 구현 세부 사항(예: 사용자가 주어진 간격 내에 몇 번 샤우트할 수 있는지)을 덜 포함합니다.

```tsx
// src/application/reply-to-shout/reply-to-shout.ts

import { hasExceededShoutLimit } from "@/domain/me";
import { hasBlockedUser } from "@/domain/user";

export async function replyToShout(
  { recipientHandle, shoutId, message, files }: ReplyToShoutInput,
  { getMe, getUser, saveImage, createReply, createShout }: typeof dependencies
) {
  const me = await getMe();
  if (hasExceededShoutLimit(me)) { // 여기
    return { error: ErrorMessages.TooManyShouts };
  }

  const recipient = await getUser(recipientHandle);
  if (!recipient) {
    return { error: ErrorMessages.RecipientNotFound };
  }
  if (hasBlockedUser(recipient, me.id)) { // 여기
    return { error: ErrorMessages.AuthorBlockedByRecipient };
  }
```

마지막으로, 우리는 이 로직을 단순한 단위 테스트로 다시 테스트할 수 있습니다:

```tsx
// src/domain/user/user.test.ts

import { describe, expect, it } from "vitest";

import { getUserById, hasBlockedUser } from "./user";

const mockUser = {
  id: "1",
  handle: "test",
  avatar: "test",
  numShoutsPastDay: 0,
  blockedUserIds: [],
  followerIds: [],
};

describe("User domain", () => {
  describe("getUserById", () => { ... });

  describe("hasBlockedUser", () => {
    it("should be false if user has not blocked the user", () => {
      const user = { ...mockUser, blockedUserIds: ["2"] };
      const hasBlocked = hasBlockedUser(user, "3");
      expect(hasBlocked).toEqual(false);
    });

    it("should be true if user has blocked the user", () => {
      const user = { ...mockUser, blockedUserIds: ["2"] };
      const hasBlocked = hasBlockedUser(user, "2");
      expect(hasBlocked).toEqual(true);
    });

    it("should be false if user is not defined", () => {
      const hasBlocked = hasBlockedUser(undefined, "2");
      expect(hasBlocked).toEqual(false);
    });

    it("should be false if user id is not defined", () => {
      const hasBlocked = hasBlockedUser(mockUser, undefined);
      expect(hasBlocked).toEqual(false);
    });
  });
});
```

## 도메인 로직 추출의 장단점

이 접근 방식의 장단점을 간략히 논의해 보겠습니다.

### 장점

- 유틸리티 함수 감소: 도메인 레이어가 없으면 위와 같은 로직을 어디에 넣어야 할지 불명확할 수 있습니다. 제 경험상, 이러한 로직은 종종 컴포넌트에 흩어져 있거나 유틸리티 파일에 존재합니다. 그러나 유틸리티 파일은 다양한 공유 코드의 덤핑장이 되기 쉽습니다.
- 가독성: `users.find(({ id }) => id === userId)`는 ID 조회로 변환하는 데 약간의 인지적 부담이 필요합니다. 대신 `getUserById(users, userId)`를 읽는 것이 훨씬 더 설명적입니다. 특히 이러한 줄이 여러 개 모여 있을 때(예: 컴포넌트 상단) 효과적입니다.
- 테스트 가능성: if/switch 문이나 삼항 연산자를 사용하는 코드를 자주 찾을 수 있습니다. 이러한 각각의 경우는 여러 테스트 분기가 필요함을 의미합니다. 모든 엣지 케이스에 대해 단위 테스트를 작성하는 것이 훨씬 더 쉽고, 꼭 필요한 통합 테스트의 수를 줄일 수 있습니다.
- 재사용성 : 이러한 작은 로직 조각들은 별도의 함수로 추출할 가치가 없어 보일 수 있습니다. 그러면 코드 내에서 무한히 반복됩니다. 그러나 요구 사항의 작은 변경은 쉽게 더 큰 리팩토링으로 이어질 수 있습니다.
  검색 가능성: `users.find(({ id }) => id === userId)`와 같은 코드를 전역 검색하기는 간단하지 않습니다. 코드가 다른 변수 이름으로 다양한 방식으로 작성될 수 있기 때문입니다. 반면에 getUserById는 전역 검색이 간단합니다.

### 단점

- 연습: 모든 개발자가 다양한 로직을 생각하는 데 익숙한 것은 아닙니다. 따라서 코드 베이스를 일관되게 유지하려면 문서화와 교육이 필요할 수 있습니다.

- 오버헤드: 위의 예시 중 하나에서 볼 수 있듯이, 특정 컴포넌트가 요구하는 것보다 도메인 함수를 더 유연하게 만들었습니다(여기서 `getUserById` 함수의 `users`와 `userId` 매개변수를 선택 사항으로 만들었습니다). 단위 테스트로 이러한 경우를 커버했기 때문에 더 많은 코드를 도입하여 **유지보수 노력**이 증가했습니다. 하지만 이는 예를 들어 서버에서 예상치 못한 응답 데이터를 갑자기 마주쳤을 때 도움이 될 수 있습니다.

## 다음 리팩토링 단계

지금까지는 React 자체 외에 다른 도구를 도입하지 않았습니다. 제 관점에서 이는 중요한데, 여러 기술 스택이 혼합되면 지식을 다른 기술 스택으로 이전하기 어려울 수 있기 때문입니다.

하지만 이제 실제 프로덕션 React 애플리케이션의 현실을 직시할 때입니다.

현장에서 가장 많이 사용되는 도구 중 하나는 `react-query`(또는 RTK query, SWR과 같은 다른 서버 상태 관리 라이브러리)입니다. 이러한 도구들은 우리의 아키텍처에 어떻게 적합할까요? 다음 글에서 다루겠습니다.
