## 🔗 [The guide to Git I never had](https://open.nytimes.com/enhancing-the-new-york-times-web-performance-with-react-18-d6f91a7c5af8)

### 🗓️ 번역 날짜: 2024.07.15

### 🧚 번역한 크루: 러기(박정우)

---

## React 18을 이용하여 New York Times 웹 성능 향상시키기

어떻게 리액트18 버전으로 업그레이드한 것이, New York Times 웹 사이트를 개선시켰는지, 그리고 그 과정에서 직면한 챌린지들을 어떻게 해결했는지에 대해서 설명합니다.

![Illustration by Ben Hickey](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*rT1gMg-9ensw1FDxaAaq_Q.gif)

### By Ilya Gurevich

뉴욕 타임스의 소프트웨어 엔지니어로서 우리는 페이지 성능, SEO, 최신 기술 유지에 큰 가치를 두고 있습니다. 이러한 우선순위를 염두에 두고, React 18의 출시가 웹 개발의 끊임없이 확장되는 세계에서 중요한 도약으로 다가왔습니다. React 기반 사이트에 대한 업그레이드는 성능 향상과 흥미로운 새로운 기능의 접근의 제공합니다. 지난 겨울, 우리는 주력 뉴스 사이트에 React 18의 기능을 도입하기로 결정했습니다. 그 과정에서 우리는 우리가 배워가면서 해결해 나가야 될 고유한 특이점을 직면했습니다. - 리액트와 우리 사이트 모두에서요.-

결국 우리는 큰 성능 향상을 이루었고, 여전히 탐구 중인 미래 개선의 가능성을 열었습니다.

업그레이드 과정을 자세히 살펴보기 전에, React 18의 주요 이점과 변경 사항 몇 가지를 살펴보겠습니다:

- **Concurrent Mode로 매끄러운 렌더링**: React 18은 업데이트와 사용자 상호 작용의 동시 렌더링을 허용하는 Concurrent Mode를 도입했습니다. 이는 더 부드러운 애니메이션, 적은 화면 지연 및 누적 레이아웃 이동, 더 응답성 있는 사용자 경험으로 이어집니다.
- **자동 배칭 및 전환**: 동시성을 최대한 활용하기 위해 React 18은 단일 렌더 사이클 내에서 상태 업데이트를 자동으로 배치하여 성능을 최적화합니다. 이는 주 스레드에서 작업을 나눔으로써 수행되며, 이전 메커니즘에서는 거의 모든 작업이 동기적으로 실행되었습니다. 새로운 `useTransition` 훅의 도입은 엔지니어들이 특정 상태를 UI를 차단하지 않고 업데이트할 수 있도록 합니다.
- **흥미로운 새로운 기능**: React 18은 서버 측 렌더링과 스트리밍 업데이트를 위한 React 서버 컴포넌트 및 선택적 Hydration과 같은 기능을 통해 혁신적인 UI 패턴과 더 빠른 초기 렌더링을 가능하게 합니다.

성능 향상은 특히 중요했는데, 이는 상호작용에서 다음 페인트(INP) 점수에서 큰 개선을 제공하기 때문이었습니다. INP는 페이지 응답성을 측정하는 지표로, 구글이 검색 결과에서 웹사이트를 순위 매기는 데 사용하는 최신 코어 웹 바이탈(Core Web Vital)중 하나입니다. SEO 점수는 뉴스 조직에게 매우 중요하며, INP 점수를 개선하는 것은 어려운 도전 과제였기 때문에 React 업그레이드는 높은 우선순위(그리고 높은 위험)로 다뤄졌습니다.

## 우리의 마이그레이션 과정

### **1. 더 이상 사용되지 않는 종속성 제거**

업그레이드를 시작하기 전에, React 18과 호환되지 않는 구식 Enzyme 테스트 라이브러리를 제거해야 했습니다. 이를 위해 모든 테스트 파일을 최신 라이브러리인 @testing-library/react로 수동으로 마이그레이션해야 했습니다. 시간 소요 면에서 이는 전체 프로젝트의 가장 큰 부분이었을 것입니다. Enzyme은 리포지토리 전반에 걸쳐 수백 개의 테스트 파일에서 사용되었으며, 이를 완전히 대체하기 위해 상당한 수작업과 수십 개의 풀 리퀘스트가 필요했습니다. 우리는 몇 개월에 걸쳐 점진적인 풀 리퀘스트를 통해 이 작업을 완료했으며, 다른 제품 작업을 수용하면서도 개발자의 피로를 줄일 수 있었습니다. 이 작업이 끝난 후 우리는 @testing-library/react API에 대해 전문가가 된 것 같았고, 드디어 React 18 업그레이드 자체로 넘어갈 수 있게 되었습니다.

