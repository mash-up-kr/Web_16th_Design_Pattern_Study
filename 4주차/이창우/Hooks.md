# Hooks

함수형 컴포넌트에서 상태, 이펙트, 로직을 구조적으로 분리하고 재사용 및 조합하기 위한 패턴

⇒ UI를 그리는 코드와, 이를 동작하기 위해 필요한 상태, 행동 로직을 분리해서 재사용 가능하게 하는 방식
단순한 문법기능 X, 로직을 어떻게 나누고, 어떤식으로 재사용할 것인가

## 클래스형 컴포넌트 ⇒ hook을 통해 함수형 컴포넌트 전환 사례

### Effect Hook

```tsx
componentDidMount() { ... }
useEffect(() => { ... }, [])

componentWillUnmount() { ... }
useEffect(() => { return () => { ... } }, [])

componentDidUpdate() { ... }
useEffect(() => { ... })
```

기존 클래스형은 생명주기 메서드를 통해 상태 관리 

```tsx
useEffect(() => { 
  console.log(`The user typed ${input}`)
}, [input])
```

의존성을 가진 Effect Hook을 통해, 의존 상태의 변경에 따라 동작하도록 생명주기 가로챔 

### Custom Hooks

- 빌트인 훅들을 이용해서 커스텀 훅을 직접 만들 수 있음
- 관습적으로 use로 시작

![image.png](attachment:d7891074-67c5-4551-9ffc-4e759f1cdf2d:image.png)

패턴스 데브에서는 기존 클래스형 컴포넌트를 통해 제어하던 생명 주기를, Hook 등장에 따라 어떻게 대체되었는지를 설명해주네요. 읽기에는 재미었지만, 정리할 필요까지는 없을 것 같습니다..

### 구조

단순한 useState, useEffect 사용에 그치지 않음

- 상태 있는 로직의 추출 ⇒ 단순함수와 다르게, 상태를 가지고 생명 주기 흐름안에서 동작
- 관심사 분리 ⇒ 컴포넌트는 렌더에 집중, 훅은 상태관리와 행동에 집중
- 재사용 가능한 인터페이스 제공  ⇒ 내부 구현을 감추고 필요한 상태와 함수만 제공

내부 상태로 useState, useReducer, useRef 등을 가지고 필요하다면 useEffect를 통해 동기화

⇒ 이를 인터페이스화 하여 접근할 값만 반환

```tsx
function useToggle(initialValue = false) {
  const [isOpen, setIsOpen] = useState(initialValue);

  const open = () => setIsOpen(true);
  const close = () => setIsOpen(false);
  const toggle = () => setIsOpen(prev => !prev);

  return { isOpen, open, close, toggle };
}
```

Good

- 하나의 관심사에 집중
- 이름이 역할을 설명
- 반환 인터페이스가 단순
- UI 와 결합 분리
- 재사용 가능한 수준에서의 추상화

→ 좋은 훅이 되려면!

## 오픈소스 사례 : DownShift

클래스형 컴포넌트 ⇒ 훅 변환 과정 정리

https://github.com/downshift-js/downshift

DownShift : 페이팔에서 개발한 autoComplete, combobox, select dropdown과 같은 UI를 만들기 위한 오픈소스 ( 이닛 9년전 ㄷㄷ )

render props 기반 ⇒ hooks 기반 전환 사례

```tsx
// ===== 기존 Render Props 방식 =====
import Downshift from 'downshift'

function MyDropdown() {
  return (
    <Downshift
      onChange={selection => console.log(`선택: ${selection.value}`)}
      itemToString={item => (item ? item.value : '')}
    >
      {({
        getInputProps,
        getItemProps,
        getLabelProps,
        getMenuProps,
        getRootProps,
        isOpen,
        inputValue,
        highlightedIndex,
        selectedItem,
      }) => (
        <div>
          <label {...getLabelProps()}>과일 검색</label>
          <div {...getRootProps({}, {suppressRefError: true})}>
            <input {...getInputProps()} />
          </div>
          <ul {...getMenuProps()}>
            {isOpen
              ? items
                  .filter(item => item.value.includes(inputValue))
                  .map((item, index) => (
                    <li
                      {...getItemProps({
                        key: item.value,
                        index,
                        item,
                        style: {
                          backgroundColor:
                            highlightedIndex === index ? '#bde4ff' : 'white',
                          fontWeight:
                            selectedItem === item ? 'bold' : 'normal',
                        },
                      })}
                    >
                      {item.value}
                    </li>
                  ))
              : null}
          </ul>
        </div>
      )}
    </Downshift>
  )
}
```

- 기존 라이브러리들이 렌더링까지 해서 API가 커져 구현이 복잡하다 ⇒ 렌더는 하지 않고, 렌더콜백과 컨트롤 프롭스로 유연성을 준다는 설계 ( headless + prop getters + controlled state ) 철학으로 시작
- 클래스형 + Render props 패턴, render prop을 통해 prop getter 함수들을 자식에게 주는 구조
- 하나의 거대한 컴포넌트가 select, combobox, autocomplete 모두 처리하다 보니 로직 복잡

