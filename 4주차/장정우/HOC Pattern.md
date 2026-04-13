# [HOC Pattern](https://patterns-dev-kr.github.io/design-patterns/hoc-pattern/)

## [What] HOC Pattern이란?

> "다른 컴포넌트를 받는 컴포넌트" _(patterns.dev)_

> HOC(Higher-Order Component)는 **고차 함수(Higher-Order Function)** 의 개념을 컴포넌트에 적용한 패턴이다. 고차 함수가 함수를 인자로 받거나 함수를 반환하는 함수인 것처럼, HOC는 컴포넌트를 인자로 받아 새로운 컴포넌트를 반환하는 함수이다. _(claude)_

함수형 프로그래밍에서 **함수 합성(Function Composition)** 은 핵심적인 개념이다. 작은 함수를 조합해 더 큰 기능을 만들어내는 것. 이 아이디어는 곳곳에 녹아 있다.

- Python의 `@decorator`
- JavaScript의 `Array.prototype.map` (함수를 인자로 받는 고차 함수)
- Express/Koa의 미들웨어 패턴

모두 "감싸서 기능을 덧붙인다"는 동일한 원리를 공유한다. HOC도 마찬가지다. 원본 컴포넌트를 감싸서, 추가적인 데이터나 기능을 부여한 새로운 컴포넌트를 반환한다.

### HOC vs Decorator

> Decorator Pattern랑 비슷하지 않나? 감싸서 덧붙인다.. but

| 기준 | HOC                   | Decorator |
|:--|:----------------------|:--|
| 인자 | 컴포넌트 + 추가 설정을 받을 수 있음 | 단일 대상을 감쌈 |
| 반환 | **새로운** 컴포넌트를 생성      | 기존 객체의 동작을 **수정** |
| 원본 변경 | 원본 컴포넌트를 변경하지 않음      | 구현에 따라 원본을 수정할 수 있음 |

HOC는 `withLoader(DogImages, url)`처럼 컴포넌트 외에 추가 인자를 받을 수 있고, 항상 **새로운 컴포넌트**를 생성한다는 점에서 Decorator와 구분된다.

## [Patterns.dev] HOC Pattern 예시로 알아보기

### withStyles: 스타일을 주입하는 HOC

가장 단순한 형태의 HOC이다. 컴포넌트에 공통 스타일을 주입한다.

```jsx
function withStyles(Component) {
  return function StyledComponent(props) {
    const style = {
      padding: '0.2rem',
      margin: '1rem',
    }

    return <Component style={style} {...props} />
  }
}
```

```jsx
const Button = ({ style, children }) => (
  <button style={style}>{children}</button>
)

const Text = ({ style, children }) => (
  <p style={style}>{children}</p>
)

const StyledButton = withStyles(Button)
const StyledText = withStyles(Text)
```

`Button`과 `Text`는 스타일에 대해 아무것도 모른다. `withStyles`가 스타일을 주입해줄 뿐이다.

### withLoader: 데이터 로딩을 처리하는 HOC

실무에서 더 유용한 패턴이다. 데이터를 fetch하고, 로딩 중에는 로딩 표시를 보여주는 로직을 HOC로 분리한다.

```jsx
function withLoader(Component, url) {
  return function LoaderComponent(props) {
    const [loading, setLoading] = useState(true)
    const [data, setData] = useState(null)

    useEffect(() => {
      async function fetchData() {
        const response = await fetch(url)
        const result = await response.json()
        setData(result)
        setLoading(false)
      }
      fetchData()
    }, [])

    if (loading) {
      return <div>Loading...</div>
    }

    return <Component {...props} data={data} />
  }
}
```

```jsx
function DogImages({ data }) {
  return (
    <div>
      {data.message.map((dog, index) => (
        <img key={index} src={dog} alt="Dog" />
      ))}
    </div>
  )
}

export default withLoader(
  DogImages,
  'https://dog.ceo/api/breed/labrador/images/random/6'
)
```

`DogImages`는 데이터를 받아 렌더링하는 일에만 집중한다. 데이터를 어떻게 가져오는지, 로딩 상태를 어떻게 처리하는지는 전혀 알 필요가 없다.

