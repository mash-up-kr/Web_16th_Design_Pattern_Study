## Provider 패턴이란?
- React의 **Context API**를 활용하여 컴포넌트 트리 전체에 데이터를 공유하는 패턴
- Prop Drilling 없이 필요한 컴포넌트가 직접 데이터에 접근할 수 있게 한다

## Prop Drilling 문제

- 데이터를 하위 컴포넌트에 전달하려면 중간 컴포넌트들이 전부 props를 넘겨줘야 한다
- 데이터가 필요 없는 컴포넌트도 props를 받아서 전달해야 하므로 불필요한 의존성이 생긴다
- prop 이름을 변경하면 모든 중간 컴포넌트를 수정해야 한다

```javascript
function App() {
  const data = { listItem: "item", title: "Title", text: "Text" };

  return (
    <div>
      <SideBar data={data} />
      <Content data={data} />
    </div>
  );
}

const SideBar = ({ data }) => <List data={data} />;
const List = ({ data }) => <ListItem data={data} />;
const ListItem = ({ data }) => <span>{data.listItem}</span>;

const Content = ({ data }) => (
  <div>
    <Header data={data} />
    <Block data={data} />
  </div>
);
const Header = ({ data }) => <div>{data.title}</div>;
const Block = ({ data }) => <Text data={data} />;
const Text = ({ data }) => <h1>{data.text}</h1>;
```

- `SideBar`, `List`, `Content`, `Block` 등은 data를 직접 사용하지 않지만 자식에게 넘기기 위해 props로 받아야 한다

## Provider 패턴 구현

### 1. Context 생성 및 Provider로 감싸기

```javascript
const DataContext = React.createContext();

function App() {
  const data = { listItem: "item", title: "Title", text: "Text" };

  return (
    <div>
      <DataContext.Provider value={data}>
        <SideBar />
        <Content />
      </DataContext.Provider>
    </div>
  );
}
```

### 2. useContext 훅으로 데이터 접근

```javascript
// 중간 컴포넌트는 더 이상 props를 전달하지 않는다
const SideBar = () => <List />;
const List = () => <ListItem />;
const Content = () => (
  <div>
    <Header />
    <Block />
  </div>
);
const Block = () => <Text />;

// 데이터가 필요한 컴포넌트만 useContext로 직접 접근
function ListItem() {
  const data = React.useContext(DataContext);
  return <span>{data.listItem}</span>;
}

function Header() {
  const data = React.useContext(DataContext);
  return <div>{data.title}</div>;
}

function Text() {
  const data = React.useContext(DataContext);
  return <h1>{data.text}</h1>;
}
```

## 실제 예시: 테마 전환

### Context 및 Provider 설정

```javascript
export const ThemeContext = React.createContext();

const themes = {
  light: { background: "#fff", color: "#000" },
  dark: { background: "#171717", color: "#fff" },
};

export default function App() {
  const [theme, setTheme] = useState("dark");

  function toggleTheme() {
    setTheme(theme === "light" ? "dark" : "light");
  }

  return (
    <div className={`App theme-${theme}`}>
      <ThemeContext.Provider value={{ theme: themes[theme], toggleTheme }}>
        <Toggle />
        <List />
      </ThemeContext.Provider>
    </div>
  );
}
```

### 컴포넌트에서 Context 사용

```javascript
import { useContext } from "react";
import { ThemeContext } from "./App";

export default function Toggle() {
  const { toggleTheme } = useContext(ThemeContext);

  return (
    <label className="switch">
      <input type="checkbox" onClick={toggleTheme} />
      <span className="slider round" />
    </label>
  );
}

export default function ListItem() {
  const { theme } = useContext(ThemeContext);
  return <li style={theme}>Lorem ipsum...</li>;
}
```

## Custom Hook + Provider 분리

- Context 로직을 커스텀 훅과 Provider 컴포넌트로 분리하면 재사용성과 안전성이 높아진다

### Provider 컴포넌트 분리

