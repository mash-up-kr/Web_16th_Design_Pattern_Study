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