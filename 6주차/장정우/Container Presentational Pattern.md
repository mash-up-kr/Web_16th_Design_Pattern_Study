# [Container/Presentational Pattern](https://patterns-dev-kr.github.io/design-patterns/container-presentational-pattern/)

## [What] Container/Presentational Pattern이란?

> "비즈니스 로직(데이터 처리)과 뷰(렌더링)를 두 종류의 컴포넌트로 분리하는 패턴" _(claude)_

> React에서 관심사의 분리를 강제하는 방법으로 Container/Presentational Pattern을 활용할 수 있다. 이 패턴을 통해 비즈니스 로직에서 뷰를 분리해낼 수 있다. _(patterns.dev)_

이 패턴은 2015년 **Dan Abramov**가 제안한 개념이다.

원래는 _"Smart and Dumb Components"_ 라고 불렸으며, "데이터를 어떻게 다룰지 아는 똑똑한 컴포넌트"와 "그저 받아서 그리기만 하는 멍청한 컴포넌트"로 역할을 나눈다는 발상에서 출발했다.

다만 **이 패턴은 현재 React 생태계에서 더 이상 적극적으로 권장되지 않는다.**   
Hooks의 등장으로 자연스럽게 대체되었기 때문이다. Dan Abramov 본인도 이후 자신의 글에 "역사적인 이유로 글을 그대로 두지만 너무 진지하게 받아들이지 말라"는 코멘트를 남겼다.

그럼에도 이 패턴을 살펴보는 이유는, 이 패턴이 시도한 핵심 아이디어 **관심사의 분리** 가 여전히 유효하기 때문이다.  
다만 분리의 단위가 "컴포넌트의 종류"에서 "Hook과 컴포넌트"로 바뀌었을 뿐이다.

### 더 넓은 관점에서

"표현(presentation)과 로직(logic)을 분리한다"는 발상은 React 이전부터 있었다. 다양한 아키텍처 패턴들이 결국 같은 철학을 공유한다.

| 패턴 / 아키텍처 | 표현 측 | 로직 측 |
|:--|:--|:--|
| **MVC** | View | Controller / Model |
| **MVVM** | View | ViewModel |
| **Clean Architecture** | UI Layer | Application / Domain Layer |
| **Container/Presentational** | Presentational | Container |
| **(현대 React)** | Component | Custom Hook |

이름과 단위는 다르지만, 모두 "**그리는 일과 결정하는 일을 같은 곳에 두지 말자**"는 같은 방향을 가리킨다.

## [Patterns.dev] 예시로 알아보기

### Container와 Presentational의 분리

강아지 사진을 가져와 화면에 보여주는 앱이 있다고 하자. 패턴 없이 만들면 데이터 fetch와 렌더링이 한 컴포넌트 안에 뒤섞인다.

```jsx
// 패턴 없이 — 데이터와 뷰가 한 곳에 섞임
class DogImages extends React.Component {
  constructor() {
    super()
    this.state = { dogs: [] }
  }

  componentDidMount() {
    fetch('https://dog.ceo/api/breed/labrador/images/random/6')
      .then(res => res.json())
      .then(({ message }) => this.setState({ dogs: message }))
  }

  render() {
    return this.state.dogs.map((dog, i) => (
      <img src={dog} key={i} alt="Dog" />
    ))
  }
}
```

Container/Presentational 패턴을 적용하면, 데이터를 담당하는 Container와 화면을 담당하는 Presentational로 분리된다.

```jsx
// Container — 데이터 로직만
class DogImagesContainer extends React.Component {
  constructor() {
    super()
    this.state = { dogs: [] }
  }

  componentDidMount() {
    fetch('https://dog.ceo/api/breed/labrador/images/random/6')
      .then(res => res.json())
      .then(({ message }) => this.setState({ dogs: message }))
  }

  render() {
    return <DogImages dogs={this.state.dogs} />
  }
}
```

```jsx
// Presentational — 뷰만
function DogImages({ dogs }) {
  return dogs.map((dog, i) => (
    <img src={dog} key={i} alt="Dog" />
  ))
}
```

`DogImagesContainer`는 데이터를 어떻게 가져올지만 신경 쓰고, 화면에는 아무것도 직접 그리지 않는다. `DogImages`는 props로 받은 `dogs` 배열을 그저 화면에 그릴 뿐, 그 데이터가 어디서 왔는지 알지 못한다.

이 분리 덕분에 `DogImages`는 어디에든 가져다 쓸 수 있다. 다른 데이터 소스에서 가져온 강아지 사진을 그릴 수도, Storybook에서 단독으로 렌더링할 수도, 테스트에서 mock 데이터로 검증할 수도 있다.

### Hooks로 대체

같은 분리를 Hooks로 구현하면 Container 컴포넌트 자체가 필요 없어진다.

```jsx
function useDogImages() {
  const [dogs, setDogs] = useState([])

  useEffect(() => {
    fetch('https://dog.ceo/api/breed/labrador/images/random/6')
      .then(res => res.json())
      .then(({ message }) => setDogs(message))
  }, [])

  return dogs
}
```

