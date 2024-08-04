## 🔗 [The Internals of Styled Components](https://jser.dev/2023-10-09-styled-components/)

### 🗓️ 번역 날짜: 2024.07.28

### 🧚 번역한 크루: 버건디(전태헌)

---

## 스타일드 컴포넌트의 내부

CSS-in-JS로 분류되는 솔루션은 상당히 많습니다. 그 중 하나가 Styled Components입니다. 이번 에피소드에서는 Styled Components의 작동 원리를 자세히 살펴보고, 다양한 스타일 스택들 중에서 더 확실하게 선택할 수 있는 기준이 생기길 바랍니다.

## Styled Components의 동기

먼저, [Styled Components의 주장](https://styled-components.com/docs/basics#motivation)에 대해 살펴보겠습니다.

> styled-components는 React 컴포넌트 시스템의 스타일링을 위해 CSS를 어떻게 개선할 수 있을지 고민한 결과입니다.

[CSS 자체에는 몇 가지 결함이 있으며, 특히 규모를 확장할 때 이러한 문제가 더욱 두드러집니다.](https://speakerdeck.com/vjeux/react-css-in-js) 이러한 문제는 컴포넌트 기반 자바스크립트 애플리케이션 시대에 더욱 명확해집니다. 많은 해결책이 존재하지만, 여기서는 Styled Components가 자신을 어떻게 소개하는지 살펴보겠습니다.

> 자동 중요한 CSS 처리: styled-components는 페이지에서 렌더링된 컴포넌트를 추적하고 그들의 스타일만 주입하며, 다른 것은 전혀 주입하지 않습니다. 이는 코드 스플리팅과 결합되어 사용자들이 필요한 최소한의 코드만 로드하게 합니다.

🤔 이는 스타일을 동적으로 주입하면 쉽게 구현할 수 있지만, 비용이 발생합니다. [React는 스타일 주입이 좋지 않다고 언급](https://react.dev/reference/react/useInsertionEffect)합니다. 또한 규모 확장을 위해 Atomic CSS와 같은 솔루션이 매우 잘 작동하므로 미리 많은 CSS를 로드하는 것이 큰 문제가 되지 않습니다.

> 클래스 이름 버그 없음: styled-components는 고유한 클래스 이름을 생성합니다. 중복, 겹침, 오타에 대해 걱정할 필요가 없습니다.

🤔 CSS 클래스 이름을 선택하는 데 시간을 낭비하는 것은 확실히 시간 낭비입니다. Styled Components는 CSS 모듈과 같은 솔루션보다 더 직관적입니다. 그러나 경험상, 여전히 Styled Component의 컴포넌트 이름을 선택하는 데 시간을 낭비해야 합니다.

> CSS 삭제 용이: 코드베이스의 어느 곳에서 클래스 이름이 사용되고 있는지 알기 어렵습니다. styled-components는 모든 스타일링이 특정 컴포넌트에 연결되어 있기 때문에 이를 명확하게 합니다. 사용되지 않는 컴포넌트는 도구가 감지하여 삭제하면 그 스타일도 함께 삭제됩니다.

🙂 이 부분은 매우 좋습니다. 스타일링에서의 공동 위치는 훌륭합니다.

> 간단한 동적 스타일링: 컴포넌트의 스타일을 props나 글로벌 테마를 기반으로 조정하는 것은 간단하고 직관적이며 수많은 클래스를 수동으로 관리할 필요가 없습니다.

🙂 매우 좋습니다. 직관적으로 느껴집니다.

> 쉬운 유지 보수: 컴포넌트에 영향을 미치는 스타일을 찾기 위해 다른 파일을 뒤질 필요가 없기 때문에 코드베이스가 얼마나 크든 유지 보수가 매우 쉽습니다.

🙂 역시 공동 위치에 관한 것입니다.

> 자동 벤더 접두사 처리 : 현재 표준에 맞게 CSS를 작성하고 나머지는 styled-components가 처리합니다.

🤔 이 부분은 postCSS를 사용하면 간단히 해결할 수 있습니다.

따라서 공동 위치와 동적 스타일링이 [CSS 모듈](https://github.com/css-modules/css-modules)보다 Styled Components를 선택하는 주요 이유가 됩니다. 이제 styled-components의 내부를 살펴볼 차례입니다.

## 스타일드 컴포넌트는 내부적으로 어떻게 동작하는가?

아래 2개의 버튼을 렌더링 시켜보겠습니다.

위의 데모를 살펴보면 아래와 같이 몇 가지 CSS 클래스 이름을 생성하는 것을 볼 수 있습니다.

```html
<div>
  <button class="sc-cyRfQX OQOsV">Normal</button>
  <button class="sc-cyRfQX hhpfzZ">Primary</button>
</div>
```

첫 번째 클래스는 컴포넌트의 ID인 "`sc-cyRFQX`"로, 두 버튼 모두 이 클래스를 가지고 있습니다. 두 번째 클래스는 서로 다른 CSS를 정의하여 다르게 나타납니다. 다음은 작동 방식에 대한 추측입니다.

`styled.button`은 CSS 코드를 `<head>`에 주입하고, 해당 클래스 이름을 가진 버튼 요소를 반환합니다.

### 비슷한 예시를 다시 시도해봅시다.

사실 이러한 방식으로 기본 컴포넌트를 구현하는 것은 꽤 간단합니다. 아래는 작동하는 데모입니다.

이를 설명해보겠습니다.

```tsx
import { useInsertionEffect } from "react";

function getClassName(css) {
  return "s" + Math.floor(Math.random() * 10 ** 8).toString(16);
  // CSS 코드를 해시하는 것이 더 좋지만, 여기서는 데모 목적으로 임의의 이름을 반환합니다.
}

function injectCSS(css) {
  const style = document.createElement("style");
  style.type = "text/css";
  style.textContent = css;
  document.head.append(style);
  // 여기서는 캐시를 사용하지 않습니다.
}

function styledButton(strings, ...expressions) {
  // 태그된 템플릿의 기초입니다.

  return (props) => {
    let css = "";
    let i = 0;
    while (i < strings.length) {
      css += strings[i];
      if (i < expressions.length) {
        css += expressions[i](props);
      }
      i += 1;
    }
    const cssClassName = getClassName(css);
    useInsertionEffect(() => {
      injectCSS(`.${cssClassName} {
  ${css}
  }
  `);
    }, [css]);
    const { className, ...rest } = props;
    const mergedClassNames = [cssClassName, className]
      .filter(Boolean)
      .join(",");
    // 간단한 핵심 로직!

    return <button className={mergedClassNames} {...rest} />;
  };
}

const Button = styledButton`
  background: ${(props) => (props.$primary ? "#BF4F74" : "white")};
  color: ${(props) => (props.$primary ? "white" : "#BF4F74")};
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid #bf4f74;
  border-radius: 3px;
`;

export default function App() {
  return (
    <div>
      <Button>Normal</Button>
      <Button $primary>Primary</Button>
    </div>
  );
}
```

굉장히 간단해 보이지만, 분명히 우리가 여기서는 autoprefixer, SSR 및 많은 다른 것들을 다루지 않았습니다. 그러나 이 코드를 이해하고 나면, 이제 styled components의 소스 코드를 살펴볼 수 있습니다.

### 2.2 styled.tagName은 래핑된 템플릿 함수입니다.

먼저 스타일 컴포넌트를 작성하는 방법인 styled.{tagName}부터 시작해 보겠습니다.

```tsx
const baseStyled = <Target extends WebTarget>(tag: Target) =>
  constructWithOptions<"web", Target>(createStyledComponent, tag);

const styled = baseStyled as typeof baseStyled & {
  [E in SupportedHTMLElements]: Styled<"web", E, JSX.IntrinsicElements[E]>;
};

// 모든 유효한 HTML 요소에 대한 약어
domElements.forEach((domElement) => {
  styled[domElement] = baseStyled<typeof domElement>(domElement);
});

// 그래서 각 태그는 이렇게 래핑됩니다. 왜 Proxy를 사용하지 않을까요?

export default styled;
```

```tsx
export default function constructWithOptions<
  R extends Runtime,
  Target extends StyledTarget<R>,
  OuterProps extends object = Target extends KnownTarget
    ? React.ComponentPropsWithRef<Target>
    : BaseObject,
  OuterStatics extends object = BaseObject,
>(
  componentConstructor: IStyledComponentFactory<
    R,
    StyledTarget<R>,
    object,
    any
  >,
  tag: StyledTarget<R>,
  options: StyledOptions<R, OuterProps> = EMPTY_OBJECT
): Styled<R, Target, OuterProps, OuterStatics> {
  /* 이것은 템플릿 함수로 직접 호출될 수 있습니다 */
  const templateFunction = <
    Props extends object = BaseObject,
    Statics extends object = BaseObject,
  >(
    initialStyles: Styles<Substitute<OuterProps, Props>>,
    ...interpolations: Interpolation<Substitute<OuterProps, Props>>[]
  ) =>
    componentConstructor<Substitute<OuterProps, Props>, Statics>(
      // 우리의 경우 createStyledComponent를 호출합니다
      tag,
      options as StyledOptions<R, Substitute<OuterProps, Props>>,
      css<Substitute<OuterProps, Props>>(initialStyles, ...interpolations)
      // 이것은 중요합니다. css()는 정적 및 동적 규칙을 처리합니다.
    );
  // 우리가 만든 템플릿 함수와 유사합니다

  templateFunction.withConfig = (config: StyledOptions<R, OuterProps>) =>
    // 이 추가 수정자는 SSR 지원에 중요합니다. 나중에 설명합니다.
    constructWithOptions<R, Target, OuterProps, OuterStatics>(
      componentConstructor,
      tag,
      {
        ...options,
        ...config,
      }
    );
  return templateFunction;
}
```

### 2.3 `createStyledComponent()`는 실제적인 컴포넌트를 만들어냅니다.

```tsx
function createStyledComponent<
  Target extends WebTarget,
  OuterProps extends object,
  Statics extends object = BaseObject,
>(
  target: Target,
  options: StyledOptions<'web', OuterProps>,
  rules: RuleSet<OuterProps> // 이는 css()에서 온 것이며, 동적 규칙으로 함수가 있을 수 있습니다.
): ReturnType<IStyledComponentFactory<'web', Target, OuterProps, Statics>> {
  ...
  const {
    attrs = EMPTY_ARRAY,
    componentId = generateId(options.displayName, options.parentComponentId),
    // 여기서 HTML에서 보는 것과 같은 안정적인 컴포넌트 ID("sc-cyRfQX")를 생성합니다.
    // 기본적으로 옵션은 비어 있으며, displayName에도 의존합니다.
    displayName = generateDisplayName(target),
  } = options;

  const styledComponentId =
    options.displayName && options.componentId
      ? `${escape(options.displayName)}-${options.componentId}`
      : options.componentId || componentId;

  ...
  const componentStyle = new ComponentStyle(
    rules,
    styledComponentId,
    // 컴포넌트 ID는 중요합니다.
    isTargetStyledComp ? (styledComponentTarget.componentStyle as ComponentStyle) : undefined
  );
  // ComponentStyle은 CSS와 관련된 모든 것을 처리합니다.

  function forwardRefRender(props: ExecutionProps & OuterProps, ref: Ref<Element>) {
    return useStyledComponentImpl<OuterProps>(WrappedStyledComponent, props, ref);
  }

  /**
   * forwardRef는 새로운 임시 컴포넌트를 생성합니다. 이를 활용하여
   * _또 다른_ 임시 클래스를 생성하는 대신 ParentComponent를 확장합니다.
   */
  let WrappedStyledComponent = React.forwardRef(forwardRefRender) as unknown as IStyledComponent<
    // 아, 이 부분을 구현에서 잊었습니다.
    'web',
    any
  > &
    Statics;

  WrappedStyledComponent.attrs = finalAttrs;
  WrappedStyledComponent.componentStyle = componentStyle;
  WrappedStyledComponent.shouldForwardProp = shouldForwardProp;
  // 이 정적 속성은 컴포넌트 선택자 목적으로 정적 클래스의 계층 구조를 보존하는 데 사용됩니다.
  // css prop 사용 시 특히 중요합니다.
  WrappedStyledComponent.foldedComponentIds = isTargetStyledComp
    ? joinStrings(styledComponentTarget.foldedComponentIds, styledComponentTarget.styledComponentId)
    : '';
  WrappedStyledComponent.styledComponentId = styledComponentId;
  // 스타일을 접었으므로 기본 StyledComponent 타겟을 접습니다.
  WrappedStyledComponent.target = isTargetStyledComp ? styledComponentTarget.target : target;

  ...
  return WrappedStyledComponent;
}
```

코드가 조금 혼란스러울 수 있습니다. `WrappedStyledComponent`는 `forwardRefRender()`의 forwardRef()이며, `forwardRefRender()` 내부에서 WrappedStyledComponent()를 사용합니다.

`useStyledComponentImpl()`의 내부를 살펴보면, 여기서 `WrappedStyledComponent()`는 단지 속성의 보관용으로 사용될 뿐이며, 렌더링에 재귀적으로 사용되지 않습니다. 실제로 렌더링을 수행하는 것은 `useStyledComponentImpl()`입니다.

```tsx
function useStyledComponentImpl<Props extends object>(
  forwardedComponent: IStyledComponent<'web', Props>,
  props: ExecutionProps & Props,
  forwardedRef: Ref<Element>
) {
  const {
    attrs: componentAttrs,
    componentStyle,
    defaultProps,
    foldedComponentIds,
    styledComponentId,
    target,
  } = forwardedComponent;
  const contextTheme = React.useContext(ThemeContext);
  const ssc = useStyleSheetContext();
  const shouldForwardProp = forwardedComponent.shouldForwardProp || ssc.shouldForwardProp;
  // 주의: 비-hooks 버전은 !componentStyle.isStatic일 때만 이것을 구독하지만,
  // 이는 hooks 규칙에 위배됩니다. 우리가 규칙을 무시하고 구독할 수도 있겠지만,
  // 현재로서는 규칙을 준수합니다.
  const theme = determineTheme(props, contextTheme, defaultProps) || EMPTY_OBJECT;
  const context = resolveContext<Props>(componentAttrs, props, theme);
  const elementToBeCreated: WebTarget = context.as || target;
  const propsForElement: Dict<any> = {};
  for (const key in context) {
    if (context[key] === undefined) {
      // 래핑된 요소에 전달되는 props에서 undefined 값을 생략합니다.
      // 예를 들어, .attrs()를 사용하여 props를 제거할 수 있습니다.
    } else if (key[0] === '$' || key === 'as' || key === 'theme') {
      // 임시 props와 실행 props를 생략합니다.
    } else if (key === 'forwardedAs') {
      propsForElement.as = context.forwardedAs;
    } else if (!shouldForwardProp || shouldForwardProp(key, elementToBeCreated)) {
      propsForElement[key] = context[key];
      ...
    }
  }
  const generatedClassName = useInjectedStyle(componentStyle, context);
  // 주입을 찾았습니다. 또한 context가 인수로 포함되어 있는 것을 주목하세요.

  let classString = joinStrings(foldedComponentIds, styledComponentId);
  if (generatedClassName) {
    classString += ' ' + generatedClassName;
  }
  if (context.className) {
    classString += ' ' + context.className;
  }

  propsForElement[
    // React가 제대로 별칭을 지정하지 않는 사용자 정의 요소를 처리합니다.
    isTag(elementToBeCreated) &&
    !domElements.has(elementToBeCreated as Extract<typeof domElements, string>)
      ? 'class'
      : 'className'
  ] = classString;
  propsForElement.ref = forwardedRef;
  return createElement(elementToBeCreated, propsForElement);
}

```

코드는 어렵지 않습니다. CSS를 주입하고, 클래스 이름을 가져오며 나머지 props를 본질적인 HTML 요소에 전달합니다.

### 2.4 컴포넌트 ID 생성

위 섹션에서 보았듯이 컴포넌트 ID는 매우 중요합니다. 이는 각 스타일드 컴포넌트를 표시하는 고유한 식별자입니다. 기본적으로 style.{tagName}는 익명 함수이기 때문입니다.

```ts
componentId = generateId(options.displayName, options.parentComponentId);
```

기본적으로 `options`는 비어 있으며, `generateId()`는 중복이 없도록 해야 합니다.

```ts
const identifiers: { [key: string]: number } = {};
// ID 중복을 처리하기 위한 간단한 전역 카운터입니다.

/* 우리는 컴포넌트가 고유한 ID를 가지는 것에 의존합니다 */
function generateId(
  displayName?: string | undefined,
  parentComponentId?: string | undefined
): string {
  const name = typeof displayName !== "string" ? "sc" : escape(displayName);
  // 기본적으로 displayName이 없기 때문에 데모에서 'sc'라는 접두사를 볼 수 있습니다.

  // displayName이 중복된 componentId로 이어지지 않도록 보장합니다.
  identifiers[name] = (identifiers[name] || 0) + 1;
  // 이론상, 각 컴포넌트는 sc0, sc1 등의 ID를 가집니다.

  const componentId = `${name}-${generateComponentId(
    // SC_VERSION은 페이지에 여러 런타임이 동시에 있을 때 격리를 제공합니다.
    // 이는 babel 플러그인의 "namespace" 기능을 사용하면 더욱 개선됩니다.
    SC_VERSION + name + identifiers[name]
  )}`;
  return parentComponentId
    ? `${parentComponentId}-${componentId}`
    : componentId;
}
```

```ts
import generateAlphabeticName from "./generateAlphabeticName";
import { hash } from "./hash";

export default function generateComponentId(str: string) {
  // 문자열을 해시하고 해시된 문자열에서 문자열을 반환합니다.
  return generateAlphabeticName(hash(str) >>> 0);
}
```

컴포넌트 ID 생성에서 두 가지를 확인할 수 있습니다:

전역 카운터로 인해 고유합니다.
전역 카운터로 인해 SSR 안전하지 않습니다. SSR이 안전하지 않은 이유에 대해서는 ["React에서 useId()는 내부적으로 어떻게 작동하나요?"](https://jser.dev/2023-04-25-how-does-useid-work/)에서 다뤘습니다.

### 2.5 ComponentStyle에서 CSS 주입

```ts
function useInjectedStyle<T extends ExecutionContext>(
  componentStyle: ComponentStyle,
  resolvedAttrs: T
) {
  const ssc = useStyleSheetContext();
  const className = componentStyle.generateAndInjectStyles(
    resolvedAttrs,
    ssc.styleSheet, // 이 컨텍스트를 기억하세요. 여기에는 CSS에 대한 모든 정보가 포함됩니다.
    ssc.stylis
  );
  return className;
}
```

```tsx
export default class ComponentStyle {
  baseHash: number;
  baseStyle: ComponentStyle | null | undefined;
  componentId: string;
  isStatic: boolean;
  rules: RuleSet<any>;
  staticRulesId: string;

  constructor(
    rules: RuleSet<any>,
    componentId: string,
    baseStyle?: ComponentStyle | undefined
  ) {
    this.rules = rules;
    this.staticRulesId = "";
    this.isStatic =
      process.env.NODE_ENV === "production" &&
      (baseStyle === undefined || baseStyle.isStatic) &&
      isStaticRules(rules);
    // 규칙이 정적이며 캐시될 수 있는지 확인합니다.

    this.componentId = componentId;
    this.baseHash = phash(SEED, componentId);
    // baseHash는 componentId에서 생성되며, componentId에 따라 안정적입니다.

    this.baseStyle = baseStyle;
    // NOTE: 이는 componentId를 등록하여 이 컴포넌트의 스타일이 다른 스타일과 비교해 일관된 순서를 유지하도록 합니다.
    StyleSheet.registerId(componentId);
  }

  generateAndInjectStyles(
    executionContext: ExecutionContext,
    styleSheet: StyleSheet,
    stylis: Stringifier
  ): string {
    let names = this.baseStyle
      ? this.baseStyle.generateAndInjectStyles(
          executionContext,
          styleSheet,
          stylis
        )
      : "";
    // 사용자가 제공한 stylis 플러그인이 사용 중인 경우 동적 클래스명을 강제로 사용합니다.
    if (this.isStatic && !stylis.hash) {
      if (
        this.staticRulesId &&
        styleSheet.hasNameForId(this.componentId, this.staticRulesId)
      ) {
        names = joinStrings(names, this.staticRulesId);
      } else {
        // StyleSheetContext는 정적 스타일에 대한 안정적인 ID를 유지합니다.
        // 이는 styled component에서 정적 규칙만 사용될 경우 합리적입니다.
        // 매번 새로운 규칙과 클래스 이름을 만들 필요가 없습니다.

        const cssStatic = joinStringArray(
          flatten(this.rules, executionContext, styleSheet, stylis) as string[]
        );
        const name = generateName(phash(this.baseHash, cssStatic) >>> 0);
        if (!styleSheet.hasNameForId(this.componentId, name)) {
          const cssStaticFormatted = stylis(
            cssStatic,
            `.${name}`,
            undefined,
            this.componentId
          );
          styleSheet.insertRules(this.componentId, name, cssStaticFormatted);
        }
        names = joinStrings(names, name);
        this.staticRulesId = name;
      }
    } else {
      let dynamicHash = phash(this.baseHash, stylis.hash);
      let css = "";
      for (let i = 0; i < this.rules.length; i++) {
        const partRule = this.rules[i];
        if (typeof partRule === "string") {
          css += partRule;
          if (process.env.NODE_ENV !== "production")
            dynamicHash = phash(dynamicHash, partRule);
        } else if (partRule) {
          const partString = joinStringArray(
            flatten(partRule, executionContext, styleSheet, stylis) as string[]
          );
          // 동일한 값이 배열 내에서 위치를 변경할 수 있으므로 해시에 "i"를 포함합니다.
          dynamicHash = phash(dynamicHash, partString + i);
          css += partString;
        }
      }
      if (css) {
        const name = generateName(dynamicHash >>> 0);
        if (!styleSheet.hasNameForId(this.componentId, name)) {
          // 주입 시 중복이 없도록 보장합니다.
          styleSheet.insertRules(
            this.componentId,
            name,
            stylis(css, `.${name}`, undefined, this.componentId)
          );
        }
        names = joinStrings(names, name);
      }
    }
    return names;
  }
}
```

`flatten()` 함수는 기본적으로 동적 부분에 대한 평가를 수행합니다.

```tsx
export default function flatten<Props extends object>(
  chunk: Interpolation<object>,
  executionContext?: (ExecutionContext & Props) | undefined,
  styleSheet?: StyleSheet | undefined,
  stylisInstance?: Stringifier | undefined
): RuleSet<Props> {
  if (isFalsish(chunk)) {
    return [];
  }
  /* 다른 컴포넌트를 처리합니다 */
  if (isStyledComponent(chunk)) {
    return [
      `.${(chunk as unknown as IStyledComponent<"web", any>).styledComponentId}`,
    ];
  }
  /* 함수 실행 또는 지연 처리 */
  if (isFunction(chunk)) {
    if (isStatelessFunction(chunk) && executionContext) {
      const result = chunk(executionContext);
      return flatten<Props>(
        result,
        executionContext,
        styleSheet,
        stylisInstance
      );
    } else {
      return [chunk as unknown as IStyledComponent<"web">];
    }
  }
  if (chunk instanceof Keyframes) {
    if (styleSheet) {
      chunk.inject(styleSheet, stylisInstance);
      return [chunk.getName(stylisInstance)];
    } else {
      return [chunk];
    }
  }
  /* 객체를 처리합니다 */
  if (isPlainObject(chunk)) {
    return objToCssArray(chunk as StyledObject<Props>);
  }
  if (!Array.isArray(chunk)) {
    return [chunk.toString()];
  }
  return flatMap(chunk, (chunklet) =>
    flatten<Props>(chunklet, executionContext, styleSheet, stylisInstance)
  );
}
```

한번 요약해봅시다.

1. 클래스 이름은 컴포넌트 ID와 CSS 규칙을 기반으로 생성됩니다.

2. 규칙이 정적(예: ${props => ...}와 같은 동적 부분이 없음)인 경우, 전체 문자열이 사용됩니다. 동적 규칙의 경우, 먼저 평가되어야 합니다.

3. 동일한 CSS는 두 번 주입되지 않습니다.

### 2.6 서버 사이드 렌더링(SSR)은 컴파일 중에 고유한 컴포넌트 ID를 할당하여 지원됩니다.

앞서 언급했듯이, 컴포넌트 ID에 대한 전역 카운터는 컴포넌트의 실행이 안정적이지 않기 때문에 SSR에 안전하지 않습니다.

`generateId()` 함수는 옵션을 받아 카운터를 건너뛸 수 있습니다.

```ts
componentId = generateId(options.displayName, options.parentComponentId);
```

따라서 SSR은 컴포넌트 ID를 설정하는 추가 줄을 삽입하여 코드를 변환하는 [Babel 플러그인](https://github.com/styled-components/babel-plugin-styled-components)을 통해 지원됩니다.

[테스트 케이스](https://github.com/styled-components/babel-plugin-styled-components/tree/main/test/fixtures/add-identifier-and-display-name)를 확인하면 자동으로 추가되는 내용을 볼 수 있습니다.

```ts
const Test = styled.div`
  width: 100%;
`;
```

위의 코드는 아래처럼 컴파일 됩니다.

```ts
const Test = styled.div.withConfig({
  displayName: "Test",
  componentId: "sc-1cza72q-0",
})`
  width: 100%;
`;
```

디스플레이 이름(displayName)과 컴포넌트 ID는 자동으로 추가됩니다. [2.2절](https://jser.dev/2023-10-09-styled-components/#22-styledtagname-are-wrapped-template-functions)에서 `withConfig()`가 어떻게 작동하는지 확인했습니다.

이 변환은 babel-plugin-styled-components에서 쉽게 찾을 수 있습니다.

```tsx
export default (t) => (path, state) => {
  if (
    path.node.tag
      ? isStyled(t)(path.node.tag, state)
      : /* styled()`` */ (isStyled(t)(path.node.callee, state) &&
          path.node.callee.property &&
          path.node.callee.property.name !== "withConfig") ||
        // styled(x)({})
        (isStyled(t)(path.node.callee, state) &&
          !t.isMemberExpression(path.node.callee.callee)) ||
        // styled(x).attrs()({})
        (isStyled(t)(path.node.callee, state) &&
          t.isMemberExpression(path.node.callee.callee) &&
          path.node.callee.callee.property &&
          path.node.callee.callee.property.name &&
          path.node.callee.callee.property.name !== "withConfig") ||
        // styled(x).withConfig({})
        (isStyled(t)(path.node.callee, state) &&
          t.isMemberExpression(path.node.callee.callee) &&
          path.node.callee.callee.property &&
          path.node.callee.callee.property.name &&
          path.node.callee.callee.property.name === "withConfig" &&
          path.node.callee.arguments.length &&
          Array.isArray(path.node.callee.arguments[0].properties) &&
          !path.node.callee.arguments[0].properties.some((prop) =>
            ["displayName", "componentId"].includes(prop.key.name)
          ))
  ) {
    const displayName =
      useDisplayName(state) &&
      getDisplayName(t)(path, useFileName(state) && state);
    addConfig(t)(
      path,
      displayName && displayName.replace(/[^_a-zA-Z0-9-]/g, ""),
      useSSR(state) && getComponentId(state)
    );
  }
};
```

기본적으로 styled components인 경우, `getDisplayName()`으로 displayName을 생성하고, `getComponentId()`로 컴포넌트 ID를 생성하며, `addConfig()`로 AST를 수정합니다.

파일 이름과 변수 이름은 `getDisplayName()`에서 고려되므로 `const Test` 때문에 `displayName: "Test"`가 표시됩니다.

```ts
const getDisplayName = (t) => (path, state) => {
  const { file } = state;
  const componentName = getName(t)(path);
  if (file) {
    const blockName = getBlockName(file, useMeaninglessFileNames(state));
    if (blockName === componentName) {
      return componentName;
    }
    return componentName
      ? `${prefixLeadingDigit(blockName)}__${componentName}`
      : prefixLeadingDigit(blockName);
  } else {
    return componentName;
  }
};
```

```ts
const getNextId = (state) => {
  const id = state.file.get(COMPONENT_POSITION) || 0;
  state.file.set(COMPONENT_POSITION, id + 1);
  return id;
};

const getComponentId = (state) => {
  // 식별자를 문자로 시작하도록 접두사를 추가합니다. 이는 CSS 클래스가 숫자로 시작할 수 없기 때문입니다.
  return `${useNamespace(state)}sc-${getFileHash(state)}-${getNextId(state)}`;
};
```

네, 컴포넌트 ID를 생성하는 또 다른 카운터가 있지만, 이는 런타임이 아닌 컴파일 시에 작동하므로 완벽하게 동작합니다.

SSR의 나머지 부분은 이제 간단합니다. 우리는 다음을 수행해야 합니다:

1. CSS 코드를 추출하여 HTML에 삽입합니다.
2. 클래스 이름을 컨텍스트에 제공하여 CSS 주입을 피합니다.

```ts
import { renderToString } from 'react-dom/server';
import { ServerStyleSheet, StyleSheetManager } from 'styled-components';
const sheet = new ServerStyleSheet();
try {
  const html = renderToString(
    <StyleSheetManager sheet={sheet.instance}>
      <YourApp />
    </StyleSheetManager>
  );
  const styleTags = sheet.getStyleTags(); // or sheet.getStyleElement();
} catch (error) {
  // handle error
  console.error(error);
} finally {
  sheet.seal();
}

```

여기서는 `StyleSheetManager`를 사용하며, `StyleSheetContext`가 사용됩니다.

```ts
export const StyleSheetContext = React.createContext<IStyleSheetContext>({
  shouldForwardProp: undefined,
  styleSheet: mainSheet,
  stylis: mainStylis,
});
```

[CSS 주입](https://jser.dev/2023-10-09-styled-components/#25-css-injection-in-componentstyle) 시 `StyleSheetContext`를 사용하여 중복을 제거하는 것을 기억해야 합니다.

### 3. 요약

인기 있는 CSS-in-JS 솔루션 중 하나인 Styled Components는 태그된 템플릿을 활용하여 CSS 코드를 컴포넌트에 바인딩합니다. 컴포넌트 ID와 CSS 코드를 해싱하여 클래스 이름을 생성하고, 렌더링 시 CSS 코드를 주입합니다. 기본적으로 컴포넌트 ID는 전역 카운터에 의해 생성되므로, CSS를 지원하기 위해서는 각 스타일드 컴포넌트에 안정적인 ID를 할당하는 Babel 플러그인이 필요합니다.

내부 구조를 살펴보면, 해싱과 CSS 주입을 포함하여 런타임에 무거운 로직이 작동하는 것을 볼 수 있습니다. 태그된 템플릿의 문제는 동적 부분에 대한 제한이 없다는 점입니다. 동적 함수가 문자열을 반환하는 것만 알 뿐, 그 내용이 무엇인지 알 수 없기 때문에 정적인 개선이 불가능합니다([관련 논의](https://github.com/styled-components/styled-components/issues/1018)). Styled Components를 사용하려면 기본적으로 SSR을 사용하는 것이 좋습니다. 이는 성능을 크게 향상시키지만, 초기 로드에만 해당됩니다.
