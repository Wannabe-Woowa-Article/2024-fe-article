## 🔗 [The Magic of Clip Path](https://emilkowal.ski/ui/the-magic-of-clip-path)

### 🗓️ 번역 날짜: 2024.08.15

### 🧚 번역한 크루: 렛서(김다은)

---

# Clip Path의 마법

`clip-path`는 종종 삼각형과 같은 특정한 모양으로 DOM 노드를 잘라내는 데 사용됩니다. 하지만 이 속성이 애니메이션에도 훌륭하게 유용하다면 믿으시겠어요?

이 아티클에서는 `clip-path`에 대해 깊이 파헤치고, 이 속성으로 할 수 있는 멋진 것들을 탐구해 보겠습니다. 이 글을 읽고 나면, 이 CSS 속성이 어디에서나 사용되는 것을 발견하게 될 거예요.

<br/>

# 기본 개념

`clip-path` 속성은 요소를 특정 모양으로 자르는 데 사용됩니다. 이 속성을 사용해 클리핑 영역을 만들면, 이 영역 외부의 콘텐츠는 숨겨지고, 내부의 콘텐츠만이 보이게 됩니다. 예를 들어, 이 속성을 사용하면 사각형을 쉽게 원으로 변환할 수 있습니다.

```css
.circle {
  clip-path: circle(50% at 50% 50%);
}
```

이것은 레이아웃에 영향을 주지 않으며, clip-path를 적용한 요소는 clip-path를 적용하지 않은 요소와 동일한 공간을 차지합니다. 이는 transform과 마찬가지입니다.

<br/>

# 위치 지정

우리는 좌표 시스템을 사용하여 원을 위치시켰습니다. 좌표 시스템은 좌측 상단 모서리(0, 0)에서 시작합니다. circle(50% at 50% 50%)는 원의 경계 반경이 50%이며, 상단에서 50%, 왼쪽에서 50% 위치, 즉 요소의 중앙에 위치하게 된다는 의미입니다.

기타 값으로는 ellipse, polygon, 또는 url()과 같은 값이 있으며, 이를 통해 사용자 지정 SVG를 클리핑 경로로 사용할 수 있지만, 이 글에서는 모든 애니메이션에 사용되는 inset에 집중할 것입니다.

inset 값은 사각형의 상단, 오른쪽, 하단, 왼쪽 오프셋을 정의합니다. 예를 들어, inset(100%, 100%, 100%, 100%) 또는 간단하게 inset(100%)을 사용하면 전체 요소를 "숨기게" 됩니다. inset(0px 50% 0px 0px)는 요소의 왼쪽 절반을 보이지 않게 만들고, 이런 방식으로 활용할 수 있습니다.

우리는 clip path가 요소의 일부분을 '숨길' 수 있다는 것을 알게 됐으니, 애니메이션의 많은 가능성을 열었습니다. 이제 그것들을 하나씩 알아보도록 하겠습니다.

<br/>

# 비교 슬라이더(Comparison Slider)

여러분도 아마 비교 슬라이더를 본 적 있을겁니다. 예를 들어, 두 개의 div를 overflow hidden으로 정의하고 너비를 조정하는 방법이 있지만, `clip-path`를 사용해 성능이 더 좋은 방법을 쓸 수도 있습니다.

먼저 두 개의 이미지를 겹칩니다. 그리고 그리고 오른쪽 이미지에 `clip-path: (0 50% 0 0)`을 적용해 숨기고, drag 위치에 따라 값을 조정합니다.

이 방법을 사용하면 추가적인 DOM 요소 없이 하드웨어 가속이 된 인터랙션을 얻을 수 있습니다. (너비 조정 방법에서는 오버플로를 숨기기 위해 추가적인 요소가 필요합니다.)

clip-path를 사용해 이런 비교 슬라이더를 만들 수 있다는 것을 알게 되면, 다양한 다른 용도로도 활용할 수 있는 가능성이 열립니다. 예를 들어, 텍스트 마스크 효과를 위해 사용할 수 있습니다.

