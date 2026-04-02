# 중재자 패턴

## 왜 사용하는가?
- 아주 멀리 존재하는 컴포넌트 트리까지 props를 내려주게 되면 prop에 의존되는 컴포넌트들을 리펙토링 하거나 어떤 데이터가 어디에서 전해져오는지 파악하기 어려운 형태가 됩니다.
- 그리고 종종 prop drilling이라 불리는 안티패턴 또한 사용하게 됩니다.
- 데이터가 필요하지 않는 컴포넌트에 props가 주입되는 경우도 존재하며, 프로퍼티의 이름을 변경하는 경우에도 모든 컴포넌트를 수정해야 합니다.

## 사용했을때 개선점
- 중재자 패턴을 사용하면 각 레이어에 직접 데이터를 전달하지 않고도 여러 컴포넌트들에서 데이터에 접근할 수 있게 구현 가능합니다.

## 사용 예시
- 중재자 패턴은 전역 데이터를 공유하기에 좋습니다. 보통은 UI 테마를 여러 컴포넌트들이 공유해 사용하기 위해 사용됩니다.
```tsx
export const ThemeContext = React.createContext()

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
- 테마의 값을 직접 props로 주입하는 대신 provider를 구성하고 테마 컬러값을 provider에 전달합니다.

```tsx
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
- 컴포넌트에서는 테마값의 변경을 위해 직접 함수를 호출할 수 있게 됩니다.

## 장점
- 컴포넌트 트리의 각 노드에 데이터를 전달하지 않아도 다수의 컴포넌트에 데이터를 전달할 수 있습니다.
- 리펙터링 과정에 개발자가 실수할 확률을 줄여줍니다.
- pros drilling이 개선됩니다.

## 단점
- 중재자 패턴을 과하게 사용할 경우 특정 상황에서 성능 이슈가 발생할 수 있습니다.
  - 컨텍스트를 참조하는 모든 컴포넌트는 컨텍스트 변경시마다 모두 리렌더링 됩니다.

## 추가 정리

## 중재자 패턴
- 객체 간의 복잡한 상호작용을 중재자 객체가 관리하도록 하는 디자인 패턴
- 컴포넌트들이 서로 직접 통신하지 않고 중재자를 통해 통신함으로서 결합도를 낮추고 유지보수성을 높임

### 프론트엔드에서의 중재자 패턴
- 컴포넌트 간 통신 간소화: prop drilling 문제 해결
- 복잡한 상태 관리 단순화: 여러 컴포넌트가 공유하는 상태를 중앙에서 관리
- 결합도 감소: 컴포넌트들이 서로를 직접 참조하지 않아도 됨

> 예시 코드는 없지만 form context를 통해 form 상태관리를 하는 경우가 대표적이지 않을까 싶습니다.

```ts
// 1. use form을 선언하고
function RouteComponent() {
  const form = useAppForm({...formOptions, /* ... */ })
  // <Outlet /> cannot be customized or receive additional props
  return <Outlet />
}

// 2. 컨텍스트 기반 폴백을 사용하여 폼 인스턴스에 액세스하고
const { useAppForm, useTypedAppFormContext } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {},
  formComponents: {},
})

// 사용하는 경우에는
const formOpts = formOptions({
  /* ... */
})

function ParentComponent() {
  const form = useAppForm({ ...formOptions /* ... */ })

  return (
    <form.AppForm>
      <ChildComponent />
    </form.AppForm>
  )
}

function ChildComponent() {
  const form = useTypedAppFormContext({ ...formOptions })

  // You now have access to form components, field components and fields
}
```