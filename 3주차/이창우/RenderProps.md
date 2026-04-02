# 1. 렌더 프롭스(Render Props) 패턴

## 1-1. 정의

컴포넌트가 직접 UI를 고정해서 렌더링하지 않고, **렌더링 결과를 반환하는 함수를 prop으로 받아** 그 결과를 사용하는 패턴

* 로직 제공 — 상태 관리나 공통 동작은 상위 컴포넌트가 담당
* UI 위임 — 실제 화면은 함수 prop을 넘긴 쪽이 결정
* 재사용성 확보 — 같은 로직을 여러 UI에서 공유 가능

핵심은 **로직을 가진 컴포넌트가 렌더링 제어권을 함수 prop으로 위임한다**는 점

```tsx
<MyComponent render={(data) => <div>{data}</div>} />

<MyComponent>
  {(data) => <div>{data}</div>}
</MyComponent>
```

`render`라는 이름의 prop을 써도 되고, `children`에 함수를 넣는 방식으로 써도 됨

## 1-2. 필요성

어떤 로직을 여러 컴포넌트가 공통으로 써야 하는 경우가 자주 있음

ex) 토글 관리, 폼 상태 관리, 드래그 추적, 마우스 위치 추적

문제는 **로직은 같지만 UI는 다를 수 있다**는 것

같은 상태 관리 로직을 매번 복붙하면 중복이 생기고, 하나의 컴포넌트가 UI까지 고정해버리면 재사용성이 떨어짐  
⇒ 그래서 **로직은 공통 컴포넌트에 두고, UI는 함수로 주입받는 구조**가 필요

## 1-3. 동작 원리

1. 상위 컴포넌트가 상태나 공통 로직을 가짐
2. 그 값을 인자로 넘겨줄 함수 prop(`render` 또는 `children`)을 받음
3. 내부에서는 직접 UI를 만들지 않고, 그 함수를 호출함
4. 사용하는 쪽은 같은 로직을 기반으로 원하는 UI를 자유롭게 렌더링함

```tsx
type MousePosition = {
  x: number;
  y: number;
};

type MouseTrackerProps = {
  render: (position: MousePosition) => React.ReactNode;
};

function MouseTracker({ render }: MouseTrackerProps) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e: React.MouseEvent<HTMLDivElement>) => {
    setPosition({
      x: e.clientX,
      y: e.clientY,
    });
  };

  return (
    <div
      onMouseMove={handleMouseMove}
      style={{ width: "100vw", height: "100vh" }}
    >
      {render(position)}
    </div>
  );
}
```

`MouseTracker`는 마우스 위치를 추적하는 로직만 갖고 있고, 실제로 무엇을 보여줄지는 `render(position)`가 결정

## 1-4. 실제 활용 예시

같은 `MouseTracker`를 사용해도 UI는 완전히 다르게 만들 수 있음

```tsx
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <h1>
          마우스 위치: ({x}, {y})
        </h1>
      )}
    />
  );
}
```

```tsx
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div
          style={{
            position: "absolute",
            left: x,
            top: y,
            width: 20,
            height: 20,
            backgroundColor: "red",
            borderRadius: "50%",
          }}
        />
      )}
    />
  );
}
```

같은 로직이지만, 한쪽은 텍스트 UI, 다른 한쪽은 움직이는 원 UI를 그림  
⇒ **로직 재사용 + UI 유연성** 확보

`children as function` 형태로도 동일하게 구현 가능

```tsx
type MouseTrackerProps = {
  children: (position: MousePosition) => React.ReactNode;
};

function MouseTracker({ children }: MouseTrackerProps) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e: React.MouseEvent<HTMLDivElement>) => {
    setPosition({
      x: e.clientX,
      y: e.clientY,
    });
  };

  return (
    <div
      onMouseMove={handleMouseMove}
      style={{ width: "100vw", height: "100vh" }}
    >
      {children(position)}
    </div>
  );
}
```

## 1-5. 기대 효과

**얻는 것**

* 로직 재사용 — 상태 관리 로직을 한 곳에 모을 수 있음
* UI 유연성 — 같은 로직으로 여러 형태의 UI를 만들 수 있음
* 관심사 분리 — 상태/행동과 화면 렌더링을 나눌 수 있음

## 1-6. 트레이드오프

**잃는 것**

* JSX 중첩 증가 — 여러 render prop이 겹치면 구조가 깊어짐
* 함수 생성 비용 — 부모가 렌더링될 때마다 새 함수가 만들어질 수 있음
* 데이터 흐름 가독성 저하 — UI가 함수 내부에서 동적으로 만들어져 직관성이 떨어질 수 있음

```tsx
<AuthProvider
  render={(auth) => (
    <ThemeProvider
      render={(theme) => (
        <MouseTracker
          render={(mouse) => (
            <Dashboard auth={auth} theme={theme} mouse={mouse} />
          )}
        />
      )}
    />
  )}
/>
```

이렇게 중첩되면 마치 콜백 지옥처럼 보일 수 있음

## 1-7. Hooks와의 비교

Hooks 등장 이후에는 비슷한 문제를 커스텀 훅으로 더 단순하게 푸는 경우가 많아짐

```tsx
function useToggle(initialValue = false) {
  const [isOn, setIsOn] = useState(initialValue);

  const toggle = () => setIsOn((prev) => !prev);

  return { isOn, toggle };
}

function App() {
  const { isOn, toggle } = useToggle();

  return <button onClick={toggle}>{isOn ? "ON" : "OFF"}</button>;
}
```

커스텀 훅은 **로직 제공 함수**, 렌더 프롭스는 **로직 제공 컴포넌트**라는 차이가 있음

* 렌더 프롭스 — 로직을 가진 컴포넌트가 상태를 제공하고, UI는 prop으로 결정
* 커스텀 훅 — 로직을 함수로 분리하고, 컴포넌트가 직접 상태를 받아 렌더

그래서 요즘 React에서는 커스텀 훅이 더 자주 선호되지만, **렌더링 제어권을 함수로 위임하는 API가 더 자연스러운 경우**에는 여전히 render props가 유효

## 1-8. 공부 중 의문

### 그냥 함수를 넘기는 props와 뭐가 다른가?

일반 함수 prop과 렌더 프롭스의 차이는, **그 함수가 렌더링할 UI를 반환하는가**에 있음

즉 단순히 이벤트 핸들러를 넘기는 것이 아니라,

* 이 prop이 렌더링 결과를 반환하는가?
* 이 함수가 UI를 구성하는 역할을 하는가?

이 질문에 해당하면 render props 패턴으로 볼 수 있음
