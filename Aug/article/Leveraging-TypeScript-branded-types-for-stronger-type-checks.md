## 🔗 [Leveraging TypeScript branded types for stronger type checks](https://blog.logrocket.com/leveraging-typescript-branded-types-stronger-type-checks/)

### 🗓️ 번역 날짜: 2024.08.04

### 🧚 번역한 크루: 렛서(김다은)

---

## TypeScript 브랜드 타입을 활용하여 더 강력한 타입 검사하기

TypeScript의 브랜드 타입은 더 명확한 코드를 작성하고 더 타입 안전한 솔루션을 제공할 수 있게 해줍니다. 이 강력한 기능은 구현하기도 매우 간단하며, 코드 유지 보수를 더 효율적으로 할 수 있게 도와줍니다.

<br/>

![](https://blog.logrocket.com/wp-content/uploads/2024/07/Leveraging-TypeScript-branded-types-stronger-type-checks.png)

<br/>

이 기사에서는 간단한 예제부터 시작하여 몇 가지 고급 사용 사례에 이르기까지, TypeScript 코드에서 브랜드 타입을 효과적으로 사용하는 방법을 배워보겠습니다. 시작해봅시다.

## TypeScript의 브랜드 타입이란 무엇인가요?

TypeScript의 브랜드 타입은 코드 가독성, 맥락, 타입 안전성을 향상시키기 위한 매우 강력하고 효율적인 기능입니다. 이 타입은 기존 타입에 추가 정의를 제공하여 구조와 파일 이름이 유사한 엔티티를 비교할 수 있게 합니다.

예를 들어, 문자열로 사용자 이메일을 저장하는 대신, 이메일 주소에 대한 브랜드 TypeScript 타입을 만들어 일반 문자열과 구분할 수 있습니다. 이렇게 하면 엔티티를 더 체계적으로 검증할 수 있고 코드도 명확해집니다.

브랜드 타입 없이 우리는 일반적으로 대부분의 경우처럼 값들을 제네릭 타입 변수에 저장합니다. 브랜드 타입을 사용하면 해당 변수를 강조하고 코드 전반에서 그 유효성을 유지할 수 있습니다.

## 간단한 TypeScript 브랜드 타입 예제

이메일 주소를 위한 브랜드 타입을 만들어봅시다. type에 브랜드 이름을 붙이는 것으로 브랜드 타입을 생성할 수 있습니다. 이 경우, 이메일 주소에 대한 브랜드 타입은 다음과 같습니다:

```ts
type EmailAddress = string & { __brand: 'EmailAddress' };
```

여기서, 우리는 \_\_brand 이름 'EmailAddress'를 붙여서 브랜드 타입 EmailAddress를 만들었습니다.

브랜드 타입을 만들 때 제네릭 문법은 필요없다는 사실을 명심하세요. 브랜드 타입의 제네릭(문자열, 숫자 등)에 따라 달라집니다. \_\_brand 문법도 예약어가 아니므로 \_\_brand 외의 다른 변수 이름을 사용할 수 있습니다.

이제 EmailAddress 타입의 객체를 만들고 문자열을 전달해 봅시다:

```ts
const email: EmailAddress = 'asd'; // error
```

보시는 바와 같이 타입 에러가 발생합니다.

```shell
Type 'string' is not assignable to type 'EmailAddress.' Type 'string' is not assignable to type '{ __brand: "EmailAddress"; }.'
```

이를 고치기 위해서 이메일 주소에 대한 기본적인 유효성 검사를 생성해봅시다.

```ts
const isEmailAddress = (email: string): email is EmailAddress => {
  return email.endsWith('@gmail.com');
};
```

여기서 boolean 값을 반환하는 대신, `email is EmailAddress`를 반환하고 있습니다. 이는 함수가 `true`를 반환하면 email이 `EmailAddress` [타입으로 캐스팅](https://blog.logrocket.com/how-to-perform-type-casting-typescript/)된다는 의미입니다. 이를 통해 문자열에 대해 어떤 작업을 수행하기 전에 유효성을 검증할 수 있습니다.

```ts
const sendVerificationEmail = (email: EmailAddress) => {
  //...
};
const signUp = (email: string, password: string) => {
  //...
  if (isEmailAddress(email)) {
    sendVerificationEmail(email); // pass
  }
  sendVerificationEmail(email); // error
};
```

보시는 바와 같이, 오류는 if 조건문 안에서 보이지 않지만, 해당 조건문의 범위를 벗어나면 발생합니다.

`assert`를 사용하여 이메일 주소를 검증할 수도 있습니다. 이렇게 하면 검증이 통과되지 않을 경우 오류를 발생시키고자 할 때 유용할 수 있습니다.

```ts
function assertEmailAddress(email: string): asserts email is EmailAddress {
  if (!email.endsWith('@gmail.com')) {
    throw new Error('Not an email addres');
  }
}

const sendVerificationEmail = (email: EmailAddress) => {
  //...
};

const signUp = (email: string, password: string) => {
  //...
  assertEmailAddress(email);
  sendVerificationEmail(email); // ok
};
```

여기서 보시다시피, 반환 타입으로 `asserts email is EmailAddress`를 명시하고 있습니다. 이는 검증이 통과하면 이메일 주소가 브랜드 타입 `EmailAddress`임을 보장합니다.

## TypeScript에서 브랜드 타입의 고급 사용 사례

위 예시는 브랜드 타입의 간단한 시연입니다. 우리는 이를 더 고급 사례에서도 사용할 수 있습니다. 예시를 하나 보겠습니다.

먼저, 다른 타입에 붙일 수 있는 공통 Branded 타입을 선언해 봅시다:

```ts
declare const __brand: unique symbol;
type Brand<B> = { [__brand]: B };
export type Branded<T, B> = T & Brand<B>;
```

여기서, 심볼을 사용해 각 타입을 구분하는 고유한 브랜드를 만듭니다. 심볼은 다른 어떤 심볼과도 구별되는 유일한 심볼임을 보장합니다. 이는 새로운 심볼을 생성할 때마다 다른 심볼과 구별된다는 것을 의미합니다. 다음은 이를 설명하는 예제입니다:

```ts
// 브랜드로 사용할 고유 심볼 정의
const metersSymbol: unique symbol = Symbol('meters');
const kilometersSymbol: unique symbol = Symbol('kilometers');

// 브랜드 타입 정의
type Meters = number & { [metersSymbol]: void };
type Kilometers = number & { [kilometersSymbol]: void };

// 브랜드 값을 생성하는 도우미 함수
function meters(value: number): Meters {
  return value as Meters;
}

function kilometers(value: number): Kilometers {
  return value as Kilometers;
}

// 브랜드 타입을 가진 변수들
const distanceInMeters: Meters = meters(100);
const distanceInKilometers: Kilometers = kilometers(1);

// 아래 할당은 타입 오류를 발생시킴
const wrongDistance: Meters = distanceInKilometers;
const anotherWrongDistance: Kilometers = distanceInMeters;

// 올바른 사용법
const anotherDistanceInMeters: Meters = meters(200);
const anotherDistanceInKilometers: Kilometers = kilometers(2);

console.log(distanceInMeters, distanceInKilometers);
```

공통 Branded 타입 인터페이스를 가지면 TypeScript에서 여러 브랜드 타입을 동시에 생성할 수 있어, 코드 구현을 줄이고 코드를 훨씬 깔끔하게 만들 수 있습니다. 이제 위의 이메일 검증 예제를 확장하여, 이 공통 Branded 타입을 사용해 EmailAddress 브랜드를 다음과 같이 정의할 수 있습니다:

```ts
type EmailAddress = Branded<string, 'EmailAddress'>;

const isEmailAddress = (email: string): email is EmailAddress => {
  return email.endsWith('@gmail.com');
};

const sendEmail = (email: EmailAddress) => {
  // ...
};

const signUp = (email: string, password: string) => {
  if (isEmailAddress(email)) {
    // 인증 메일 전송
    sendEmail(email);
  }
};
```

이제 이 Branded 타입을 사용하여 새로운 브랜드 TypeScript 타입을 만들 수 있습니다. Branded 타입을 사용하는 또 다른 예를 살펴봅시다. 사용자가 게시물을 좋아할 수 있도록 하는 함수를 작성한다고 가정해 보겠습니다. userId와 postId 모두에 Branded 타입을 사용할 수 있습니다:

```ts
type UserId = Branded<string, 'UserId'>;
type PostId = Branded<string, 'PostId'>;

type User = {
  userId: UserId;
  username: string;
  email: string;
};

type Post = {
  postId: PostId;
  title: string;
  description: string;
  likes: Like[];
};

type Like = {
  userId: UserId;
  postId: PostId;
};

const likePost = async (userId: UserId, postId: PostId) => {
  const response = await fetch(`/posts/${postId}/like/${userId}`, {
    method: 'post',
  });
  return await response.json();
};

// 가상의 객체
const user: User = {
  userId: '1' as UserId,
  email: 'a@email.com',
  username: 'User1',
};
const post: Post = {
  postId: '2' as PostId,
  title: 'Sample Title',
  description: 'Sample post description',
  likes: [],
};

likePost(user.userId, post.postId); // ok
likePost(post.postId, user.userId); // error
```

## TypeScript 5.5-beta에서 브랜드 타입으로 작업하기

새로운 TypeScript 5.5-beta 릴리스에서는 TypeScript의 제어 흐름 분석이 코드가 진행됨에 따라 변수의 타입이 어떻게 변하는지 추적할 수 있게 되었습니다.

이는 변수의 타입이 코드 로직에 따라 변경될 수 있으며, TypeScript가 코드 로직의 각 수정 체인에서 변수 타입을 추적한다는 것을 의미합니다. 변수가 가질 수 있는 두 가지 가능한 타입이 있는 경우, 필요한 조건을 적용하여 변수의 타입을 분리할 수 있습니다.

아래 코드를 보면서 이해해봅시다:

```tsx
interface ItemProps {
  // ...
}

declare const items: Map<string, ItemProps>;

function getItem(id: string) {
  const item = items.get(id); // item은 ItemProps | undefined 타입으로 선언됨
  if (item) {
    // if 문 안에서 item은 ItemProps 타입을 가짐
  } else {
    // 여기서 item은 undefined 타입을 가짐
  }
}

function getAllItemsByIds(ids: string[]): ItemProps[] {
  return ids.map((id) => items.get(id)).filter((item) => item !== undefined); // 이전에는 이런 오류가 발생했습니다: Type '(ItemProps | undefined)[]' is not assignable to type 'ItemProps[]'. Type 'ItemProps | undefined' is not assignable to type 'ItemProps'. Type 'undefined' is not assignable to type 'ItemProps'
}
```

다음 예제를 살펴보겠습니다. 이 예제에서는 이메일 주소 목록을 가져와서 검증된 이메일 주소를 데이터베이스에 저장합니다. 이전 예제를 사용하여 브랜드 EmailAddress 타입을 생성하고 검증된 이메일을 데이터베이스에 저장할 수 있습니다:

```ts
type EmailAddress = Branded<string, 'EmailAddress'>;
const isEmailAddress = (email: string): email is EmailAddress => {
  return email.endsWith('@gmail.com');
};

const storeToDb = async (emails: EmailAddress[]) => {
  const response = await fetch('/store-to-db', {
    body: JSON.stringify({
      emails,
    }),
    method: 'post',
  });
  return await response.json();
};

const emails = ['a@gmail.com', 'b@gmail.com', '...'];
const validatedEmails = emails.filter((email) => isEmailAddress(email));
storeToDb(validatedEmails); // error
```

여기서 검증된 이메일 주소를 validatedEmails 배열에 나열하고 있지만, storeToDb 함수에 전달할 때 오류가 발생하는 것을 볼 수 있습니다. 이 오류는 특히 v5.5-beta 이전의 TypeScript 버전을 사용할 때 나타납니다.

낮은 버전의 TypeScript에서는 validatedEmails 배열의 타입이 원래 변수인 emails 배열에서 파생됩니다. 그래서 validatedEmails 배열의 타입이 string[]로 나타납니다. 그러나 이 문제는 현재 TypeScript 베타 버전(현재 기준으로 5.5-beta)에서 해결되었습니다.

현재 베타 버전에서는 검증된 이메일을 필터링한 후에 validatedEmails가 자동으로 EmailAddress[]로 타입 캐스팅됩니다. 따라서 TypeScript 5.5-beta 버전에서는 오류가 발생하지 않습니다. 프로젝트에 TypeScript 베타 버전을 설치하려면 터미널에서 다음 명령어를 실행하십시오:

```shell
npm install -D typescript@beta
```

<br/>

## 결론

브랜드 타입은 TypeScript에서 매우 유용한 기능입니다. 이들은 런타임 타입 안전성을 제공하여 코드 무결성을 보장하고 가독성을 향상시킵니다. 또한 도메인 수준에서 오류를 발생시켜 코드의 버그를 줄이는 데 매우 유용합니다.

브랜드 타입을 쉽게 검증하고, 검증된 객체를 프로젝트 전반에 걸쳐 안전하게 사용할 수 있습니다. 최신 TypeScript 베타 버전 기능을 사용하면 코딩 경험을 더욱 원활하게 활용할 수 있습니다.