### withHover: 호버 상태를 추적하는 HOC

```jsx
function withHover(Component) {
  return function HoverComponent(props) {
    const [hovering, setHover] = useState(false)

    return (
      <div
        onMouseEnter={() => setHover(true)}
        onMouseLeave={() => setHover(false)}
      >
        <Component {...props} hovering={hovering} />
      </div>
    )
  }
}
```

### HOC 합성하기

여러 HOC를 합성하여 하나의 컴포넌트에 여러 기능을 덧붙일 수 있다.

```jsx
export default withHover(
  withLoader(DogImages, 'https://dog.ceo/api/breed/labrador/images/random/6')
)
```

`DogImages`는 이제 데이터 로딩 기능과 호버 감지 기능을 모두 갖추게 된다. 컴포넌트 자체의 코드는 전혀 변경하지 않았다.

## [How] HOC Pattern은 어떻게 구현하는가?

핵심은 **컴포넌트를 인자로 받아, 추가 로직을 적용한 새로운 컴포넌트를 반환하는 함수를 만드는 것**이다.

### 기본 구조

```jsx
function withSomething(WrappedComponent) {
  return function EnhancedComponent(props) {
    // 추가 로직 (상태, 이벤트, 데이터 fetch 등)
    const [extraData, setExtraData] = useState(null)

    return <WrappedComponent {...props} extraData={extraData} />
  }
}
```

1. **함수 이름**: `with___` 접두사를 사용하는 것이 관례이다. `withAuth`, `withLayout`, `withLogger` 등.
2. **props 전달**: `{...props}`로 원본 컴포넌트의 props를 그대로 전달한다.
3. **추가 props 주입**: HOC가 관리하는 데이터를 새로운 prop으로 주입한다.

### 추가 설정을 받는 HOC

컴포넌트 외에 설정값을 함께 받을 수도 있다.

```jsx
function withLoader(Component, url) {
  return function LoaderComponent(props) {
    // url을 사용해 데이터를 fetch
    // ...
    return <Component {...props} data={data} />
  }
}

// 사용
const EnhancedList = withLoader(UserList, '/api/users')
```

또는 커링(currying) 형태로 작성하기도 한다.

```jsx
function withLoader(url) {
  return function (Component) {
    return function LoaderComponent(props) {
      // ...
      return <Component {...props} data={data} />
    }
  }
}

// 사용
const EnhancedList = withLoader('/api/users')(UserList)
```

### HOC 합성

여러 HOC를 합성할 때는 함수 중첩으로 작성한다.

```jsx
// 직접 중첩
const Enhanced = withAuth(withLayout(withLogger(MyComponent)))

// compose 유틸리티 사용
const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x)
const Enhanced = compose(withAuth, withLayout, withLogger)(MyComponent)
```

## [Why] HOC Pattern은 왜 사용하는가?

Q. 언제 사용하면 좋을까?
- 핵심은 **여러 컴포넌트에 동일한 로직을 적용하되, 각 컴포넌트의 본래 역할은 건드리지 않고 싶을 때.**

Q. 왜 컴포넌트 내부에 직접 로직을 작성하지 않고 HOC를 쓰는가?

1. **DRY — 반복 제거**
    - 인증 체크, 로딩 처리, 에러 핸들링 같은 로직이 10개의 컴포넌트에 필요하다면, 각각에 복붙하는 대신 HOC 하나로 해결할 수 있다.

2. **관심사의 분리**
    - 데이터 로딩, 인증, 로깅 같은 **횡단 관심사(Cross-Cutting Concerns)** 를 컴포넌트 본연의 렌더링 로직에서 분리한다. 컴포넌트는 "무엇을 보여줄 것인가"에만 집중하고, "데이터를 어떻게 가져올 것인가"는 HOC에 위임한다.

3. **개방-폐쇄 원칙(OCP)**
    - 원본 컴포넌트의 코드를 수정하지 않고 기능을 확장한다. 새로운 기능이 필요하면 새로운 HOC를 감싸기만 하면 된다.

Q. HOC의 단점은 무엇인가?

