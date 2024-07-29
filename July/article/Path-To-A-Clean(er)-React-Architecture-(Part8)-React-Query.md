## 🔗 [Path To A Clean(er) React Architecture (Part 8) - React-Query](https://profy.dev/article/react-architecture-tanstack-query)

### 🗓️ 번역 날짜: 2024.07.21

### 🧚 번역한 크루: 마스터위(명재위)

---

## 깔끔한 리액트 설계로의 길(Part8) React-Query

Johannes Kettmann  
Updated On July 19, 2024  
Keyword : **React**, **Refactoring**, **Component**, **React-Query**

리액트의 비의존적인 특성은 양날의 검입니다:

- 한편으로는 선택의 자유를 제공합니다.
- 다른 한편으로는 많은 프로젝트가 커스텀화되고 종종 혼란스러운 아키텍처로 끝나게 됩니다.

이 글은 소프트웨어 아키텍처와 리액트 앱에 대한 시리즈의 여덟 번째 부분으로, 많은 잘못된 관행이 있는 코드베이스를 단계적으로 리팩토링하는 과정을 다룹니다.

이전의 글에서는,

- [we created the initial API layer and extracted fetch functions](https://profy.dev/article/react-architecture-api-layer-and-fetch-functions)
- [added data transformations](https://profy.dev/article/react-architecture-api-layer-and-data-transformations)
- [separated domain entities and DTOs](https://profy.dev/article/react-architecture-domain-entities-and-dtos)
- [introduced infrastructure services using dependency injection and](https://profy.dev/article/react-architecture-infrastructure-services-and-dependency-injection)
- separated [business logic](https://profy.dev/article/react-architecture-business-logic-and-dependency-injection) and [domain logic](https://profy.dev/article/react-architecture-business-logic-and-dependency-injection) from the components.

이를 통해 우리는 UI 코드를 서버로부터 분리하고, 비즈니스 로직을 UI 프레임워크와 독립적으로 만들며, 테스트 가능성을 높일 수 있었습니다.

하지만 우리는 아직 프로덕션 리액트 앱에서 가장 중요한 도구 중 하나인 react-query 또는 다른 서버 상태 관리 라이브러리를 소개하지 않았습니다.

이것이 이번 글의 주제입니다.

시작하기 전에 간단한 메모: react-query의 설정 방법이나 기능을 자세히 설명하지는 않겠습니다. 이에 익숙하다고 가정합니다. 그렇지 않다면 문서나 많은 튜토리얼을 참고할 수 있습니다.

또한 이 시리즈의 다른 글을 읽지 않았다면, 계속하기 전에 읽어보는 것을 권장합니다.

## 목차

- 문제가 있는 코드 예시 1: 서버 데이터를 수동으로 관리

  - 문제: 보일러플레이트 및 상태 관리
  - 해결책: react-query 훅 생성

- 문제가 있는 코드 예시 2: 비즈니스 로직과 react-query

  - 문제: 중복 요청
  - 해결책: react-query 데이터와 변이 사용

- 다음 리팩토링 단계

[유튜브 링크](https://youtu.be/A2FOOudW5GQ)

## 문제가 있는 코드 예시 1: 서버 데이터를 수동으로 관리

문제가 있는 코드 예시를 살펴보겠습니다. 다음은 현재 로그인된 사용자를 가져오고, 사용자가 메시지(aka shout)에 답장할 수 있는 폼을 다이얼로그에 렌더링하는 컴포넌트입니다.

[전체 소스 코드를 포함한 모든 변경 사항은 여기에 있습니다.](https://github.com/jkettmann/react-architecture/pull/15)

```tsx
import { useEffect, useState } from "react";

import { isAuthenticated as isUserAuthenticated } from "@/domain/me";
import UserService from "@/infrastructure/user";

...

export function ReplyDialog({
  recipientHandle,
  children,
  shoutId,
}: ReplyDialogProps) {
  const [open, setOpen] = useState(false);
  const [isLoading, setIsLoading] = useState(true);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [hasError, setHasError] = useState(false);

  ...
 // 삭제될 부분
  useEffect(() => {
    UserService.getMe()
      .then(isUserAuthenticated)
      .then(setIsAuthenticated)
      .catch(() => setHasError(true))
      .finally(() => setIsLoading(false));
  }, []);
  // 삭제될 부분

  if (hasError || !isAuthenticated) {
    return <LoginDialog>{children}</LoginDialog>;
  }

  async function handleSubmit(event: React.FormEvent<ReplyForm>) {
      // 이 부분을 나중에 다시 볼겁니다
  }

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      {/* 컴포넌트의 나머지 부분 */}
    </Dialog>
  );
}
```

## 문제: 보일러플레이트 및 상태 관리

react-query를 사용해 본 적이 있다면 아마도 이 문제를 알고 있을 것입니다.

- 로딩 및 오류 상태를 수동으로 관리하면 보일러플레이트 코드가 발생합니다.
- 동시에, API 응답을 캐시하지 않아서 중복 API 요청이 발생하게 됩니다.
- 이 두 가지 문제를 react-query가 해결해 줄 수 있습니다.

## 해결책: react-query 훅 생성

다음은 `useEffect`를 대체할 수 있는 query 훅입니다.

```tsx
import { useQuery } from '@tanstack/react-query';

import UserService from '@/infrastructure/user';

export function getQueryKey() {
  return ['me'];
}

export function useGetMe() {
  return useQuery({
    queryKey: getQueryKey(),
    queryFn: () => UserService.getMe(),
  });
}
```

쿼리 함수는 변환된 API 응답을 반환하는 `UserService`([이전 글에서 생성된](https://profy.dev/article/react-architecture-domain-entities-and-dtos))를 호출합니다. 이 부분은 여기서 크게 중요하지 않습니다.

이제 컴포넌트에서 `useEffect` 대신 새 쿼리 훅을 간단히 사용할 수 있습니다.

```tsx
import { useState } from "react";

import { useGetMe } from "@/application/queries/get-me";
import { useReplyToShout } from "@/application/reply-to-shout";
import { isAuthenticated } from "@/domain/me";

...

export function ReplyDialog({
  recipientHandle,
  children,
  shoutId,
}: ReplyDialogProps) {
  const [open, setOpen] = useState(false);
  const [isLoading, setIsLoading] = useState(true);
  const [replyError, setReplyError] = useState<string>();
  const replyToShout = useReplyToShout();
  const me = useGetMe(); // 해당 코드 추가

  if (me.isError || !isAuthenticated(me.data)) { // 해당 코드 추가
    return <LoginDialog>{children}</LoginDialog>;
  }

  async function handleSubmit(event: React.FormEvent<ReplyForm>) {
    // 이 부분을 곧 다시 볼겁니다
  }

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      {/* 컴포넌트의 나머지 부분 */}
    </Dialog>
  );
}
```

우리는 오류 및 로딩 상태 처리와 같은 많은 보일러플레이트 코드를 제거했을 뿐만 아니라, 응답 캐싱, 재시도 등 많은 기능을 기본적으로 얻을 수 있습니다.

요약하면, 쿼리 훅을 컴포넌트와 서비스 레이어 사이의 일종의 프록시로 사용합니다.

이제 더 복잡한 예제를 살펴보겠습니다.

## 문제가 있는 코드 예시 2: 비즈니스 로직과 react-query

이전 코드 예제에서는 submit 핸들러를 살펴보지 않았습니다. 여기를 보면:

```tsx
import { useReplyToShout } from "@/application/reply-to-shout";

...

export function ReplyDialog({
  recipientHandle,
  children,
  shoutId,
}: ReplyDialogProps) {
    ...

   const replyToShout = useReplyToShout(); // 삭제될 부분

  async function handleSubmit(event: React.FormEvent<ReplyForm>) {
    event.preventDefault();
    setIsLoading(true);

    const message = event.currentTarget.elements.message.value;
    const files = Array.from(event.currentTarget.elements.image.files ?? []);

    // 삭제될 부분
    const result = await replyToShout({
      recipientHandle,
      message,
      files,
      shoutId,
    });
    // 삭제될 부분

    if (result.error) {
      setReplyError(result.error);
    } else {
      setOpen(false);
    }
    setIsLoading(false);
  }

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      {/* 컴포넌트의 나머지 부분 */}
    </Dialog>
  );
}
```

[이전 글](https://profy.dev/article/react-architecture-business-logic-and-dependency-injection)에서 우리는 submit 핸들러의 많은 비즈니스 로직을 위에서 강조한 `useReplyToShout` 함수로 추출했습니다.

현재 `useReplyToShout` 훅은 의존성 주입을 통해 `replyToShout` 함수에 몇 가지 서비스 함수를 제공합니다.

```tsx
import { useCallback } from "react";

import { hasExceededShoutLimit } from "@/domain/me";
import { hasBlockedUser } from "@/domain/user";
import MediaService from "@/infrastructure/media";
import ShoutService from "@/infrastructure/shout";
import UserService from "@/infrastructure/user";

...

// 삭제될 부분
const dependencies = {
  getMe: UserService.getMe,
  getUser: UserService.getUser,
  saveImage: MediaService.saveImage,
  createShout: ShoutService.createShout,
  createReply: ShoutService.createReply,
};
// 삭제될 부분

export async function replyToShout(
  { recipientHandle, shoutId, message, files }: ReplyToShoutInput,
  { getMe, getUser, saveImage, createReply, createShout }: typeof dependencies
) {
  const me = await getMe(); // 삭제될 부분
  if (hasExceededShoutLimit(me)) {
    return { error: ErrorMessages.TooManyShouts };
  }

  const recipient = await getUser(recipientHandle); // 삭제될 부분
  if (!recipient) {
    return { error: ErrorMessages.RecipientNotFound };
  }
  if (hasBlockedUser(recipient, me.id)) {
    return { error: ErrorMessages.AuthorBlockedByRecipient };
  }

  try {
    let image;
    if (files?.length) {
      image = await saveImage(files[0]); // 삭제될 부분
    }

    const newShout = await createShout({ // 삭제될 부분
      message,
      imageId: image?.id,
    });

    await createReply({ // 삭제될 부분
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
    (input: ReplyToShoutInput) => replyToShout(input, dependencies), // 삭제될 부분
    []
  );
}
```

## 문제: 중복 요청

`replyToShout` 함수는 여러 API 요청을 보냅니다. 그 중 `UserService.getMe(...)`와 `UserService.getUser(...)` 호출이 포함됩니다. 그러나 이 데이터는 이미 애플리케이션의 다른 부분에서 가져와 react-query 캐시에 존재합니다.

또한, 다시 로딩 상태를 수동으로 관리해야 합니다.

## 해결책: react-query 데이터와 mutation 사용

이전 예제에서 `useGetMe` 쿼리 훅을 도입했습니다. 이제 사용자의 핸들을 기반으로 사용자를 가져오는 또 다른 쿼리 훅을 추가해 보겠습니다.

```tsx
import { useQuery } from '@tanstack/react-query';

import UserService from '@/infrastructure/user';

interface GetUserInput {
  handle?: string;
}

export function getQueryKey(handle?: string) {
  return ['user', handle];
}

export function useGetUser({ handle }: GetUserInput) {
  return useQuery({
    queryKey: getQueryKey(handle),
    queryFn: () => UserService.getUser(handle),
  });
}
```

그런 다음 필요한 mutation 훅을 생성합니다. 여기 shout를 생성하는 예제 훅이 있습니다.

```tsx
import { useMutation } from '@tanstack/react-query';

import ShoutService from '@/infrastructure/shout';

interface CreateShoutInput {
  message: string;
  imageId?: string;
}

export function useCreateShout() {
  return useMutation({
    mutationFn: (input: CreateShoutInput) => ShoutService.createShout(input),
  });
}
```

이제 이러한 훅을 `useReplyToShout` 훅 내에서 사용할 수 있습니다.

첫 번째 단계로, `dependencies` 객체를 TypeScript 인터페이스로 대체하고, `replyToShout` 함수를 이에 맞게 조정합니다.

```tsx
import { Me, hasExceededShoutLimit, isAuthenticated } from "@/domain/me";
import { Image } from "@/domain/media";
import { Shout } from "@/domain/shout";
import { User, hasBlockedUser } from "@/domain/user";

import { useCreateShout } from "../mutations/create-shout";
import { useCreateShoutReply } from "../mutations/create-shout-reply";
import { useSaveImage } from "../mutations/save-image";
import { useGetMe } from "../queries/get-me";
import { useGetUser } from "../queries/get-user";

...

// 추가할 부분
interface Dependencies {
  me: ReturnType<typeof useGetMe>["data"];
  recipient: ReturnType<typeof useGetUser>["data"];
  saveImage: ReturnType<typeof useSaveImage>["mutateAsync"];
  createShout: ReturnType<typeof useCreateShout>["mutateAsync"];
  createReply: ReturnType<typeof useCreateShoutReply>["mutateAsync"];
}

export async function replyToShout(
  { shoutId, message, files }: ReplyToShoutInput,
  { me, recipient, saveImage, createReply, createShout }: Dependencies // 추가할 부분
) {
  if (!isAuthenticated(me)) {  // 추가할 부분
    return { error: ErrorMessages.NotAuthenticated };
  }
  if (hasExceededShoutLimit(me)) {  // 추가할 부분
    return { error: ErrorMessages.TooManyShouts };
  }

  if (!recipient) {  // 추가할 부분
    return { error: ErrorMessages.RecipientNotFound };
  }
  if (hasBlockedUser(recipient, me.id)) {  // 추가할 부분
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
```

다음으로, `useReplyToShout` 훅을 다시 작성해야 합니다.

단순히 dependencies 객체를 `replyToShout` 함수에 제공하고 반환하는 대신,

- 쿼리와 mutation 훅을 통해 모든 종속성을 수집합니다.
- `mutateAsync` 함수를 반환합니다(이름은 임의이지만 react-query mutation 훅과 API 일관성을 유지합니다).
- 모든 쿼리와 mutation 훅의 로딩 상태를 병합(merge)합니다.
- 두 쿼리 훅의 오류를 병합합니다.

```tsx
interface UseReplyToShoutInput {
  recipientHandle: string;
}

export function useReplyToShout({ recipientHandle }: UseReplyToShoutInput) {
  const me = useGetMe();
  const user = useGetUser({ handle: recipientHandle });
  const saveImage = useSaveImage();
  const createShout = useCreateShout();
  const createReply = useCreateShoutReply();

  return {
    mutateAsync: (input: ReplyToShoutInput) =>
      replyToShout(input, {
        me: me.data,
        recipient: user.data,
        saveImage: saveImage.mutateAsync,
        createShout: createShout.mutateAsync,
        createReply: createReply.mutateAsync,
      }),
    isLoading:
      me.isLoading ||
      user.isLoading ||
      saveImage.isPending ||
      createShout.isPending ||
      createReply.isPending,
    isError: me.isError || user.isError,
  };
}
```

이 변경으로 `ReplyDialog` 컴포넌트가 렌더링되자마자 `me`와 `user`를 얻기 위한 API 요청이 전송됩니다. 동시에, 이전에 데이터를 가져왔으면 두 응답 모두 캐시에서 제공될 수 있습니다.

사용자가 샤우트에 답장할 때 이러한 요청을 기다릴 필요가 없어 전반적인 사용자 경험이 개선됩니다.

이 접근 방식의 또 다른 장점은, 서버 데이터 관리를 위해 react-query를 사용하면서도 `replyToShout` 함수와 모든 비즈니스 로직을 개별적으로 테스트할 수 있다는 것입니다.

이러한 변경 사항으로 `ReplyToDialog` 컴포넌트를 간소화할 수 있습니다. 조정된 useReplyToShout 훅에서 `isLoading`과 `hasError` 상태를 제공하므로 더 이상 필요하지 않습니다.

```tsx
import { useState } from "react";

import { useGetMe } from "@/application/queries/get-me";
import { useReplyToShout } from "@/application/reply-to-shout";
import { LoginDialog } from "@/components/login-dialog";
import { Button } from "@/components/ui/button";
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { isAuthenticated } from "@/domain/me";

...

export function ReplyDialog({
  recipientHandle,
  children,
  shoutId,
}: ReplyDialogProps) {
  const [open, setOpen] = useState(false);
  const [replyError, setReplyError] = useState<string>();
  const replyToShout = useReplyToShout({ recipientHandle });
  const me = useGetMe();

  if (me.isError || !isAuthenticated(me.data)) {
    return <LoginDialog>{children}</LoginDialog>;
  }

  async function handleSubmit(event: React.FormEvent<ReplyForm>) {
    event.preventDefault();

    const message = event.currentTarget.elements.message.value;
    const files = Array.from(event.currentTarget.elements.image.files ?? []);

    const result = await replyToShout.mutateAsync({  // 추가될 부분
      recipientHandle,
      message,
      files,
      shoutId,
    });

    if (result.error) {
      setReplyError(result.error);
    } else {
      setOpen(false);
    }
  }

    ...

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      {/* 컴포넌트의 나머지 부분 */}
    </Dialog>
  );
}
```

비즈니스 로직을 `useReplyToShout` 훅으로 추출한 또 다른 장점이 이제 명확해집니다:

서버 데이터 관리의 기본 메커니즘을 훅 내부에서 상당히 변경했지만, 컴포넌트에서의 조정은 최소한이었습니다.

## 다음 리팩토링 단계

이번 글은 여기까지입니다. 우리는 react-query를 React 애플리케이션 아키텍처에 성공적으로 통합했습니다. 다음 번에는 더 일반적인 기능 중심의 폴더 구조에 맞추기 위해 폴더 구조를 리팩토링할 것입니다.
