# 컴파운드 패턴

## Context API
- React의 Context API 를 활용해 컴파운드 패턴을 활용하여 예제를 구현해 보겠읍니다.
  - 예시 코드
  ```jsx
  const FlyOutContext = createContext()

  function FlyOut(props) {
    const [open, toggle] = useState(false)

    return (
      <FlyOutContext.Provider value={{ open, toggle }}>
        {props.children}
      </FlyOutContext.Provider>
    )
  }

  function Toggle() {
    const { open, toggle } = useContext(FlyOutContext)

    return (
      <div onClick={() => toggle(!open)}>
        <Icon />
      </div>
    )
  }

  function List({ children }) {
    const { open } = useContext(FlyOutContext)
    return open && <ul>{children}</ul>
  }

  function Item({ children }) {
    return <li>{children}</li>
  }

  FlyOut.Toggle = Toggle
  FlyOut.List = List
  FlyOut.Item = Item
  ```

  - 사용처
  ```jsx
  import React from 'react'
  import { FlyOut } from './FlyOut'

  export default function FlyoutMenu() {
    return (
      <FlyOut>
        <FlyOut.Toggle />
        <FlyOut.List>
          <FlyOut.Item>Edit</FlyOut.Item>
          <FlyOut.Item>Delete</FlyOut.Item>
        </FlyOut.List>
      </FlyOut>
    )
  }
  ```
  - `FlyOutMenu` 자체에는 아무런 상태를 가지고 있지 않습니다.

## React.Children.map
- 자식 컴포넌트들을 순회 처리 하는데에도 컴파운드 패턴을 사용할 수 있읍니다
- `React.cloneElement`를 사용하여 자식 컴포넌트를 복제하여 각각에게 open과 toggle 메서드를 넘길 수 있습니다.
  ```jsx
  import React from "react";
  import Icon from "./Icon";

  export function FlyOut(props) {
    const [open, toggle] = React.useState(false);

    return (
      <div className={`flyout`}>
        {React.Children.map(props.children, child =>
          React.cloneElement(child, { open, toggle })
        )}
      </div>
    );
  }

  function Toggle({ open, toggle }) {
    return (
      <div className="flyout-btn" onClick={() => toggle(!open)}>
        <Icon />
      </div>
    );
  }

  function List({ children, open }) {
    return open && <ul className="flyout-list">{children}</ul>;
  }

  function Item({ children }) {
    return <li className="flyout-item">{children}</li>;
  }

  FlyOut.Toggle = Toggle;
  FlyOut.List = List;
  FlyOut.Item = Item;
  ```
  - 모든 메뉴 컴포넌트들은 복제되며 `open`, `toggle`메서드를 전달받았습니다. 위에서 해당 값을 받기 위해 Context API를 사용했던 것에 비교하여 그냥 prop에서 두 값을 사용하면 됩니다.

## 장점
- 컴파운드 패턴은 동작 구현에 필요한 상태를 내부적으로 가지고 있는데 이 것을 사용하는 쪽에서는 드러나지 않아 걱정 없이 사용할 수 있습니다.
- 또 해당 패턴을 사용하면 NameSpace 하위 자식 컴포넌트들을 일일히 import 할 필요 없이 기능을 이용할 수 있습니다.

## 단점
- 내부에서 `React.Children.map`을 사용하고 있기 때문에 사용하는 쪽에서 자식 컴포넌트를 약속된 형태로 넘겨야 하는 제약이 생깁니다.
- 엘리먼트를 복제하는 경우, 복제 대상 컴포넌트가 기존에 갖고 있는 prop과 이름이 충돌될 수 있습니다. 이 경우 React.cloneElement를 사용할 때 넘어간 값으로 해당 prop은 덮어씌워질 것 입니다.

# 추가 보충

