# [Provider Pattern](https://patterns-dev-kr.github.io/design-patterns/provider-pattern/)

## [What] Provider Pattern이란?

> "데이터를 필요로 하는 컴포넌트에게 중간 단계를 거치지 않고 직접 공급하는 구조적 패턴" _(claude)_

> Provider 패턴은 앱 내의 여러 컴포넌트들에게 데이터를 전달해야 할 때, props를 통해 일일이 내려주는 대신 Context를 사용하여 컴포넌트 트리 전체에 데이터를 공급하는 방식이다. _(patterns.dev)_

## [Patterns.dev] Provider Pattern 예시로 알아보기

### Prop Drilling의 문제

아래와 같은 컴포넌트 트리가 있다고 해보자.

```jsx
function App() {
  const data = { ... }

  return (
    <div>
      <SideBar data={data} />
      <Content data={data} />
    </div>
  )
}

const SideBar = ({ data }) => <List data={data} />
const List = ({ data }) => <ListItem data={data} />
const ListItem = ({ data }) => <span>{data.listItem}</span>

const Content = ({ data }) => (
  <div>
    <Header data={data} />
    <Block data={data} />
  </div>
)
const Header = ({ data }) => <div>{data.title}</div>
const Block = ({ data }) => <Text data={data} />
const Text = ({ data }) => <h1>{data.text}</h1>
```

`App`에서 `data`를 사용해야 하는 컴포넌트까지 모든 중간 컴포넌트를 거쳐 `props`로 전달하고 있다.
이것이 바로 **Prop Drilling**이다.

`SideBar`나 `Content`, `Block` 같은 중간 컴포넌트들은 `data`를 **직접 사용하지 않음에도 불구하고**, 단지 하위 컴포넌트에 전달하기 위해 `data` prop을 받고 있다. 이런 구조에서는:

- `data`의 속성명을 변경하면 모든 컴포넌트를 수정해야 한다.
- 중간 컴포넌트들이 불필요한 props에 의존하게 된다.
- 컴포넌트 트리가 깊어질수록 유지보수가 어려워진다.

### Provider Pattern 적용

React의 `createContext`를 활용하면, 중간 컴포넌트를 거치지 않고 데이터가 필요한 컴포넌트에서 직접 접근할 수 있다.

```jsx
const DataContext = React.createContext()

function App() {
  const data = { ... }

  return (
    <div>
      <DataContext.Provider value={data}>
        <SideBar />
        <Content />
      </DataContext.Provider>
    </div>
  )
}
```

`DataContext.Provider`로 컴포넌트 트리를 감싸고, `value` prop에 공유할 데이터를 넘긴다.
이제 중간 컴포넌트들은 더 이상 `data` prop을 받을 필요가 없다.

```jsx
const SideBar = () => <List />
const List = () => <ListItem />
const Content = () => <div><Header /><Block /></div>
const Block = () => <Text />
```

데이터가 필요한 컴포넌트에서는 `useContext` 훅으로 직접 접근한다.

```jsx
function ListItem() {
  const { data } = useContext(DataContext)
  return <span>{data.listItem}</span>
}

function Header() {
  const { data } = useContext(DataContext)
  return <div>{data.title}</div>
}

function Text() {
  const { data } = useContext(DataContext)
  return <h1>{data.text}</h1>
}
```

중간 컴포넌트들이 깔끔해졌다. 데이터를 사용하지 않는 컴포넌트는 `data`의 존재 자체를 모르게 되었다.

### 테마 전환 예시

Provider Pattern이 가장 흔하게 사용되는 사례 중 하나가 **테마(Theme) 관리**이다.

```jsx
export const ThemeContext = createContext()

const themes = {
  light: {
    background: '#fff',
    color: '#000',
  },
  dark: {
    background: '#171717',
    color: '#fff',
  },
}

export default function App() {
  const [theme, setTheme] = useState('dark')

  function toggleTheme() {
    setTheme(theme === 'light' ? 'dark' : 'light')
  }

  const providerValue = {
    theme: themes[theme],
    toggleTheme,
  }

  return (
    <div className={`App theme-${theme}`}>
      <ThemeContext.Provider value={providerValue}>
        <Toggle />
        <List />
      </ThemeContext.Provider>
    </div>
  )
}
```

`Toggle` 컴포넌트에서는 `useContext`를 통해 `toggleTheme`에 접근하고, `ListItem`에서는 현재 테마의 스타일에 접근한다.

```jsx
// Toggle.jsx
import React, { useContext } from 'react'
import { ThemeContext } from './App'

export default function Toggle() {
  const theme = useContext(ThemeContext)

  return (
    <label className="switch">
      <input type="checkbox" onClick={theme.toggleTheme} />
      <span className="slider round" />
    </label>
  )
}
```