1. **Props 이름 충돌**
    - 여러 HOC가 같은 이름의 prop을 주입하면, 나중에 감싸는 HOC의 값이 앞의 것을 덮어쓴다.

    ```jsx
    function withStylesA(Component) {
      return (props) => <Component {...props} style={{ color: 'red' }} />
    }

    function withStylesB(Component) {
      return (props) => <Component {...props} style={{ backgroundColor: 'blue' }} />
    }

    // style prop이 충돌 — withStylesA가 주입한 style이 사라짐
    const Enhanced = withStylesA(withStylesB(MyComponent))
    ```

2. **Props 출처 불명확**
    - HOC가 여러 겹 감싸지면, 컴포넌트가 받는 props 중 어떤 것이 부모에서 온 것이고, 어떤 것이 HOC에서 주입된 것인지 코드만 보고는 알 수 없다.

    ```jsx
    // MyComponent에서 data, hovering, style 등의 props를 받지만
    // 어디에서 온 건지 추적하려면 HOC 체인을 하나하나 따라가야 한다
    export default withAuth(withHover(withStyles(withLoader(MyComponent, url))))
    ```

3. **Wrapper Hell**
    - HOC를 많이 사용하면 React DevTools에서 컴포넌트 트리가 깊어진다.

    ```
    <withAuth>
      <withLayout>
        <withLogging>
          <withLoader>
            <MyComponent />
          </withLoader>
        </withLogging>
      </withLayout>
    </withAuth>
    ```

    디버깅할 때 실제 컴포넌트를 찾기 어려워진다.

4. **정적 메서드 손실**
    - HOC는 새로운 컴포넌트를 반환하기 때문에, 원본 컴포넌트에 정의된 static method가 사라진다. `hoist-non-react-statics` 같은 라이브러리를 별도로 사용해야 한다.

## [Question] HOC vs Hooks 비교

**HOC가 여전히 적합한 경우**
- 앱 전반에 걸쳐 **동일하며 커스터마이징이 불필요한** 횡단 관심사
- 인증 체크: 로그인하지 않으면 리다이렉트 (`withAuth`)
- 레이아웃 래핑: 모든 페이지에 동일한 레이아웃 적용 (`withLayout`)
- 에러 바운더리: 컴포넌트 단위 에러 격리 (`withErrorBoundary`)
- 이런 경우 "각 컴포넌트에서 커스터마이징할 것이 없고, 일괄적으로 적용"되어야 하므로 HOC가 간결하다.

**Hooks가 적합한 경우**
- 각 컴포넌트에서 **커스터마이징이 필요**하거나, 하나 또는 소수의 컴포넌트에서만 사용하는 로직
- 폼 상태 관리: 컴포넌트마다 필드 구성이 다름 (`useForm`)
- 데이터 fetch: 각각 다른 URL, 다른 옵션 (`useSWR`, `useQuery`)
- 타이머, 인터섹션 옵저버 등 컴포넌트별 설정이 다른 로직

| 기준 | HOC | Hooks |
|:--|:--|:--|
| 적용 방식 | 컴포넌트를 감싸서 새 컴포넌트 반환 | 컴포넌트 내부에서 훅 호출 |
| Props 출처 | 불명확 (자동 주입) | 명확 (반환값을 직접 할당) |
| 조합 | 함수 합성 (중첩) | 순차적 호출 (평탄) |
| 타입 안정성 | 제네릭이 복잡해짐 | 비교적 간단 |
| 적합한 경우 | 전역적, 일괄적 횡단 관심사 | 지역적, 커스터마이징 가능한 로직 |

```jsx
// HOC 방식: 선언적이고 일괄적
export default withAuth(MyPage)

// Hooks 방식: 명시적이고 유연함
function MyPage() {
  const { user, isLoading } = useAuth()
  if (isLoading) return <Loading />
  if (!user) return <Redirect to="/login" />
  // ...
}
```

HOC와 Hooks는 대체 관계가 아니라 **적합한 영역이 다른 도구**이다.   
"모든 곳에 동일하게, 자동으로" 적용해야 하는 로직은 HOC가, "각 컴포넌트에서 세밀하게 제어"해야 하는 로직은 Hooks가 적합.