다시 두 개의 요소를 겹쳐 배치하되, 이번에는 clip-path: inset(0 0 50% 0)을 사용해 점선 텍스트의 하단 절반을 숨기고, clip-path: inset(50% 0 0 0)을 사용해 실선 텍스트의 상단 절반을 숨깁니다. 그런 다음, 마우스 위치에 따라 이 값을 조정합니다.

#### 점선 텍스트

```css
clip-path: inset(0 0 50% 0);
```

#### 실선 택스트

```css
clip-path: inset(50% 0 0 0);
```

이 예시는 기술적으로는 세로 비교 슬라이더와 같지만 그렇게 보이지 않습니다. 이건 단지 창의력의 문제일 뿐입니다.

여기서 점선 텍스트는 Figma에서 스트로크를 적용해 SVG로 변환한 것입니다.

<br/>

# 이미지 애니메이션

clip-path를 사용하여 이미지 나타내기 애니메이션을 연출할 수도 있습니다. 처음에는 이미지를 완전히 가리는 clip-path를 설정해 이미지를 보이지 않게 한 후, 애니메이션을 통해 이미지가 드러나도록 할 수 있습니다.

```css
.image-reveal {
  clip-path: inset(0 0 100% 0);
  animation: reveal 1s forwards cubic-bezier(0.77, 0, 0.175, 1);
}

@keyframes reveal {
  to {
    clip-path: inset(0 0 0 0);
  }
}
```

마찬가지로 `height` 속성을 사용해 구현할 수도 있지만, `clip-path`를 사용해 얻는 몇 가지 이점이 있습니다. clip-path는 하드웨어 가속이 되기 때문에 이미지의 높이를 애니메이션하는 것보다 성능이 더 좋습니다. 또한, 이미지를 드러낼 때 layout이 이동하는 현상을 방지할 수 있는데, 이미지는 이미 존재하지만 clip-path로 인해 가려져 있을 뿐이기 때문입니다.

<br/>

# 스크롤 애니메이션

이미지 드러내기 효과는 이미지가 뷰포트에 들어왔을 때 트리거되어야 합니다. 그렇지 않으면 사용자는 이미지가 애니메이션되는 것을 절대 볼 수 없게 됩니다. 그렇다면 어떻게 해야 할까요?

저는 보통 애니메이션을 위해 Framer Motion을 사용하므로, 이를 이용하는 방법을 보여드리겠습니다. 하지만 만약 프로젝트에서 이 라이브러리를 사용하지 않고 있다면, Intersection Observer API를 사용하는 것을 권장합니다. Framer Motion은 꽤 무거운 라이브러리이기 때문입니다.

Framer Motion은 useInView라는 훅을 제공하는데, 이 훅은 요소가 뷰포트 안에 있는지 여부를 나타내는 불리언 값을 반환합니다. 우리는 이 값을 사용해 애니메이션을 트리거할 수 있습니다.

```jsx
'use client';

import { useInView } from 'framer-motion';
import { useRef } from 'react';

export default function ImageRevealInner() {
  const ref = useRef(null);
  // 이미지가 뷰포트 안에 100px 이상 들어왔을 때만 한 번 true로 변경
  const isInView = useInView(ref, { once: true, margin: '-100px' });

  if (isInView && ref.current) {
    ref.current.animate(
      [{ clipPath: 'inset(0 0 100% 0)' }, { clipPath: 'inset(0 0 0 0)' }],
      {
        duration: 1000,
        fill: 'forwards',
        easing: 'cubic-bezier(0.77, 0, 0.175, 1)',
      }
    );
  }

  return (
    <>
      <h1>스크롤을 내려보세요</h1>
      <img
        className="image-reveal"
        alt="대각선으로 교차하는 검은색과 흰색 줄무늬가 부드러운 그라디언트 효과로 표현된 이미지입니다. 교차하는 밝고 어두운 밴드는 깊이와 움직임의 느낌을 주며, 빛의 선이나 표면에 비친 그림자를 연상케 합니다. 전반적인 미학은 추상적이며 높은 대비를 이루고 있으며, 세련되고 현대적인 느낌을 줍니다."
        src="https://emilkowalski-git-the-magic-of-clip-path-emilkowalski-s-team.vercel.app/clip-path/raycast.jpg"
        height={430}
        ref={ref}
        width={644}
      />
    </>
  );
}
```

