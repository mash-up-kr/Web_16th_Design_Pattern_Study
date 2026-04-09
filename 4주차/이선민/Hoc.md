## HOC(Higher Order Component) 패턴이란?

- **다른 컴포넌트를 인자로 받아 새로운 컴포넌트를 반환하는 함수**
- 여러 컴포넌트에서 공통으로 사용하는 로직을 한 곳에 구현하여 재사용할 수 있다
- 컴포넌트 자체를 수정하지 않고 **래핑(wrapping)** 방식으로 기능을 주입하므로 기존 코드를 건드리지 않는다
- Decorator 패턴의 함수형 Decorator와 동일한 아이디어다

## 기본 구조

```javascript
function withStyles(Component) {
  return props => {
    const style = { padding: '0.2rem', margin: '1rem' }
    return <Component style={style} {...props} />
  }
}

const StyledButton = withStyles(Button)
const StyledText = withStyles(Text)
```

- `Button`, `Text` 컴포넌트를 수정하지 않고도 공통 스타일을 주입할 수 있다
- `with` 접두사를 붙이는 것이 HOC 네이밍 관례다

## 실제 사용 예시: 데이터 로딩

HOC를 활용해 로딩 상태 처리 로직을 컴포넌트로부터 분리할 수 있다.

```javascript
function withLoader(Element, url) {
  return props => {
    const [data, setData] = useState(null)

    useEffect(() => {
      fetch(url)
        .then(res => res.json())
        .then(data => setData(data))
    }, [])

    if (!data) return <div>Loading...</div>
    return <Element data={data} {...props} />
  }
}

function DogImages(props) {
  return props.data.message.map((dog, index) => (
    <img src={dog} alt="Dog" key={index} />
  ));
}

export default withLoader(
  DogImages,
  "https://dog.ceo/api/breed/labrador/images/random/6"
);


// DogImages 컴포넌트가 데이터 페칭 로직을 전혀 몰라도 됨
const DogImagesWithLoader = withLoader(DogImages, 'https://dog.ceo/api/breed/labrador/images/random/6')
```

## HOC 조합

여러 HOC를 중첩하여 기능을 쌓을 수 있다.

```javascript
// 마우스 호버 상태를 제공하는 HOC
function withHover(Component) {
  return props => {
    const [hovering, setHovering] = useState(false)
    return (
      <Component
        {...props}
        hovering={hovering}
        onMouseEnter={() => setHovering(true)}
        onMouseLeave={() => setHovering(false)}
      />
    )
  }
}

function DogImages(props) {
  return (
    <div {...props}>
      {props.hovering && <div id="hover">Hovering!</div>}
      <div id="list">
        {props.data.message.map((dog, index) => (
          <img src={dog} alt="Dog" key={index} />
        ))}
      </div>
    </div>
  );
}

// 두 HOC를 조합
export default withHover(withLoader(DogImages, 'https://dog.ceo/api/breed/labrador/images/random/6'))
```

- 안쪽 HOC(`withLoader`)가 먼저 적용되고, 바깥쪽 HOC(`withHover`)가 그 결과를 감싼다
- 조합이 많아질수록 컴포넌트 트리가 깊어지고 데이터 흐름 파악이 어려워진다

## Hooks와의 비교

몇몇 상황에서 HOC패턴은 Hooks로 대체 가능하다.
withHover HOC을 useHover로 대체해보자

```javascript
import { useState, useRef, useEffect } from "react";

export default function useHover() {
  const [hovering, setHover] = useState(false);
  const ref = useRef(null);

  const handleMouseOver = () => setHover(true);
  const handleMouseOut = () => setHover(false);

  useEffect(() => {
    const node = ref.current;
    if (node) {
      node.addEventListener("mouseover", handleMouseOver);
      node.addEventListener("mouseout", handleMouseOut);

      return () => {
        node.removeEventListener("mouseover", handleMouseOver);
        node.removeEventListener("mouseout", handleMouseOut);
      };
    }
  }, [ref.current]);

  return [ref, hovering];
}

function DogImages(props) {
  const [hoverRef, hovering] = useHover();

  return (
    <div ref={hoverRef} {...props}>
      {hovering && <div id="hover">Hovering!</div>}
      <div id="list">
        {props.data.message.map((dog, index) => (
          <img src={dog} alt="Dog" key={index} />
        ))}
      </div>
    </div>
  );
}

export default withLoader(
  DogImages,
  "https://dog.ceo/api/breed/labrador/images/random/6"
);
```

| | HOC | Hooks |
|---|---|---|
| 로직 재사용 방식 | 컴포넌트를 감싸는 래퍼 반환 | 컴포넌트 내부에서 직접 사용 |
| 컴포넌트 트리 | 중첩될수록 깊어짐 | 트리 구조 변화 없음 |
| Props 커스터마이징 | 어려움 (충돌 가능성) | 용이함 |
| 적합한 상황 | 모든 컴포넌트에 동일한 로직 적용 | 컴포넌트마다 다른 동작이 필요할 때 |

```javascript
// HOC 방식 — 트리가 깊어짐
<withAuth>
  <withLayout>
    <withTheme>
      <Component />
    </withTheme>
  </withLayout>
</withAuth>

// Hooks 방식 — 트리 변화 없음
function Component() {
  const auth = useAuth()
  const theme = useTheme()
  // ...
}
```

**HOC 추천 상황**
- 모든 컴포넌트에 동일하고 고정된 로직이 필요한 경우
- 컴포넌트가 HOC의 구현 세부 사항을 몰라도 독립적으로 작동해야 하는 경우

**Hooks 추천 상황**
- 로직이 컴포넌트마다 커스터마이징이 필요한 경우
- 일부 컴포넌트만 해당 로직을 사용하는 경우
- 여러 props를 전달해야 해서 충돌 위험이 있는 경우

## 장점

- **로직 재사용**: 한 곳에 구현한 로직을 여러 컴포넌트에서 중복 없이 사용할 수 있다
- **관심사의 분리**: UI 렌더링과 데이터 페칭, 인증 등의 로직을 분리할 수 있다
- **DRY 원칙 준수**: 동일한 코드가 여러 컴포넌트에 흩어지는 것을 방지한다

## 단점

- **Props 충돌**: HOC가 주입하는 prop 이름이 기존 prop과 겹칠 경우 덮어써질 수 있다
  ```javascript
    function withStyles(Component) {
        return props => {
            const style = { padding: '0.2rem', margin: '1rem' }
            return <Component style={style} {...props} />
        }
    }

    const Button = () = <button style={{ color: 'red' }}>Click me!</button>
    const StyledButton = withStyles(Button)
    ```
    HOC를 만들 땐 이런 상황을 고려해야 하며 prop 병합을 통해 해결한다.
    ```javascript
    // 기존 스타일을 병합해 충돌을 방지하는 방법
    function withStyles(Component) {
        return props => {
        const style = {
            padding: '0.2rem',
            margin: '1rem',
            ...props.style  // 기존 스타일을 우선 병합
        }
        return <Component style={style} {...props} />
        }
    }
    ```
- **복잡한 데이터 흐름**: 여러 HOC를 조합하면 어떤 HOC가 어떤 prop을 제공하는지 추적하기 어렵다
- **디버깅 어려움**: 중첩 구조로 인해 어느 HOC에서 문제가 발생했는지 파악이 어렵다
- **React DevTools 가독성 저하**: HOC를 많이 쓸수록 컴포넌트 트리가 복잡하게 보인다

## 참고자료

- https://patterns-dev-kr.github.io/design-patterns/hoc-pattern/