```javascript
const ThemeContext = React.createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("dark");

  function toggleTheme() {
    setTheme(theme === "light" ? "dark" : "light");
  }

  return (
    <ThemeContext.Provider value={{ theme: themes[theme], toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

### Custom Hook으로 안전하게 접근

```javascript
function useThemeContext() {
  const theme = useContext(ThemeContext);
  if (!theme) {
    throw new Error("useThemeContext must be used within ThemeProvider");
  }
  return theme;
}
```

### 사용

```javascript
// App — Provider로 감싸기만 하면 됨
export default function App() {
  return (
    <div className="App">
      <ThemeProvider>
        <Toggle />
        <List />
      </ThemeProvider>
    </div>
  );
}

// 컴포넌트 — 커스텀 훅으로 간결하게 접근
function ListItem() {
  const { theme } = useThemeContext();
  return <li style={theme}>...</li>;
}
```

## styled-components의 ThemeProvider

- styled-components도 내부적으로 Provider 패턴을 사용한다
- 아래는 emotion 라이브러리 예시

```javascript
// ThemeProvider로 최상위에서 감싸기
import { Provider } from 'react-redux';
import { Global, ThemeProvider } from '@emotion/react';
import theme from '@/styles/theme';

export default function App({ Component, pageProps }: AppProps) {
  return (
    <ThemeProvider {...{ theme }}>
      <Global styles={globalStyle} />
      <AppStyle>
        <Provider store={store}>
          <Component {...pageProps} />
        </Provider>
      </AppStyle>
    </ThemeProvider>
  );
}

// useTheme 훅으로 theme 사용
import { useTheme } from '@emotion/react';
import styled from '@emotion/styled';

export default function DetailBackIcon() {
  const { color: themeColor } = useTheme();
  const { gray400 } = themeColor;
  ...
  return (
    <Cancel onClick={onCancelDetail}>
      <IoIosArrowBack color={gray400} size={25} />
    </Cancel>
  );
}

// 혹은 아래와 같은 사용도 가능
const Wrapper = styled.div`
  width: 50px;
  height: 30px;
  display: inline-block;
  overflow: hidden;
  background: ${({ theme }) => theme.color.white};
`;
```

## 장점
- Prop Drilling을 제거하여 **불필요한 props 전달 없이** 필요한 컴포넌트만 데이터에 접근할 수 있다
- 리팩토링이 용이하다 — prop 이름 변경 시 중간 컴포넌트를 모두 수정할 필요가 없다
- 데이터 흐름이 명확하다 — 어떤 컴포넌트가 어떤 데이터를 사용하는지 바로 알 수 있다

## 단점: 성능 이슈

- Context의 value가 변경되면, 해당 Context를 구독하는 **모든 컴포넌트가 리렌더링**된다

```javascript
const CountContext = createContext(null);

function CountProvider({ children }) {
  const [count, setCount] = useState(0);
  return (
    <CountContext.Provider value={{ count, setCount }}>
      {children}
    </CountContext.Provider>
  );
}

function useCountContext() {
  const context = useContext(CountContext);
  if (!context) {
    throw new Error("useCountContext must be within CountProvider");
  }
  return context;
}

// count를 보여주는 컴포넌트
function Button() {
  const { count, setCount } = useCountContext();
  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}

// count를 사용하지 않지만, setCount 때문에 Context를 구독
function Reset() {
  const { setCount } = useCountContext();
  return <button onClick={() => setCount(0)}>Reset</button>;
}
```

- `Button`에서 count를 증가시키면 `Reset` 컴포넌트도 같은 Context를 구독하고 있기 때문에 **불필요하게 리렌더링**된다
- 해결책: Context를 역할별로 분리하여 관련 없는 컴포넌트의 리렌더링을 방지한다

### Redux는 어떻게 리렌더링을 방지하는가

- Redux도 `<Provider>`로 감싸지만, 내부적으로 Context가 아닌 **subscribe 기반 구독**으로 리렌더링을 제어한다
- `useSelector`는 store가 변경될 때마다 selector의 반환값을 이전 값과 비교(===)하고, **실제로 값이 바뀐 컴포넌트만** 리렌더링한다

```javascript
import { createStore } from "redux";
import { Provider, useSelector, useDispatch } from "react-redux";

// action
const INCREMENT = "INCREMENT";
const RESET = "RESET";

const increment = (payload) => ({ type: INCREMENT, payload });
const reset = () => ({ type: RESET });

