# Container/Presentational 패턴

- React 에서 관심사의 분리를 강제하는 방법은 Container/Presentational 패턴을 이용하는 방법이 있습니다.
- 이 패턴을 통해 비즈니스 로직에서 뷰를 분리해낼 수 있습니다.

## 정의
1. Presentational Components: 데이터가 어떻게 사용자에게 보여질지에 대해서만 다루는 컴포넌트
2. Container Component: 어떤 데이터가 보여질지에 대해 다루는 컴포넌트

## Presentational Component
- Presentational 컴포넌트의 주요 기능은 데이터를 화면에 표현하는 것이며 그 목적을 위해 스타일시트를 포함합니다.
  - 데이터는 건드리지 안습니다.
- Presentational 컴포넌트는 UI 변경을 위한 상태 외에는 상태를 갖지 않습니다.
- Presentional 컴포넌트는 Container 컴포넌트로부터 데이터를 받습니다.

## Container Component
- Container Component의 주요 기능은 Presentational 컴포넌트에 데이터를 전달하는 것 입니다.
- Container Component 자체는 화면에 아무것도 렌더링하지 안습니다.
- Container Component는 아무것도 화면에 그리지 않으니 스타일 시트도 포함하지 않습니다.

> 컴포넌트 네이밍이 명시해주고 사용하는게 패턴을 알고있으면 이해하고 사용하기 쉬운 구조일것 같습니다.

## Hooks
- 대게 Container/Presentational 패턴은 React Hooks로 대체 가능합니다.
- Hooks의 등장으로 Container 컴포넌트 없이도 stateless 컴포넌트를 쉽게 만들 수 있게 되었습니다.
- 훅을 사용하여 Container/Presentational 패턴을 사용한 것 같이 비즈니스 로직과 뷰를 분리할 수 있습니다.
  ```ts
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

## 장점
- 해당 패턴을 사용하면 자연스럽게 관심사의 분리를 구현할 수 있게 됩니다.
  - Presentational Component는 UI를 담당하는 순수함수로 작성하게 되는 반면 Container Component는 상태와 기타 데이터를 책임지게 됩니다.
- Presentational Component는 데이터 변경 없이 화면에 출력할 수 있으므로 앱의 여러곳에서 다양한 목적으로 재사용 가능합니다.
- Presentational Component는 앱의 비즈니스 로직을 수정하지 않으므로 코드베이스에 대한 이해가 깊지 않은 개발자더라도 쉽게 수정이 가능합니다.
  - 공통으로 쓰이는 Presentational Component가 디자인의 요구사항에 따라 수정하면 앱 전체에서 반영됩니다.
- Presentational Component는 테스트하기도 쉽습니다.
- 일반적으로 순수함수로 구현되므로 전체 Mock data store 또한 만들 필요 없이 요구하는 데이터만 인자로 넘겨주면 됩니다.

## 단점
- Container/Presentational 패턴은 비즈니스 로직과 렌더링 로직을 쉽게 분리할 수 있지만 훅을 활용하면 클래스형 컴포넌트를 사용하지 않고도, 또 이 패턴을 따르지 않아도 동일한 효과를 볼 수 있습니다.
- 훅을 사용하더라도 이 패턴을 사용할 수는 있지만 너무 작은 규모의 앱에서는 오버엔지니어링일 수 있습니다.

# 추가 정리

- 책임을 명확히 분리하여 코드의 가독성과 유지보수성을 챙긴다!!

## 특징
- 데이터 처리와 비즈니스 로직에 집중
- 상태를 관리하고 데이터를 가져오는 역할
Presentational 컴포넌트에게 데이터와 함수를 props로 전달
- Redux의 connect, React 훅 등을 사용합니다.
  > 보통 Container에서 사용합니다.

## Presentational Component - 매우 중요 ⭐️
- UI를 어떻게 보여줄지에 집중
- 주로 props를 받아 화면을 렌더링하는 역할
- 내부 상태를 거의 가지지 않음
- 재사용성이 높음
- 스타일링이 포함되는 경우가 많음
- 함수 컴포넌트로 구현하는것이 일반적

