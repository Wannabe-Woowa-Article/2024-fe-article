## 🔗 [Mastering Error Handling in Frontend Applications: A Comprehensive Guide](https://medium.com/carousell-insider/mastering-error-handling-in-frontend-applications-a-comprehensive-guide-2df73846385b)

### 🗓️ 번역 날짜: 2024.07.31

### 🧚 번역한 크루: 렛서(김다은)

---

## 프론트엔드 애플리케이션의 오류 처리 마스터하기: 종합 가이드

### 소개

생기발랄한 사용자 인터페이스와 눈부신 기능으로 애플리케이션이 생명을 얻는 매혹적인 프론트엔드 개발의 세계에 오신 것을 환영합니다. 그러나 모든 모험과 마찬가지로 이 여정에도 도전 과제가 존재합니다. 가장 강력한 적 중 하나는 잡기 어렵고 예측할 수 없는 “오류”입니다. 이 블로그 게시물에서는 이러한 오류를 정복하고 프론트엔드 애플리케이션에서 원활한 사용자 경험을 창출하기 위한 지식과 도구를 제공하겠습니다.

<br/>

![아름다운 프로그램이 에러에게 공격받고 있습니다.](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*2ogOoE_T8Z6jsVtf8_xl0w.jpeg)

<br/>

### 1장: 영웅들을 만나다 — CustomError와 그 동반자들

오류를 다루는 여정에서, 우리는 "CustomError"가 이끄는 믿음직한 동반자들로 구성된 팀을 만들면서 시작합니다. 마치 마법사의 지팡이처럼, CustomError는 내장 JavaScript "Error" 객체를 확장하여 우리의 애플리케이션 고유의 요구에 맞춥니다. "code"와 "data" 같은 마법 같은 속성들을 통해 오류에 대한 정확한 정보를 전달하여 디버깅에 도움을 줄 수 있습니다.

<br/>

```tsx
export class CustomError extends Error {
  code;
  data;
  constructor(code: string, message: string, data?: any) {
    super(message);
    // target이 es6 미만이면 타입스크립트에서 오류가 프로토타입 체인으로 전달되지 않습니다.
    // https://github.com/Microsoft/TypeScript-wiki/blob/main/Breaking-Changes.md#extending-built-ins-like-error-array-and-map-may-no-longer-work
    this.name = code;
    this.code = code;
    this.data = data;
  }
}

export function isCustomError(candidate: any): candidate is CustomError {
  return candidate instanceof CustomError || 'code' in candidate;
}
```

하지만 CustomError는 이 여정에서 혼자가 아닙니다. 그것은 "AuthenticationError", "AuthorisationError", "NotFoundError"와 같은 특수 오류 클래스 군단과 여러 부대를 생성합니다. 각 클래스는 명확한 목적을 가지고 있어 오류를 식별하고 우아하게 처리하기가 더 쉬워집니다.

```tsx
export class NotFoundError extends CustomError {
  constructor(message = '404 - Not Found', data?: any) {
    super(ERROR_CODES.NOT_FOUND, message, data);
  }
}

export function isNotFoundError(candidate: any): candidate is NotFoundError {
  return (
    candidate instanceof NotFoundError ||
    candidate?.code === ERROR_CODES.NOT_FOUND
  );
}
```

<br/>

### 2장: API 레이어 오류 처리 마스터하기

오류 처리 마스터를 향한 여정에서, 우리는 API 레이어가 수행하는 중요한 역할을 잊어서는 안 됩니다. 이 레이어는 외부 자원과 서비스로 가는 관문 역할을 하므로, 오류를 효과적으로 처리하는 것이 필수적입니다.

```tsx
const GeneralErrorMessage =
  'Error: Please refresh and try again, or contact the support team.';

export const errorHandler = (error) => {
  const { request, response, code } = error;
  if (code) {
    if (code === 'ECONNABORTED')
      throw new ServiceError('The server is taking too long to respond!');
  }
  if (response) {
    const { data, status } = response;
    const readableError = getReadableError(status, data);
    throw readableError;
  } else if (request) {
    // 요청은 보내졌지만 응답이 오지 않았을 때
    throw new ServiceError(GeneralErrorMessage);
  } else {
    const { status } = { ...error.toJSON(), ...{ status: 500 } };
    const readableError = getReadableError(status);
    throw readableError;
  }
};
const getReadableError = (status: number, data?: any) => {
  if (status <= 500) {
    if (status === 404)
      return new NotFoundError(data?.errorMsg ?? ErrorMessages[404]);
    if (status === 500)
      return new ServiceError(data?.errorMsg ?? ErrorMessages[500]);
    if (status === 401)
      return new AuthenticationError(data?.errorMsg ?? ErrorMessages[401]);
    if (status === 403)
      return new AuthorisationError(data?.errorMsg ?? ErrorMessages[403]);
    if (status === 409 || status === 422)
      return new ValidationError(data?.errorMsg ?? ErrorMessages[409]);
    return new ClientError(data?.errorMsg ?? ErrorMessages[status]);
  }
  return new ServiceError(data?.errorMsg ?? GeneralErrorMessage, data);
};
```

