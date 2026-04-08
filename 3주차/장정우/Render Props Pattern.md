# [Render Props Pattern](https://patterns-dev-kr.github.io/design-patterns/render-props-pattern/)

## [What] Render Props Pattern이란?

> "컴포넌트에게 무엇을 렌더링할지를 함수로 알려주는 패턴"

> render prop이란 JSX 엘리먼트를 리턴하는 함수 값을 가지는 컴포넌트의 prop이다. 컴포넌트 자체는 아무것도 렌더링하지 않고, render prop 함수를 호출하여 그 결과를 반환한다. _(patterns.dev)_

즉, **데이터를 가진 컴포넌트**가 직접 UI를 그리는 것이 아니라, "나는 데이터만 줄 테니, 뭘 그릴지는 네가 정해"라고 함수를 통해 외부에 위임하는 패턴이다.

## [Patterns.dev] Render Props Pattern 예시로 알아보기

가장 기본적인 형태는 다음과 같다.

```jsx
<Title render={() => <h1>I am a render prop!</h1>} />
```

```jsx
const Title = (props) => props.render()
```

`Title` 컴포넌트는 자체적으로 아무것도 렌더링하지 않는다.
단지 `render`라는 이름의 prop으로 받은 함수를 호출하고, 그 결과를 반환할 뿐이다.

prop의 이름이 반드시 `render`일 필요도 없다. JSX를 반환하는 함수라면 어떤 이름이든 render prop으로 볼 수 있다.

### 온도 변환 앱으로 알아보기

`Input` 컴포넌트에서 섭씨 온도를 입력하면, `Kelvin`과 `Fahrenheit` 컴포넌트가 변환된 값을 보여주는 앱이다.

#### 문제 상황: 상태 끌어올리기(Lifting State Up)

가장 먼저 떠오르는 방법은 상태를 부모로 끌어올리는 것이다.

```jsx
function Input({ value, handleChange }) {
  return <input value={value} onChange={(e) => handleChange(e.target.value)} />
}

export default function App() {
  const [value, setValue] = useState("")

  return (
    <div className="App">
      <h1>Temperature Converter</h1>
      <Input value={value} handleChange={setValue} />
      <Kelvin value={value} />
      <Fahrenheit value={value} />
    </div>
  )
}
```

작동은 하지만, 앱의 규모가 커지면 문제가 생긴다.  
상태를 최상위로 끌어올릴수록 **상태 변경 시 모든 자식 컴포넌트가 리렌더링**되어 성능에 영향을 줄 수 있다.

#### 해결: Render Props 적용

이때 Render Props 패턴을 적용하면, `Input` 컴포넌트가 상태를 직접 갖되 **무엇을 렌더링할지는 외부에서 결정**하도록 위임할 수 있다.

```jsx
function Input(props) {
  const [value, setValue] = useState("")

  return (
    <>
      <input
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Temp in °C"
      />
      {props.render(value)}
    </>
  )
}

export default function App() {
  return (
    <div className="App">
      <h1>Temperature Converter</h1>
      <Input
        render={(value) => (
          <>
            <Kelvin value={value} />
            <Fahrenheit value={value} />
          </>
        )}
      />
    </div>
  )
}
```

`Input`은 상태(`value`)를 관리하면서, `props.render(value)`를 통해 그 값을 외부에 전달한다.  
외부에서는 그 값을 받아 `Kelvin`과 `Fahrenheit`를 렌더링할지, 아니면 전혀 다른 컴포넌트를 그릴지 자유롭게 결정할 수 있다.

### Children을 함수로 전달하기

`render`라는 이름의 prop 대신, **자식 엘리먼트 자체를 함수로 전달**하는 방법도 있다.

```jsx
export default function App() {
  return (
    <div className="App">
      <h1>Temperature Converter</h1>
      <Input>
        {(value) => (
          <>
            <Kelvin value={value} />
            <Fahrenheit value={value} />
          </>
        )}
      </Input>
    </div>
  )
}
```