```jsx
// ListItem.jsx
import React, { useContext } from 'react'
import { ThemeContext } from './App'

export default function ListItem() {
  const theme = useContext(ThemeContext)

  return <li style={theme.theme}>...</li>
}
```

이처럼 props 전달 없이도 어떤 컴포넌트든 테마 데이터에 접근할 수 있다.

### 커스텀 훅으로 개선하기

각 컴포넌트에서 `useContext(ThemeContext)`를 직접 호출하는 것보다, 커스텀 훅으로 감싸면 더 깔끔하고 안전하다.

```jsx
function useThemeContext() {
  const theme = useContext(ThemeContext)
  if (!theme) {
    throw new Error('useThemeContext must be used within ThemeProvider')
  }
  return theme
}
```

Provider 로직도 별도의 컴포넌트로 분리할 수 있다.

```jsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('dark')

  function toggleTheme() {
    setTheme(theme === 'light' ? 'dark' : 'light')
  }

  const providerValue = {
    theme: themes[theme],
    toggleTheme,
  }

  return (
    <ThemeContext.Provider value={providerValue}>
      {children}
    </ThemeContext.Provider>
  )
}
```

이제 `App`은 테마 로직을 신경 쓸 필요 없이, `ThemeProvider`로 감싸기만 하면 된다.

```jsx
export default function App() {
  return (
    <div className="App">
      <ThemeProvider>
        <Toggle />
        <List />
      </ThemeProvider>
    </div>
  )
}
```

하위 컴포넌트에서는 커스텀 훅으로 간결하게 접근한다.

```jsx
export default function ListItem() {
  const theme = useThemeContext()

  return <li style={theme.theme}>...</li>
}
```

## [How] Provider Pattern은 어떻게 구현하는가?

핵심은 **React의 Context API를 사용해 데이터를 공급하는 Provider를 만들고, 소비하는 쪽에서 useContext로 접근하는 것**이다.

### 기본 구조

```tsx
// 1. Context 생성
const MyContext = createContext<MyContextType | null>(null)

// 2. Provider 컴포넌트 정의
function MyProvider({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState(initialValue)

  const value = { state, setState }

  return (
    <MyContext.Provider value={value}>
      {children}
    </MyContext.Provider>
  )
}

// 3. 커스텀 훅으로 소비 인터페이스 제공
function useMyContext() {
  const context = useContext(MyContext)
  if (!context) {
    throw new Error('useMyContext must be used within MyProvider')
  }
  return context
}
```

### 사용처

```tsx
// 최상위에서 Provider로 감싸기
function App() {
  return (
    <MyProvider>
      <ChildComponent />
    </MyProvider>
  )
}

// 데이터가 필요한 곳에서 훅으로 접근
function ChildComponent() {
  const { state, setState } = useMyContext()

  return <div>{state}</div>
}
```

이 구조에서 `createContext` → `Provider` → `useContext`(또는 커스텀 훅) 흐름이 React에서의 Provider Pattern이다.

## [Why] Provider Pattern은 왜 사용하는가?

Q. 언제 사용되면 좋을까?
- 핵심은 **여러 컴포넌트가 동일한 데이터를 공유해야 하는데, 그 사이에 데이터를 사용하지 않는 중간 컴포넌트가 있는 경우.**

Q. 왜 props로 직접 전달하지 않고 Provider를 쓰는가?
> Prop Drilling의 고통에서 벗어나기 위해

- **Prop Drilling 제거**: 데이터를 사용하지 않는 중간 컴포넌트가 불필요한 props를 받지 않아도 된다. 데이터가 필요한 컴포넌트에서 직접 접근하므로, 컴포넌트 트리가 깊어져도 전달 경로가 복잡해지지 않는다.
- **리팩토링 안전성**: 속성명을 변경할 때 중간 컴포넌트를 모두 수정할 필요가 없다. Provider의 value와 소비하는 컴포넌트만 수정하면 된다.
- **관심사의 분리**: 데이터 공급 로직(Provider)과 데이터 소비 로직(useContext)이 명확히 나뉘어, 각 컴포넌트가 자신의 역할에만 집중할 수 있다.

Q. Provider Pattern의 단점은 무엇인가?
- **불필요한 리렌더링**: Context의 `value`가 변경되면, 해당 Context를 `useContext`로 구독하는 **모든 컴포넌트**가 리렌더링된다. 값을 실제로 사용하지 않는 부분까지 리렌더링될 수 있어, 대규모 앱에서 자주 업데이트되는 값을 Context로 관리하면 성능에 악영향을 줄 수 있다.
- **과도한 사용 주의**: 모든 상태를 Context로 관리하려 하면 Provider가 중첩되어 "Provider Hell"이 발생할 수 있다. 전역적으로 공유되어야 하는 데이터(테마, 인증, 언어 등)에만 사용하고, 지역적인 상태는 props나 로컬 상태로 관리하는 것이 바람직하다.

