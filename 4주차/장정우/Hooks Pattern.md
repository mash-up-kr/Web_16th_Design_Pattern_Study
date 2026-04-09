# [Hooks Pattern](https://patterns-dev-kr.github.io/design-patterns/hooks-pattern/)

## [What] Hooks Pattern이란?

> "클래스 없이 상태와 생명주기 로직을 함수 단위로 캡슐화하고 조합하는 패턴" _(claude)_

> Hooks를 통해 React의 state와 생명 주기 함수를 클래스를 사용하지 않고 쓸 수 있게 되었다. _(patterns.dev)_

Hooks는 GoF 디자인 패턴에 속하지 않는다. 그러나 그 근본 원리인 **Composition over Inheritance(상속보다 조합)**는 소프트웨어 설계에서 가장 중요한 원칙 중 하나이다.  

클래스 상속 계층 없이 행위를 공유하는 방법을 찾는 것은 React만의 고민이 아니다. Go는 인터페이스 임베딩을, Rust는 트레이트를, 함수형 프로그래밍은 함수 합성을 사용한다. React Hooks는 이 동일한 철학을 UI 컴포넌트에 적용한 것이다.
> "상속 계층 없이, 필요한 행위만 골라서 조합할 수 없을까?"

Hooks는 클래스 컴포넌트가 가진 세 가지 문제를 해결하기 위해 등장했다.

1. **ES2015 class 학습 비용** — `this` 바인딩, `constructor`, `super()` 등 클래스 문법 자체의 진입 장벽이 높다.
2. **Wrapper Hell** — HOC와 Render Props를 중첩하면 컴포넌트 트리가 깊어지고, props 출처를 추적하기 어려워진다.
3. **생명주기에 얽힌 복잡한 로직** — 관련 있는 로직이 `componentDidMount`와 `componentWillUnmount`에 흩어지고, 관련 없는 로직이 하나의 생명주기 메서드에 뒤섞인다.

그 시절 문제점
1. 생명주기 메서드가 "시점" 기준이지 "관심사" 기준이 아니다 -> 로직 관리 너무 힘듬
```jsx
  class ChatRoom extends React.Component {
    componentDidMount() {
      // 채팅 소켓 연결
      this.socket = connectSocket(this.props.roomId)

      // 윈도우 리사이즈 감지 (채팅과 무관)
      window.addEventListener('resize', this.handleResize)

      // 분석 로그 전송 (채팅과 무관)
      analytics.track('enter_room', this.props.roomId)
    }

    componentWillUnmount() {
      // 채팅 소켓 해제
      this.socket.disconnect()

      // 윈도우 리사이즈 해제
      window.removeEventListener('resize', this.handleResize)
    }

    componentDidUpdate(prevProps) {
      // 방이 바뀌면 소켓 재연결
      if (prevProps.roomId !== this.props.roomId) {
        this.socket.disconnect()
        this.socket = connectSocket(this.props.roomId)
      }
    }
  }
```
Hooks로 바꾸면 관심사 기준으로 묶인다.

```jsx
function ChatRoom({ roomId }) {
   // 관심사 1: 채팅 소켓
   useEffect(() => {
   const socket = connectSocket(roomId)
   return () => socket.disconnect()
   }, [roomId])

    // 관심사 2: 윈도우 리사이즈
    useEffect(() => {
      window.addEventListener('resize', handleResize)
      return () => window.removeEventListener('resize', handleResize)
    }, [])

    // 관심사 3: 분석 로그
    useEffect(() => {
      analytics.track('enter_room', roomId)
    }, [roomId])
}
```
2. React.Component를 상속해야만 setState, componentDidMount 등 생명주기 메서드를 쓸 수 있다.

## [Patterns.dev] Hooks Pattern 예시로 알아보기

### 클래스 컴포넌트의 한계

클래스 컴포넌트로 간단한 `Input`을 만들어 보자.