잘 설계된 컴포넌트인지에 대한 식별은 불가능.. 😭 하지만

1. 상태가 직접 안보임 ⇒ DownShift 내부에 있기 때문, 상태 소유자가 지금 컴포넌트가 아님
2. Props를 함수로 받아 뿌림. 또 이는 단순 요소가 아닌, downShift 시스템에 등록하는 함수 

```tsx
src/hooks/
├── useSelect/           ← 단순 드롭다운 select용
│   ├── index.js         ← 메인 hook
│   └── reducer.js       ← 상태 변경 로직
├── useCombobox/         ← 검색 가능한 combobox용
│   ├── index.js
│   └── reducer.js
├── useMultipleSelection/ ← 다중 선택용
│   ├── index.js
│   └── reducer.js
└── utils.js             ← 공유 유틸리티
```

훅 등장과 함께 분리, useReducer를 사용해 상태 관리하고, prop getter 인터페이스 그대로
하나의 만능 컴포넌트 DownShift 에서 useSelect, useCombobox로 분리 ⇒ 코드가 더 명확해짐

```tsx
import { useSelect } from 'downshift'

function SelectDropdown() {
  const books = [
    { id: 1, title: '노르웨이의 숲', author: '무라카미 하루키' },
    { id: 2, title: '1Q84', author: '무라카미 하루키' },
    { id: 3, title: '해변의 카프카', author: '무라카미 하루키' },
  ]

  const {
    isOpen,
    selectedItem,
    highlightedIndex,
    getToggleButtonProps,
    getLabelProps,
    getMenuProps,
    getItemProps,
  } = useSelect({
    items: books,
    itemToString: item => (item ? item.title : ''),
  })

  return (
    <div>
      <label {...getLabelProps()}>책을 선택하세요:</label>
      <div {...getToggleButtonProps()}>
        <span>{selectedItem ? selectedItem.title : '선택...'}</span>
        <span>{isOpen ? '↑' : '↓'}</span>
      </div>
      <ul {...getMenuProps()}>
        {isOpen &&
          books.map((item, index) => (
            <li
              key={item.id}
              {...getItemProps({ item, index })}
              style={{
                backgroundColor:
                  highlightedIndex === index ? '#bde4ff' : 'white',
                fontWeight: selectedItem === item ? 'bold' : 'normal',
              }}
            >
              {item.title} — {item.author}
            </li>
          ))}
      </ul>
    </div>
  )
}
```

```tsx
import { useCombobox } from 'downshift'

function ComboboxDropdown() {
  const allColors = ['Red', 'Green', 'Blue', 'Yellow', 'Purple']
  const [inputItems, setInputItems] = React.useState(allColors)

  const {
    isOpen,
    selectedItem,
    highlightedIndex,
    getToggleButtonProps,
    getLabelProps,
    getMenuProps,
    getInputProps,    // ← useSelect에는 없고 useCombobox에만 있음
    getItemProps,
  } = useCombobox({
    items: inputItems,
    onInputValueChange: ({ inputValue }) => {
      setInputItems(
        allColors.filter(item =>
          item.toLowerCase().startsWith(inputValue.toLowerCase()),
        ),
      )
    },
  })

  return (
    <div>
      <label {...getLabelProps()}>색상 선택:</label>
      <div>
        <input {...getInputProps()} />
        <button {...getToggleButtonProps()}>{isOpen ? '↑' : '↓'}</button>
      </div>
      <ul {...getMenuProps()}>
        {isOpen &&
          inputItems.map((item, index) => (
            <li key={item} {...getItemProps({ item, index })}>
              {item}
            </li>
          ))}
      </ul>
    </div>
  )
}
```

- 실제 동작 로직은 커스텀 훅에 위임, 렌더만 담당!
- 공유 로직은 유틸로 추출해서 여러 훅에 주입

그러면 DownShift가 구조를 변경한것은 단순한 유행인가…?

⇒ 아니다!

1. 같은 철학을 더 리액트 스럽고 단순한 인터페이스로 옮긴 것. 렌더 프롭스는 정의된 로직을 공유하는 용도였고 그 용도를 hooks가 훨씬 단순하게 해결한다고 판단
2. 범용 컴포넌트 하나보다, 더 작고 목적이 뚜렷한 hook API 들이 사용성과 유지보수성 면에서 더 낫다고 판단
갓 컴포넌트 ⇒ 용도별로 쪼갠 headless hook
3. 최신 ARIA 스펙에 맞춰 접근성 모델을 재정비 하기 위해서.. 등

구조는 변경하더라도, 여전히 headless + prop getters + controlled state 철학은 유지되는것이 핵심 