// reducer
const initialState = { count: 0, name: "선민" };

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case INCREMENT:
      return { ...state, count: state.count + action.payload };
    case RESET:
      return { ...state, count: 0 };
    default:
      return state;
  }
}

const store = createStore(counterReducer);

function App() {
  return (
    <Provider store={store}>
      <Counter />
      <ResetButton />
      <Name />
    </Provider>
  );
}

// count만 구독 — count가 바뀔 때만 리렌더링
function Counter() {
  const count = useSelector((state) => state.count);
  const dispatch = useDispatch();
  return <button onClick={() => dispatch(increment(1))}>Count: {count}</button>;
}

// dispatch만 사용, 상태를 구독하지 않음 — 리렌더링 안 됨
function ResetButton() {
  const dispatch = useDispatch();
  return <button onClick={() => dispatch(reset())}>Reset</button>;
}

// name만 구독 — count가 바뀌어도 리렌더링 안 됨
function Name() {
  const name = useSelector((state) => state.name);
  return <div>{name}</div>;
}
```

**동작 방식:**
1. `store.dispatch(action)` → reducer 실행 → 새로운 state 생성
2. store가 내부 listener들에게 변경을 알림 (`store.subscribe`)
3. 각 `useSelector`가 selector를 다시 실행하고 이전 반환값과 `===` 비교
4. 값이 **실제로 바뀐 컴포넌트만** 리렌더링

Context와의 핵심 차이: Context는 value 객체의 참조가 바뀌면 구독자 전체가 리렌더링되지만, Redux는 selector 반환값 단위로 비교하므로 관련 없는 컴포넌트는 리렌더링되지 않는다.

### Recoil은 어떻게 리렌더링을 방지하는가

- Recoil은 상태를 **atom 단위**로 쪼개고, 각 atom을 구독하는 컴포넌트만 리렌더링한다
- 하나의 거대한 store가 아니라, 독립적인 atom들의 그래프로 상태를 관리한다

```javascript
import { RecoilRoot, atom, useRecoilState, useRecoilValue } from "recoil";

// 상태를 atom 단위로 분리
const countState = atom({ key: "count", default: 0 });
const nameState = atom({ key: "name", default: "선민" });

function App() {
  return (
    <RecoilRoot>
      <Counter />
      <ResetButton />
      <Name />
    </RecoilRoot>
  );
}

// countState만 구독 — count가 바뀔 때만 리렌더링
function Counter() {
  const [count, setCount] = useRecoilState(countState);
  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}

// countState의 setter만 사용 — useSetRecoilState는 값을 구독하지 않으므로 리렌더링 안 됨
function ResetButton() {
  const setCount = useSetRecoilState(countState);
  return <button onClick={() => setCount(0)}>Reset</button>;
}

// nameState만 구독 — count가 바뀌어도 리렌더링 안 됨
function Name() {
  const name = useRecoilValue(nameState);
  return <div>{name}</div>;
}
```

**동작 방식:**
1. `setCount(newValue)` → 해당 atom의 값이 변경됨
2. Recoil이 해당 atom을 구독하는 컴포넌트 목록을 확인
3. **해당 atom을 구독하는 컴포넌트만** 리렌더링

Context와의 핵심 차이: Context는 하나의 value 객체에 여러 상태를 담기 때문에 일부만 바뀌어도 전체가 리렌더링되지만, Recoil은 atom 단위로 구독이 분리되어 있어 관련 없는 컴포넌트는 영향을 받지 않는다.

### Context vs Redux vs Recoil 리렌더링 비교

| | Context | Redux | Recoil |
|---|---|---|---|
| 구독 단위 | Provider의 value 전체 | selector 반환값 | atom 단위 |
| count 변경 시 Reset 리렌더링 | O (같은 Context 구독) | X (count를 구독 안 함) | X (다른 atom / setter만 사용) |
| count 변경 시 Name 리렌더링 | O (같은 Context 구독) | X (name만 구독) | X (다른 atom 구독) |
| 최적화 방식 | Context 분리 (수동) | selector === 비교 (자동) | atom 그래프 구독 (자동) |

## 참고자료
- https://patterns-dev-kr.github.io/design-patterns/provider-pattern/