```jsx
class Input extends React.Component {
  constructor(props) {
    super(props)
    this.state = { value: '' }
    this.handleChange = this.handleChange.bind(this)
  }

  handleChange(e) {
    this.setState({ value: e.target.value })
  }

  render() {
    return (
      <input
        type="text"
        value={this.state.value}
        onChange={this.handleChange}
      />
    )
  }
}
```

`constructor`에서 `super(props)`를 호출하고, `this.state`를 초기화하고, 이벤트 핸들러에 `this`를 바인딩해야 한다. 단순히 입력값 하나를 관리하는 것치고는 보일러플레이트가 많다.

같은 컴포넌트를 Hooks로 작성하면 다음과 같다.

```jsx
function Input() {
  const [value, setValue] = useState('')

  return (
    <input
      type="text"
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  )
}
```

`constructor`, `this`, `bind` — 전부 사라졌다. 상태 관리 로직이 `useState` 한 줄로 줄어들었다.

### Wrapper Hell

HOC와 Render Props를 여러 개 조합하면 다음과 같은 구조가 만들어진다.

```jsx
// HOC 방식의 Wrapper Hell
export default withAuth(
  withTheme(
    withRouter(
      withLogging(
        MyComponent
      )
    )
  )
)
```

```jsx
// Render Props 방식의 Wrapper Hell
<Auth>
  {(auth) => (
    <Theme>
      {(theme) => (
        <Router>
          {(router) => (
            <Logging>
              {(logger) => (
                <MyComponent
                  auth={auth}
                  theme={theme}
                  router={router}
                  logger={logger}
                />
              )}
            </Logging>
          )}
        </Router>
      )}
    </Theme>
  )}
</Auth>
```

depth가 깊어질수록 코드를 읽기 어렵고, 각 prop이 어디에서 오는지 추적하기 어렵다. Hooks를 사용하면 이 모든 것이 평탄해진다.

```jsx
function MyComponent() {
  const auth = useAuth()
  const theme = useTheme()
  const router = useRouter()
  const logger = useLogging()

  // 모든 값의 출처가 명확하다
  return <div>...</div>
}
```

### useState

`useState`는 클래스 컴포넌트의 `this.state`와 `this.setState`를 대체한다.

```jsx
const [value, setValue] = useState(initialValue)
```

배열 구조 분해를 통해 현재 상태값(`value`)과 상태 업데이트 함수(`setValue`)를 받는다.

클래스 컴포넌트와 비교하면 다음과 같다.

```jsx
// 클래스 컴포넌트
class Counter extends React.Component {
  constructor(props) {
    super(props)
    this.state = { count: 0 }
  }

  render() {
    return (
      <div>
        <p>{this.state.count}</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          +1
        </button>
      </div>
    )
  }
}
```

```jsx
// Hooks
function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  )
}
```

### useEffect

`useEffect`는 `componentDidMount`, `componentDidUpdate`, `componentWillUnmount` 세 가지 생명주기 메서드를 하나로 통합한다.

```jsx
useEffect(() => {
  // 마운트 시, 또는 의존성이 변경될 때 실행
  console.log('effect 실행')

  return () => {
    // 클린업: 언마운트 시 또는 다음 effect 실행 전에 호출
    console.log('cleanup 실행')
  }
}, [dependency]) // 의존성 배열
```

의존성 배열에 따라 동작이 달라진다.

| 의존성 배열 | 동작 |
|:--|:--|
| 생략 | 매 렌더링마다 실행 |
| `[]` (빈 배열) | 마운트 시 1회만 실행 |
| `[a, b]` | `a` 또는 `b`가 변경될 때 실행 |

클래스 컴포넌트에서는 관련 로직이 여러 생명주기에 흩어진다.

```jsx
class WindowWidth extends React.Component {
  constructor(props) {
    super(props)
    this.state = { width: window.innerWidth }
    this.handleResize = this.handleResize.bind(this)
  }

  handleResize() {
    this.setState({ width: window.innerWidth })
  }

  componentDidMount() {
    window.addEventListener('resize', this.handleResize)
  }

  componentWillUnmount() {
    window.removeEventListener('resize', this.handleResize)
  }

  render() {
    return <p>Window width: {this.state.width}</p>
  }
}
```

