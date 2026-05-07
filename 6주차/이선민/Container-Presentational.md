# Container/Presentational 패턴

## 정의

Container/Presentational 패턴은 React에서 **관심사의 분리(Separation of Concerns)** 를 구현하기 위한 디자인 패턴이다. 비즈니스 로직과 UI 렌더링 로직을 서로 다른 컴포넌트로 분리하여 코드의 재사용성과 유지보수성을 높인다.

## 기본 개념

이 패턴은 두 가지 역할의 컴포넌트로 구성된다.

1. **Container 컴포넌트**: 데이터 관리 및 비즈니스 로직 담당
2. **Presentational 컴포넌트**: UI 렌더링만 담당

## 동작 방식

예시로 6장의 강아지 사진을 가져와 표시하는 앱을 생각해보자.

- **Container**는 API에서 사진을 다운로드한다.
- **Presentational**은 받은 사진을 화면에 표시한다.

## 코드 예시

### 1. Presentational Component (`DogImages`)

UI 렌더링만 담당한다. props로 받은 데이터를 화면에 그릴 뿐이다.

```javascript
export default function DogImages({ dogs = [] }) {
  return (
    <>
      {dogs.map((dog, index) => (
        <img key={index} src={dog} />
      ))}
    </>
  )
}
```

**특징**

- props를 통해서만 데이터를 수신한다.
- 자체 상태를 관리하지 않는다 (UI 상태 제외).
- 데이터를 수정하지 않는다.
- 스타일시트를 포함한다.

### 2. Container Component (`DogImagesContainer`)

데이터를 가져오고 상태를 관리하며, 그 결과를 Presentational 컴포넌트에 props로 전달한다.

```javascript
import { useState, useEffect } from 'react'
import DogImages from './DogImages'

export default function DogImagesContainer() {
  const [dogs, setDogs] = useState([])

  useEffect(() => {
    fetch('https://dog.ceo/api/breed/labrador/images/random/6')
      .then(res => res.json())
      .then(({ message }) => setDogs(message))
  }, [])

  return <DogImages dogs={dogs} />
}
```

**특징**

- API 데이터 로딩을 담당한다.
- 상태 관리를 담당한다.
- 직접 렌더링하지 않는다.
- 스타일시트를 포함하지 않는다.

## Hooks를 활용한 현대적 접근

React Hooks가 도입된 이후에는 Container 컴포넌트를 따로 두지 않고도 같은 효과를 낼 수 있다. **커스텀 훅(custom hook)** 으로 비즈니스 로직을 분리하는 방식이다.

### 커스텀 훅 작성

```javascript
export default function useDogImages() {
  const [dogs, setDogs] = useState([])

  useEffect(() => {
    fetch('https://dog.ceo/api/breed/labrador/images/random/6')
      .then(res => res.json())
      .then(({ message }) => setDogs(message))
  }, [])

  return dogs
}
```

### Presentational 컴포넌트에서 직접 사용

```javascript
import useDogImages from './useDogImages'

export default function DogImages() {
  const dogs = useDogImages()

  return (
    <>
      {dogs.map((dog, index) => (
        <img key={index} src={dog} />
      ))}
    </>
  )
}
```

Container 래핑 없이도 비즈니스 로직과 뷰를 효과적으로 분리할 수 있다.

## 장점

| 장점 | 설명 |
| --- | --- |
| **관심사의 분리** | Presentational은 순수 UI, Container는 상태/데이터 관리 |
| **재사용성** | 데이터 변경 없이 다양한 목적으로 여러 곳에서 재사용 가능 |
| **유지보수성** | 비즈니스 로직을 모르는 개발자도 UI 컴포넌트만 쉽게 수정 가능 |
| **테스트 용이** | 순수 함수에 가까워 필요한 데이터만 넣으면 테스트하기 쉬움 |
| **일관성** | 공통 Presentational 컴포넌트를 수정하면 전체 앱에 일괄 반영 |

## 단점

| 단점 | 설명 |
| --- | --- |
| **Hooks 대체** | 최근의 React Hooks로 동일한 효과를 달성할 수 있음 |
| **과도한 추상화** | 소규모 앱에서는 오버엔지니어링이 될 수 있음 |
| **보일러플레이트** | 컴포넌트가 추가로 생기면서 파일 수가 늘어남 |

## 활용 사례

1. **대규모 앱**: 여러 페이지에서 같은 UI를 다른 데이터로 보여줘야 할 때
2. **디자인 시스템**: 공통 컴포넌트 라이브러리 구축
3. **팀 협업**: 디자이너와 로직 개발자의 역할 분리
4. **테스트 커버리지**: UI 컴포넌트의 단위 테스트 극대화

## 결론

Container/Presentational 패턴은 **관심사 분리의 명확한 틀**을 제공한다. 다만 최근에는 **React Hooks와 커스텀 훅**으로 더 간단하게 같은 목적을 달성할 수 있기 때문에, 프로젝트 규모와 팀의 필요에 따라 선택적으로 활용하는 것이 효과적이다.

## 참고

- [patterns-dev-kr - Container/Presentational Pattern](https://patterns-dev-kr.github.io/design-patterns/container-presentational-pattern/)
