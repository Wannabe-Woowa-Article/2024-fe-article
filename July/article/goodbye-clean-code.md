## 🔗 [Goodbye, Clean Code](https://overreacted.io/goodbye-clean-code/)

### 🗓️ 번역 날짜: 2024.07.08

### 🧚 번역한 크루: 소하(최소연)

---

## 잘가, Clean Code

늦은 저녁이었습니다.

동료가 일주일 내내 작성하던 코드를 막 확인했습니다. 우리는 그래픽 편집기 캔버스를 작업하고 있었고, 동료는 사각형과 타원 같은 도형의 크기를 가장자리의 작은 핸들을 드래그하여 조정할 수 있는 기능을 구현했습니다.

코드는 작동했습니다.

하지만 반복적이었습니다. 각 도형(예: 사각형 또는 타원)에는 다른 핸들 세트가 있었고, 각 핸들을 다른 방향으로 드래그하면 도형의 위치와 크기가 다르게 영향을 받았습니다. 사용자가 Shift 키를 누르고 있으면 비율을 유지하면서 크기를 조정해야 했습니다. 많은 수학이 있었습니다.

코드는 다음과 같이 생겼습니다:

```jsx
let Rectangle = {
	resizeTopLeft(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
	resizeTopRight(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
	resizeBottomLeft(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
	resizeBottomRight(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
};

let Oval = {
	resizeLeft(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
	resizeRight(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
	resizeTop(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
	resizeBottom(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
};

let Header = {
	resizeLeft(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
	resizeRight(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
};

let TextBlock = {
	resizeTopLeft(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
	resizeTopRight(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
	resizeBottomLeft(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
	resizeBottomRight(position, size, preserveAspect, dx, dy) {
		// 10 repetitive lines of math
	},
};
```

반복되는 수학이 정말 신경 쓰였습니다.

깨끗하지 않았습니다.

대부분의 반복은 유사한 방향 사이에서 발생했습니다. 예를 들어, `Oval.resizeLeft()`는 `Header.resizeLeft()`와 유사한 점이 있었습니다. 이는 둘 다 왼쪽 가장자리의 핸들을 드래그하는 것을 다루기 때문입니다.

또 다른 유사점은 같은 도형에 대한 메서드 사이에 있었습니다. 예를 들어, `Oval.resizeLeft()`는 다른 `Oval` 메서드와 유사한 점이 있었습니다. 이는 모두 타원을 다루기 때문입니다. 텍스트 블록이 사각형이었기 때문에 `Rectangle`, `Header`, `TextBlock` 사이에도 중복이 있었습니다.

아이디어가 떠올랐습니다.

다음과 같이 코드를 그룹화하면 모든 중복을 제거할 수 있습니다:

```jsx
let Directions = {
  top(...) {
    // 5 unique lines of math
  },
  left(...) {
    // 5 unique lines of math
  },
  bottom(...) {
    // 5 unique lines of math
  },
  right(...) {
    // 5 unique lines of math
  },
};

let Shapes = {
  Oval(...) {
    // 5 unique lines of math
  },
  Rectangle(...) {
    // 5 unique lines of math
  },
}
```

그리고 나서 동작을 구성하는 것입니다.

```jsx
let { top, bottom, left, right } = Directions;

function createHandle(directions) {
	// 20 lines of code
}

let fourCorners = [
	createHandle([top, left]),
	createHandle([top, right]),
	createHandle([bottom, left]),
	createHandle([bottom, right]),
];
let fourSides = [createHandle([top]), createHandle([left]), createHandle([right]), createHandle([bottom])];
let twoSides = [createHandle([left]), createHandle([right])];

function createBox(shape, handles) {
	// 20 lines of code
}

let Rectangle = createBox(Shapes.Rectangle, fourCorners);
let Oval = createBox(Shapes.Oval, fourSides);
let Header = createBox(Shapes.Rectangle, twoSides);
let TextBox = createBox(Shapes.Rectangle, fourCorners);
```

코드의 크기는 절반으로 줄었고, 중복은 완전히 사라졌습니다! 정말 깔끔하네요. 특정 방향이나 도형의 동작을 변경하려면 여기저기서 메서드를 업데이트하는 대신 한 곳에서 처리할 수 있습니다.

이미 밤이 늦었었습니다 (저는 푹 빠져 있었죠). 리팩터링한 내용을 master에 확인하고 잠자리에 들었습니다. 동료의 지저분한 코드를 깔끔하게 정리한 것이 자랑스러워 하면서요.

### 다음날 아침

... 예상대로 되지 않았습니다.

상사가 저를 일대일로 불러 제가 변경한 내용을 되돌리라고 정중하게 요청했습니다. 저는 충격을 받았습니다. 기존 코드는 엉망이었고 제가 수정한 것은 깔끔했는데 말이죠!

