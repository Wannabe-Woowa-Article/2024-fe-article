## 🔗 [You must know about principle of z-index](https://dev.to/moondaeseung/you-must-know-about-principle-of-z-index-4akj)

### 🗓️ 번역 날짜: 2024.08.18

### 🧚 번역한 크루: 버건디(전태헌)

---

안녕하세요 저는 [Flitter](https://flitter.dev)의 메인테이너이고 최근에 z-index 위젯을 만들었습니다.

![](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fru1rp878soehm61vnhoh.png)

이 글에서는 z-index 속성들의 작동 방식과 어떻게 SVG들과 함께 활용할수 있는지 알아보겠습니다.

### 대상 독자

- z-index가 예상대로 적용되지 않아 어려움을 겪는 사람들
- 스택 컨텍스트(Stacking Context)의 개념에 익숙하지 않은 사람들
- SVG에서 직접적으로 z-index를 구현하려는 사람들
- 차트나 다이어그램에 z-index 동작을 추가하려는 사람들

### 키워드

- [z-index](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index)
- [스택 컨텍스트](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_positioned_layout/Understanding_z-index/Stacking_context)

## z-index란?

z-index는 요소들의 수직적 쌓임 순서를 조정하는 CSS 속성입니다. 기본값은 `auto`이며, 이는 `z-index: 0`과 동일합니다. 그러나 `z-index: 0`은 새로운 스택 컨텍스트를 생성하여, 자식 요소들에 대한 우선 순위를 `z-index: auto`와 다르게 변경합니다. 스택 컨텍스트는 아래에서 더 자세히 설명될 것입니다.

### 일반적인 Z 순서

지정된 z-index가 없는 경우, 스택 요소들의 수직적 순서는 DOM 노드가 렌더링되는 순서와 일치합니다. 이 순서는 깊이 우선 탐색(DFS)과 일관되며, 나중에 렌더링된 요소가 이전에 렌더링된 요소보다 우선순위를 갖습니다. 예를 들어, 분홍색 상자가 주황색 상자 위에 그려진다면, 분홍색 상자가 스택에서 더 높은 위치에 있게 됩니다.

![](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmt5a46d9omist0unkbc6.png)

### Z 요소 추가

보라색 원에 z-index를 9999로 지정하면, 해당 원이 가장 위에 위치하게 됩니다(일러스트에서처럼). 비록 깊이 우선 탐색에서 분홍색 상자가 마지막에 도달하더라도, z-index 재배열로 인해 보라색 원이 최우선으로 그려지게 됩니다.

![](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flr6pin0fximwkryvs2k6.png)

### 스택 컨텍스트란?

스택 컨텍스트는 요소의 z-index와 함께 해당 요소의 수직적 쌓임 우선순위를 결정합니다.
간단히 말해, 스택 컨텍스트를 부모의 z-index 값 배열로 생각할 수 있습니다.
자식 요소가 높은 z-index를 가지고 있더라도, 스택 컨텍스트에 의해 우선순위가 더 낮게 배치될 수 있습니다.

![](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1qb8q0o3qyx0f75jg2mv.png)

보라색 원이 z-index 값이 9999임에도 불구하고, 위의 일러스트에서 보이는 것처럼 부모의 z-index가 0으로 설정되어 새 스택 컨텍스트가 생성되었기 때문에 분홍색 상자에 의해 가려지게 됩니다.

- 스택 컨텍스트를 생성하는 상황

  스택 컨텍스트는 부모 요소의 z-index가 지정될 때뿐만 아니라 다음과 같은 조건에서도 형성될 수 있습니다:

  - 문서의 루트 요소 `<html>`
  - [position](<(https://developer.mozilla.org/en-US/docs/Web/CSS/position)>) 값이 `absolute` 또는 `relative`이고, [z-index](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index) 값이 auto가 아닌 요소.
  - [position](<(https://developer.mozilla.org/en-US/docs/Web/CSS/position)>) 값이 `fixed` 또는 `sticky`인 요소(모든 모바일 브라우저에서 sticky가 적용되지만, 이전의 데스크탑 브라우저에서는 그렇지 않음).
  - [container queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries)를 위해 [container-type](https://developer.mozilla.org/en-US/docs/Web/CSS/container-type) 값이 size 또는 inline-size로 설정된 요소.
  - [z-index](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index) 값이 `auto`가 아닌 [flex](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox) 컨테이너의 자식 요소.
  - [z-index](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index) 값이 `auto`가 아닌 [grid](https://developer.mozilla.org/en-US/docs/Web/CSS/grid) 컨테이너의 자식 요소.
  - [opacity](https://developer.mozilla.org/en-US/docs/Web/CSS/opacity) 값이 1보다 작은 요소([투명도에 대한 사양 참조](https://www.w3.org/TR/css-color-3/#transparency)).
  - [mix-blend-mode](https://developer.mozilla.org/en-US/docs/Web/CSS/mix-blend-mode) 값이 `normal`이 아닌 요소.
  - 다음 속성 중 하나라도 none이 아닌 값을 가지는 요소:
    - [transform](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)
    - [filter](https://developer.mozilla.org/en-US/docs/Web/CSS/filter)
    - [backdrop-filter](https://developer.mozilla.org/en-US/docs/Web/CSS/backdrop-filter)
    - [perspective](https://developer.mozilla.org/en-US/docs/Web/CSS/perspective)
    - [clip-path](https://developer.mozilla.org/en-US/docs/Web/CSS/clip-path)
    - [mask](https://developer.mozilla.org/en-US/docs/Web/CSS/mask) / [mask-image](https://developer.mozilla.org/en-US/docs/Web/CSS/mask-image) / [mask-border](https://developer.mozilla.org/en-US/docs/Web/CSS/mask-border)
  - [isolation](https://developer.mozilla.org/en-US/docs/Web/CSS/isolation) 값이 `isolate`인 요소.
  - 스택 컨텍스트를 생성할 수 있는 속성을 지정하는 [will-change](https://developer.mozilla.org/en-US/docs/Web/CSS/will-change) 값을 가진 요소([자세한 내용은 이 게시물 참조](https://blogs.opera.com/news/)).
  - `layout`, `paint` 또는 이 둘 중 하나를 포함하는 복합 값을 가지는 [contain](https://developer.mozilla.org/en-US/docs/Web/CSS/contain) 값이 설정된 요소(예: `contain: strict`, `contain: content`).
  - [top layer](https://developer.mozilla.org/en-US/docs/Glossary/Top_layer)에 배치된 요소와 그에 대응하는 [::backdrop](https://developer.mozilla.org/en-US/docs/Web/CSS/::backdrop). 예로는 [full screen](https://developer.mozilla.org/en-US/docs/Web/API/Fullscreen_API) 및 [popover](https://developer.mozilla.org/en-US/docs/Web/API/Popover_API) 요소가 포함됩니다.

## 스택 컨텍스트의 작동 규칙

이제 스택 컨텍스트가 어떻게 형성되고 노드들이 그려지는 순서에 어떤 영향을 미치는지 단계별로 설명해 보겠습니다.
z-index의 구현 논리에 바로 관심이 있는 분들은 다음 섹션인 "Z-Index 구현 논리"로 넘어가시기 바랍니다.

![](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fommypzrt2hg6893zsj7a.png)

위의 일러스트는 DOM 구조의 개요를 나타냅니다. 각 노드에 표시된 숫자는 전위 순회에서 방문되는 순서를 나타냅니다. 노드 2와 4는 각각 독립된 스택 컨텍스트를 가지고 있습니다. 노드 3은 노드 2로부터 **스택 컨텍스트**를 상속받습니다.

노드 2와 4를 비교할 때, 노드 4가 동일한 z-index를 가지고 있더라도, 더 높은 노드 번호를 가지므로 스택 순서에서 우선권을 가집니다. 노드 3은 노드 2로부터 **스택 컨텍스트**를 상속받기 때문에, 아무리 높은 z-index를 가지더라도 노드 4에 비해 스택 순서에서 낮게 평가됩니다.

### 예시: 복잡한 스택 컨텍스트

![](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5j84xznn4y473yfz04sp.png)

일러스트에서, 노드 3과 4는 노드 2 아래에 위치하고, 노드 7은 노드 6 아래에 위치해 있습니다.
특히, 노드 6과 같이 z-index가 -1인 스택 컨텍스트가 존재하며, 노드 5와 같이 별도의 스택 컨텍스트에 속하지 않는 노드도 있습니다. 스택 컨텍스트가 없는 노드들은 z-index가 0으로 간주됩니다.

![](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F11bnjd2phd9ks9i8g2j6.png)

노드들은 위에서 보여진 그리기 순서에 따라 나눌 수 있습니다.
노드 1은 수직 레이어에서 노드 6과 7보다 우선하며, 노드 5는 수직 레이어에서 노드 2, 3, 4보다 우선합니다.
노드 3과 4는 모두 노드 2 아래에 있기 때문에, 더 높은 z-index를 가진 노드 3이 수직 레이어에서 노드 4보다 우선합니다.
따라서, 그리기 순서는 다음과 같습니다:

![](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmrhd0m7l10ls8tzpk7le.png)

### Z-Index 구현 논리

z-index를 구현하는 것은 스택 컨텍스트를 고려하여 DOM 그리기 순서를 정렬하는 논리를 만드는 것과 같습니다.
스택 컨텍스트는 DOM 트리의 전위 순회를 통해 식별할 수 있습니다.
먼저, 스택 컨텍스트를 정의해 보겠습니다.

```ts
type StackingContext = {
  zIndex: number; //stacking context를 형성한 dom의 z-index
  domOrder: number; //stacking context를 형성한 dom의 전위순회 순서
};

type CollectedDom = {
  contexts: StackingContext[]; //dom이 물려받은 stacking context
  domOrder: number; //dom의 전위순회 순서
  dom: HTMLELEMENT;
};
```

스택 컨텍스트는 z-index를 가진 DOM 요소에 의해 형성됩니다. 이는 생성된 DOM의 zIndex와 domOrder를 스택 컨텍스트에 수집합니다. 자식 요소는 부모로부터 스택 컨텍스트를 상속받습니다. 자식이 새로운 z-index를 가지고 있다면, 부모로부터 상속받은 스택 컨텍스트에 새로운 컨텍스트를 추가합니다. 그런 다음, 노드 7에 해당하는 CollectedDom 정보는 다음과 같을 것입니다:

(여기에 이어서 노드 7의 CollectedDom 정보가 설명될 것입니다.)

```ts
const 7Node = {
  contexts: [{zIndex: -1, domOrder: 6}] //6번 노드로부터 물려받음
  domOrder: 7
  dom: div태그
}
```

DOM을 순회하면서 `const collectedDoms: CollectedDom[]`을 수집했다면, 스택 컨텍스트를 고려하여 올바른 순서로 정렬할 수 있습니다. 이때 유의해야 할 세 가지 사항이 있습니다:

1. 동일한 스택 컨텍스트에 있는 경우, 순서는 DOM 순서에 의해 결정됩니다.
2. 스택 컨텍스트가 없는 경우, z-index는 0으로 간주됩니다.
3. 스택 컨텍스트 내에서는 부모 요소가 자식 요소의 z-index에 상관없이 먼저 그려집니다. (수직 레이어 우선순위에서 자식에게 우선권을 부여합니다.)

### 정렬 로직

```ts
function sort(a: CollectedDom, b: CollectedDom) {
  const limit = Math.min(a.contexts.length, b.contexts.length);

  /*
     stacking context를 순회하며 비교합니다.
   */
  for (let i = 0; i < limit; ++i) {
    const aContext = a.contexts[i];
    const bContext = b.contexts[i];

    /*
     * stacking context가 서로 다른 경우의 노드는 context의
     * zIndex를 먼저 비교하고, 그 다음의 dom 순서를 비교합니다.
     */
    if (aContext.zIndex !== bContext.zIndex) {
      return aContext.zIndex - bContext.zIndex;
    } else if (aContext.domOrder - bContext.domOrder) {
      return aContext.domOrder - bContext.domOrder;
    }
  }

  /*
   * 이 아래부터는 a,b는 동일한 stacking context 있음을 의미한다.
   * a와 b의 관계는 아래 둘 중 하나이다.
   *   1. 부모 - 자식
   *   2. 형제
   */

  // 1. 부모-자식인 경우
  if (limit > 0) {
    const lastContext = a.contexts[limit - 1]; // 값이 같기 때문에 a,b 둘 중 어느것이여도 상관없다.
    //둘 중 하나가 부모임
    if (
      lastContext.domOrder === a.domOrder ||
      lastContext.domOrder === b.domOrder
    ) {
      return a.domOrder - b.domOrder;
    }
  }

  // 2. 형제인 경우
  if (a.contexts.length !== b.contexts.length) {
    const aContext = a.contexts[limit] || { zIndex: 0, domOrder: a.domOrder };
    const bContext = b.contexts[limit] || { zIndex: 0, domOrder: b.domOrder };

    /*
     * stacking context가 서로 다른 경우의 노드는 context의
     * zIndex를 먼저 비교하고, 그 다음의 dom 순서를 비교합니다.
     */
    if (aContext.zIndex !== bContext.zIndex) {
      return aContext.zIndex - bContext.zIndex;
    } else if (aContext.domOrder - bContext.domOrder) {
      return aContext.domOrder - bContext.domOrder;
    }
  }

  // 그 외는 돔 순서로 결정한다.
  return a.domOrder - b.domOrder;
}
```

위의 코드는 z-index에 따라 정렬하는 논리를 표현합니다.

먼저, 스택 컨텍스트가 다른지 확인합니다.

만약 다르다면, zIndex를 우선적으로 비교하고, 그 다음으로 DOM 순서를 비교합니다.

스택 컨텍스트의 길이는 서로 다를 수 있습니다.

가장 짧은 컨텍스트를 기준으로 순차적으로 비교합니다.

비교된 스택 컨텍스트가 모두 동일하다면, 다음 단계로 진행합니다.

비교된 스택 컨텍스트가 동일한 값을 가지는 경우, 두 DOM이 부모-자식 관계인지 확인합니다.

만약 같은 스택 컨텍스트에 속해 있다면, 자식의 z-index 값에 관계없이 자식이 수직 레이어에서 부모보다 우선권을 가집니다.

![](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmqppd4vxd8d3alif1mu4.png)

다이어그램에서, 비록 빨간 원의 z-index 값이 -1로 부모보다 낮지만, 수직 레이어링에서 더 높은 우선순위를 가진다는 것이 분명합니다. 만약 부모의 `z-index: 0`이 제거된다면, 빨간 원은 파란 상자 아래에 위치하게 될 것입니다.

파란 상자의 스택 컨텍스트는 `[{zIndex: 0, domOrder: 2}]`이고, 빨간 원의 스택 컨텍스트는 `[{zIndex: 0, domOrder: 2}, {zIndex: -1, domOrder: 3}]`입니다. 이제 파란 상자의 자식으로 보라색 원을 추가하는 것을 고려해 보겠습니다.

![](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxa1yajrdi7gx7pyn2bji.png)

보라색 원의 스택 컨텍스트는 파란 상자와 마찬가지로 `[{zIndex: 0, domOrder: 2}]`입니다.

그러나 보라색 원의 domOrder가 3이기 때문에 새로운 스택 컨텍스트를 생성하지 않습니다.

위의 다이어그램에서 볼 수 있듯이, 보라색 원은 수직 레이어링에서 빨간 원보다 더 높은 우선순위를 가집니다.

지정되지 않은 z-index는 0으로 간주되므로, `z-index가 -1`인 빨간 원은 결과적으로 우선순위가 더 낮아집니다.

## SVG에서 구현하기

SVG는 z-index 속성을 지원하지 않습니다.

대신, DOM에서 나중에 위치한 요소들이 수직 스택에서 더 높은 우선순위를 가지게 됩니다.

SVG에서 z-index를 적용하려면 DOM 요소들의 순서를 수동으로 조작해야 합니다. 이는 캔버스에서도 유사합니다.

저는 SVG 트리를 DOM 트리와 유사하게 생성하는 **flutterjs** 라이브러리를 사용하여, 방문자 패턴(Visitor pattern)과 스택 컨텍스트를 활용해 z-index를 구현했습니다.

![](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb477j56gdn68xdxmybww.png)

## 결론

이 글에서는 z-index 속성과 그것이 SVG에서 실제로 어떻게 적용되는지에 대한 통찰을 제공했습니다.

이 글을 작성하면서 스택 컨텍스트의 작동 규칙에 대한 오해를 바로잡고, 제가 개발 중인 ERD 다이어그램에서 드래그 동작을 개선할 수 있었습니다.

![](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvwmzf5ndd0mq72ykq1sj.gif)