`useEffect`를 사용하면 등록과 해제 로직이 한 곳에 모인다.

```jsx
function WindowWidth() {
  const [width, setWidth] = useState(window.innerWidth)

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth)
    window.addEventListener('resize', handleResize)

    return () => window.removeEventListener('resize', handleResize)
  }, [])

  return <p>Window width: {width}</p>
}
```

### Custom Hooks

커스텀 훅은 `use`로 시작하는 함수로, 내부에서 다른 훅을 사용하여 재사용 가능한 로직을 캡슐화한다.

#### useKeyPress

특정 키가 눌렸는지 감지하는 훅이다.

```jsx
function useKeyPress(targetKey) {
  const [keyPressed, setKeyPressed] = useState(false)

  useEffect(() => {
    const downHandler = ({ key }) => {
      if (key === targetKey) setKeyPressed(true)
    }
    const upHandler = ({ key }) => {
      if (key === targetKey) setKeyPressed(false)
    }

    window.addEventListener('keydown', downHandler)
    window.addEventListener('keyup', upHandler)

    return () => {
      window.removeEventListener('keydown', downHandler)
      window.removeEventListener('keyup', upHandler)
    }
  }, [targetKey])

  return keyPressed
}
```

```jsx
function App() {
  const enterPressed = useKeyPress('Enter')
  const escPressed = useKeyPress('Escape')

  return (
    <div>
      <p>Enter: {enterPressed ? '눌림' : '안 눌림'}</p>
      <p>Escape: {escPressed ? '눌림' : '안 눌림'}</p>
    </div>
  )
}
```

#### useCounter와 useWindowWidth

앞서 본 `WindowWidth` 클래스 컴포넌트에는 카운터 로직과 윈도우 크기 감지 로직이 하나의 클래스에 뒤섞여 있을 수 있다. 커스텀 훅을 사용하면 이를 독립적인 단위로 분리할 수 있다.

```jsx
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue)

  const increment = () => setCount((prev) => prev + 1)
  const decrement = () => setCount((prev) => prev - 1)
  const reset = () => setCount(initialValue)

  return { count, increment, decrement, reset }
}
```

```jsx
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth)

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth)
    window.addEventListener('resize', handleResize)
    return () => window.removeEventListener('resize', handleResize)
  }, [])

  return width
}
```

```jsx
function Dashboard() {
  const { count, increment, decrement } = useCounter(0)
  const width = useWindowWidth()

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <p>Window width: {width}px</p>
    </div>
  )
}
```

각 훅은 하나의 관심사만 담당하며, 어떤 컴포넌트에서든 독립적으로 재사용할 수 있다. 훅의 이름은 반드시 `use`로 시작해야 한다. 이는 React가 훅의 규칙(Rules of Hooks)을 검증하기 위한 컨벤션이다.

## [How] Hooks Pattern은 어떻게 구현하는가?

핵심은 **상태와 사이드 이펙트를 독립적인 함수 단위로 캡슐화하고, 필요한 컴포넌트에서 조합하여 사용하는 것**이다.

### 커스텀 훅의 기본 구조

```jsx
function useCustomHook(params) {
  // 1. 상태 선언
  const [state, setState] = useState(initialValue)

  // 2. 사이드 이펙트 처리
  useEffect(() => {
    // 구독, 데이터 페칭, DOM 조작 등
    return () => {
      // 클린업
    }
  }, [dependencies])

  // 3. 상태와 핸들러를 반환
  return { state, setState, /* 기타 핸들러 */ }
}
```

### 조합의 평탄한 구조

HOC는 감싸는 구조이므로 중첩이 깊어지지만, 훅은 함수 호출이므로 평탄하게 나열된다.

```jsx
// HOC: 중첩 구조
const EnhancedComponent = withAuth(withTheme(withLogger(Component)))

// Hooks: 평탄한 구조
function Component() {
  const auth = useAuth()
  const theme = useTheme()
  const logger = useLogger()
  // ...
}
```