## [Hum..🤔] Props Drilling, 무조건 Context로 해결해야 할까?

**Props Drilling이 발생한다고 해서 모든 값을 Context로 관리해야 하는 것은 아니다.**

### Props Drilling이 문제가 아닌 경우

컴포넌트는 props를 통해 **어떤 데이터를 사용하는지 명확하게 표현**한다.  
컴포넌트의 역할과 의도를 담고 있는 props라면 Drilling이 되더라도 문제가 되지 않을 수 있다.

```jsx
// UserProfilePage → ProfileCard → ProfileAvatar
// 3단계를 거치지만, 각 컴포넌트에서 props는 역할과 의도를 명확히 드러낸다.

function UserProfilePage({ user }) {
  return (
    <div>
      <ProfileCard user={user} />
      <ActivityFeed userId={user.id} />
    </div>
  )
}

function ProfileCard({ user }) {
  return (
    <div className="card">
      <ProfileAvatar name={user.name} imageUrl={user.imageUrl} />
      <span>{user.bio}</span>
    </div>
  )
}

function ProfileAvatar({ name, imageUrl }) {
  return <img src={imageUrl} alt={name} />
}
```

이 경우 `ProfileCard`는 `user`를 단순 전달이 아니라 **직접 사용**하고 있고(`user.bio`), `ProfileAvatar`에는 필요한 필드만 골라서 넘기고 있다.  
각 props가 **"이 컴포넌트는 무엇을 보여주는가"** 를 설명하고 있기 때문에, Drilling이 되더라도 문제가 되지 않는다.

오히려 이걸 Context로 감춰버리면 `ProfileAvatar`만 보고는 어디서 데이터가 오는지 알 수 없게 된다.

### Context 이전에 시도해볼 것: 조합(Composition) 패턴

Context API를 사용하기 전에, `children` prop을 이용해서 컴포넌트를 전달함으로써 depth를 줄일 수 있다.

```jsx
// Before: ItemEditModal → ItemEditBody → ItemEditList
// ItemEditBody는 데이터를 사용하지 않고 그냥 넘기기만 한다.

function ItemEditModal({ items, keyword, onKeywordChange }) {
  return (
    <Modal>
      <ItemEditBody 
        items={items} 
        keyword={keyword} 
        onKeywordChange={onKeywordChange} 
      />
    </Modal>
  )
}

function ItemEditBody({ items, keyword, onKeywordChange }) {
  return (
    <>
      <Input value={keyword} onChange={onKeywordChange} />
      <ItemEditList items={items} />
    </>
  )
}
```

```jsx
// After: 조합 패턴으로 중간 컴포넌트 제거

function ItemEditModal({ items, keyword, onKeywordChange }) {
  return (
    <Modal>
      <Input value={keyword} onChange={onKeywordChange} />
      <ItemEditList items={items} />
    </Modal>
  )
}
```

데이터를 사용하지 않고 **단순히 값을 전달하기 위한 용도의 중간 컴포넌트**는, props가 컴포넌트의 역할과 의도를 나타내지 않을 수 있다.  
이런 경우에 조합 패턴을 사용하면 불필요한 depth를 줄일 수 있다.

### Props Drilling을 해결하는 단계별 접근

정리하자면, Props Drilling이 발생했을 때 바로 Context를 꺼내는 것이 아니라 다음의 순서로 접근하는 것이 바람직하다.

| 순서 | 접근 방법                  | 설명 |
|:--:|:-----------------------|:--|
| 1 | **그대로 두기**             | props가 컴포넌트의 역할과 의도를 명확히 드러내고 있다면, Drilling이 되어도 문제가 아닐 수 있다. |
| 2 | **조합(Composition) 패턴** | 데이터를 사용하지 않는 중간 컴포넌트가 있다면, `children`을 활용해 불필요한 depth를 제거한다. |
| 3 | **컴포넌트 구조 재설계**        | 상태를 소유하는 컴포넌트의 위치가 적절한지, 불필요한 추상화가 있는지 점검한다. |
| 4 | **Context API **       | 위 방법들로 해결되지 않을 때, 최후의 방법으로 Context를 사용한다. |

불필요한 Props Drilling을 제거하는 과정은 단순히 전달 경로를 줄이는 것이 아니다.  
**불필요한 중간 추상화를 제거하여 개발자가 컴포넌트의 역할과 의도를 명확히 이해할 수 있도록 하는 것**이 핵심이다.