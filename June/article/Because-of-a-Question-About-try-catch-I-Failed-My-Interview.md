# [try-catch 질문 때문에 면접에서 탈락했다](https://medium.com/@haiou-a/because-of-a-question-about-try-catch-i-failed-my-interview-2cea0225820c)

### 🗓️ 번역 날짜: 2024.06.03

### 🧚 번역한 크루: 렛서(김다은)

<br/>

---

`try...catch`는 코드 블록의 오류를 잡을 때 주로 사용되기 때문에 매우 친숙하게 느껴지고, 저도 꽤 자주 사용합니다.

하지만, 실은 `try...catch`에 대한 지식이 부족했기 때문에 의도치 않게 이런 기본 개념을 면접에서 배웠습니다.: `try...catch`는 동기 코드 블럭에서만 에러를 잡을 수 있습니다.

질문은 다음과 같았습니다: 다음 코드에 문제가 있나요? 그렇다면 어떻게 수정해야 하나요?

<br/>

```js
try {
  setTimeout(() => {
    throw new Error('err');
  }, 200);
} catch (err) {
  console.log(err);
}

try {
  Promise.resolve().then(() => {
    throw new Error('err');
  });
} catch (err) {
  console.log(err);
}
```

<br/>

'`try...catch` 자체는 동기 블록이기 때문에 비동기 에러를 잡을 수 없다.'.기본적으로 저는 이 지점을 몰랐기 때문에 무슨 일이 일어나고 있는지 알지 못했습니다. 그래서 이 질문을 들었을 때 혼란스러웠습니다.

오류를 잡기 위해 `try...catch`를 사용하는 것이 우리가 일반적으로 코드를 작성하는 방식이 아닌가요? 그래서 저는 그냥 모른다고 대답했고 오류가 없다고 생각했습니다... 면접관은 무력한 표정으로 저를 쳐다보며 나중에 찾아보라고 제안했고, 그게 끝이었습니다.

빠르게 조사한 결과 `try...catch`는 비동기 코드의 오류를 포착할 수 없다는 사실을 알게 되었습니다.

**JavaScript에서 setTimeout은 비동기 함수이며, 해당 콜백 함수는 지정된 지연 후 이벤트 큐에 배치되고 현재 실행 스택이 지워진 후에만 실행됩니다.**

따라서 setTimeout 콜백이 실행되어 오류가 발생하면 try...catch는 이미 실행이 완료되어 비동기 콜백의 오류를 잡을 수 없습니다.

올바른 접근 방식은 비동기 작업 내에서 직접 오류를 처리하는 것입니다. 예를 들어, 콜백 함수, Promise 또는 try...catch와 결합된 async/await을 사용하는 것 말이죠.

<br/>

```js
new Promise((resolve, reject) => {
  setTimeout(() => {
    try {
      throw new Error('err');
    } catch (err) {
      reject(err);
    }
  }, 200);
})
  .then(() => {
    // 예상대로 실행되었을 때 수행될 로직
  })
  .catch((err) => {
    console.log(err); // 에러는 여기서 잡힙니다
  });
```

<br/>

---

<br/>

두 번째 예시의 경우, `try...catch`를 사용해 프로미스 체인에서 발생한 오류를 잡으려는 시도도 비효율적입니다. 왜냐하면 `try...catch`는 프로미스 체인 내에서 비동기 오류를 잡을 수 없기 때문입니다.

프로미스 객체는 비동기 작업의 최종 완료(또는 실패)와 그 결과 값을 나타냅니다. 프라미스의 상태는 다음 중 하나일 수 있습니다:

- Pending(대기 상태): 초기 상태, 이행되지도 거부되지도 않은 상태.
- Fulfilled(완료된 상태): 작업이 성공적으로 완료되었음을 의미합니다.
- Reject(실패 상태): 작업이 실패했음을 의미합니다.

프로미스에서 에러를 발생시키면(예: throw 문을 통해) 프로미스가 거부(또는 실패)됩니다. 이 에러를 올바르게 처리하려면 프라미스 체인에서 `.catch` 메서드를 사용하거나 비동기 함수에서 `try...catch`를 사용해야 합니다.

<br/>

```js
// 메서드 1
Promise.resolve()
  .then(() => {
    throw new Error('err');
  })
  .catch((err) => {
    console.log(err); // 에러는 여기서 처리됩니다.
  });

// 메서드 2
async function handleError() {
  try {
    await Promise.resolve().then(() => {
      throw new Error('err');
    });
  } catch (err) {
    console.log(err); // 에러는 여기서 처리됩니다.
  }
}

handleError();
```

<br/>

### >> 결론

위의 사례에서 `try...catch` 구조는 비동기 코드에서 오류를 포착할 수 없다는 것을 이해해야 합니다.

비동기 연산을 다룰 때는 Promise에서 `.catch` 메서드를 사용하거나 `async/await` 구조 내에서 `try...catch`를 사용하여 오류를 처리하는 등 적절한 오류 처리 메커니즘을 채택하는 것이 중요합니다.

이 접근 방식은 비동기 작업 중에 발생할 수 있는 예외를 효과적으로 포착하고 처리할 뿐만 아니라 코드의 견고성과 유지보수성을 보장합니다.

일상적인 개발 작업에서 자바스크립트 런타임 메커니즘과 비동기 프로그래밍 모델에 대한 이해는 개인의 커리어 성장에 중요한 역할을 하므로 의식적으로 탐구해야 합니다.