훅의 반환값을 직접 변수에 할당하므로, 각 값의 출처가 명시적이다. HOC에서는 props가 자동으로 병합되기 때문에 `this.props.theme`이 어느 HOC에서 온 건지 알기 어렵지만, 훅에서는 `const theme = useTheme()`으로 출처가 즉시 드러난다.

## [Why] Hooks Pattern은 왜 사용하는가?

Q. 언제 사용하면 좋을까?
- 핵심은 **컴포넌트 간에 상태를 가진 로직을 공유하고 싶지만, 컴포넌트 구조를 변경하고 싶지 않을 때.**

Q. 왜 클래스 컴포넌트나 HOC 대신 Hooks를 쓰는가?

1. **평탄한 구조**
    - HOC의 Wrapper Hell 없이 로직을 조합할 수 있다. 훅은 함수 호출이므로 중첩이 발생하지 않는다.

2. **Props 출처의 명확성**
    - HOC는 props를 자동으로 병합하기 때문에, 특정 prop이 어느 HOC에서 주입된 것인지 추적이 어렵다. 훅은 반환값을 직접 변수에 할당하므로 출처가 명시적이다.

3. **코드 간결성**
    - `constructor`, `bind`, `this`, `super()` 등의 보일러플레이트가 불필요하다.

4. **로직의 재사용 단위가 작아짐**
    - 커스텀 훅으로 작은 단위의 로직을 분리하고 공유할 수 있다. HOC는 컴포넌트 전체를 감싸야 하지만, 훅은 필요한 로직만 가져다 쓸 수 있다.

Q. Hooks의 단점은 무엇인가?

- **규칙 준수 필요**: 훅은 컴포넌트의 최상위에서만 호출해야 하며, 조건문이나 반복문 안에서 사용할 수 없다. (Rules of Hooks) React가 훅의 호출 순서에 의존하여 상태를 관리하기 때문이다.

```jsx
// 잘못된 사용
function Component({ isLoggedIn }) {
  if (isLoggedIn) {
    const [user, setUser] = useState(null) // 조건문 안에서 훅 호출 — 금지
  }
}
```

- **클로저 함정**: `useEffect`나 이벤트 핸들러 내부에서 state를 참조할 때, 클로저가 이전 렌더링의 값을 캡처하여 의도치 않은 값을 참조할 수 있다.

```jsx
function Timer() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    const id = setInterval(() => {
      console.log(count) // 항상 0을 출력 (stale closure)
      setCount(count + 1) // 항상 1로만 업데이트됨
    }, 1000)
    return () => clearInterval(id)
  }, []) // 빈 의존성 배열: count의 초기값만 캡처

  // 해결: setCount(prev => prev + 1) 사용
}
```

- **useEffect 의존성 배열 관리가 까다로움**: 의존성을 빠뜨리면 stale closure가 발생하고, 불필요한 의존성을 넣으면 effect가 과도하게 실행된다. `eslint-plugin-react-hooks`의 `exhaustive-deps` 규칙이 이를 검증해주지만, 때로는 의도적으로 특정 의존성을 제외해야 하는 경우도 있어 판단이 필요하다.


- **잘못된 추상화 위험**: 모든 로직을 커스텀 훅으로 분리하면 오히려 코드 추적이 어려워진다. 컴포넌트의 핵심 로직까지 훅으로 빼버리면, 컴포넌트를 읽을 때 훅의 내부 구현을 계속 따라가야 하는 문제가 생긴다.

## [Hum..🤔] 커스텀 훅, 언제 만들어야 할까?

커스텀 훅은 강력한 추상화 도구이지만, 모든 로직을 훅으로 분리해야 하는 것은 아니다.

### useEffect를 나눈 것만으로는 부족하다

