# HOC 패턴
- 종종 여러 컴포넌트에서 같은 로직을 사용해야 하는 경우가 있습니다.
  - 이런 로직은 컴포넌트의 스타일시트를 설정하는 것일 수 있고, 권한을 요청하거나, 전역 상태를 추가하는 것일 수 있습니다.
- 동일한 로직을 여러 컴포넌트에서 재사용하는 방법 중 하나로 고차 컴포넌트 패턴을 활용하는 방법이 있습니다. 
  - 이 패턴은 앱 전반적으로 재사용 가능한 로직을 여러 컴포넌트들이 사용할 수 있게 해줍니다.
- 고차 컴포넌트란?
  - 고차 컴포넌트란 다른 컴포넌트를 받는 컴포넌트를 뜻합니다.
  - HOC는 인자로 넘긴 컴포넌트에게 추가되길 원하는 로직을 갖고 있습니다.
  - HOC는 로직이 적용된 Element를 반환하게 됩니다.

## Composilng
```jsx
import React from "react";
import withLoader from "./withLoader";
import withHover from "./withHover";

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

export default withHover(
  withLoader(DogImages, "https://dog.ceo/api/breed/labrador/images/random/6")
);
```

## Hooks와 HOC
- 일반적으로 React의 훅은 HOC 패턴을 완전 대체할 수 없습니다.

### HOC의 사용 사례
- 앱 전반적으로 동일하며 커스터마이징 불가한 동작이 여러 컴포넌트에 필요한 경우
- 컴포넌트가 커스텀 로직 추가 없이 단독으로 동작할 수 있어야 하는 경우

### Hooks의 사용 사례
- 공통 기능이 각 컴포넌트에서 쓰이기 전에 커스터마이징 되어야 하는 경우
- 공통 기능이 앱전반적으로 쓰이는 것이 아닌 하나나 혹은 몇개의 컴포넌트에서 요구되는 경우
- 해당 기능이 기능을 사용하는 컴포넌트에게 여러 프로퍼티를 전달해야 하는 경우

## 장점
- 고차 컴포넌트를 사용하면 한 곳에 구현한 로직들을 여러 컴포넌트에서 재사용할 수 있게 됩니다.
- 동일 구현을 여러군데에 직접 구현하며 버그를 만들어낼 확률을 줄일 수 있습니다.
- 로직을 한 곳에서 관리하여 코드를 DRY 하면서 관심사의 분리도 적용할 수 있게 됩니다.

## 단점
- HOC가 반환하는 컴포넌트에 전달하는 props의 이름이 겹칠 수 있습니다.
- HOC를 여러번 조합하여 사용하는 경우 모든 prop이 안에서 병합되므로 어떤 HOC가 어떤 props에 관련되어 있는지 파악하기가 어렵습니다. 앱의 디버깅이나 규모를 키울 때 방해가 될 수 있습니다.
- dev tool에서 컴포넌트 이름이 잡히지 않아 display name 을 사용해야 하는 번거로움까지...

# 추가 정리

## HOC 기본 개념
- HOC는 함수형 패러다임 입니다. HOC는 컴포넌트를 감싸는(wrapping) 함수 
  > (예: withLogger(Button))
- props 위임: 기존 컴포넌트에 추가 props를 주입하거나 동작을 확장
- 순수성 유지: 부수 호과를 최소화하고, 입력 컴포넌트를 수정하지 않음

## 일반적인 규칙과 특징
- 이름 접두사 관례: with[기능] 형태의 컨벤션을 유지해야 함
- props 전달: 원본 컴포넌트의 props를 그대로 유지하며 추가 prop을 주입합니다.
- 정적 구성: 일반적으로 컴포넌트 외부에서 한번만 생성/실행 이 됩니다.

## HOC 장점
- 로직 재사용: 여러 컴포넌트에 동일한 기능을 일관되게 적용 가능
  > 인증, fetching 등등
- 관심사 분리: 비즈니스 로직과 UI 컴포넌트를 분리합니다.
- 조합 가능: 여러 HOC를 체이닝하여 복합적인 기능 추가 가능
  (예: withRouter(withAuth(Profile)))

## HOC 단점
- props 충돌: 여러 HOC가 같은 prop 이름을 사용할 경우 문제가 발생할 수 있습니다.
- 디버깅 어려움: HOC 계층이 깊어지면 React DevTools에서 추적이 복잡해짐
- 명시성 부족: HOC 내부에서 어떤 props가 주입되는지 외부에서 확인하기 어려워짐