우리의 API 오류 핸들러는 오류 객체를 검사하고, 다양한 오류 상황을 처리하며, 특정 상황에 대해 사용자 정의 오류 클래스를 던져 일관되고 구조화된 오류 처리 접근 방식을 보장합니다.

<br/>

### 3장: 왕국 관리 — 오류를 위한 상태 관리

광활한 프론트엔드 개발 왕국에서는 다양한 오류를 만나게 됩니다. 지혜롭고 우아하게 통치하기 위해서는 이러한 오류를 정밀하게 관리해야 합니다. 그럴 때 Redux와 같은 상태 관리 시스템이 필요합니다.

<br/>

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*fBeBKJmxlnDIqlPS3BqsXA.jpeg)

<br/>

상태 관리를 통합함으로써 오류를 상태에 원활하게 전파하여 사용자에게 실시간 피드백을 제공할 수 있습니다. 예기치 않은 오류를 우아하게 처리하든, 정보성 메시지를 제공하든, 잘 구조화된 오류 처리 전략은 조화로운 왕국을 보장합니다.

```tsx
export const errorSlice = createSlice({
  name: ERROR_SLICE_NAMESPACE,
  initialState,
  reducers: {
    setError(
      state,
      { payload: { code, message, data } }: PayloadAction<ErrorState>
    ) {
      state.code = code;
      state.message = message;
      state.data = data;
    },
    clearError(state) {
      const { code, message, data } = initialState;
      state.code = code;
      state.message = message;
      state.data = data;
    },
  },
});
```

<br/>

### 4장: 전장에서 오류 가로채기 — Redux Saga

우리의 여정은 API 오류가 강력한 적으로 등장하는 전장으로 이어집니다. API 오류 처리를 중앙 집중화하기 위해, 우리는 "Redux Saga"라는 현명한 마법사를 호출합니다. 그 신비한 능력을 통해 Redux Saga는 HTTP 요청과 응답을 가로채고 공통 오류 처리 로직을 적용합니다.

```tsx
const sagaTask = sagaMiddleware.run(saga);
sagaTask.toPromise().catch((error) => {
  // error 상태에 따라 적절한 액션을 부착하세요.
  store.dispatch(setError(error));
});
```

<br/>

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*nf6pFSS7GSvTcViwcRSBCw.jpeg)

<br/>

실용적인 예제로 들어가 보겠습니다 — Redux Saga를 사용하여 애플리케이션에 새로운 항목 추가하기:

<br/>

```tsx
export function* addNewItem(action) {
  const {
    payload: { name },
  } = action;

  // UI 상태를 업데이트하여 항목 추가가 진행 중임을 나타냅니다
  yield put(addNewItemPending());

  // 새로운 항목을 추가하기 위해 API를 호출합니다
  const data = yield call(ItemsClient.addNewItem, {
    data: { name },
  });

  if (data?.id) {
    // 성공 상태로 UI 상태를 업데이트합니다
    yield put(addNewItemSuccess());
    // 사용자에게 성공 메시지를 표시합니다
    yield put(
      showToastMessageAction({
        type: 'success',
        message: `Successfully added item: ${name}!`,
      })
    );
  } else {
    throw new IAMError("Couldn't add new item");
  }
}
```

<br/>

이 예제에서는 Redux Saga를 사용하여 새로운 항목을 추가합니다. UI 상태를 업데이트하고, API 호출을 수행하며, 성공 및 오류 시나리오를 처리합니다. Redux Saga를 통해 적절한 오류를 발생시킬 수 있으며, 이러한 오류를 Redux 스토어에 액션으로 디스패치하고, 오류 상태 관리 솔루션을 사용하여 관리할 수 있습니다. 이를 통해 오류 처리를 통합된 방식으로 제공합니다.

<br/>

### 5장: 에러 바운더리 — 애플리케이션의 방패

