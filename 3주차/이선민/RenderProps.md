## Render Props 패턴이란?

- **JSX 엘리먼트를 반환하는 함수를 컴포넌트의 prop으로 전달**하는 패턴
- 컴포넌트 자체는 아무것도 렌더링하지 않고, 전달받은 함수를 호출하여 그 결과를 렌더링한다
- 상태 로직과 렌더링 로직을 분리하여 재사용성을 높인다

```javascript
// 사용 방법
<Title render={() => <h1>I am a render prop!</h1>} />

// 컴포넌트 구현
const Title = props => props.render()
```

## 데이터 전달

패턴의 진정한 가치는 **render prop 함수의 인자로 데이터를 전달**할 때 나타난다

```javascript
// 컴포넌트: 내부 상태를 인자로 전달
function Component(props) {
  const data = { ... }
  return props.render(data)
}

// 사용: 인자로 데이터 수신
<Component render={data => <ChildComponent data={data} />} />
```

## 실제 예시: 온도 변환기

### 문제 상황

- 온도 입력을 받는 `Input` 컴포넌트와, 이를 섭씨/화씨로 변환해서 보여주는 `Kelvin`, `Fahrenheit` 컴포넌트가 형제 관계일 때 상태 공유가 어렵다

### Render Props 해결책

```javascript
function Input(props) {
  const [value, setValue] = useState('')

  return (
    <>
      <input
        value={value}
        onChange={e => setValue(e.target.value)}
        placeholder="Temp in °C"
      />
      {props.render(value)}
    </>
  )
}

// Input이 value를 관리하고, 렌더링은 외부에서 결정
<Input
  render={value => (
    <>
      <Kelvin value={value} />
      <Fahrenheit value={value} />
    </>
  )}
/>
```

- `Input`은 상태 관리만 담당하고, 어떤 컴포넌트를 렌더링할지는 사용하는 쪽에서 결정한다

## Children Props 방식

- render prop 이름을 `render`로 지정하지 않고 **`children`에 함수를 전달**하는 방식도 있다

```javascript
<Input>
  {value => (
    <>
      <Kelvin value={value} />
      <Fahrenheit value={value} />
    </>
  )}
</Input>

// 컴포넌트 내부
{props.children(value)}
```

## HOC와의 비교

- HOC 패턴은 props가 어디서 왔는지 불명확해지는 문제가 있다
- Render Props는 함수 인자로 데이터를 받기 때문에 **props의 출처가 명확**하다

```javascript
// HOC — props가 자동으로 병합되어 출처 불명확
function withHover(Component) {
  return function(props) {
    const [hovering, setHovering] = useState(false)
    return (
      <Component
        {...props}
        hovering={hovering}    // 어디서 왔는지 Component 내부에서 알기 어렵다
        onMouseOver={...}
        onMouseOut={...}
      />
    )
  }
}

// Render Props — 인자로 명시적으로 전달
function Hover(props) {
  const [hovering, setHovering] = useState(false)
  return (
    <div onMouseOver={...} onMouseOut={...}>
      {props.render(hovering)}   // hovering이 어디서 오는지 명확하다
    </div>
  )
}
```

| | Render Props | HOC |
|---|---|---|
| props 출처 | 함수 인자로 명시적 | 자동 병합되어 불명확 |
| props 충돌 | 없음 | 이름 충돌 가능 |
| 구조 | 중첩 가능 | Wrapper hell 위험 |

## 장점

- **명시성**: render 함수의 인자로 데이터를 받기 때문에 어떤 데이터가 어디서 오는지 명확하다
- **재사용성**: 상태 로직을 가진 컴포넌트를 여러 곳에서 다른 UI로 재사용할 수 있다
- **HOC 문제 해결**: prop 이름 충돌이나 출처 불명확 문제가 없다

## 단점

### 깊은 중첩 (Callback Hell)

- 여러 render prop이 필요한 경우 콜백 지옥과 비슷한 중첩 구조가 만들어진다

```javascript
<Mutation mutation={FIRST}>
  {first => (
    <Mutation mutation={SECOND}>
      {second => (
        <Mutation mutation={THIRD}>
          {third => <Element />}
        </Mutation>
      )}
    </Mutation>
  )}
</Mutation>
```

### 생명주기 제약

- render prop 함수 내부에서는 생명주기 메서드(`componentDidMount` 등)를 사용할 수 없다

## Hooks로의 대체

- React Hooks 등장 이후 Render Props의 대부분의 사용 사례가 **Custom Hook**으로 대체되었다

```javascript
// 과거 Render Props 방식 (Apollo Client 예시)
<Query query={GET_DATA}>
  {({ data, loading }) => (
    loading ? <Spinner /> : <Component data={data} />
  )}
</Query>

// 현대 Hooks 방식
function Component() {
  const { data, loading } = useQuery(GET_DATA)
  return loading ? <Spinner /> : <DataComponent data={data} />
}
```

- Hooks는 중첩 없이 여러 상태 로직을 조합할 수 있어 더 간결하다
- 다만 특정 상황(예: 부모가 자식의 렌더링을 동적으로 제어해야 할 때)에서는 Render Props가 여전히 유용하다

## 참고자료

- https://patterns-dev-kr.github.io/design-patterns/render-props-pattern/
