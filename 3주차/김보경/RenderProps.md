# Render Props 패턴
- 컴포넌트를 재사용 가능하게 만드는 패턴중 하나입니다.
- render prop은 컴포넌트의 prop으로 함수이며 JSX Element를 리턴합니다. 컴포넌트 자체는 아루것도 렌더링하지 않지만 render prop함수를 호출합니다.

```jsx
// Title 컴포넌트는 prop으로 넘어온 함수를 호출하여 반환하는 것 외에는 아무런 동작을 하지 않습니다.
// Title 컴포넌트에 render prop을 아래와 같이 넣어보겠습니다.
<Title render={() => <h1>I am a render prop!</h1>} />

// Title 컴포넌트 내에서는 단순히 prop의 render함수를 호출하여 반환합니다.
const Title = props => props.render()

// 컴포넌트 요소에 React 요소를 반환하는 render라는 이름의 prop을 넘깁니다.
import React from "react";
import { render } from "react-dom";

import "./styles.css";

const Title = (props) => props.render();

render(
  <div className="App">
    <Title
      render={() => (
        <h1>
          <span role="img" aria-label="emoji">
            ✨
          </span>
          I am a render prop!{" "}
          <span role="img" aria-label="emoji">
            ✨
          </span>
        </h1>
      )}
    />
  </div>,
  document.getElementById("root")
);


// render prop 패턴의 장점은 prop을 받는 컴포넌트가 재사용성이 좋다는 점 입니다. Title 컴포넌트는 인제 render prop만 바꿔가며 여러번 재사용할 수 있습니다.
import React from "react";
import { render } from "react-dom";
import "./styles.css";

const Title = (props) => props.render();

render(
  <div className="App">
    <Title render={() => <h1>✨ First render prop! ✨</h1>} />
    <Title render={() => <h2>🔥 Second render prop! 🔥</h2>} />
    <Title render={() => <h3>🚀 Third render prop! 🚀</h3>} />
  </div>,
  document.getElementById("root")
);
```

## 상태를 부모 컴포넌트로 올리기
- 좋지 않은 사용 예시
  ```jsx
  function Input({ value, handleChange }) {
    return <input value={value} onChange={e => handleChange(e.target.value)} />
  }

  export default function App() {
    const [value, setValue] = useState('')

    return (
      <div className="App">
        <h1>☃️ Temperature Converter 🌞</h1>
        <Input value={value} handleChange={setValue} />
        <Kelvin value={value} />
        <Fahrenheit value={value} />
      </div>
    )
  }
  ```
  - 여러 자식 컴포넌트를 가지고 있는 경우 상태의 변경으로 인해 모든 자식 컴포넌트의 리렌더링을 유발할 수 있고 이런 상황이 쌓여 앱의 전체적인 성능을 떨어트릴 수 있습니다.

- Render props 사용 예시
  ```jsx
  function Input(props) {
    const [value, setValue] = useState('')

    return (
      <>
        <input
          type="text"
          value={value}
          onChange={e => setValue(e.target.value)}
          placeholder="Temp in °C"
        />
        {props.render(value)}
      </>
    )
  }

  export default function App() {
    return (
      <div className="App">
        <h1>☃️ Temperature Converter 🌞</h1>
        <Input
          render={value => (
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
  - ender props 패턴을 활용하여 Input 컴포넌트에서 render prop을 받을 수 있도록 리펙터링 해보았습니다.
  - 이로써 Kelvin과 Fahrenheight 컴포넌트는 사용자의 입력 값을 받을 수 있게 되었습니다.

## 자식 컴포넌트를 함수로 받아보자
```jsx
function Input(props) {
  const [value, setValue] = useState('')

  return (
    <>
      <input
        type="text"
        value={value}
        onChange={e => setValue(e.target.value)}
        placeholder="Temp in °C"
      />
      {props.children(value)}
    </>
  )
}


// 사용처 예시
export default function App() {
  return (
    <div className="App">
      <h1>☃️ Temperature Converter 🌞</h1>
      <Input>
        {value => (
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
- headless UI 라이브러리에서 자주 사용되던 패턴으로 보이네요

## 장점
- render prop을 사용하여 몇몇 컴포넌트간 데이터를 공유하는 것은 간단합니다. render 혹은 children prop을 활용하는 것으로 해당 컴포넌트를 재사용할 수 있게 됩니다.
- HOC 패턴을 사용할 때 마주칠 수 있는 몇 가지 이슈을 또한 해결 가능합니다.
  - HOC 패턴을 사용할 때 prop이 어디에서 만들어져 오는지 구별하기 힘들었던 이슈 개선
    - 왜 why? 부모 컴포넌트로부터 받은 prop을 명시적으로 받아 처리하기 때문
  - 함수의 인자에서 명시적으로 prop이 전달되기 때문에 HOC를 사용할 때 prop이 모호한 문제가 해결됩니다. 이 때문에 prop이 어디로부터 오는지 확실히 알 수 있습니다.
  - render props를 활용하여 렌더링 컴포넌트와 앱의 로직을 분리할 수 있습니다.
    - 상태를 가진 컴포넌트는 render prop을 받고, 상태가 없는 컴포넌트를 렌더할 수 있습니다.

## 단점
- render props로 해결하려 했던 문제들은 이미 React Hooks로 대체되었습니다.
- render prop 내에서는 생명 주기 함수를 사용할 수 없기 때문에 render  prop 패턴은 받은 데이터를 수정할 필요가 없는 컴포넌트들에 대하여 사용할 수 있습니다.

> 사실 내용을 보아도 메모이제이션으로 해결 가능할것 처럼 보이네요 🤔

## 추가 정리

## Render Props 패턴
- 컴포넌트가 children을 렌더링하는 대신, props로 전달된 함수를 호출하여 UI를 생성합니다.

### 함수를 호출하여 UI를 생성
```tsx
// 로직을 가진 부모 컴포넌트
const MouseTracker = ({ render }) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };

  return <div onMouseMove={handleMouseMove}>{render(position)}</div>;
};

// 사용 예시
<MouseTracker render={({ x, y }) => (
  <h1>마우스 위치: {x}, {y}</h1>
)} />
```
- MouseTracker는 마우스 위치를 추적하는 로직만 책임지고, UI는 render props로 전달된 함수가 담당
- 그냥 마우스 위치 바로 주입하면 되는거 아님?
  - 라고 생각할 수 있는데 이걸 굳이 나누는 이유는 "관심사 분리"가 핵심 포인트 입니다.
  - 정적 UI 요소가 아닌 다이나믹한 동적으로 변경되는 렌더에 주로 사용

### Render Props의 장점
- 관심사 분리: 로직과 UI를 분리하여 컴포넌트를 더 유연하게 재사용 가능해집니다.
- 명시적 제어: HOC보다 직관적으로 데이터 흐름을 추적할 수 있습니다.
- 동적 UI: 런타임 상황에 렌더링 방식을 변경할 수 있습니다.

#### 변형 패턴 (children을 함수로 적용)
```tsx
<MouseTracker>
  {({ x, y }) => <p>{x}, {y}</p>}
</MouseTracker>
```
- 두 방식의 트레이드 오프를 말해본다면 변형 패턴에서는 관계없는 UI도 추가될 수 있기 때문에 확장성이 더 있지만, 추천하는 방식은 아님
- 동적으로 변경되는 UI를 명시적으로 보호하기 위해서는 render prop 으로 전달하는게 더 좋다!