프론트엔드 개발의 세계에서는 에러가 애플리케이션 전체를 중단시킬 수 있는 위험한 존재로 나타날 수 있습니다. 두려워하지 마세요! 우리는 "에러 바운더리"라는 마법의 방패를 소개합니다. 이 수호자들은 React에서 영감을 받아 애플리케이션의 일부를 감싸고, 에러의 치명적인 영향을 막아줍니다.

<br/>

```tsx
interface Props extends PropsFromRedux {
  children: React.ReactNode;
  ifNotAuthenticated: () => void;
  onSupportClick: (errorMessage?: string) => void;
}

class ModuleErrorBoundary extends Component<Props> {
  state: {
    error: Error | null;
    errorInfo: ErrorInfo;
    hasError: boolean;
  };
  // 렌더링 중 에러가 발생했을 때 호출됩니다
  static getDerivedStateFromError(error: Error) {
    // 다음 렌더를 위한 상태를 설정합니다
    return { hasError: true, error };
  }
  // 스토어에서 에러가 업데이트될 때 호출됩니다
  static getDerivedStateFromProps(props: Props) {
    const { error } = props;
    logger.info(error);
    if (error.code) return { hasError: true, error };
    return null;
  }
  constructor(props: Props) {
    super(props);
    this.state = {
      hasError: false,
      errorInfo: { componentStack: '' },
      error: null,
    };
  }
  // 에러를 로깅합니다
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    logger.error(error);
    logger.error(errorInfo.componentStack);
  }
  render(): ReactNode {
    const { hasError, error } = this.state;
    const { ifNotAuthenticated, children, onSupportClick } = this.props;
    if (hasError) {
      if (isAuthenticationError(error) && ifNotAuthenticated)
        ifNotAuthenticated();
      const message = isCustomError(error)
        ? error.message
        : `Sorry, something went wrong
Try to refresh the page or contact your admin`;
      return (
        <>
          <Box
            sx={{
              alignItems: 'center',
              justifyContent: 'center',
              display: 'flex',
              height: '100%',
            }}
          >
            <Stack spacing={4}>
              <Image alt="Error Image" loader={CustomLoader} src={ErrorImage} />
              <P align="center" variant="caption">
                {message}
              </P>
              {isNotFoundError(error) || isClientError(error) ? (
                <Button
                  icon={
                    <Image
                      alt="reload"
                      loader={CustomLoader}
                      src={RealoadIcon}
                    />
                  }
                >
                  {RETRTY_LABEL}
                </Button>
              ) : (
                <Button
                  color="secondary"
                  icon={
                    <Icon variant="ContactSupportOutlined" fontSize="small" />
                  }
                  onClick={() => onSupportClick(error?.message)}
                  variant="text"
                >
                  {SUPPORT}
                </Button>
              )}
            </Stack>
          </Box>
        </>
      );
    }
    return children;
  }
}
const mapState = (state: AppState) => ({ error: state.ERROR_SLICE });
const mapDispatch = {
  clearError,
};
const connector = connect(mapState, mapDispatch);
type PropsFromRedux = ConnectedProps<typeof connector>;
export const ConnectedModuleErrorBoundary = connector(ModuleErrorBoundary);
```

<br/>

에러 바운더리를 사용하면 애플리케이션이 에러를 우아하게 처리하고, 사용자 친화적인 메시지를 표시하며, 복구 옵션을 제공할 수 있습니다. 이는 마치 숙련된 기사처럼 왕국을 방어하는 것과 같습니다.

<br/>

### 제 6장: 실전에서의 에러 처리 적용

프론트엔드 개발에서 에러 처리를 위한 지식과 도구를 갖추었으니, 이제 실제 예제를 통해 이를 실전에서 적용해 보겠습니다. 애플리케이션에 새로운 항목을 추가하는 사가가 있다고 가정해 봅시다. 우리가 논의한 에러 처리 기술을 어떻게 적용하는지 단계별로 안내해 드리겠습니다.

<br/>

