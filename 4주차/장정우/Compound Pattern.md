# [Compound Pattern](https://patterns-dev-kr.github.io/design-patterns/compound-pattern/)

## [What] Compound Pattern이란?

> "여러 컴포넌트가 모여 하나의 완전한 동작을 구성하는 패턴" _(claude)_

> Compound 컴포넌트 패턴은 여러 컴포넌트들이 하나의 작업을 수행하기 위해 함께 동작하는 패턴이다. _(patterns.dev)_

사실 이 개념은 React 이전부터 존재했다.
HTML의 `<select>`와 `<option>`이 바로 원시적인 Compound Component이다.

```html
<select>
  <option value="apple">사과</option>
  <option value="banana">바나나</option>
  <option value="orange">오렌지</option>
</select>
```

`<select>`는 `<option>` 없이는 의미가 없고, `<option>`도 `<select>` 밖에서는 동작하지 않는다.
이처럼 **개별적으로는 불완전하지만, 함께 조합되었을 때 비로소 완전한 동작을 이루는 구조**가 Compound Pattern이다.

조금 더 넓은 관점에서 보면,  
GOF의 Composite Pattern은 "부분-전체(part-whole) 계층 구조"를 표현하며, 개별 객체와 복합 객체를 동일하게 다루는 것이 핵심이다.  
Compound Pattern 역시 서브 컴포넌트(부분)와 부모 컴포넌트(전체)가 하나의 통합된 인터페이스를 형성한다는 점에서 같은 사고방식에 기반한다.

|![Composite & Compound.png](images/Composite%20%26%20Compound.png)|
|:--:|

## [Patterns.dev] Compound Pattern 예시로 알아보기

### FlyOut 메뉴 예시

패턴 없이 하나의 거대한 컴포넌트로 FlyOut 메뉴를 만들면 다음과 같다.

```jsx
function FlyOut() {
  const [open, setOpen] = useState(false)

  return (
    <div>
      <button onClick={() => setOpen(!open)}>메뉴 열기</button>
      {open && (
        <ul>
          <li onClick={() => console.log("편집")}>편집</li>
          <li onClick={() => console.log("삭제")}>삭제</li>
        </ul>
      )}
    </div>
  )
}
```

작동은 하지만, 메뉴 항목의 구성을 바꾸거나 토글 버튼의 UI를 변경하려면 **`FlyOut` 컴포넌트 자체를 수정**해야 한다.  
사용하는 쪽에서 구조를 제어할 수 없다.

### Context API를 활용한 Compound Pattern 적용

먼저 `FlyOutContext`를 만들어 열림/닫힘 상태를 공유한다.

```jsx
const FlyOutContext = createContext()

function FlyOut({ children }) {
  const [open, setOpen] = useState(false)

  return (
    <FlyOutContext.Provider value={{ open, setOpen }}>
      <div>{children}</div>
    </FlyOutContext.Provider>
  )
}
```

토글 버튼, 메뉴 리스트, 메뉴 아이템을 각각 서브 컴포넌트로 분리한다.

```jsx
function Toggle() {
  const { open, setOpen } = useContext(FlyOutContext)

  return (
    <button onClick={() => setOpen(!open)}>
      {open ? "닫기" : "열기"}
    </button>
  )
}

function List({ children }) {
  const { open } = useContext(FlyOutContext)

  return open ? <ul>{children}</ul> : null
}

function Item({ children, onClick }) {
  return <li onClick={onClick}>{children}</li>
}
```

### Static Property 패턴으로 서브 컴포넌트 연결

서브 컴포넌트를 부모 컴포넌트의 **정적 속성(Static Property)** 으로 연결하면, 사용하는 쪽에서 `FlyOut`만 import하면 된다.

```jsx
FlyOut.Toggle = Toggle
FlyOut.List = List
FlyOut.Item = Item
```

이제 사용하는 쪽에서는 다음과 같이 선언적으로 조합한다.

```jsx
export default function FlyOutMenu() {
  return (
    <FlyOut>
      <FlyOut.Toggle />
      <FlyOut.List>
        <FlyOut.Item onClick={() => console.log("편집")}>편집</FlyOut.Item>
        <FlyOut.Item onClick={() => console.log("삭제")}>삭제</FlyOut.Item>
      </FlyOut.List>
    </FlyOut>
  )
}
```