### **2. 기초 작업**

테스트 파일 마이그레이션을 마친 후, React 18을 통합하는 작업을 시작할 수 있었습니다. 이를 안전하게 수행하기 위해, 우리는 먼저 모든 주요 종속성, 타입 및 테스트를 React 18에 맞게 업그레이드했습니다. 이는 최신 기능 자체를 구현하지 않고 @types/react, react-test-renderer, react-dom, @testing-library 등의 모든 것을 최신 버전으로 업그레이드하는 작업을 포함했습니다. 주요 종속성을 업그레이드하는 과정에서는 최신 버전에 맞게 일부 테스트와 타입 정의를 리팩토링하는 작업도 포함되었습니다.

### **3. 엔진 가동**

패키지 업그레이드에 자신감이 생기면서, React 18의 새로운 기능을 안전하게 통합할 준비가 되었습니다. 기능을 실현하기 위해 우리는 최신 API인 `createRoot`와 `hydrateRoot`를 사용해야 했습니다. 우리는 여러 웹 서버에 React Hydration을 통합했으며, 모든 서버에서 공통으로 렌더링되는 공유 UI 컴포넌트를 갖고 있기 때문에 가능한 많은 곳에서 React 18 기능을 활성화하는 것이 중요했습니다. 처음에는 `ReactDOM.hydrate`에서 `hydrateRoot`로 참조를 변경하는 것만으로 간단해 보였습니다. 그러나 실제로는 어땠을까요?

### 예상치 못한 도전 과제

개발자로서 "배포" 버튼을 누를 때 자신감이 넘치기 쉽습니다. 통합 테스트와 유닛 테스트가 통과되고, 다양한 표면과 장치에서 QA를 완료했으며, 최신 기능을 출시하기 직전입니다. 우리가 뉴욕 타임스 웹사이트에 React의 최신 버전을 배포할 때 모두 그렇게 느꼈습니다. 그러나 새로운 업그레이드를 처음 배포한 직후, 우리는 "임베디드 인터랙티브"라는 콘텐츠 유형에서 문제가 발생했습니다.

### React 18에 임베디드 인터랙티브 적응

![A custom embedded interactive built by our graphics developers](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*yqosD8m1lIRtZRh4)