```tsx
export function* addNewItem(action: PayloadAction<AddItemRequest>) {
  try {
    const {
      payload: { name },
    } = action;

    // 항목 추가가 진행 중임을 나타내기 위해 UI 상태를 업데이트합니다
    yield put(addNewItemPending());
    // 새로운 항목을 추가하기 위해 API를 호출합니다
    const data = yield call(ItemsClient.addNewItem, {
      data: {
        name,
      },
    });
    if (data?.id) {
      // 성공 상태로 UI 상태를 업데이트합니다
      yield put(addNewItemSuccess());
      // 사용자에게 성공 토스트 메시지를 표시합니다
      yield put(
        showToastMessageAction({
          type: 'success',
          message: `Successfully added item: ${name}!`,
        })
      );
      // 항목 테이블의 데이터를 새로 고치거나 필요한 다른 작업을 수행합니다
      // (예: 다른 페이지로 이동)
      // yield put(refreshItemsTableAction());
    } else {
      // API 응답에 'id'가 포함되어 있지 않으면, 사용자 정의 IAMError를 발생시킵니다
      throw new IAMError("Couldn't add new item");
    }
  } catch (err) {
    // 에러를 처리하고 UI 상태를 적절히 업데이트합니다
    yield put(addNewItemFailed());
    // 특정 에러 유형 (IAMError, ValidationError 등)을 처리합니다
    if (!isIAMError(err) && !isValidationError(err)) {
      // 예상된 에러 유형이 아닌 경우, 에러를 다시 발생시킵니다
      throw err;
    }
    // 에러 유형에 따라 사용자 친화적인 에러 메시지를 표시합니다
    if (isValidationError(err)) {
      // 유효성 검사 에러 토스트 메시지를 표시합니다
      yield put(
        showToastMessageAction({
          type: 'error',
          message: 'This item already exists!',
        })
      );
    } else if (isIAMError(err)) {
      // 일반 IAM 에러 토스트 메시지를 표시합니다
      yield put(
        showToastMessageAction({
          type: 'error',
          message: "Couldn't add new item. Please try again later.",
        })
      );
    }
  }
}
```

<br/>

이 예제에서는 애플리케이션에 새로운 항목을 추가하는 사가가 있습니다. 여기에서 논의한 에러 처리 기술을 다음과 같이 사용할 수 있습니다:

- 우선, 항목 추가가 진행 중임을 나타내기 위해 UI 상태를 업데이트합니다(addNewItemPending()).
- 그런 다음, 새로운 항목을 추가하기 위해 API 호출을 하고, API 응답을 검사하여 작업이 성공했는지 확인합니다.
- 응답에 'id'가 포함된 경우, 성공 상태로 UI 상태를 업데이트하고(addNewItemSuccess()), 사용자에게 성공 토스트 메시지를 표시하며, 데이터를 새로 고치는 등의 필요한 다른 작업을 수행할 수 있습니다.
- 응답에 'id'가 포함되지 않은 경우, 특정 에러 케이스를 처리하기 위해 사용자 정의 IAMError를 발생시킵니다. 이를 통해 에러를 우아하게 관리하고 사용자 친화적인 피드백을 제공합니다.
- catch 블록에서는 에러를 처리하고, 실패 상태로 UI 상태를 업데이트하며(addNewItemFailed()), 에러 유형(유효성 검사 에러 또는 IAM 에러)에 따라 사용자 친화적인 에러 메시지를 표시합니다.
- 예상치 못한 에러가 발견되면 다시 발생시킵니다. 이 에러는 사가 미들웨어에 의해 잡히고, 액션을 디스패치하여 상태를 업데이트합니다. 이 상태는 에러 바운더리에 의해 구독되며, 이에 따라 올바른 에러 뷰를 표시할 수 있습니다.

이 예제를 따르면, 논의한 에러 처리 기술을 프론트엔드 애플리케이션의 실제 시나리오에 적용하여 부드러운 사용자 경험과 견고한 에러 관리를 보장할 수 있습니다.

<br/>

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*QyyaPwiMnQb4Bi4TgasVDw.jpeg)

<br/>

### 결론: 영웅의 여정

프론트엔드 개발의 마법 세계를 탐험하면서, 여러분은 강력한 "CustomError"와 그 동료들을 다루고, 에러 바운더리로 애플리케이션을 보호하며, Redux Saga의 힘을 활용하고, 상태 관리를 통해 왕국을 다스리는 법을 배웠습니다.

프론트엔드 애플리케이션에서 에러 처리를 마스터하는 것은 단순히 에러를 물리치는 것만이 아니라, 사용자 경험을 향상시키고 소프트웨어의 안정성을 보장하는 것입니다. 이러한 도구와 기술을 갖추게 된 여러분은 이제 프론트엔드 원더랜드에서 마주치는 어떤 에러도 헤쳐 나갈 준비가 되었습니다.

다음 블로그 시리즈에서는 에러 로깅 및 보고의 숨겨진 영역을 탐험하는 여정을 시작합니다. 프론트엔드 애플리케이션에 숨어 있는 에러에 대한 귀중한 통찰력을 발견하는 여정에 함께하세요. 그때까지, 행복한 코딩 되시길 바라며, 여러분의 코드가 버그 없이 깨끗하기를 바랍니다!