구조가 한눈에 읽히고, 항목의 순서나 구성을 사용하는 쪽에서 자유롭게 변경할 수 있다.  
`FlyOut`이라는 하나의 import로 모든 서브 컴포넌트에 접근할 수 있기 때문에, API가 깔끔하다.

### React.Children.map 방식

Context 대신 `React.Children.map`과 `React.cloneElement`를 사용하여 부모가 자식에게 암묵적으로 props를 전달하는 방법도 있다.

```jsx
function FlyOut({ children }) {
  const [open, setOpen] = useState(false)

  return (
    <div>
      {React.Children.map(children, (child) =>
        React.cloneElement(child, { open, setOpen })
      )}
    </div>
  )
}
```

하지만 이 방식에는 제약이 있다.  
`React.Children.map`은 **직접 자식(direct children)** 만 순회하기 때문에, 서브 컴포넌트를 `<div>`로 감싸는 것만으로도 동작하지 않게 된다.

```jsx
// 동작하지 않음
<FlyOut>
  <div>
    <FlyOut.Toggle />  {/* open, setOpen을 받지 못한다 */}
  </div>
</FlyOut>
```

이러한 한계 때문에 **Context 방식이 더 유연하고 실무에서 선호**된다.

### Children을 함수로 전달하기

Render Props와 결합하여 children을 함수로 전달하는 변형도 가능하다.

```jsx
function FlyOut({ children }) {
  const [open, setOpen] = useState(false)

  return <div>{children({ open, setOpen })}</div>
}
```

```jsx
<FlyOut>
  {({ open, setOpen }) => (
    <>
      <button onClick={() => setOpen(!open)}>메뉴</button>
      {open && (
        <ul>
          <li>편집</li>
          <li>삭제</li>
        </ul>
      )}
    </>
  )}
</FlyOut>
```

이 방식은 상태를 명시적으로 받을 수 있다는 장점이 있지만, Compound Pattern의 핵심인 **서브 컴포넌트 단위의 선언적 조합**이 사라지기 때문에 Compound Pattern보다는 Render Props Pattern에 가깝다.

## [How] Compound Pattern은 어떻게 구현하는가?

핵심은 **여러 서브 컴포넌트가 하나의 부모 컴포넌트의 상태를 암묵적으로 공유하며, 사용하는 쪽에서는 선언적으로 조합하는 것**이다.

### 기본 구조

```tsx
// 1. 공유 상태를 위한 Context 생성
const CompoundContext = createContext<CompoundContextType | null>(null)

// 2. 부모 컴포넌트: 상태를 관리하고 Provider로 공급
function Compound({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState(initialValue)

  return (
    <CompoundContext.Provider value={{ state, setState }}>
      {children}
    </CompoundContext.Provider>
  )
}

// 3. 커스텀 훅: Context 접근을 캡슐화
function useCompoundContext() {
  const context = useContext(CompoundContext)
  if (!context) {
    throw new Error("서브 컴포넌트는 Compound 내부에서만 사용할 수 있습니다.")
  }
  return context
}

// 4. 서브 컴포넌트: 공유 상태를 사용하여 각자의 역할 수행
function SubA() {
  const { state, setState } = useCompoundContext()
  return <button onClick={() => setState(!state)}>Toggle</button>
}

function SubB({ children }: { children: React.ReactNode }) {
  const { state } = useCompoundContext()
  return state ? <div>{children}</div> : null
}

// 5. Static Property로 연결
Compound.SubA = SubA
Compound.SubB = SubB
```

### 사용 예시

```tsx
<Compound>
  <Compound.SubA />
  <Compound.SubB>
    <p>열려 있을 때만 보이는 콘텐츠</p>
  </Compound.SubB>
</Compound>
```

Static Property 패턴 덕분에 `Compound` 하나만 import하면 모든 서브 컴포넌트에 접근할 수 있다.  
사용하는 쪽에서 서브 컴포넌트의 순서, 구성, 중간에 추가할 요소 등을 자유롭게 결정할 수 있다.

## [Why] Compound Pattern은 왜 사용하는가?