## 컴파운드 패턴
- 여러 컴포넌트를 조합하여 하나의 완성된 기능을 제공
  - 대표적으로는 Tab 컴포넌트가 존재합니다.
    ```jsx
    import { createContext, useContext, useState } from 'react';

    const TabContext = createContext();

    function Tabs({ children }) {
      const [activeTab, setActiveTab] = useState(0);

      return (
        <TabContext.Provider value={{ activeTab, setActiveTab }}>
          <div className="tabs-container">{children}</div>
        </TabContext.Provider>
      );
    }

    function TabList({ children }) {
      return <div className="tab-list">{children}</div>;
    }

    function Tab({ index, children }) {
      const { activeTab, setActiveTab } = useContext(TabContext);

      return (
        <button
          className={`tab ${activeTab === index ? 'active' : ''}`}
          onClick={() => setActiveTab(index)}
        >
          {children}
        </button>
      );
    }

    function TabPanels({ children }) {
      return <div className="tab-panels">{children}</div>;
    }

    function TabPanel({ index, children }) {
      const { activeTab } = useContext(TabContext);
      return activeTab === index ? <div className="tab-panel">{children}</div> : null;
    }

    // 사용 예시
    function App() {
      return (
        <Tabs>
          <TabList>
            <Tab index={0}>Tab 1</Tab>
            <Tab index={1}>Tab 2</Tab>
          </TabList>
          <TabPanels>
            <TabPanel index={0}>Content 1</TabPanel>
            <TabPanel index={1}>Content 2</TabPanel>
          </TabPanels>
        </Tabs>
      );
    }
    ```
### 특징
- 암묵적 상태 공유: 부모 컴포넌트가 상태를 관리하고 자식 컴포넌트와 암묵적으로 공유
  > 합성 컴포넌트를 구성할 때 Context API를 사용하는게 가장 바람직한 사용 예시중 하나로 언급
- 선언적 API: 사용자가 컴포넌트의 구조를 직관적으로 이해할 수 있게 구성
- 유연한 조합: 자식 컴포넌트들을 필요에 따라 자유롭게 배열하거나 생략 가능
- 컨텍스트 활용: 주로 React의 Context API를 사용하여 상태 공유

### 장점
- 사용 편의성: 복잡한 props 전달 없이 직관적인 API 제공 
  > ⭐️ 가장 중요한 포인트 ⭐️
- 유연성: 자식 컴포넌트들의 순서나 구서을 자유롭게 변경 가능
- 관심사 분리: 상태 관리와 UI 표현이 자연스럽게 분리됨
- 확장성: 새로운 자식 컴포넌트 타입을 쉽게 추가 가능

### 단점
- 초기 설정 복잡성: Context API 와 여러 컴포넌트를 구성해야 해서 초기 구현이 복잡함
- 과도한 추상화: 간단한 컴포넌트에 적용하면 오히려 코드가 복잡해질 수 있음
- 디버깅 어려움: 암묵적인 상태 공유로 인하여 데이터 흐름 추적이 어려울 수 있음

### 일상적으로 사용되는 예시
- React Router
  ```jsx
  <Router>
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="about" element={<About />} />
    </Routes>
  </Router>
  ```

- Headless UI (Combobox)
  ```jsx
  <Combobox>
    <Combobox.Input />
    <Combobox.Options>
      <Combobox.Option value="1">Option 1</Combobox.Option>
    </Combobox.Options>
  </Combobox>
  ```

### 컴파운드 컴포넌트 패턴으로 개발 시 주의 사항
- 명확한 컴포넌트 계층: 부모 - 자식 관계를 명확히 정의
- 의미 있는 에러 메세지: Context가 없는 경우 사용자에게 명확한 지침 제공 필요
- 필수 컴포넌트 검증: 필수 자식 컴포넌트가 없는 경우 경고 또는 에러 발생
- 유연한 스타일링: className이나 style props를 통해 외부에서 스타일 조정 가능하게 구현
- 문서화: 컴포넌트 조합 방식을 명확히 문서화