```jsx
function DogImages() {
  const dogs = useDogImages()
  return dogs.map((dog, i) => <img src={dog} key={i} alt="Dog" />)
}
```

`useDogImages` 훅이 데이터 fetch 로직을 캡슐화하고, `DogImages`는 그 결과를 받아 화면에 그린다. **로직과 뷰의 분리**라는 목적은 동일하게 달성하면서, **컴포넌트 계층은 추가되지 않았다.**

훅은 Container 컴포넌트가 했던 일을 그대로 해주면서도, 별도의 wrapper 컴포넌트를 만들 필요가 없다. 이것이 현대 React에서 Container/Presentational이 점차 사라진 이유이다.

## [How] Container/Presentational Pattern은 어떻게 구현하는가?

핵심은 **데이터를 가져오고 가공하는 책임과, UI를 렌더링하는 책임을 별개의 단위로 분리하는 것**이다.

### 전통적인 방식 — Container 컴포넌트

기본 구조는 다음과 같다.

```jsx
// Container: 상태와 데이터 흐름 관리
class FeatureContainer extends React.Component {
  state = { /* ... */ }

  componentDidMount() {
    // 데이터 fetch
  }

  handleSomething = () => {
    // 비즈니스 로직
  }

  render() {
    return (
      <FeaturePresentation
        data={this.state.data}
        onAction={this.handleSomething}
      />
    )
  }
}

// Presentational: props만 받아 렌더링
function FeaturePresentation({ data, onAction }) {
  return (
    <div>
      {data.map(item => <Item key={item.id} {...item} />)}
      <button onClick={onAction}>Click</button>
    </div>
  )
}
```

### 현대적인 방식 — Custom Hook

같은 분리를 더 가볍게 구현할 수 있다.

```jsx
// Custom Hook: 로직 캡슐화
function useFeature() {
  const [data, setData] = useState([])

  useEffect(() => {
    // 데이터 fetch
  }, [])

  const handleSomething = useCallback(() => {
    // 비즈니스 로직
  }, [])

  return { data, handleSomething }
}

// Component: 훅을 호출해 사용
function Feature() {
  const { data, handleSomething } = useFeature()

  return (
    <div>
      {data.map(item => <Item key={item.id} {...item} />)}
      <button onClick={handleSomething}>Click</button>
    </div>
  )
}
```

### 분리의 기준

전통적인 패턴에서 두 컴포넌트의 차이는 다음과 같이 정리된다.

| 기준 | Presentational | Container |
|:--|:--|:--|
| 받는 것 | props만 | (없음) — 직접 fetch / 상태 관리 |
| 상태 | 없음 (UI 상태는 OK) | 비즈니스 상태 보유 |
| 외부 의존성 | 없음 (순수함수에 가까움) | API, store 등에 의존 |
| 스타일 | 포함 | 포함하지 않음 |
| 렌더링 | UI 직접 그림 | Presentational에 위임 |

현대적인 방식에서는 이 경계가 다음과 같이 옮겨간다.

| 역할 | 담당 |
|:--|:--|
| 데이터 / 로직 | **Custom Hook** |
| UI / 렌더링 | **Component** |

## [Hum..🤔] 그래도 이 패턴을 써야 할 때가 있을까?

Hooks가 거의 모든 경우를 커버하지만, "**컴포넌트 단위의 분리**" 자체가 여전히 의미 있는 맥락들이 있다.

### 1. React Server Components (RSC)

Next.js 13+의 App Router가 도입한 Server/Client 컴포넌트 분리는, 어떤 면에서는 Container/Presentational의 부활에 가깝다.

- **Server Component**: 데이터 fetch, DB 접근, 비즈니스 로직 — Container의 역할
- **Client Component**: 인터랙션, 상태, 이벤트 핸들링 — Presentational(+ 인터랙션)의 역할

```jsx
// app/dogs/page.tsx — Server Component (Container 성격)
export default async function DogsPage() {
  const dogs = await fetchDogs() // 서버에서 직접 fetch

  return <DogImages dogs={dogs} />
}
```

```jsx
// components/DogImages.tsx — Client Component (Presentational 성격)
'use client'
export function DogImages({ dogs }) {
  return dogs.map((dog, i) => <img src={dog} key={i} alt="Dog" />)
}
```

훅으로 하나의 컴포넌트에 합칠 수 없는 환경(서버/클라이언트 경계)에서는, **데이터 담당자**와 **렌더링 담당자**를 다시 컴포넌트 단위로 나눠야만 한다.

### 2. 디자인 시스템

순수한 Presentational 컴포넌트의 가치는 디자인 시스템에서 여전히 크다. Storybook 같은 도구는 **외부 의존성이 없는 컴포넌트**일수록 잘 다룰 수 있다. props만으로 모든 상태가 표현되는 컴포넌트라면 카탈로그화하기가 훨씬 쉽다.

```jsx
// 디자인 시스템의 Button — 완전한 Presentational
function Button({ variant, size, disabled, onClick, children }) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  )
}
```

이런 컴포넌트는 데이터 소스에 대해 아무것도 모르는 것이 **장점**이다.