# 스크롤 진행

다른 스크롤 인터랙션으로는, 스크롤할 때 길어지는 수직선을 clip-path로 만들 수 있습니다. 겉보기에는 SVG가 그려지는 것처럼 보일 수 있지만, 사실 이 효과는 점점 드러나는 잘라진(div) 요소일 뿐입니다.

이 효과는 처음 Rauno의 트윗에서 보았습니다. 그 트윗에서 영감을 받아 이 효과를 재현해 보고 이 글에 포함시켰습니다.

이번 예제에서는 Framer Motion의 useScroll 훅을 사용해 컨테이너의 스크롤 진행 상황을 가져왔습니다. offset 옵션을 사용하면 요소의 상단이 뷰포트 하단에 도달할 때 측정을 시작하고, 요소의 하단이 뷰포트 하단에 도달할 때 측정을 종료합니다. 이렇게 하면 사용자가 요소를 지나쳐 스크롤해도 애니메이션이 되돌아가지 않습니다.

useTransform 덕분에, 스크롤 진행 상황(0에서 1까지)을 `clip-path` 속성에서 사용할 수 있는 값으로 매핑할 수 있습니다. 기본적으로 0을 100%로, 1을 0%로 매핑하며, 그 사이의 값들은 자동으로 계산됩니다.

```jsx
const { scrollYProgress } = useScroll({
  target: containerRef,
  offset: ['start end', 'end end'],
});

const clipPathY = useTransform(scrollYProgress, [0, 1], ['100%', '0%']);
```

`scrollYProgress`는 Framer Motion의 내부 값인 모션 값(motion value)으로, 컴포넌트를 다시 렌더링하지 않고 최신 값을 유지합니다. 이 방식은 매우 유용하지만, 접근 방법에 제약이 따르기도 합니다.

모션 값을 사용해 `clip-path` 속성에서 사용할 새로운 모션 값을 만들어야 했습니다. 이를 위해 `useMotionTemplate` 함수를 사용했으며, 이 함수는 다른 모션 값이 포함된 문자열 템플릿에서 새로운 모션 값을 생성할 수 있게 해줍니다.

```jsx
const motionClipPath = useMotionTemplate`inset(0 0 ${clipPathY} 0)`;
```

모션 값의 장점은 인라인 스타일링으로 사용할 때 자동으로 업데이트된다는 점입니다. 그래서 `scrollYProgress`를 모션 값으로 유지해야 했습니다. 만약 이를 const 변수에 저장했다면, 업데이트를 받지 못했을 겁니다.

```jsx
<motion.div
  ref={containerRef}
  // 이 스타일 값은 자동으로 업데이트됩니다.
  // 왜냐하면 `motionClipPath`가 모션 값이기 때문이죠.
  style={{ clipPath: motionClipPath }}
>
  ...
</motion.div>
```

이 예제는 이번 글에서 가장 고급스러운 예제로, Framer Motion의 복잡한 기능들을 다루고 있습니다. 이 라이브러리를 사용해 무엇이 가능한지 보여주기 위해 포함했지만, 만약 이해하기 어려우시다면 걱정하지 마세요! 앞으로 Framer Motion에 대해 더 깊이 있는 글을 작성할지도 모릅니다.

<br/>

# 탭 전환

이것도 분명 본 적이 있을 겁니다. 여기서의 문제는 활성 탭의 텍스트 색상이 비활성 탭의 색상과 다르다는 점입니다. 보통 사람들은 텍스트 색상에 transition을 적용해서 이 문제를 어느정도 해결하곤 합니다.

이 방법도 괜찮긴 하지만, 더 나은 방법이 있습니다.

리스트를 복제한 후, 복제된 리스트의 스타일을 활성 상태로 보이도록 변경할 수 있습니다. (파란색 배경, 흰색 텍스트) 그런 다음, `clip-path`를 사용해 복제된 리스트를 잘라내어 그 리스트에서 활성 탭만 보이게 합니다. 클릭 시 `clip-path`값을 애니메이션으로 변경하여 새로운 활성 탭을 드러낼 수 있습니다.