## HOC는 테스트에도 사용할 수 있습니다!
- 기존 컴포넌트에 신규 기능을 테스팅 해봐야 할 때, 원본 컴포넌트 소스 코드에는 아무 영향을 주지 않으면서 HOC를 통해 Wrapping하여 테스트를 처리할 수 있습니다.
- 일반적으로 로깅을 Wrapping하는 방식으로 내부에서 일어나는 일을 모니터링 하기도 합니다.
- 마찬가지로 비즈니스 로직에는 전혀 영향을 주지 않으므로 테스팅 및 개발 환경에서 로깅에 매우 적합합니다.
- Before: 컴포넌트 안에 테스트용 로직이 그대로 들어간 경우
  ```jsx
  import { useEffect, useState } from 'react';

  type User = { id: string; name: string };

  export function UserPanel() {
    const [user, setUser] = useState<User>({ id: '1', name: 'Guest' });

    // ❌ 프로덕션 컴포넌트에 테스트 전용 부작용이 섞임
    useEffect(() => {
      if (process.env.NODE_ENV !== 'test') return;
      (window as unknown as { __TEST_SET_USER__?: (u: User) => void }).__TEST_SET_USER__ =
        setUser;
      return () => {
        delete (window as unknown as { __TEST_SET_USER__?: (u: User) => void }).__TEST_SET_USER__;
      };
    }, []);

    return (
      <section>
        <p data-testid="user-name">{user.name}</p>
        <button type="button" onClick={() => setUser((u) => ({ ...u, name: 'Clicked' }))}>
          이름 바꾸기
        </button>
      </section>
    );
  }
  ```
  - 컴포넌트 안에 테스트용 로직이 그대로 들어간 경우 배포 코드에도 클라이언트 환경 분기 처리 등에 대한 내용들이 한 파일에 섞입니다.
- After: withTestingUserUpdate HOC로 테스트 전용 통로만 분리
  ```jsx
  import { useEffect, useState } from 'react';

  type User = { id: string; name: string };

  export type UserPanelProps = {
    user: User;
    onUserUpdate: (next: User | ((prev: User) => User)) => void;
  };

  // ✅ 테스트/윈도우와 무관한 표현만
  export function UserPanel({ user, onUserUpdate }: UserPanelProps) {
    return (
      <section>
        <p data-testid="user-name">{user.name}</p>
        <button type="button" onClick={() => onUserUpdate((u) => ({ ...u, name: 'Clicked' }))}>
          이름 바꾸기
        </button>
      </section>
    );
  }

  type WindowWithTestUser = Window & { __TEST_SET_USER__?: (u: User) => void };

  export function withTestingUserUpdate(
    Component: React.ComponentType<UserPanelProps>
  ): React.FC {
    return function UserPanelWithTestingUserUpdate() {
      const [user, setUser] = useState<User>({ id: '1', name: 'Guest' });

      useEffect(() => {
        if (process.env.NODE_ENV !== 'test') return;
        const w = window as WindowWithTestUser;
        w.__TEST_SET_USER__ = setUser;
        return () => {
          delete w.__TEST_SET_USER__;
        };
      }, []);

      return <Component user={user} onUserUpdate={setUser} />;
    };
  }
  ```
  - 사용처 
    ```jsx
    // 앱에서는 HOC 없이 쓰거나, 상위에서 user 상태만 넘김
    import { UserPanel } from './UserPanel';

    // 테스트 전용 래퍼
    const UserPanelForTest = withTestingUserUpdate(UserPanel);
    ```
  - 순수 UI는 user와 onUserUpdate만 받고, 테스트에서만 필요한 window 세터는 HOC 한 곳에 둡니다.

### 마무리
- [일전에 했던 HOC에 대한 생각 정리](https://github.com/refactor-readers/Modern-React-DeepDive/tree/main/%EA%B9%80%EB%B3%B4%EA%B2%BD/03.%EB%A6%AC%EC%95%A1%ED%8A%B8%20%ED%9B%85%20%EA%B9%8A%EA%B2%8C%20%EC%82%B4%ED%8E%B4%EB%B3%B4%EA%B8%B0#032-%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%A0%95%EC%9D%98-%ED%9B%85%EA%B3%BC-%EA%B3%A0%EC%B0%A8-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EC%A4%91-%EB%AC%B4%EC%97%87%EC%9D%84-%EC%8D%A8%EC%95%BC-%ED%95%A0%EA%B9%8C)
- 디자인 패턴을 공부하면서 이런 개인적인 견해가 얼마나 짧은 생각이었는지 다시 머리를 봉합하게 되는것 같습니다 🤯