### 컴파운드 패턴의 고급 패턴
- 부모 - 자식 관계를 명확히 정의하고, 필수 자식 컴포넌트가 없는 경우 경고 또는 에러를 발생시킬줄 알아야 한다...!!
  ```jsx
  import React, {
    Children,
    cloneElement,
    createContext,
    isValidElement,
    useContext,
    useState,
  } from 'react';

  interface TabProps {
    index?: number;
    children?: React.ReactNode;
  }

  interface TabPanelProps {
    index: number;
    children?: React.ReactNode;
  }

  interface TabContextValue {
    activeTab: number;
    setActiveTab: React.Dispatch<React.SetStateAction<number>>;
  }

  const TabContext = createContext<TabContextValue>({
    activeTab: 0,
    setActiveTab: () => {},
  });

  function Tab({ index, children }: TabProps) {
    return <div className="tab">{children}</div>;
  }

  function TabPanel({ index, children }: TabPanelProps) {
    const { activeTab } = useContext(TabContext);
    return (
      <div
        className="tab-panel"
        style={{ display: activeTab === index ? 'block' : 'none' }}
      >
        {children}
      </div>
    );
  }

  // 이 부분 코드가 핵심이에요
  // Children, cloneElement, isValidElement 는 알고 넘어가시는게 좋을거같습니다.
  function Tabs({ children }: { children: React.ReactNode }) {
    const [activeTab, setActiveTab] = useState(0);
    const tabs = Children.toArray(children).filter(
      (child) => isValidElement(child) && child.type === Tab
    );
    // React.Children 리액트에서 가져오는 Children에 toArray 후 props.children(ReactNode)을 주입하면 배열로 변경된 값에 대한 검증을 진행할 수 있습니다.
    // 즉, 배열로 변경되어 이터리이션을 돌며 검증이 가능해진다는 이야기 입니다.
    // isValidElement 를 통해 유효한 child인지 그리고 Tab 에 해당하는 값인지에 대한 검증을 진행하게 됩니다.
    // 즉, tabs는 유효성 검증을 통과한 값만 구성됩니다.

    return (
      <TabContext.Provider value={{ activeTab, setActiveTab }}>
        <div className="tabs">
          {tabs.map((tab, index) =>
            isValidElement(tab)
              ? cloneElement(tab as React.ReactElement<TabProps>, { index, key: index })
              : null
          )}
        </div>
        {Children.toArray(children).filter(
          (child) => isValidElement(child) && child.type === TabPanel
        )}
      </TabContext.Provider>
    );

    // 유효성 검증을 통과한 tabs에 대해서만 element를 바인딩 하고, 그 이외에는 null 처리를 진행하게 됩니다.
    // 유효하지 않은 값에 대한 null 처리는 isValidElement를 사용해서 error로 처리 가능하도록 구조를 개선할 수 있습니다.
    // 알잘딱 알아서 설계 가넝
  }

  export { Tab, TabPanel, Tabs };
  ```
  - 허용되지 않은 형태의 UI 혹은 children 구성을 했다는 내용이 error 처리 되거나 null로 감춰지는 처리가 되는 구성입니다.

### 컴파운드 패턴의 네임스페이스 전략
#### HOC를 이용한 네임스페이스 사용 개선
```js
function withNamespace(Component, components) {
  Object.assign(Component, components);
  return Component;
}

// 사용처 예시
export default withNamespace(Tabs, { TabList, Tab, TabPanel })
```
- 부모 컴포넌트 프로퍼티로 자식 컴포넌트를 할당하는 경우 해당 HOC 함수를 사용하면 한줄로 딸깍 가넝

#### 네임스페이스를 적용할 때의 장점
- 논리적 그룹화: 관련 컴포넌트들이 자연스럽게 응집됨
- import 간소화
- 자동완성 지원

#### 네임스페이스 적용 시 주의사항
- 명확한 문서화: 네임스페이스로 제공되는 컴포넌트들을 명확히 문서화
- 일괄된 네이밍: 계층 구조를 반영한 일관된 네이밍 규칙 적용
- 과도한 중첩 방지: 2~3 단계 이상의 깊은 중첩은 피하는 것이 좋음
- Context 공유: 네임스페이스 컴포넌트들 간의 상태 공유는 Context API로 처리
- TypeScript 지원: 타입 정의를 통해 안전한 처리 필요