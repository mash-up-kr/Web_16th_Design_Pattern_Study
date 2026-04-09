## Compound 패턴이란?

- **여러 컴포넌트가 하나의 동작을 위해 상태를 공유하며 협력하는 패턴**
- `<select>`와 `<option>`처럼 서로 분리되어 있지만 하나의 동작을 구성하는 컴포넌트 관계를 표현한다
- 내부 상태는 Context를 통해 캡슐화하고, 사용하는 쪽에서는 상태를 직접 관리할 필요가 없다
- 부모 컴포넌트에 서브 컴포넌트를 프로퍼티로 붙이는 방식(`Component.SubComponent`)으로 사용한다

## 구현 방법 1: Context API

Context를 통해 상태를 공유하는 방식으로, **자식 컴포넌트의 위치가 자유롭다**는 장점이 있다.

```javascript
const FlyOutContext = createContext()

function FlyOut({ children }) {
  const [open, setOpen] = useState(false)

  return (
    <FlyOutContext.Provider value={{ open, toggle: setOpen }}>
      {children}
    </FlyOutContext.Provider>
  )
}

function Toggle() {
  const { open, toggle } = useContext(FlyOutContext)
  return <div onClick={() => toggle(!open)}><Icon /></div>
}

function List({ children }) {
  const { open } = useContext(FlyOutContext)
  return open && <ul>{children}</ul>
}

function Item({ children }) {
  return <li>{children}</li>
}

// 서브 컴포넌트로 등록
FlyOut.Toggle = Toggle
FlyOut.List = List
FlyOut.Item = Item
```

**사용 예시:**

```javascript
<FlyOut>
  <FlyOut.Toggle />
  <FlyOut.List>
    <FlyOut.Item>Edit</FlyOut.Item>
    <FlyOut.Item>Delete</FlyOut.Item>
  </FlyOut.List>
</FlyOut>
```

- `FlyOut`만 import하면 하위 컴포넌트를 모두 사용할 수 있다
- 사용하는 쪽에서는 `open` 상태를 전혀 알 필요가 없다

## 구현 방법 2: React.Children.map

`React.cloneElement`로 자식에게 직접 props를 주입하는 방식이다.

```javascript
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

- Context 없이 props를 주입하므로 구조가 단순하지만, **직계 자식에게만 주입**된다
- 추가 `div`로 자식을 감싸면 주입이 끊기기 때문에 자식 구조에 제약이 생긴다

## 실제 사용 예시: Modal 컴포넌트

디자인 시스템 프로젝트의 Modal 컴포넌트. `onClose` 핸들러를 Context에 담아 `Header`에서 별도 props 없이 꺼내 쓴다.

```typescript
const ModalContext = createContext<{ onClose: MouseEventHandler } | null>(null)

const Modal = ({ children, isOpen, onClose }: ModalProps) => {
  return (
    <Portal id={MODAL_PORTAL_ROOT_ID}>
      <ModalContext.Provider value={{ onClose }}>
        <ModalWrapper isOpen={isOpen}>
          <Dimmed onClick={onClose} />
          <Center>
            <ModalBox role="dialog" aria-modal="true">
              {children}
            </ModalBox>
          </Center>
        </ModalWrapper>
      </ModalContext.Provider>
    </Portal>
  )
}

// Header가 Context에서 onClose를 직접 꺼내므로 사용 측에서 prop을 내려줄 필요 없다
const Header = ({ title }: HeaderProps) => {
  const { onClose } = useSafeContext(ModalContext)
  return (
    <Flexbox>
      <Button onClick={onClose}>
        <MdArrowBackIosNew />
      </Button>
      <Typography variant="body">{title}</Typography>
    </Flexbox>
  )
}

const Content = ({ children }: ChildrenProps) => {
  return <ContentWrapper>{children}</ContentWrapper>
}

// 서브 컴포넌트 등록
Modal.Header = Header
Modal.Content = Content

export default Modal
```

**useSafeContext**
```typescript
import { Context, useContext } from 'react';

// eslint-disable-next-line @typescript-eslint/no-explicit-any
const useSafeContext: <T extends Context<any>>(
  Context: T,
) => T extends Context<infer U> ? Exclude<U, null> : never = (Context) => {
  const context = useContext(Context);

  if (!context) {
    throw new Error('CDS 컴포넌트에 필요한 컨텍스트가 없어요!');
  }

  return context;
};

export default useSafeContext;
```

**사용 예시:**

```typescript
<Modal isOpen={isOpen} onClose={handleClose}>
  <Modal.Header title="설정" />
  <Modal.Content>
    <p>내용</p>
  </Modal.Content>
</Modal>
```

- `Modal.Header`가 `onClose`를 내부적으로 알고 있으므로 사용 측에서 매번 전달하지 않아도 된다
- `Modal`, `Modal.Header`, `Modal.Content`를 조합해 구조를 자유롭게 구성할 수 있다

**참조**
- https://github.com/c-h-w-h/cds/blob/main/src/components/Modal/index.tsx

## 장점

- **상태 캡슐화**: 공유 상태가 Context 내부에 숨겨져 있어, 사용하는 쪽의 코드가 단순해진다
- **유연한 구성**: 서브 컴포넌트를 원하는 순서와 구조로 배치할 수 있다
- **단순한 import**: 루트 컴포넌트 하나만 가져오면 서브 컴포넌트를 모두 쓸 수 있다

## 단점

- **자식 구조 제약** (`React.Children.map` 방식): 직계 자식이 아닌 경우 props 주입이 끊긴다
- **Props 충돌** (`React.Children.map` 방식): 기존 props와 이름이 겹칠 경우 덮어써진다
- **서브 컴포넌트 강제**: 루트 컴포넌트 외부에서 서브 컴포넌트만 단독 사용하면 Context가 없어 오류가 발생한다

## 참고자료

- https://patterns-dev-kr.github.io/design-patterns/compound-pattern/
