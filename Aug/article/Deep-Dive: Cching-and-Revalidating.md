# [Deep Dive: Caching and Revalidating ](https://github.com/vercel/next.js/discussions/54075)

### 🗓️ 번역 날짜: 2024.08.11

### 🧚 번역한 크루: 버건디(전태헌)

---

다음은 Next.js App Router에서 캐싱과 재검증(revalidating)과 관련하여 새롭게 도입된 휴리스틱(heuristics)에 대한 내용입니다. 캐싱이 어떻게 작동하도록 설계되었는지에 대해 모두가 동일한 이해를 할 수 있도록 이 주제에 대해 심도 있는 논의를 진행하는 것이 도움이 될 것이라고 생각합니다.

이 논의는 최근에 공개된 [캐싱 관련 문서](https://nextjs.org/docs/app/building-your-application/caching)와 보완적으로 작용합니다. 이는 커뮤니티의 일부 구성원들이 클라이언트 측 라우터 캐시(client-side router cache)에 대해 혼란을 겪거나, 약간 다른 동작을 원했던 문제에 대한 후속 작업으로, 이 글을 통해 캐싱이 어떻게 설계되었는지에 대한 배경을 설명하고 [커뮤니티와의 논의](https://github.com/vercel/next.js/issues/42991#issuecomment-1673691751)를 열기 위한 기회를 제공하고자 합니다.

**답변을 작성하시기 전에 이 글 전체를 읽어보시길 부탁드립니다. 이 문맥을 이해하는 것이 중요하기 때문입니다.**

---

## - 데이터 변이(Mutations)

App Router는 거의 1년 전에 처음 발표되었습니다. 당시에는 데이터 변이를 어떻게 처리할 계획인지에 대해 아직 공유하지 않았습니다. 그때는 Server Actions가 지원되지 않았고, `router.refresh()`가 예상대로 라우트 캐시를 지우지 않았습니다.

이로 인해 약간의 혼란이 있었음을 이해하며, 이에 대해 명확히 하고자 합니다.

그 이후로 우리는 [Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)를 추가했습니다. 이것이 안정화되면 대부분의 변이가 발생할 것으로 기대됩니다. 또한, `router.refresh()`가 예상대로 작동하도록 개선했습니다.

아래에서는 각 요소에 대한 설명과 그것들이 어떻게 함께 작동하는지, 그리고 다른 도구에서 사용했던 몇 가지 동작(때로는 암묵적으로 사용된)을 어떻게 재현할 수 있는지에 대한 섹션을 찾을 수 있습니다.

이 섹션은 두 부분으로 나뉘어 있습니다:

- 내가 애플리케이션에서 트리거하는 변이, 예를 들어 폼 제출 등.
- 동료와 같은 다른 사람들이 트리거하는 변이, 예를 들어 CMS에서 제품을 추가하는 것과 같은 변경 사항.

## - 내가 트리거하는 변이

### 무효화(Invalidation)

**Server Actions (revalidatePath / revalidateTag)**

- 언제 유용한가

  - 서버가 필요한 사용자 상호작용

            - 예시: 폼(form) 제출

              - 제품 추가
                - 사용자 세션의 일부로 제품을 추가한 후, revalidateTag 또는 revalidatePath를 호출하여 페이지가 다시 렌더링되도록 합니다. 이는 장바구니 컴포넌트가 서버 컴포넌트로 렌더링되기 때문에 최신 상태로 유지될 수 있습니다(useSWR / react-query의 mutate()와 유사한 동작).

                - 로그인 / 로그아웃
                    폼을 통해 사용자 이름/비밀번호(또는 다른 인증 흐름)를 제출하고, 서버 액션에 의해 처리됩니다. 쿠키를 설정한 후, revalidatePath 또는 revalidateTag를 호출하여 라우터 캐시가 삭제되고, 새로 설정된 쿠키로 새 렌더링이 이루어지도록 합니다.

            - 예시: 페이지 내 타이프어헤드(실시간 검색) / 검색 결과 상자
                - 예를 들어 [nextjs.org/docs](https://nextjs.org/docs)에서 검색할 때 볼 수 있는 것(현재는 Algolia에 의해 지원되지만, 이를 직접 작성한 코드로 지원한다고 가정합니다).

            - 예시: 권한에 기반한 툴팁
                - Facebook, Instagram, GitHub에서 볼 수 있는 예시로, 툴팁이 클릭 시 서버 데이터에 기반해 로드됩니다.

- 작동 원리

1. `"use server"`는 컴파일 시 함수에 마커를 추가하고, 해당 함수를 별도의 모듈로 이동시킵니다. 모든 Server Actions는 각각 고유한 ID를 가지므로, 개별적으로 참조될 수 있습니다. (이 설명은 매우 단순화된 버전이며, 실제로는 더 복잡합니다.)

2. 함수를 호출하면, 현재 URL로 Server Action ID를 포함한 POST 요청을 보내며, 이를 통해 Next.js가 올바른 기본 함수를 호출할 수 있게 됩니다.

- 함수 호출 예시
  `<form action={serverAction}>`이 제출되거나, 클라이언트 컴포넌트에서 직접 `serverAction()`을 호출하는 경우
  서버 액션으로 정의된 함수를 호출할 때 몇 가지 일이 발생할 수 있습니다:

- 함수가 `revalidatePath` 또는 `revalidateTag`를 호출합니다.

  - 지정된 항목이 전체 라우트 캐시 또는 데이터 캐시(서버 캐시)에서 삭제됩니다.
    - 라우트가 정적 렌더링(전체 라우트 캐시 내) 중일 때 사용된 모든 fetch 태그는 전체 라우트 캐시의 태그이기도 하므로, revalidateTag를 호출하면 전체 라우트 캐시도 올바르게 삭제됩니다.

- `revalidatePath`에 있는 경로에 따라 현재 페이지는 서버에서 다시 렌더링됩니다. `revalidateTag`의 경우, 현재 페이지는 항상 다시 렌더링됩니다.

  - 이 동작이 `router.refresh()`를 호출하는 것과 매우 유사하다는 것을 나중에 깨닫게 될 수도 있습니다.
  - 이는 동일한 라운드 트립의 일부로 발생하므로 추가적인 클라이언트 측 요청이 필요하지 않습니다.

- 제공된 경로에 따라 클라이언트 측 캐시가 삭제됩니다. `revalidateTag`의 경우, 전체 클라이언트 측 라우터 캐시가 삭제됩니다.

  - Server Actions를 사용하는 예시

```tsx
export async function updateUser(config) {
  "use server";
  await prisma.user.update(config);
  revalidateTag("prisma-user");
}
```

- 이 스레드에서 혼란이 많았던 이유는 구현 과정에서 몇 가지 버그가 있었기 때문입니다. 이러한 버그는 최신 버전의 Next.js에서 수정되었습니다. (예를 들어, 이전에는 뒤로/앞으로 가기 버튼을 사용할 때의 캐시가 삭제되지 않고, 프리페치된 데이터만 삭제되어 오래된 데이터가 남아 있을 수 있었습니다. 이제는 더 이상 그렇지 않지만, 이 점을 강조하고자 합니다.)

- 이 동작은 서버로부터의 fetch 요청이 반환된 후 `router.refresh()`와 사실상 동일한 코드 경로를 사용하므로, 라우터 캐시를 삭제하는 방식도 본질적으로 동일합니다.

  - 미래에는 캐시 삭제 방식을 더 스마트하게 개선할 계획입니다. 예를 들어, 현재 `/products` 경로에 있고 서버 액션에서 `revalidatePath('/dashboard/settings')`를 호출하면 전체 라우터 캐시가 삭제됩니다. 하지만 실제로는 `/dashboard/settings` 경로와 그 하위 세그먼트만 삭제하는 것이 더 적절합니다.

    - 여기서 "하위 세그먼트"란 `/dashboard/settings/`environment와 같은 경로를 의미합니다. 제공된 경로 아래의 레이아웃이 변경되었는지 알 수 없기 때문에, 해당 경로에 대한 라우터 캐시를 삭제해야 합니다.

    - 이와 같은 이유로, `revalidatePath('/')`를 호출하면 라우터 캐시에 있는 모든 경로가 삭제됩니다. 이는 `/` 경로와 그 하위 세그먼트 모두가 포함되기 때문입니다. 따라서 `router.refresh()`를 호출하는 것은 `revalidatePath('/')`와 마찬가지로 라우터 캐시를 삭제하고 다시 채우는 것과 같습니다.

- 함수가 `redirect()`를 호출하는 경우

  - redirect()를 호출하면, 서버 액션에서 결과를 반환할 수 없게 됩니다. 왜냐하면 리디렉션이 즉시 적용되기 때문입니다.

- 함수가 `cookies().set()`을 호출하는 경우

  - 이 경우, 라우터 캐시가 완전히 무효화되고 페이지가 다시 렌더링됩니다. 이는 `revalidateTag`를 호출하는 것과 유사합니다. 쿠키를 변경하면 레이아웃에서 렌더링되는 내용에 영향을 미칠 수 있기 때문입니다.

        - 예시: 로그인 / 로그아웃
        - 예시: 기능 플래그(Feature flags) 변경

- 함수가 결과를 반환하는 경우

  - 서버 액션에서 직접 응답을 반환할 수 있습니다. 이 응답은 객체일 수도 있고, JSX일 수도 있으며, 서버 컴포넌트에서 클라이언트 컴포넌트로 직렬화할 수 있는 모든 것이 서버 액션의 유효한 반환 값이 될 수 있습니다.

  - 이 경우에도 revalidatePath 또는 revalidateTag를 호출하는 것을 금지하지 않습니다. 만약 이러한 호출이 이루어진다면, 서버로부터의 응답은 함수의 결과와 새로 렌더링된 페이지를 모두 포함하게 되며, 두 가지가 모두 애플리케이션에 적용됩니다.

  - 결과를 사용하기 위해서는 서버 액션을 수동으로 호출해야 합니다. 이는 `<form action={serverAction}>`에서 서버 액션을 `action` 속성에 직접 전달하는 경우에는 해당되지 않습니다.

- **Route Handlers (revalidatePath / revalidateTag)**

언제 유용한가:

- 외부 이벤트를 기반으로 서버 측 캐시(전체 경로 캐시 및 데이터 캐시)를 삭제할 때 유용합니다.

  - 예시: CMS에서 항목을 수정할 때, 현재의 전체 경로 캐시/데이터 캐시를 삭제하여 새 데이터를 가져오고 전체 경로 캐시/데이터 캐시에 새 데이터를 채우고 싶을 때 사용합니다.
    - 비록 캐싱 문서에서 전체 경로 캐시와 데이터 캐시를 별도로 설명하고 있지만, 실제로는 기본적으로 하나의 데이터 캐시입니다. 우리가 이를 두 개의 개념으로 문서화한 주된 이유는 전체 경로 캐시 부분은 HTML/RSC 페이로드를 저장하고, 데이터 캐시 부분은 fetches와 unstable_cache 호출을 저장하기 때문입니다(이 API는 가까운 미래에 안정화될 예정입니다).

- 작동 방식

  - `revalidatePath` 또는 `revalidateTag`를 호출하면 지정된 항목이 전체 경로 캐시/데이터 캐시에서 삭제됩니다.

    - 해당 경로가 정적 렌더링(전체 경로 캐시에 있음)일 때, 사용된 모든 fetch 태그가 전체 경로 캐시의 태그로도 적용됩니다. 즉, revalidateTag를 호출하면 전체 경로 캐시도 올바르게 삭제됩니다.

  - 사용자의 브라우저에서 경로 핸들러를 fetch()하는 경우, Next.js에서는 어떤 것이 변경되었는지 알 수 없기 때문에 해당 사용자의 라우터 캐시에 영향을 미치지 않습니다. 이 경우, router.refresh()를 호출해야 합니다.

    - 그러나, 애플리케이션의 API 엔드포인트인 경로 핸들러를 호출할 예정이라면, 두 번의 왕복 대신 한 번의 왕복으로 이 작업을 수행하기 위해 서버 액션을 사용할 수 있습니다.

```js
onClick={async () => {
	const res = await fetch('/my-route-handler')
  const json = await res.json()
  // etc
  router.refresh()
}}
```

**router.refresh()**

- 언제 유용한가

  - 사용자 상호작용으로 인해 발생하는 서버 액션에 의해 처리되지 않는 이벤트를 기반으로 라우터 캐시를 삭제하고 싶을 때 유용합니다.

    - 예시: 폼이나 클릭 핸들러가 외부 API를 직접 호출하여 예를 들어 항목을 추가하는 경우, 서버 액션을 사용하지 않은 상황입니다. 이는 useSWR 또는 react-query를 사용할 때 mutate()를 사용하는 경우와 유사합니다.
      - 이 상황에서는 서버 액션을 사용하는 것이 더 좋습니다. 위의 예시에서는 두 번의 왕복(한 번은 API에, 한 번은 업데이트된 데이터를 반영한 페이지를 렌더링하는 작업)이 필요하지만, 서버 액션을 사용하면 이 작업을 한 번의 왕복으로 처리할 수 있습니다. 서버 액션을 사용하여 항목을 추가하고 `revalidatePath` 또는 `revalidateTag`를 호출하여 동일한 서버 요청에서 새 결과를 렌더링할 수 있습니다. 하지만 서버 액션을 사용하는 것이 항상 가능하지는 않을 수 있습니다.

- 페이지가 렌더링된 후 최신 항목으로 업데이트되는 stale-while-revalidate (useSWR/react-query) 동작을 원할 때도 유용합니다.

- 예시: 아래 코드에서 볼 수 있듯이, 이는 useSWR가 내부적으로 수행하는 작업과 매우 유사하지만, 여기서는 revalidation을 언제, 어떻게 트리거할지 완전히 제어할 수 있습니다. 이 점은 라우터 캐시에만 영향을 미치며, 서버 측 캐시에는 영향을 미치지 않습니다.

```tsx
import { useRouter, usePathname } from "next/navigation";
import { debounce } from "lodash";

interface DataRevalidatorProps {
  /**
   * 데이터를 다시 검증하기 전에 대기할 시간(밀리초 단위)입니다.
   */
  timeout?: number;
}

export function DataRevalidator({
  timeout = 5000,
}: DataRevalidatorProps): null {
  const router = useRouter();
  const pathname = usePathname();

  const getRefresher = useCallback(
    () =>
      debounce(
        () => {
          // 이 코드는 언제 router.refresh()가 트리거되는지를 보여주기 위해 추가된 것입니다.
          // 실제로는 제거해도 됩니다.
          console.log(
            `[${new Date().toLocaleTimeString()}] Triggering router.refresh()...`
          );
          router.refresh();
        },
        timeout,
        { leading: false, trailing: true }
      ),
    [router, timeout]
  );

  // `pathname`이 변경될 때마다 `router.refresh()`를 호출합니다.
  // 참고: 이 코드는 `searchParams`를 처리하지 않습니다. 필요시 `useSearchParams`를 추가할 수 있습니다.
  useEffect(() => {
    const refresher = getRefresher();
    refresher();
  }, [revalidate, pathname]);

  // 창으로 포커스가 돌아올 때 `router.refresh()`를 호출합니다.
  // (이는 `useSWR` 또는 `react-query`와 유사한 동작입니다.)
  useEffect(() => {
    if (process.env.NODE_ENV === "development") {
      return;
    }

    const refresher = getRefresher();

    window.addEventListener("focus", refresher);
    return () => {
      refresher.cancel();
      window.removeEventListener("focus", refresher);
    };
  }, [revalidate]);

  return null;
}
```

- 어떻게 동작할까요

  - router.refresh()를 호출하면 다음과 같은 일이 발생합니다:

    - 서버로 현재 페이지 전체를 가져오기 위한 요청이 전송됩니다. 이 때, HTML 대신 RSC Payload 형식으로 페이지의 모든 내용이 포함됩니다.
    - 라우터의 모든 캐시가 제거됩니다:
      - 프리페치 캐시(모든 프리페치를 저장하는 캐시)
      - 라우터 네비게이션 캐시(30초 동안 유지되는 캐시)
      - 컴포넌트 캐시(네비게이션했던 모든 세그먼트를 저장하며, 부분 렌더링 및 스크롤 위치를 보존하면서 뒤로/앞으로 이동할 수 있게 하는 캐시)
    - 가져온 RSC Payload가 제거된 캐시에 적용되며, 이로 인해 해당 캐시는 유일한 경로로 설정됩니다

## 다른 사람이 트리거하는 변동 사항 (Mutations)

---

### 데이터 소스 변경

변동 사항(Mutations)에 관한 이야기의 다른 측면으로, 만약 내가 아닌 내 동료가 CMS에서 업데이트를 한다면 어떻게 될까요? 만약 그 변동 사항이 `revalidatePath` 또는 `revalidateTag`로 백업되었다면, 이는 전체 경로 캐시와 데이터 캐시를 무효화할 것입니다. 하지만 이것들은 서버 측에 있는 것이며, 내 브라우저 탭에 있는 라우터 캐시는 여전히 서버의 이전 결과를 유지할 수 있습니다. 즉, 가장 최신 데이터가 반영되지 않을 수 있습니다.

이 이슈에 대한 댓글 중 상당수는 다른 사람이 변동 사항을 만들었을 때 어떤 일이 일어나는지에 대해 이야기하고 있었기 때문에, 먼저 이 상황에서의 동작을 명확히 해보겠습니다:

- 내가 `/products` 경로로 이동합니다.
- 다른 사람이 서버 액션을 트리거하여 전자상거래 스토어에 새 제품을 추가합니다.
- 나는 메뉴에서 계정(account)을 클릭하여 `/account` 경로로 이동합니다.
- 나는 다시 제품을 보고 싶어져서, [https://demo.vercel.store](https://demo.vercel.store) 에서 그 후드티를 정말로 사고 싶어합니다.
- 나는 메뉴에서 제품(products)을 클릭합니다.
- 나는 즉시 /products 경로로 다시 이동하며, 이는 마치 뒤로가기 버튼을 클릭한 것과 비슷합니다.
- 내가 처음 /products 경로를 열었을 때 보았던 것과 동일한 페이지가 보입니다.

경로가 "정적"인 경우(예: 정적 렌더링 또는 페이지가 동적 렌더링을 사용할 때 `prefetch={true}`로 설정한 경우), `/products`로 즉시 다시 돌아갈 수 있는 시간은 5분입니다.

경로가 동적 렌더링을 사용하는 경우(예: 쿠키, 헤더 사용, `export const dynamic = 'force-dynamic'` 설정) `/products`로 즉시 다시 돌아갈 수 있는 시간은 30초입니다.

이 30초 또는 5분이 지나면, 해당 캐시 항목은 router.push / router.replace 내비게이션(여기에는 `<Link>`도 포함됨)에 대해 제거됩니다.

이제 `/products`가 동적 렌더링을 사용하도록 설정되어 있다고 가정해봅시다. 이는 캐시 시간이 30초임을 의미하며, 만약 내가 `/account`에서 조금 더 오래 머물렀다면 캐시가 있다는 것을 눈치채지 못할 것입니다.

이 동작이 존재하는 이유는 여러 가지가 있습니다:

- 메뉴를 빠르게 클릭하거나 탭할 때, 링크를 클릭한 후 다른 링크를 클릭하고 다시 돌아왔을 때 여러 로딩 상태를 다시 보는 것은 UX에 좋지 않습니다.

- 느린 네트워크 연결에서 이 동작은 링크를 사용해 앞뒤로 이동할 때 여전히 빠르게 느껴지도록 보장해줍니다.

- 또한 라우터 사이를 빠르게 이동할 때 위치를 유지하는 것과도 관련이 있습니다. 예를 들어, 다른 페이지에서 무언가를 찾아보고 이전 페이지로 돌아가는 경우에 해당합니다.

이 타이밍과 관련된 이전 연구 및 사례가 있으며, 예를 들어 페이스북에서 애플리케이션 내에서 클릭할 때 이 타이밍을 사용합니다. react-query도 이 타이밍을 채택하고 있습니다.
`/products` 경로는 헤더/푸터 데이터를 위해 데이터 캐시를 여전히 활용할 수 있으며, 이 경우 제품 자체에 대한 요청만이 블로킹될 수 있으므로 괜찮습니다.

이것이 라우터 캐시를 무효화할 수 없다는 것을 의미하지는 않습니다. 여전히 stale-while-revalidate 및 focus 동작(useSWR / react-query에 있는)을 사용할 수 있습니다 (위의 router.refresh 섹션 참조).

전체적으로 이 동작의 기본 설정이 올바르다고 믿지만, 이 문제에 대한 사람들이 이 동작을 옵트아웃(opt-out)하려는 방법을 파악하고자 하며, 이 동작과 혼동될 수 있는 여러 관련 기능이 있는 것 같습니다.

## 부분 렌더링

문서에서 고수준 개념을 잘 설명하고 있으므로, [문서에서 발췌한 내용](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#3-partial-rendering)을 소개합니다:

부분 렌더링(Partial Rendering)은 네비게이션 시 변경되는 경로 세그먼트만 클라이언트에서 다시 렌더링되고, 공유된 세그먼트는 그대로 유지되는 것을 의미합니다.

예를 들어, 두 형제 경로 `/dashboard/settings`와 `/dashboard/analytics` 사이를 네비게이션할 때, `설정 페이지(settings)`와 `분석 페이지(analytics)`가 렌더링되며, 공유된 대시보드 레이아웃은 유지됩니다.

[partial-rendering.avif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33037b65-dd75-41a0-b974-2d73a569d5eb/partial-rendering.avif)

부분 렌더링이 없을 경우, 각 네비게이션은 서버에서 전체 페이지를 다시 렌더링하게 됩니다. 변경되는 세그먼트만 렌더링하면 전송되는 데이터 양과 실행 시간이 줄어들어 성능이 향상됩니다.

이것이 가능한 이유는 앱 라우터(App Router)가 "라우터 트리(Router Tree)"라고 부르는 개념을 가지고 있기 때문입니다. 이 트리는 현재 보고 있는 화면에 무엇이 렌더링되는지를 결정하는 형식입니다.

라우터 트리는 다음과 같은 형태를 가지고 있습니다 (단순화된 형태):

```ts
// Array format as it's transferred between server/client often.
type RouterTree = [
	// The segment, e.g. 'dashboard' or for dynamic params ['user', 'timneutkens', 'd' /* d for dynamic */].
  // Final segment is called __PAGE__
	segment: Segment
  // In the simple case this would only hold `children`.
  parallelRoutes: {
		// E.g. `children` or `modal` (when you add `@modal` as a parallel route slot).
		[parallelRouteKey: string]: RouterTree
	}
]
```

각 경로(route)에는 라우터 트리(Router Tree)가 연결되어 있습니다. 예를 들어, /dashboard/settings/page.tsx와 같은 경로에 대한 라우터 트리는 다음과 같은 형태일 것입니다:

```ts
const routerTreeForSettings: RouterTree = [
  "dashboard",
  {
    children: [
      "settings",
      {
        children: ["__PAGE__", {}],
      },
    ],
  },
];
```

처음 페이지를 요청할 때, 서버 측의 라우터 트리는 컴포넌트 목록(예: `레이아웃`, `로딩`, `페이지`)을 포함하며, 이를 사용해 전체 페이지를 렌더링합니다.

네비게이션을 할 때, 현재 화면에 있는 라우터 트리가 페이로드의 일부로 서버에 전송됩니다. 이를 통해 서버는 현재 라우터 트리와 요청하는 페이지의 라우터 트리를 비교(diff)할 수 있습니다. 이 비교 결과는 라우터 트리에서 변경된 부분만 렌더링하는 데 사용됩니다.

라우터 트리는 브라우저 히스토리에 저장됩니다(뒤로/앞으로 이동 섹션에서 더 자세히 설명).

현재 화면에 있는 트리만으로는 이전에 렌더링된 페이지의 개별 세그먼트를 재사용할 수 없기 때문에 충분하지 않습니다. 이때 클라이언트 측 라우터의 "컴포넌트 캐시" 부분이 필요합니다.

라우터 상태에 라우터 트리를 유지하는 것 외에도, 우리는 컴포넌트 캐시도 그곳에 유지합니다. 이 캐시는 다음과 같은 형태를 가지고 있습니다(단순화된 형태).

```ts
type ChildSegmentMap = Map<
  // For explanation purposes showing Segment
  // Real value is a serialized version of Segment (string)
  Segment,
  ComponentCacheNode
>;

interface ComponentCacheNode {
  // There is also 'DATAFETCH' and 'LAZYINITIALIZED' but not going into those in this post.
  status: "READY";
  // This property only exists on the `__PAGE__` nodes.
  head?: React.ReactNode;
  subTreeData: React.ReactNode;
  parallelRoutes: Map<ParallelRouteKey, ChildSegmentMap>;
}
```

컴포넌트 캐시는 각각의 렌더링된 세그먼트를 개별적으로 추적하며, 이를 통해 여러 가지 기능이 가능해집니다:

- 세그먼트를 재사용하며 페이지 간 네비게이션 (부분 네비게이션).
- 브라우저의 스크롤 위치를 보존하는 즉시 뒤로/앞으로 이동.
- 병렬 라우트/인터셉션, 여러 페이지의 세그먼트 캐시 노드를 동시에 유지할 수 있습니다. 라우터 트리는 병렬 라우트를 작동하게 하는 또 다른 요소로, 화면에 무엇이 표시되는지를 결정합니다.
- 라우터 캐시의 개별 세그먼트 삭제(예: `router.refresh()`는 전체 캐시를 삭제하지만, `revalidatePath('/dashboard')`는 가까운 미래에 지정된 대시보드 경로 아래의 클라이언트 측 라우터 캐시만 삭제할 것입니다).

## 뒤로/앞으로 네비게이션

이 동작은 브라우저에 내장된 bfcache와 매우 유사합니다: [https://web.dev/bfcache/](https://web.dev/articles/bfcache?hl=ko).

경로 간 네비게이션 시 클라이언트 측 라우터는 pushState 또는 replaceState를 호출하여 브라우저 주소 표시줄에 보이는 URL을 업데이트합니다. 이때 메타데이터 일부도 함께 푸시되며, 특히 라우터 트리가 히스토리 항목에 첨부됩니다.

라우터 트리는 화면에 무엇이 렌더링될지를 결정합니다(더 깊은 설명은 이전 섹션을 참조하세요).

뒤로/앞으로 이동([popstate](https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event)) 시 히스토리 항목의 라우터 트리가 라우터에 적용됩니다. 이로 인해 해당 라우터 트리가 렌더링됩니다.

컴포넌트 캐시를 활용하여 브라우저의 기본 `scrollRestoration` 기능을 이용할 수 있으며, 이를 통해 뒤로/앞으로 이동할 때 정확히 이전에 있던 위치로 돌아갈 수 있습니다.

브라우저에서의 이 scrollRestoration 동작은 일정한 휴리스틱을 가지고 있으며, 이 작업을 제대로 수행하지 않으면 스크롤 위치 복원이 실패할 수 있습니다.

이제 Pages Router와 이 동작이 어떻게 다른지 궁금할 것입니다.

Pages Router에서는 두 가지 동작이 있었으며, 둘 다 클라이언트 측 라우터에 적용될 수 있는 문제가 있었습니다.

`getStaticProps`

- 네비게이션 시, 데이터가 가져와져 단순한 캐시에 저장됩니다. 이 캐시는 본질적으로 { [url: string]: Data } 형태입니다.
- 뒤로/앞으로 네비게이션 시, 이 캐시에서 데이터를 재사용합니다.
- 이 동작에는 한 가지 문제점이 있습니다: 클라이언트 측 라우터 캐시를 무효화할 수 없기 때문에, 데이터를 변경해도 사용자가 탭을 새로 고치지 않으면 결과를 볼 수 없습니다. 이 캐시를 삭제할 API가 없었습니다.
- 스크롤 위치는 복원됩니다.

`getServerSideProps`

- 네비게이션 시, 데이터를 가져오기 위해 서버에 요청이 전송됩니다.
- 뒤로/앞으로 네비게이션 시에도 데이터 요청을 위해 서버로 요청이 전송됩니다.
- 뒤로/앞으로 네비게이션이 즉시 적용되지 않았습니다. 이는 요청이 진행 중일 때 스크롤 복원 휴리스틱에 도달하기 전이기 때문에, UX가 깨진 것처럼 느껴질 수 있습니다.

우리는 스크롤 위치가 보존되고 네비게이션이 즉각적으로 이루어지는 새로운 동작이 올바른 동작이라고 믿습니다. 이 동작은 브라우저의 bfcache 동작을 반영합니다.

이 동작에서 기억해야 할 주요 사항은, 뒤로/앞으로 네비게이션 시 사용되는 클라이언트 측 컴포넌트 캐시는 `router.refresh()`, `revalidatePath`, `revalidateTag`와 같은 무효화 함수들을 호출할 때만 삭제된다는 점입니다.

이는 이전에 해왔던 것과 크게 다르지 않습니다. 즉, 클라이언트 측 SPA를 완전히 구축할 때 상태 관리를 사용하여 페치 결과를 추적하는 자체 캐시를 구축하는 것과 유사합니다. 이러한 캐시도 마찬가지로 삭제되어야 하며, 이는 useSWR 또는 react-query에서도 마찬가지입니다.

## 라우터에 대한 단기 작업

현재 우리의 주요 초점은 앱 라우터(App Router)의 안정성을 개선하는 데 있으며, 보고된 버그를 조사하고 수정하는 데 상당한 시간을 할애하고 있습니다. 이 작업이 완료되면 클라이언트 측 라우터 성능을 개선하는 여러 기능에 대한 추가 작업이 진행될 예정입니다.

### 정적 레이아웃 최적화

현재 경로가 완전히 정적일 때(정적 렌더링), RSC 페이로드(RSC Payload)에 대해 URL을 가져오고, [전체 경로 캐시에서 HIT(캐시 적중)](https://nextjs.org/docs/app/building-your-application/caching#full-route-cache)가 발생합니다. 이는 요청이 정적 파일로 처리되어 상당히 빠르게 제공된다는 것을 의미합니다. 그러나 이 응답은 레이아웃 렌더링을 분할함으로써 더 최적화될 수 있습니다. 구체적으로, 각 레이아웃에 대해 RSC 페이로드를 생성할 수 있으며, 이는 동적 렌더링을 수행할 때처럼 더 세분화된 네비게이션을 가능하게 합니다.

### 네비게이션/새로고침 배칭 처리

현재 네비게이션 페치는 여러 React 전환(React Transitions) 간 상태가 올바르게 유지되도록 순차적으로 발생합니다. 그러나 React Experimental에서 새로운 기능이 있으며(서버 액션을 활성화하면 사용됩니다), 이 기능은 startTransition 함수 자체를 비동기(async)로 사용할 수 있게 하고, React는 여러 전환을 동시에 트리거하면서 그 순서를 추적할 수 있게 합니다. 우리는 이 기능을 활용하여 네비게이션을 여러 번 호출하는 경우의 속도를 높일 계획입니다. 예로, 입력 필드에 타이핑할 때 searchParams를 업데이트하는 경우가 있습니다.

### 서버 액션의 배칭 처리

현재 서버 액션은 순차적으로 실행됩니다. 일반적으로 일관성을 위해 호출된 순서대로 실행되어야 하지만, 여러 액션을 실행하기 위해 단일 요청으로 서버에 보낼 수 있습니다.

### 클라이언트/오프라인 아일랜드

애플리케이션의 일부 섹션을 SPA(단일 페이지 애플리케이션)와 유사하게 표시하여 네비게이션 시 서버에 요청을 보내지 않도록 하는 제안을 작업 중입니다. 이에 따른 트레이드오프는 해당 "아일랜드"(경로 집합)에서 로드할 수 있는 모든 가능한 JS의 매니페스트를 보내야 한다는 점입니다.

## 결론

이제 동적 렌더링 RSC 페이로드의 30초 캐싱을 구성하는 방법에 대해 이야기해 보겠습니다.

먼저, 현재 동작에서 발생하는 버그에 대해 설명하겠습니다. 피드백을 읽어보니, 30초가 지나기 전에 페이지로 다시 네비게이션할 때마다 캐시 시간이 30초씩 추가되는 버그가 있음을 발견했습니다. 이 버그를 수정할 예정이며, 페이지에서 벗어난 시점부터 30초로 변경할 것입니다. 예를 들어, `/account`로 네비게이션한 후 10초를 기다렸다가 `/products`로 돌아오면 시간이 30초 추가되지 않고, 대신 20초 후에 캐시 노드가 삭제됩니다(10초 후에 다시 클릭한 경우).

모든 것이 기본적으로 동적으로 설정된 또 다른 선택지가 있었을 수 있습니다. 이는 네비게이션할 때마다 애플리케이션의 루트부터 다시 렌더링하는 방식입니다.

이는 Pages Router에서 getServerSideProps가 작동하는 방식과 유사합니다. 하지만 이렇게 하면 몇 가지 문제가 발생할 수 있습니다. 예를 들어:

- 클라이언트에서 서버로 더 많은 컴포넌트를 이동시킬수록 RSC 페이로드가 증가하게 되며, 네비게이션이나 변동(mutation) 등 어떤 작업을 할 때마다 이를 다시 가져와야 합니다.
  - 특히 여러 페이지에서 공유되는 레이아웃이 있을 때 이러한 문제가 더 두드러집니다. 이것은 `<html>`을 포함하는 루트 레이아웃이나 대시보드 페이지 간 네비게이션 시의 대시보드 레이아웃일 수 있습니다. 부분 네비게이션(Partial Navigation)이 없다면 이러한 모든 상호작용에 대해 `<html>`부터 다시 가져와야 합니다.
    - 이로 인해 상당한 대역폭 사용이 발생하며, 이는 느린 네트워크에서 성능에 영향을 미치고 비용을 증가시킬 수 있습니다.
- 앞으로 우리는 이러한 부분 네비게이션을 더 작게 만드는 방법을 제공할 계획입니다. 예를 들어, 툴팁, 검색 상자, 탭, 페이지네이션, 아코디언과 같은 경우에 하나의 컴포넌트 또는 소수의 컴포넌트만 렌더링되도록 할 것입니다.
  - 이러한 네비게이션이 페이지와 레이아웃을 다시 가져와야 할지 여부는 아직 명확하지 않습니다. 라우터 캐시가 없고 모든 상호작용이 페이지를 완전히 다시 렌더링해야 한다면, 툴팁을 표시하기 위해서라도 모든 경로에 대한 데이터 페칭이 실행되어야 하므로 이 패턴은 느려질 수 있습니다.
- 모든 것이 동적으로 렌더링되고 캐싱 및 부분 네비게이션이 없을 경우, 서버에서 항상 전체 페이지를 다시 렌더링해야 합니다. 이는 모든 컴포넌트(예: 서드파티 npm 모듈 포함)가 자체 캐싱 메커니즘을 도입하여 속도를 유지해야 한다는 것을 의미합니다. 예를 들어, react-tweet은 디스크나 외부 저장소에 기록해야 할 수 있습니다.
  - 마찬가지로 브라우저에서 캐시가 없는 경우, 결국 직접 캐시를 도입하게 됩니다. 예를 들어, useSWR 또는 react-query는 자체 캐싱 동작을 도입했습니다. 혹은 Redux와 같은 상태 관리 라이브러리를 사용할 경우, 변경이 발생할 때마다 관리해야 하는 클라이언트 측 캐시를 스스로 구현하게 됩니다(mutate() 또는 dispatch() 호출 등).
    - 서버 컴포넌트와 함께 사용하는 SPA에서는 괜찮을 수 있지만, 라우터가 서버 컴포넌트 렌더링 방식에 깊이 통합되어 있기 때문에 외부 라이브러리에서 이러한 네비게이션/캐시를 수동으로 추적하는 것은 번거로울 수 있습니다. 이 때문에 react-query의 관리자가 서버 컴포넌트 환경에서는 [react-query가 필요하지 않을 수도 있다](https://tkdodo.eu/blog/you-might-not-need-react-query)고 언급한 것입니다.

우리는 Next.js와 RSC의 동작을 가능한 한 포크(fork)하지 않으려고 합니다. 그렇지 않으면 RSC 생태계에서 한 애플리케이션에서 사용하는 컴포넌트와 다른 애플리케이션에서 사용하는 컴포넌트 간의 차이가 발생할 수 있기 때문입니다.

또한, 가까운 미래에는 revalidatePath나 revalidateTag를 수동으로 호출할 필요가 줄어들 것으로 기대하고 있습니다. ORMs/SDKs와 같은 데이터 페칭 솔루션이 데이터를 삽입하거나 업데이트할 때 무효화를 자동으로 처리할 수 있기 때문입니다. 예를 들어, Prisma의 SDK가 자동으로 이러한 작업을 처리할 수 있을 것입니다.

```ts
// Without integration into the SDK
export async function updateUser(config) {
  "use server";
  await prisma.user.update(config);
  revalidateTag("prisma-user");
}

// With integration into the SDK
// If the SDK called revalidateTag and applied tags instead:
export async function updateUser(config) {
  "use server";
  // Automatically calls `revalidateTag` on your behalf.
  await prisma.user.update(config);
}
```

이 추가적인 맥락과 내부 작동 방식에 대한 설명, 변동 사항 처리 방법, stale-while-revalidate 패턴을 사용하여 신선한 데이터를 가져오는 방법, 그리고 버그 수정 사항을 고려했을 때, 이 설명이 이해가 되시나요? 아니면 여전히 해결되지 않은 사례가 있나요? 시간을 30초가 아닌 0초로 설정할 수 있다면, 비록 뒤로 네비게이션에는 영향을 미치지 않겠지만, 현재 직면하고 있는 문제를 해결할 수 있을까요? 알려주세요!

게시물의 마지막 부분부터 읽기 시작하셨다면, 이 토론에 대해 의견을 남기기 전에 처음부터 끝까지 전체 설명을 읽어보시기 바랍니다.