Hooks를 사용하면 클래스의 생명주기 메서드처럼 "시점 기준"이 아니라 "관심사 기준"으로 로직을 분리할 수 있다. 하지만 단순히 `useEffect`를 여러 개 나열하는 것만으로는 절반만 해결한 것이다.

```jsx
function ChatRoom({ roomId }) {
  // 관심사 1: 채팅 소켓
  useEffect(() => {
    const socket = connectSocket(roomId)
    return () => socket.disconnect()
  }, [roomId])

  // 관심사 2: 윈도우 리사이즈
  useEffect(() => {
    window.addEventListener('resize', handleResize)
    return () => window.removeEventListener('resize', handleResize)
  }, [])

  // 관심사 3: 분석 로그
  useEffect(() => {
    analytics.track('enter_room', roomId)
  }, [roomId])
}
```

클래스 컴포넌트보다는 낫지만, 여전히 **서로 무관한 3개의 관심사가 하나의 컴포넌트 안에 나열되어 있을 뿐**이다. 컴포넌트가 커질수록 "이 컴포넌트가 뭘 하는 건지" 한눈에 파악하기 어려워진다.

커스텀 훅으로 추출하면 컴포넌트는 **"무엇을 하는지"만 선언**하게 된다.

```jsx
function ChatRoom({ roomId }) {
  useChatSocket(roomId)
  useWindowResize(handleResize)
  useEnterTracking('enter_room', roomId)
}
```

"어떻게 하는지"는 각 훅 내부에 캡슐화되어 있으므로, 컴포넌트를 읽는 사람은 세부 구현을 알 필요가 없다.

다만, 이것이 **모든 `useEffect`를 무조건 훅으로 빼라는 의미는 아니다.** `useEffect`가 하나뿐이거나, 해당 컴포넌트에서만 사용하는 로직이라면 굳이 분리할 필요 없다. 위 ChatRoom처럼 **서로 무관한 관심사가 여러 개 섞여 있을 때** 분리의 효과가 크다.

### 커스텀 훅을 만들어야 하는 경우

1. **동일한 상태 로직이 2개 이상의 컴포넌트에서 반복될 때**
    - 여러 컴포넌트에서 윈도우 크기를 감지하거나, 키보드 입력을 추적하거나, API를 호출하는 동일한 패턴이 반복된다면 커스텀 훅으로 추출할 타이밍이다.

2. **컴포넌트의 본질적 역할과 무관한 부수적 로직을 분리하고 싶을 때**
    - 이벤트 리스너 등록/해제, 타이머 관리, API 호출 등은 컴포넌트가 "무엇을 보여주는가"와 직접적인 관련이 없다. 이런 로직을 훅으로 분리하면 컴포넌트는 렌더링에만 집중할 수 있다.

### 만들지 않아도 되는 경우

- **한 컴포넌트에서만 사용하는 로직** — 재사용 가능성이 없다면 굳이 훅으로 분리할 필요 없다. 그냥 컴포넌트 안에 두면 된다.
- **상태가 없는 단순한 유틸리티 함수** — `formatDate`, `debounce` 같은 순수 함수는 훅이 아니다. `use` 접두어는 내부에서 `useState`, `useEffect` 등 React 훅을 사용할 때만 붙여야 한다.

### 커스텀 훅 vs 유틸리티 함수

| 기준 | 커스텀 훅 | 유틸리티 함수 |
|:--|:--|:--|
| 상태 관리 | `useState`, `useReducer` 사용 | 상태 없음 |
| 사이드 이펙트 | `useEffect` 사용 | 사이드 이펙트 없음 |
| React 의존성 | React 런타임 필요 | 순수 JS, 어디서든 사용 가능 |
| 테스트 | 렌더링 환경 필요 (`renderHook`) | 단순 함수 호출로 테스트 |
| 예시 | `useWindowWidth`, `useFetch` | `formatDate`, `debounce` |

"이 함수 안에서 React 훅을 호출하는가?"가 가장 단순한 판단 기준이다. 호출한다면 커스텀 훅, 호출하지 않는다면 유틸리티 함수로 분류하면 된다.