Q. 언제 사용하면 좋을까?
- 핵심은 **여러 서브 컴포넌트가 하나의 상태를 공유하면서도, 사용하는 쪽에서 구조를 자유롭게 조합하고 싶을 때.**
- Dropdown, Accordion, Tabs, Dialog 등 "여러 파트가 하나의 동작을 이루는" UI에 적합하다.

Q. 왜 하나의 거대한 컴포넌트로 만들지 않고 Compound Pattern을 쓰는가?

1. **선언적 API**
    - 사용하는 쪽에서 컴포넌트의 구조가 한눈에 보인다. props로 `showHeader`, `showFooter`, `items` 등을 넘기는 것보다, 서브 컴포넌트를 직접 배치하는 것이 의도를 더 명확하게 드러낸다.

    ```jsx
    {/* props 기반: 구조가 숨겨져 있다 */}
    <Menu items={items} showIcon={true} onEdit={handleEdit} onDelete={handleDelete} />

    {/* Compound 기반: 구조가 드러난다 */}
    <Menu>
      <Menu.Toggle icon={<HamburgerIcon />} />
      <Menu.List>
        <Menu.Item onClick={handleEdit}>편집</Menu.Item>
        <Menu.Item onClick={handleDelete}>삭제</Menu.Item>
      </Menu.List>
    </Menu>
    ```

2. **유연한 구조 변경**
    - 서브 컴포넌트의 순서, 구성을 사용하는 쪽에서 자유롭게 변경할 수 있다. 새로운 요소를 추가하거나 순서를 바꿀 때 부모 컴포넌트를 수정할 필요가 없다.

3. **관심사의 분리**
    - 상태 관리는 부모 컴포넌트가, UI 렌더링은 각 서브 컴포넌트가 담당한다. 각자의 책임이 명확하게 분리된다.
    - 그래서 그런지 코드만 보고 구조를 파악하기에 매우 편하다.

Q. Compound Pattern의 단점은 무엇인가?

- **React.Children.map 방식의 제약**: 이 방식을 사용하면 서브 컴포넌트가 반드시 직접 자식이어야 한다. 중간에 `<div>` 하나만 감싸도 상태 전달이 끊어진다.
- **Context 방식의 제약**: Provider 밖에서 서브 컴포넌트를 사용하면 동작하지 않는다. 에러 핸들링을 하지 않으면 디버깅이 어려워질 수 있다. (커스텀 훅에서 `throw new Error`를 넣는 이유이다.)
- **과도한 분리**: 단순한 컴포넌트까지 Compound로 만들면 오히려 복잡도가 증가한다. 구조가 고정되어 있고 변경 가능성이 없는 컴포넌트라면 하나로 두는 것이 낫다.

## [Hum..🤔] Compound Pattern, 어디까지 쪼개야 할까?

모든 컴포넌트를 Compound Pattern으로 만들 필요는 없다.

구조가 고정되어 있고, 사용하는 쪽에서 변경할 일이 없는 컴포넌트라면 단일 컴포넌트로 충분하다.  
예를 들어 `<ProfileCard>`가 항상 아바타 + 이름 + 소개글의 고정 구조라면, 이를 `ProfileCard.Avatar`, `ProfileCard.Name`, `ProfileCard.Bio`로 쪼개는 것은 불필요한 복잡도만 추가하는 것이다.

판단 기준은 **"사용하는 쪽에서 구조를 바꿀 필요가 있는가?"** 이다.

- 구조가 고정 → 단일 컴포넌트
- 구조가 유동적, 사용처마다 다른 조합이 필요 → Compound Pattern

실제로 이 패턴을 적극적으로 활용하는 라이브러리들이 있다.

| 라이브러리 | 특징 |
|:--|:--|
| **Radix UI** | `Dialog.Root`, `Dialog.Trigger`, `Dialog.Content` 등 Compound 구조를 기본으로 사용 |
| **Headless UI** | 스타일 없이 동작만 제공하는 Compound Component |
| **shadcn/ui** | Radix 기반으로 Compound 구조를 유지하면서 스타일을 입힌 형태 |

이 라이브러리들의 공통점은 **"동작은 라이브러리가 관리하고, 구조와 스타일은 사용자가 결정한다"** 는 철학이다.  
Compound Pattern이 그 철학을 구현하는 핵심 수단이 된다.

