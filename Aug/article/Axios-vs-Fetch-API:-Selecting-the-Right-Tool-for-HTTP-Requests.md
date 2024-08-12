## 🔗 [Axios vs. Fetch API: Selecting the Right Tool for HTTP Requests](https://medium.com/@johnnyJK/axios-vs-fetch-api-selecting-the-right-tool-for-http-requests-ecb14e39e285)

### 🗓️ 번역 날짜: 2024.08.05

### 🧚 번역한 크루: 소하(최소연)

---

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*quTpcb5i1NuOx235KQynOg.jpeg"/>

소프트웨어 개발 분야에서는 웹을 통해 원격 서버와 원활하게 상호 작용하고 데이터를 교환하는 능력이 매우 중요합니다. API에서 데이터를 가져오거나, CRUD 작업을 수행하거나, 기타 네트워크 관련 작업을 수행하는 경우 HTTP 요청을 만드는 것의 중요성은 아무리 강조해도 지나치지 않습니다. Fetch와 Axios라는 두 개의 널리 사용되는 JavaScript 라이브러리는 HTTP 요청을 처리하고자 하는 개발자들에게 최고의 선택이 되었습니다.

이 기사에서는 Axios와 Fetch를 비교하여 다양한 작업을 수행하는 방법을 살펴보겠습니다. 이 글을 마칠 때쯤이면 두 API에 대해 더 잘 이해하게 될 것입니다.

### AXIOS

Axios는 네트워크 요청을 만드는 데 사용되는 third-party HTTP 클라이언트 라이브러리입니다. Promise를 기반으로 하며 요청 및 응답을 처리하기 위한 깔끔하고 일관된 API를 제공합니다.

콘텐츠 배포 네트워크(CDN) 또는 패키지 관리자(예: npm)를 사용하여 프로젝트에 추가할 수 있습니다. 핵심 기능 중 일부는 다음과 같습니다:
브라우저 내에서 XMLHttpRequests 수행, node.js 환경 내에서 http 요청 수행, 요청 취소, 요청 및 응답 가로채기

### FETCH