마지못해 따랐지만, 상사가 옳았다는 것을 깨닫는 데 수년이 걸렸습니다.

### 이것은 한 단계입니다.

"깨끗한 코드"에 집착하고 중복을 제거하는 것은 우리 중 많은 사람들이 겪는 단계입니다. 코드에 자신감이 없을 때, 측정 가능한 무언가에 자아 가치와 전문적인 자부심을 부여하는 것이 유혹적입니다. 엄격한 lint 규칙, 명명 스키마, 파일 구조, 중복 없음 등이 그 예입니다.

중복 제거는 자동화할 수 없지만, 연습을 통해 더 쉬워집니다. 매번 변경 후 중복이 적거나 많은지 알 수 있습니다. 그 결과 중복 제거는 코드에 대한 객관적인 메트릭을 개선하는 것처럼 느껴집니다. 더 나쁜 것은 사람들의 정체성을 혼란스럽게 만든다는 것입니다: "나는 깨끗한 코드를 작성하는 사람입니다". 이것은 일종의 자기기만만큼 강력합니다.

일단 추상화를 만드는 방법을 배우면, 그 능력에 도취되어 반복되는 코드를 볼 때마다 추상화를 허공에서 뽑아내는 것이 유혹적입니다. 몇 년 동안 코딩을 하다 보면 어디에서나 반복을 보게 되고, 추상화는 우리의 새로운 초능력이 됩니다. 누군가 추상화가 미덕이라고 말하면, 우리는 그것을 받아들일 것입니다. 그리고 "깨끗함"을 숭배하지 않는 다른 사람들을 판단하기 시작할 것입니다.

이제 저는 제 "리팩터링"이 두 가지 측면에서 재앙이었다는 것을 깨달았습니다.

- 첫째, 저는 그것을 작성한 사람과 이야기하지 않았습니다. 코드를 다시 작성하고 그들의 의견을 듣지 않고 체크인했습니다. 개선된 것이었다 하더라도(이제는 더 이상 그렇게 믿지 않지만), 이는 그것을 처리하는 끔찍한 방법입니다. 건강한 엔지니어링 팀은 끊임없이 신뢰를 쌓아갑니다. 팀원과 상의 없이 코드를 다시 작성하는 것은 코드베이스를 효과적으로 함께 협업하는 능력에 큰 타격을 줍니다.

- 둘째, 공짜는 없습니다. 제 코드는 요구 사항을 변경할 수 있는 능력을 중복 감소와 교환했지만, 이는 좋은 거래가 아니었습니다. 예를 들어, 나중에 다양한 도형의 다양한 핸들에 대한 많은 특수 케이스와 동작이 필요했습니다. 제 추상화는 그러한 변경을 용이하게 하기 위해 몇 배 더 복잡해져야 했을 것입니다. 반면에 원래의 "지저분한" 버전에서는 그러한 변경이 케이크처럼 쉽게 유지되었습니다.

제가 "더러운" 코드를 작성해야 한다고 말하고 있는 걸까요? 아닙니다. "깨끗함" 또는 "더러움"이라고 말할 때 의미하는 바를 깊이 생각해 보라고 제안합니다. 반항심, 정의감, 아름다움, 우아함을 느끼시나요? 그러한 품질에 해당하는 구체적인 엔지니어링 결과를 이름 지을 수 있다고 얼마나 확신하시나요? 그것들이 코드 작성 및 수정 방식에 정확히 어떤 영향을 미치나요?

저는 이러한 것들에 대해 깊이 생각하지 않았습니다. 코드가 어떻게 생겼는지에 대해서는 많이 생각했지만, 말랑말랑한 인간 팀과 함께 어떻게 발전하는지에 대해서는 생각하지 않았습니다.

코딩은 여정입니다. 첫 번째 코드 한 줄부터 지금까지 얼마나 멀리 왔는지 생각해 보세요. 함수를 추출하거나 클래스를 리팩터링하여 복잡한 코드를 간단하게 만들 수 있는 방법을 처음 보았을 때 기쁨을 느꼈을 것입니다. 기술에 자부심을 느낀다면, 코드의 깔끔함을 추구하는 것이 유혹적일 것입니다. 한동안 그렇게 해보세요.

하지만 거기서 멈추지 마세요. 깨끗한 코드 광신도가 되지 마세요. 깨끗한 코드는 목표가 아닙니다. 우리가 다루는 시스템의 막대한 복잡성에서 약간의 의미를 찾으려는 시도입니다. 변경이 코드베이스에 어떤 영향을 미칠지 아직 확신하지 못하지만 미지의 바다에서 안내가 필요할 때 사용하는 방어 메커니즘입니다.

깨끗한 코드가 여러분을 인도하게 하세요. **그리고 놓아주세요.**