```jsx
function Input(props) {
  const [value, setValue] = useState("")

  return (
    <>
      <input
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Temp in °C"
      />
      {props.children(value)}
    </>
  )
}
```

`props.render(value)` 대신 `props.children(value)`를 호출하는 것 외에는 동일하다.  
이 방식을 사용하면 JSX 구조가 좀 더 자연스럽게 읽히는 장점이 있다.

### Render Props의 중첩 문제

Render Props는 여러 개를 사용하게 되면 **콜백 지옥**과 비슷한 구조가 만들어질 수 있다.

Apollo Client를 예로 들면, 여러 Mutation을 동시에 사용해야 할 때 다음과 같은 코드가 만들어진다.

```jsx
<Mutation mutation={FIRST_MUTATION}>
  {(firstMutation) => (
    <Mutation mutation={SECOND_MUTATION}>
      {(secondMutation) => (
        <Mutation mutation={THIRD_MUTATION}>
          {(thirdMutation) => (
            <Element
              firstMutation={firstMutation}
              secondMutation={secondMutation}
              thirdMutation={thirdMutation}
            />
          )}
        </Mutation>
      )}
    </Mutation>
  )}
</Mutation>
```

depth가 깊어질수록 코드를 읽기 어려워지고, 유지보수도 힘들어진다.

## [How] Render Props Pattern은 어떻게 구현하는가?

핵심은 **데이터를 가진 컴포넌트가 UI를 직접 그리지 않고, 함수 prop을 통해 데이터를 전달하며 렌더링을 위임하는 것**이다.

### 기본 구조

```jsx
// 데이터를 관리하는 컴포넌트 (Provider 역할)
function DataProvider({ render }) {
  const [data, setData] = useState(null)

  useEffect(() => {
    fetchData().then(setData)
  }, [])

  return <>{render(data)}</>
}

// 사용하는 쪽에서 렌더링 방식을 결정
<DataProvider
  render={(data) => (
    data ? <DataView data={data} /> : <Loading />
  )}
/>
```

### children 함수 방식

```jsx
function DataProvider({ children }) {
  const [data, setData] = useState(null)

  useEffect(() => {
    fetchData().then(setData)
  }, [])

  return <>{children(data)}</>
}

<DataProvider>
  {(data) => (
    data ? <DataView data={data} /> : <Loading />
  )}
</DataProvider>
```

두 방식 모두 동일한 결과를 만들어내며, 팀의 컨벤션이나 가독성에 따라 선택하면 된다.

## [Why] Render Props Pattern은 왜 사용하는가?

Q. 언제 사용하면 좋을까?
- 핵심은 **여러 컴포넌트 간에 동일한 데이터나 로직을 공유하되, 각 컴포넌트가 그 데이터를 어떻게 렌더링할지는 독립적으로 결정하고 싶을 때.**

Q. 왜 Render Props를 쓰는가?

1. **prop 출처의 명확성**
    - HOC(Higher-Order Component)와 달리 prop이 어디에서 오는지 명시적으로 알 수 있다. HOC는 자동으로 prop을 병합하기 때문에 "이 prop이 어디에서 온 건지" 추적이 어려운 반면, Render Props는 함수의 인자로 직접 전달받기 때문에 출처가 분명하다.

2. **관심사의 분리**
    - 데이터를 관리하는 컴포넌트와 데이터를 표현하는 컴포넌트를 깔끔하게 분리할 수 있다.

3. **재사용성**
    - 동일한 데이터 소스에 대해 완전히 다른 UI를 그려야 하는 상황에서, 데이터 컴포넌트를 재사용하면서 렌더링만 바꿀 수 있다.

Q. Render Props의 단점은 무엇인가?

- **콜백 지옥**: 여러 Render Props를 중첩해서 사용하면 코드의 depth가 깊어지고 가독성이 크게 떨어진다.
- **생명주기 함수 사용 불가**: render prop 함수 내부에서는 React의 생명주기 메서드를 사용할 수 없다. 받은 데이터를 그대로 렌더링하는 것만 가능하다.