Axios와 마찬가지로 Fetch는 Promise 기반 HTTP 클라이언트입니다. 내장 API이므로 설치하거나 가져올 필요가 없습니다. 현대의 모든 브라우저에서 사용할 수 있으며 [caniuse](https://caniuse.com/fetch)에서 확인할 수 있습니다. 또한 node.js에서도 사용할 수 있습니다.

이제 두 인기 있는 라이브러리를 이해하기 위해 기본 구문, 기능 및 사용 사례를 비교해 보겠습니다.

### 기본 구문

Axios는 요청 매개변수(헤더, 데이터, 요청 메서드 등)를 간단하게 구성할 수 있는 연결 가능한 API를 제공하여 HTTP 요청을 간소화합니다.

다음은 Axios를 사용하여 사용자 정의 헤더와 함께 URL에 [POST] 요청을 보내는 방법입니다. Axios는 데이터를 자동으로 JSON으로 변환하므로 직접 변환할 필요가 없습니다.

```jsx
const axios = require('axios');

const url = 'https://jsonplaceholder.typicode.com/posts';
const data = {
	title: 'Hello World',
	body: 'This is a test post.',
	userId: 1,
};

axios
	.post(url, data, {
		headers: {
			Accept: 'application/json',
			'Content-Type': 'application/json;charset=UTF-8',
		},
	})
	.then(({ data }) => {
		console.log('POST request successful. Response:', data);
	})
	.catch((error) => {
		console.error('Error:', error);
	});
```

해당 코드를 fetch API 코드와 비교해 보면, 정확히 동일한 결과를 얻을 수 있습니다.

```jsx
const url = 'https://jsonplaceholder.typicode.com/todos';

const options = {
	method: 'POST',
	headers: {
		Accept: 'application/json',
		'Content-Type': 'application/json;charset=UTF-8',
	},
	body: JSON.stringify({
		title: 'Hello World',
		body: 'This is a test post.',
		userId: 1,
	}),
};

fetch(url, options)
	.then((response) => response.json())
	.then((data) => {
		console.log('POST request successful. Response:', data);
	});
```

Fetch는 POST 요청에 데이터를 보내는 데 body 속성을 사용하는 반면, Axios는 data 속성을 활용합니다. Axios는 서버의 응답 데이터를 자동으로 변환하지만, Fetch를 사용할 때는 response.json() 메서드를 호출하여 데이터를 JavaScript 객체로 파싱해야 합니다. 또한 Axios는 데이터 응답을 data 객체 내에 전달하는 반면, Fetch는 최종 데이터를 어떤 변수에도 저장할 수 있도록 합니다.

### 응답 및 오류 처리

Fetch는 로딩 프로세스에 대한 정확한 제어를 제공하지만 두 개의 약속을 처리해야 하므로 복잡성이 추가됩니다. 또한 응답을 다룰 때 Fetch는 .json() 메서드를 사용하여 JSON 데이터를 파싱해야 합니다. 그런 다음 검색된 최종 데이터는 어떤 변수에도 저장할 수 있습니다.

HTTP 응답 오류의 경우 Fetch는 자동으로 오류를 던지지 않습니다. 대신 서버에서 오류 상태 코드(예: 404 Not found)를 반환하더라도 응답을 성공한 것으로 간주합니다.

Fetch에서 HTTP 오류를 명시적으로 처리하려면 개발자는 `.then()` 블록 내에서 조건문을 사용하여 `response.ok` 속성을 확인해야 합니다. `response.ok`가 false이면 서버가 오류 상태 코드로 응답했음을 의미하므로 개발자는 해당 오류를 적절히 처리할 수 있습니다.

다음은 Fetch를 사용한 [GET] 요청입니다:

```jsx
fetch('https://jsonplaceholder.typicode.com/todos')
	.then((response) => {
		if (!response.ok) {
			throw Error(`HTTP error: ${response.status}`);
		}
		return response.json();
	})
	.then((data) => {
		console.log('Data received:', data);
	})
	.catch((error) => {
		console.error('Error message:', error.message);
	});
```

반면에 Axios는 data 속성에 직접 액세스할 수 있게 하여 응답 처리를 간소화합니다. 200-299 범위(성공적인 응답)를 벗어난 응답은 자동으로 거부합니다. `.catch()` 블록을 사용하면 응답을 받았는지, 받았다면 상태 코드가 무엇인지 등 오류에 대한 정보를 얻을 수 있습니다. 이는 실패한 응답도 해결되는 fetch와 대조됩니다.

다음은 Axios를 사용한 [GET] 요청입니다:

```jsx
const axios = require('axios');

axios
	.get('https://jsonplaceholder.typicode.com/todos')
	.then((response) => {
		console.log('Data received:', response.data);
	})
	.catch((error) => {
		if (error.response) {
			console.error(`HTTP error: ${error.response.status}`);
		} else if (error.request) {
			console.error('Request error: No response received');
		} else {
			console.error('Error:', error.message);
		}
	});
```

Axios는 자동으로 실패한 응답에 대해 오류를 발생시키므로, 오류 처리를 간소화하고 개발자가 각 응답의 성공 또는 실패를 수동으로 확인하지 않고도 적절한 오류 처리 로직을 구현하는 데 집중할 수 있도록 합니다.

### HTTP 요청 및 응답 가로채기(Intercepting)

Axios의 주요 기능 중 하나는 HTTP 요청을 인터셉트할 수 있다는 것입니다. HTTP 인터셉터는 애플리케이션에서 서버로 또는 그 반대로 HTTP 요청을 검사하거나 변경해야 할 때 유용합니다. 로깅, 인증 또는 실패한 HTTP 요청을 다시 시도하는 것과 같은 다양한 작업에 필수적입니다.

인터셉터를 사용하면 각 HTTP 요청에 대해 별도의 코드를 작성할 필요가 없습니다. HTTP 인터셉터는 요청 및 응답을 처리하는 방법에 대한 전역 전략을 설정하고자 할 때 유용합니다.

다음은 Axios를 사용하여 HTTP 요청을 인터셉트하는 방법입니다:

```jsx
const axios = require('axios');

// 인터셉터 요청 등록
axios.interceptors.request.use((config) => {
	// HTTP 요청이 전송되기 전에 메시지를 기록합니다.
	console.log('Request was sent');
	return config;
});

// GET 요청 전송
axios
	.get('https://jsonplaceholder.typicode.com/todos')
	.then(({ data }) => {
		console.log('Data received:', data);
	})
	.catch((error) => {
		console.error('Error:', error.message);
	});
```

이 코드에서는 `axios.interceptors.request.use()` 메서드를 사용하여 HTTP 요청이 전송되기 전에 실행할 코드를 정의합니다.

또한 `axios.interceptors.response.use()`를 사용하여 서버의 응답을 인터셉터할 수도 있습니다. 예를 들어 네트워크 오류가 발생한 경우 응답 인터셉터를 사용하여 동일한 요청을 다시 시도할 수 있습니다.

기본적으로 fetch()는 요청을 인터셉트할 방법을 제공하지 않지만, 우회책을 찾는 것은 어렵지 않습니다. 전역 fetch() 메서드를 재정의하고 다음과 같이 인터셉터를 정의할 수 있습니다.

다음은 Fetch를 사용하여 HTTP 요청을 인터셉트하는 방법입니다.

```jsx
fetch = ((originalFetch) => {
	return (...arguments) => {
		return originalFetch.apply(this, arguments).then((response) => {
			if (!response.ok) {
				throw new Error(`HTTP error: ${response.status}`);
			}
			console.log('Request was sent');
			return response;
		});
	};
})(fetch);

fetch('https://jsonplaceholder.typicode.com/todos')
	.then((response) => response.json())
	.then((data) => {
		console.log('Data received:', data);
	})
	.catch((error) => {
		console.error('Error:', error.message);
	});
```

Fetch는 Axios에 비해 더 많은 보일러플레이트 코드를 가지는 경향이 있습니다. 하지만 선택은 사용 사례와 개인적인 필요에 따라 이루어져야 합니다.

### 응답 시간 초과

응답 시간 초과는 Fetch와 Axios를 비교할 때 또 다른 중요한 영역입니다. 응답 시간 초과는 클라이언트가 서버의 응답을 기다리는 시간으로, 이 시간이 지나면 요청이 실패한 것으로 간주합니다.

Axios에서 시간을 설정하는 간결함은 일부 개발자가 Fetch보다 Axios를 선호하는 이유 중 하나입니다. Axios에서는 config 객체의 선택적 timeout 속성을 사용하여 요청이 중단되기 전에 밀리초 단위로 시간을 설정할 수 있습니다. 이 직관적인 접근 방식은 개발자가 요청 구성 내에서 직접 시간 초과 기간을 정의할 수 있게 하여 더 큰 제어와 구현 용이성을 제공합니다.

다음은 Axios를 사용하여 지정된 시간 제한이 있는 [GET] 요청입니다.

```jsx
const axios = require('axios');

// 타임아웃 시간을 밀리초 단위로 정의합니다.
const timeout = 5000; // 5초

// 타임아웃 속성이 있는 config 객체를 생성합니다.
const config = {
	timeout: timeout,
};

// 지정된 타임아웃으로 GET 요청을 보냅니다.
axios
	.get('https://jsonplaceholder.typicode.com/todos', config)
	.then((response) => {
		console.log('Data received:', response.data);
	})
	.catch((error) => {
		console.error('Error fetching data:', error.message);
	});
```

Axios를 사용하여 GET 요청을 보내고 있으며, 두 번째 인수로 config 객체를 전달하여 시간 제한을 지정하고 있습니다. 요청이 성공하면 수신한 데이터가 콘솔에 로그됩니다. 요청 중에 오류가 발생하거나 시간이 초과되면 오류 메시지가 콘솔에 로그됩니다.

Fetch는 AbortController 인터페이스를 통해 유사한 기능을 제공하지만 Axios 버전만큼 직관적이지는 않습니다.

다음은 Fetch를 사용하여 HTTP 요청을 인터셉트하는 방법입니다.

```jsx
const timeout = 5000; // 5초
// AbortController 인스턴스를 생성합니다.
const controller = new AbortController();
const signal = controller.signal;

// 지정된 타임아웃 후에 fetch 요청을 중단하기 위해 setTimeout 함수를 설정합니다.
const timeoutId = setTimeout(() => {
	controller.abort();
}, timeout);

// 지정된 URL과 신호로 fetch 요청을 보냅니다.
fetch('https://jsonplaceholder.typicode.com/todos', { signal })
	.then((response) => {
		if (!response.ok) {
			throw new Error('Network response was not ok');
		}
		return response.json();
	})
	.then((data) => {
		console.log('Data received:', data);
	})
	.catch((error) => {
		// 오류가 요청이 중단된 것 때문인지 확인합니다.
		if (error.name === 'AbortError') {
			console.error('Request timed out');
		} else {
			console.error('Error fetching data:', error.message);
		}
	})
	.finally(() => {
		// 요청이 완료된 후에 타임아웃이 발생하지 않도록 타임아웃을 해제합니다.
		clearTimeout(timeoutId);
	});
```

5초 타임아웃이 fetch 요청에 설정되며, 이 제한 시간을 초과할 경우 요청을 취소하기 위해 `AbortController`가 생성됩니다. `setTimeout`을 사용하여 이 기간이 지나면 요청을 중단시키는 타이머가 시작됩니다. 그런 다음 fetch 요청이 전송되며, 중단 신호가 포함됩니다. 응답이 도착하면 성공 여부를 확인하고 데이터를 파싱합니다. `timeouts`과 같은 오류는 적절하게 처리됩니다. 마지막으로, 요청이 완료된 후 불필요한 지연을 피하기 위해 타임아웃이 해제됩니다.

### 동시 요청

Fetch와 Axios를 비교할 때 또 다른 중요한 영역은 동시 요청 또는 여러 요청을 동시에 할 수 있는 기능입니다. 이 기능은 성능과 반응성이 중요한 애플리케이션에서 특히 중요합니다.

Axios는 `axios.all()` 메서드를 제공하여 여러 요청을 동시에 할 수 있습니다. 이 메서드에 요청 배열을 전달한 후 `axios.spread()`를 사용하여 응답을 개별 인수로 펼칠 수 있습니다. 이를 통해 각 응답을 개별적으로 처리할 수 있습니다.

다음은 Axios를 사용하여 동시에 HTTP 요청을 보내는 방법입니다:

```jsx
const axios = require('axios');

// 요청의 기본 URL 정의
const baseURL = 'https://jsonplaceholder.typicode.com/todos';

// 요청을 위한 URL 정의
const urls = [`${baseURL}/1`, `${baseURL}/2`, `${baseURL}/3`];

// Axios 요청 프라미스 배열 생성
const axiosRequests = urls.map((url) => axios.get(url));

// `axios.all()`을 사용하여 여러 요청을 동시에 전송
axios
	.all(axiosRequests)
	.then(
		axios.spread((...responses) => {
			// 모든 요청의 응답 처리
			responses.forEach((response, index) => {
				console.log(`Response from ${urls[index]}:`, response.data);
			});
		})
	)
	.catch((error) => {
		console.error('Error fetching data:', error.message);
	});
```

Fetch를 사용하여 동일한 결과를 얻기 위해서는 내장된 `Promise.all()` 메서드에 모든 fetch 요청을 배열로 전달해야 합니다. 그런 다음 async 함수를 사용하여 응답을 처리합니다.

다음은 Fetch를 사용하여 동시에 HTTP 요청을 보내는 방법입니다:

```jsx
// 요청할 base URL 정의
const baseURL = 'https://jsonplaceholder.typicode.com/todos';

// 요청할 URLs 정의
const urls = [`${baseURL}/1`, `${baseURL}/2`, `${baseURL}/3`];

// fetch 요청의 프로미스를 저장할 배열 생성
const fetchRequests = urls.map((url) => fetch(url));

// `Promise.all()`을 사용하여 여러 요청을 동시에 보냄
Promise.all(fetchRequests)
	.then((responses) => {
		// 모든 요청의 응답을 처리
		responses.forEach((response, index) => {
			if (!response.ok) {
				throw new Error(`Request ${urls[index]} failed with status ${response.status}`);
			}
			response.json().then((data) => {
				console.log(`Response from ${urls[index]}:`, data);
			});
		});
	})
	.catch((error) => {
		console.error('Error fetching data:', error.message);
	});
```

이 접근 방식은 실행 가능하지만, 비동기 프로그래밍 개념에 익숙하지 않은 개발자들에게는 추가적인 복잡성과 오버헤드를 초래할 수 있습니다.

### 하위 호환성

하위 호환성은 소프트웨어 시스템이나 제품이 종속성의 이전 버전과 함께 사용되거나 이전 환경에서 사용될 때에도 올바르게 또는 효과적으로 작동할 수 있는 능력을 말합니다.

Axios는 기본적으로 더 나은 하위 호환성을 제공하며, 이전 시스템이나 코드베이스와의 호환성을 용이하게 하는 추가 기능을 제공합니다. Axios의 주요 장점 중 하나는 XMLHttpRequest를 기반으로 하여 IE11과 같은 오래된 브라우저를 포함한 광범위한 브라우저를 지원한다는 것입니다.

반면에 Fetch는 Chrome, Firefox, Edge, Safari와 같은 현대 브라우저를 주로 대상으로 하는 더 제한된 브라우저 지원을 제공합니다. Fetch의 기능이 지원되지 않는 브라우저에서 프로젝트를 진행해야 하는 경우 'whatwg-fetch'와 같은 polyfill을 통합하여 격차를 해소할 수 있습니다. 이 폴리필은 Fetch 지원을 오래된 브라우저로 확장하여 호환성을 보장합니다.

사용하려면 npm 명령을 사용하여 설치할 수 있습니다:

```bash
npm install whatwg-fetch - save
```

그다음에는 다음과 같이 요청을 보낼 수 있습니다:

```jsx
import 'whatwg-fetch'

window.fetch(…)
```

일부 오래된 브라우저에서는 promise polyfill이 필요할 수도 있다는 점을 유의해 주세요.

### 결론

Axios는 간결함, 견고함, 광범위한 브라우저 지원으로 인해 오래된 브라우저와의 하위 호환성이 필요한 프로젝트에 이상적인 선택입니다. 반면에 Fetch는 스트리밍 기능과 다른 웹 플랫폼 API와 원활하게 작업할 수 있는 기능을 지원하는 브라우저에 직접 내장된 보다 네이티브한 접근 방식을 제공합니다.

Axios와 Fetch 중에서 선택할 때 개발자는 사용 편의성, 성능 고려사항, 브라우저 호환성 및 필요한 추가 기능 등 프로젝트 요구 사항을 신중하게 평가해야 합니다.