이렇게 하면 탭 간의 전환이 매끄럽게 이루어지고, 색상 전환의 타이밍을 맞추는 것에 대해 걱정할 필요가 없습니다. 어차피 색상 전환은 완전히 매끄럽게 처리하기 어렵기 때문이죠.

이해를 돕기 위해 "Toggle clip path" 버튼을 눌러 클리핑 없이 어떻게 보이는지 확인할 수 있습니다. 옆에 있는 버튼을 클릭하여 속도를 줄이면 전환의 차이를 더 잘 알아차릴 수 있습니다.

이 기술을 처음 본 것은 [Paco의 트윗](https://x.com/pacocoursey/status/1522639642155266048) 중 하나였는데, 그의 프로필에는 이와 같은 보석 같은 기술들이 많이 있습니다.

물론, 모든 사람이 이 차이를 알아차리는 것은 아니라고 할 수 있지만, 저는 이러한 작은 디테일들이 쌓여 전체 경험을 더 세련되게 만든다고 믿습니다. 비록 그것들이 눈에 띄지 않더라도 말이죠.

아래에서는 이 기술을 제가 구현한 예를 볼 수 있습니다. 이 코드가 클립 패스 부분에 집중하기 위해 단순화되었음을 기억해 주세요. 실제 구현에서는 더 많은 작업이 필요할 것입니다. 예를 들어, 접근성을 높이기 위해 [Radix's Tabs](https://www.radix-ui.com/primitives/docs/components/tabs)를 사용할 것입니다.

```jsx
'use client';

import { useEffect, useRef, useState } from 'react';

export default function TabsClipPath() {
  const [activeTab, setActiveTab] = useState(TABS[0].name);
  const containerRef = useRef(null);
  const activeTabElementRef = useRef(null);

  useEffect(() => {
    const container = containerRef.current;

    if (activeTab && container) {
      const activeTabElement = activeTabElementRef.current;

      if (activeTabElement) {
        const { offsetLeft, offsetWidth } = activeTabElement;

        const clipLeft = offsetLeft;
        const clipRight = offsetLeft + offsetWidth;
        container.style.clipPath = `inset(0 ${Number(
          100 - (clipRight / container.offsetWidth) * 100
        ).toFixed()}% 0 ${Number(
          (clipLeft / container.offsetWidth) * 100
        ).toFixed()}% round 17px)`;
      }
    }
  }, [activeTab, activeTabElementRef, containerRef]);

  return (
    <div className="wrapper">
      <ul className="list">
        {TABS.map((tab) => (
          <li key={tab.name}>
            <button
              ref={activeTab === tab.name ? activeTabElementRef : null}
              data-tab={tab.name}
              onClick={() => {
                setActiveTab(tab.name);
              }}
              className="button"
            >
              {tab.icon}
              {tab.name}
            </button>
          </li>
        ))}
      </ul>

      <div aria-hidden className="clip-path-container" ref={containerRef}>
        <ul className="list list-overlay">
          {TABS.map((tab) => (
            <li key={tab.name}>
              <button
                data-tab={tab.name}
                onClick={() => {
                  setActiveTab(tab.name);
                }}
                className="button-overlay button"
                tabIndex={-1}
              >
                {tab.icon}
                {tab.name}
              </button>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}

const TABS = [
  {
    name: 'Payments',
    icon: (
      <svg
        aria-hidden="true"
        width="16"
        height="16"
        viewBox="0 0 16 16"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path
          fillRule="evenodd"
          clipRule="evenodd"
          fill="currentColor"
          d="M0 3.884c0-.8.545-1.476 1.306-1.68l.018-.004L10.552.213c.15-.038.3-.055.448-.055.927.006 1.75.733 1.75 1.74V4.5h.75A2.5 2.5 0 0 1 16 7v6.5a2.5 2.5 0 0 1-2.5 2.5h-11A2.5 2.5 0 0 1 0 13.5V3.884ZM10.913 1.67c.199-.052.337.09.337.23v2.6H2.5c-.356 0-.694.074-1 .208v-.824c0-.092.059-.189.181-.227l9.216-1.984.016-.004ZM1.5 7v6.5a1 1 0 0 0 1 1h11a1 1 0 0 0 1-1V7a1 1 0 0 0-1-1h-11a1 1 0 0 0-1 1Z"
        ></path>
        <path
          fillRule="evenodd"
          clipRule="evenodd"
          fill="currentColor"
          d="M10.897 1.673 1.681 3.657c-.122.038-.181.135-.181.227v.824a2.492 2.492 0 0 1 1-.208h8.75V1.898c0-.14-.138-.281-.337-.23m0 0-.016.005Zm-9.59.532 9.23-1.987c.15-.038.3-.055.448-.055.927.006 1.75.733 1.75 1.74V4.5h.75A2.5 2.5 0 0 1 16 7v6.5a2.5 2.5 0 0 1-2.5 2.5h-11A2.5 2.5 0 0 1 0 13.5V3.884c0-.8.545-1.476 1.306-1.68l.018-.004ZM1.5 13.5V7a1 1 0 0 1 1-1h11a1 1 0 0 1 1 1v6.5a1 1 0 0 1-1 1h-11a1 1 0 0 1-1-1ZM13 10.25c0 .688-.563 1.25-1.25 1.25-.688 0-1.25-.55-1.25-1.25 0-.688.563-1.25 1.25-1.25.688 0 1.25.562 1.25 1.25Z"
        ></path>
      </svg>
    ),
  },
  {
    name: 'Balances',
    icon: (
      <svg
        data-testid="primary-nav-item-icon"
        aria-hidden="true"
        width="16"
        height="16"
        viewBox="0 0 16 16"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path
          fill="currentColor"
          d="M1 2a.75.75 0 0 1 .75-.75h7.5a.75.75 0 0 1 0 1.5h-7.5A.75.75 0 0 1 1 2Zm0 8a.75.75 0 0 1 .75-.75h5a.75.75 0 0 1 0 1.5h-5A.75.75 0 0 1 1 10Zm2.25-4.75a.75.75 0 0 0 0 1.5h7.5a.75.75 0 0 0 0-1.5h-7.5ZM2.5 14a.75.75 0 0 1 .75-.75h4a.75.75 0 0 1 0 1.5h-4A.75.75 0 0 1 2.5 14Z"
        ></path>
        <path
          fillRule="evenodd"
          clipRule="evenodd"
          fill="currentColor"
          d="M16 11.5a3.5 3.5 0 1 1-7 0 3.5 3.5 0 0 1 7 0Zm-1.5 0a2 2 0 1 1-4 0 2 2 0 0 1 4 0Z"
        ></path>
      </svg>
    ),
  },
  {
    name: 'Customers',
    icon: (
      <svg
        aria-hidden="true"
        width="16"
        height="16"
        viewBox="0 0 16 16"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path
          fillRule="evenodd"
          clipRule="evenodd"
          fill="currentColor"
          d="M2.5 14.4h11a.4.4 0 0 0 .4-.4 3.4 3.4 0 0 0-3.4-3.4h-5A3.4 3.4 0 0 0 2.1 14c0 .22.18.4.4.4Zm0 1.6h11a2 2 0 0 0 2-2 5 5 0 0 0-5-5h-5a5 5 0 0 0-5 5 2 2 0 0 0 2 2ZM8 6.4a2.4 2.4 0 1 0 0-4.8 2.4 2.4 0 0 0 0 4.8ZM8 8a4 4 0 1 0 0-8 4 4 0 0 0 0 8Z"
        ></path>
      </svg>
    ),
  },
  {
    name: 'Billing',
    icon: (
      <svg
        aria-hidden="true"
        width="16"
        height="16"
        viewBox="0 0 16 16"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path
          fill="currentColor"
          d="M0 2.25A2.25 2.25 0 0 1 2.25 0h7.5A2.25 2.25 0 0 1 12 2.25v6a.75.75 0 0 1-1.5 0v-6a.75.75 0 0 0-.75-.75h-7.5a.75.75 0 0 0-.75.75v10.851a.192.192 0 0 0 .277.172l.888-.444a.75.75 0 1 1 .67 1.342l-.887.443A1.69 1.69 0 0 1 0 13.101V2.25Z"
        ></path>
        <path
          fill="currentColor"
          d="M5 10.7a.7.7 0 0 1 .7-.7h4.6a.7.7 0 1 1 0 1.4H7.36l.136.237c.098.17.193.336.284.491.283.483.554.907.855 1.263.572.675 1.249 1.109 2.365 1.109 1.18 0 2.038-.423 2.604-1.039.576-.626.896-1.5.896-2.461 0-.99-.42-1.567-.807-1.998a.75.75 0 1 1 1.115-1.004C15.319 8.568 16 9.49 16 11c0 1.288-.43 2.54-1.292 3.476C13.838 15.423 12.57 16 11 16c-1.634 0-2.706-.691-3.51-1.64-.386-.457-.71-.971-1.004-1.472L6.4 12.74v2.56a.7.7 0 1 1-1.4 0v-4.6ZM2.95 4.25a.75.75 0 0 1 .75-.75h2a.75.75 0 0 1 0 1.5h-2a.75.75 0 0 1-.75-.75ZM3.7 6.5a.75.75 0 0 0 0 1.5h4.6a.75.75 0 0 0 0-1.5H3.7Z"
        ></path>
      </svg>
    ),
  },
];
```

<br/>

# 한 단께 더 나아가서

2021년에 X에 테마 애니메이션을 공유한 적이 있습니다. 그 구현은 다소 억지스러웠는데, 페이지 전체를 복제했기 때문입니다. 하지만 빠른 프로토타입을 위해서는 잘 작동했습니다.

이 방식은 기본적으로 클립 패스의 애니메이션을 사용하여 라이트 테마나 다크 테마 중 하나를 애니메이션으로 드러내는 것입니다. 어느 테마를 애니메이션할지 알 수 있는 이유는 현재 테마를 상태에 저장하고, 사용자가 버튼을 클릭하면 이를 변경하기 때문입니다.

코드는 이미지 공개 효과와 매우 유사합니다. 클립 패스의 기본 원리를 이해하면, 이와 같은 훌륭한 애니메이션을 많이 만들 수 있습니다. 결국, 창의력의 문제일 뿐입니다.

```css
.clipPathReveal {
  clip-path: inset(0 0 100% 0);
  animation: revealClipPath 1s cubic-bezier(0.77, 0, 0.175, 1) forwards;
}

@keyframes revealClipPath {
  from {
    clip-path: inset(0 0 100% 0);
  }
  to {
    clip-path: inset(0 0 0 0);
  }
}
```

이 구현 방식은 애니메이션을 적용하려는 요소를 복제해야 하므로 다소 억지스러운 면이 있지만, [View Transitions API](https://theme-toggle.rdsx.dev/)를 사용하면 동일한 효과를 얻을 수 있습니다. 이 기사에서는 내용을 간결하게 유지하기 위해 그 부분은 다루지 않겠습니다.

<br/>

# Clip Ptah는 어디에나 있습니다

이제 클립 패스를 사용한 애니메이션 방법을 알게 되었으니, 많은 곳에서 그것이 사용되는 것을 발견할 수 있을 것입니다. 예를 들어, Vercel은 그들의 보안 페이지에서 클립 패스를 사용하고 있습니다.

Tuple은 우리가 앞서 논의한 너비 방식을 사용하고 있지만, 더 성능이 좋은 솔루션을 위해 이곳에서 클립 패스를 사용할 수도 있습니다.

그리고 앞서 논의했던 바로 그 탭 컴포넌트가 Stripe의 블로그에서 사용되고 있습니다.

<br/>

# 빙산의 일각

이 글은 여러분에게 영감을 주고, 틀을 벗어난 사고가 훌륭한 애니메이션으로 이어질 수 있음을 보여주기 위해 작성되었습니다. 클립 패스는 매우 강력한 속성이지만, 그것만이 전부는 아닙니다! 웹에서 요소를 애니메이션화할 수 있는 창의적인 방법이 많이 있습니다. 저는 "Animations on the Web"이라는 제 강의에서 그 방법들을 다루고 있습니다.