A custom embedded interactive built by our graphics developers: [https://www.nytimes.com/article/hurricane-norma-baja-california.html](https://www.google.com/url?q=https%3A%2F%2Fwww.nytimes.com%2Farticle%2Fhurricane-norma-baja-california.html&sa=D&ust=1719425021820790&usg=AOvVaw0Wy7DlTRRsTiyaAjg31xhe)

뉴욕 타임스에서는 서버 측에서 [`dangerouslySetInnerHTML`](https://react.dev/reference/react-dom/components/common#dangerously-setting-the-inner-html)을 사용하여 렌더링된 맞춤형 임베디드 인터랙티브를 사용합니다. 이러한 인터랙티브는 자체 HTML, 링크 및 스크립트를 가지고 있으며, React 트리와 독립적으로 실행됩니다. 이를 통해 편집자와 기자는 핵심 인프라를 변경하거나 재배포하지 않고도 개별적이고 독립적인 시각적 및 인터랙티브 요소를 페이지에 주입할 수 있습니다. 임베디드 인터랙티브는 우리의 가장 영향력 있는 보도의 핵심이지만, 개발자에게는 챌린지가가 되었습다.

간단한 예시는 다음과 같습니다 (script 태그가 페이지가 열리자마자 DOM을 수정함):

```jsx
const embeddedInteractiveString = `
  <div id="server-test">server</div>
  <script>
    document.addEventListener("DOMContentLoaded", () => {
      const serverTestElement = document.getElementById("server-test");
      serverTestElement.textContent = "client";
    });
  </script>
`;
return <div dangerouslySetInnerHTML={{ __html: embeddedInteractiveString }} />;
```

이 설정에서는 스크립트가 페이지 로드 후 "server-test" 요소의 내용을 "server"에서 "client"로 수정합니다. 이는 브라우저에서 렌더링된 스크립트가 React가 DOM을 Hydration하기 전에 실행되기 때문에 작동합니다. 이는 기본적으로 "블랙 박스"와 같으며, 주입된 HTML과 그 스크립트가 의도한 대로 동작할 것이라고 신뢰합니다.

### Hydration 문제

React 18의 도입과 함께 더 엄격한 hydration 불일치 요구 사항이 등장했습니다. 새로운 규칙에 따르면, 초기 브라우저 로드와 클라이언트 측 hydration 사이에 발생하는 모든 DOM 수정은 클라이언트 측 렌더링으로의 fallback을 트리거합니다. 예를 들어, 스크립트 태그가 hydration 전에 "server-test" 요소를 수정하더라도 hydration 불일치가 발생하면 React는 서버 렌더링된 콘텐츠를 폐기하고 클라이언트 측 렌더링으로 fallback하여 스크립트의 영향을 실질적으로 무효화합니다. 이전 버전의 React에서는 hydration 불일치가 발생하더라도 DOM의 버전을 무효 상태로 두는 것을 선택했기 때문에 이러한 문제를 경험하지 않았습니다.

실제로 이것이 의미하는 바는 무엇일까요? 클라이언트에서 `dangerouslySetInnerHTML` 속성을 사용하여 컴포넌트를 렌더링할 때, 그 안에 `<script>` 태그가 포함된 HTML 조각은 브라우저 보안 고려 사항으로 인해 실행되지 않습니다. 이는 `dangerouslySetInnerHTML` 속성을 사용하여 hydration 불일치로 인해 클라이언트에서 다시 렌더링된 모든 임베디드 인터랙티브가 자바스크립트가 실행되지 않은 것처럼 렌더링된다는 것을 의미합니다. 위의 예시에서 텍스트 콘텐츠는 "server"에서 "client"로 변경되지만, hydration 불일치가 발생하면 "server"로 다시 렌더링됩니다. 이는 일부 임베디드 인터랙티브가 예상 렌더링과 크게 다르게 보이게 만들었습니다.

### 예상 값

![예상 이미지](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*iL0ORQWV_QeM9ty5)

실제 값

![실제 이미지](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*kcvw3Bu9X7OLN-1x)

### 그래서 우리가 해야하는 것은?

React 18이 React 16보다 hydration 불일치에 훨씬 민감하다는 점을 감안할 때, 우리에게는 두 가지 선택지가 있었습니다. 첫 번째는 웹사이트의 모든 잠재적 hydration 불일치를 수정하는 것이고, 두 번째는 hydration 불일치가 발생할 경우 임베디드 인터랙티브가 클라이언트에서 다시 마운트되도록 적응하는 것이었습니다. 수백 가지의 다른 컴포넌트와 수만 개의 맞춤형 임베디드 인터랙티브를 포함한 수백만 개의 기사를 게시한 뉴욕 타임스에게 이는 상당한 딜레마였습니다. 물론 우리는 모든 hydration 불일치를 수정하고 싶었지만, 이를 안전하게 수행할 수 있는 방법은 무엇이었을까요?

결국 우리는 두 가지 문제를 동시에 해결하기로 결정했습니다.

## 임베디드 인터랙티브 스크립트를 수동으로 추출하고 실행하기

우리는 innerHTML 속성을 통해 추가된 스크립트 태그가 자동으로 실행되지 않는다는 것을 알고 있습니다. 그렇다면 이를 어떻게 해결할까요? 스크립트 태그는 다른 요소의 자식 노드로 수동으로 추가되거나 교체될 때만 실행됩니다. 즉, 스크립트 태그를 제대로 실행하려면 먼저 인터랙티브 HTML에서 스크립트를 추출하고 제거한 다음, 컴포넌트가 다시 렌더링될 때 올바른 위치에 다시 추가해야 합니다.

```jsx
// 이 함수는 일반적인 html에서 스크립트 태그를 빈 자리 표시자로 교체합니다.
// 이를 통해 클라이언트 마운트 시 실제 스크립트로 스크립트 태그 참조를 제자리에 교체할 수 있습니다.
export const addsPlaceholderScript = (scriptText, id, scriptCounter) => {
  let replacementToken = "";
  let hoistedText = scriptText;

  replacementToken = `<script id="${id}-script-${scriptCounter}"></script>`;
  hoistedText = hoistedText.replace("<script", `<script id="${id}-script-${scriptCounter}"`);

  return {
    replacementToken,
    hoistedText,
  };
};

// 이 함수는 인터랙티브 HTML 문자열에서 <script> 태그를 추출하고 제거하며,
// 다음을 포함하는 객체를 반환합니다:
// - `scriptsToRunOnClient`: 클라이언트 마운트 시 실행할 스크립트 텍스트 배열.
// - `scriptlessHtml`: 스크립트가 제거된 수정된 HTML 문자열로, 빈 스크립트 참조가 포함됩니다.
export const extractAndReplace = (html, id) => {
  const SCRIPT_REGEX = /<script[\s\S]*?>[\s\S]*?<\/script>/gi;
  let lastMatchAdjustment = 0;
  let scriptlessHtml = html;
  let match;
  const scriptsToRunOnClient = [];
  let scriptCounter = 0;
  while ((match = SCRIPT_REGEX.exec(html))) {
    const [matchText] = match;
    if (matchText) {
      let hoistedText = matchText;
      let replacementToken = "";
      ({ hoistedText, replacementToken } = addsPlaceholderScript(hoistedText, id, scriptCounter));
      scriptCounter += 1;
      const start = match.index - lastMatchAdjustment;
      const end = match.index + matchText.length - lastMatchAdjustment;
      scriptlessHtml = `${scriptlessHtml.substring(0, start)}${replacementToken}${scriptlessHtml.substring(
        end,
        scriptlessHtml.length
      )}`;
      scriptsToRunOnClient.push(hoistedText);
      lastMatchAdjustment += matchText.length - replacementToken.length;
    }
  }

  return {
    scriptsToRunOnClient,
    scriptlessHtml,
  };
};

// 클라이언트에서 스크립트 실행
const runScript = (clonedScript) => {
  const script = document.getElementById(document.getElementById(`${clonedScript.id}`));
  script.parentNode.replaceChild(clonedScript, script);
};
```

왜 스크립트를 서버에 두고 클라이언트에서 다시 실행하지 않느냐고 묻는 분도 있을 것입니다. 일부 시나리오에서는 일부 스크립트 태그가 함수 클로저 내에서가 아니라 전역적으로 변수를 선언하기 때문에 불가능합니다. 이러한 스크립트 태그를 서버에서 미리 렌더링한 다음 클라이언트에서 다시 실행하려고 하면, 전역 변수를 재선언할 수 없기 때문에 오류가 발생합니다.

이 초기 솔루션은 많은 임베디드 인터랙티브를 수정했습니다. 하지만 모든 인터랙티브가 임의의 순서로 스크립트를 실행하는 데 잘 맞지 않았습니다. 여기서 우리는 몇 가지 세부 사항을 조정했습니다:

### 스크립트 로드 순서

일부 인터랙티브 스크립트는 임베디드 인터랙티브 HTML에 다시 추가될 때 올바른 순서로 로드되어야 합니다. 이전 스크립트 실행 전략은 모든 <script> 태그가 이미 서버에서 선언되고 미리 렌더링되었음을 자동으로 가정했습니다. 이제 스크립트 태그를 제거하고 클라이언트에서 다시 마운트함에 따라 이러한 원리에 기반한 고유한 로직이 깨질 수 있습니다. 예를 들어보겠습니다.

```html
<script>
  const results = document.getElementById("RESULTS_MANIFEST").innerHTML.ELECTION_RESULTS;
  // 결과를 이용한 추가 로직
</script>
<div>인터랙티브 DOM 콘텐츠가 여기에 들어갑니다.</div>
<script id="RESULTS_MANIFEST">
  {"ELECTION_RESULTS": ['result1', 'result2', ....]}
</script>
```

위 시나리오에서, 초기 스크립트는 다른 스크립트 태그를 ID로 검색한 다음 두 번째 스크립트 태그의 innerHTML을 기반으로 일부 기존 로직을 실행합니다. 이전 버전에서는 스크립트 태그가 서버에서 미리 렌더링되었기 때문에 ID로 스크립트 태그를 참조하는 데 문제가 없었습니다.

최적의 상호작용을 위해 스크립트 실행은 특정 순서를 따라야 합니다. 여기에는 다음이 포함됩니다:

1. 정적 데이터를 포함하는 비기능적으로 드러난 스크립트를 먼저 추가합니다.
2. src 속성이 있는 스크립트를 비동기적으로 실행합니다.
3. 마지막으로, innerHTML에 순수 자바스크립트가 포함된 스크립트를 추가하고 실행합니다.

이 순서는 스크립트가 제대로 로드되기 전에 서로를 참조하는 것을 방지합니다.

```jsx
// 제공된 스크립트 태그를 분석하여 정렬 우선 순위를 반환합니다.
// 우선 순위 1: JSON 또는 기타 메타데이터 콘텐츠.
// 우선 순위 2: 기타 순수 JS 또는 src 콘텐츠
export const getPriority = (template) => {
  let priority;
  try {
    JSON.parse(template.innerHTML);
    priority = 1;
  } catch (err) {
    priority = 2;
  }
  return priority;
};

scripts.sort((a, b) => getPriority(a) - getPriority(b));
```

### 즉각적인 성능 이점

임베디드 인터랙티브 코드의 이러한 세밀한, 거의 외과적인 조작을 통합한 후, 우리는 React 18을 다시 안전하게 출시할 수 있다고 느꼈습니다. 40,000개에 가까운 맞춤형 임베디드 인터랙티브를 광범위하게 QA할 수는 없었지만, 그래픽 팀이 자주 사용하는 몇 가지 재사용 가능한 템플릿에 의존할 수 있었습니다. 이를 통해 Svelte 또는 Adobe Illustrator 기반의 임베디드 인터랙티브 내에서 특정 동작을 검증할 수 있었습니다. 장기적으로는 남아 있는 hydration 불일치를 완전히 제거하고 완전한 안심을 달성할 계획입니다. 하지만 단기적으로는 다시 "배포" 버튼을 누를 준비가 되었습니다.

새로운 기능을 출시하고 (이슈에 대한 내부 경고를 한 시간 동안 긴장하며 모니터링한 후) 우리는 거의 즉각적인 성능 향상을 확인할 수 있었습니다.

![INP 성능 이미지](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*MtMev3JHmEYYsbne)

이 차트에서 볼 수 있듯이, p75 범위의 INP 점수가 약 30% 하락했습니다!

업그레이드 이전에, 우리의 가장 큰 도전 과제 중 하나는 뉴스 사이트가 페이지를 로드할 때 자주 발생하는 재렌더링이었습니다. 이는 사용자가 아직 로딩 중인 페이지와 상호작용하려 할 때 나쁜 사용자 경험을 초래했고, 그로 인해 INP 점수가 기대에 미치지 못했습니다.

React 18 업그레이드 이후, 우리의 재렌더링 횟수는 사실상 절반으로 줄었습니다!

![리렌더링 성능 이미지](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*_jaE9Y5ugAi1TfSS)

이 두 가지 매우 눈에 띄고 중요한 개선 사항은 React 18의 자동 배칭 및 동시성 기능의 직접적인 결과입니다. 이는 우리가 올바른 방향으로 나아가고 있다는 매우 명확하고 긍정적인 신호를 주었습니다.

## 앞으로의 계획

React 18 통합은 이미 우리에게 상당한 개선을 가져왔고, 이전에는 불가능했던 다양한 가능성을 열어주었습니다. 이제 우리는 `startTransition` 및 React `Server Component`와 같은 새로운 기능의 잠재적 이점을 탐구하는 데 집중하고 있습니다. 우리의 핵심 목표는 INP 점수를 지속적으로 낮추고 전체 기능성을 개선하는 것입니다. 그러나 이러한 개선 사항에 대해 아직 답변이 필요한 질문들이 있음을 인식하고 있습니다. 현재로서는 우리가 사용하는 React 버전의 안정적이고 신뢰할 수 있는 성능을 보장하는 것이 주요 과제입니다.

뉴스 사이트에서의 결과를 바탕으로, 우리는 다른 사이트들에서도 유사한 성능 향상을 기대하며 업그레이드를 진행할 자신감을 얻었습니다. 우리는 구글의 [March deadline](https://developers.google.com/search/blog/2023/05/introducing-inp?hl=ko) 이전에 INP 점수를 "나쁨" 영역에서 벗어나게 할 수 있었고, 검색 알고리즘의 일부가 되었을 때도 부정적인 SEO 결과를 보지 않았습니다. 우리는 독자들이 조금 더 빠른 경험을 즐기고 있다고 생각합니다. 그리고 우리의 뉴스룸은 렌더링 프레임워크에 대해 다시 생각할 필요 없이 강력하고 흥미로운 인터랙티브 콘텐츠를 계속해서 제공할 것입니다.
