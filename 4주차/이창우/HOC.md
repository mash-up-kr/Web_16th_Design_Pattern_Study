# HOC

Higher-Order-Component의 줄임말 ⇒ 고차 컴포넌트

> 컴포넌트를 입력받아서, 새로운 컴포넌트를 반환하는 함수
> 
- 어떤 컴포넌트가 있고, 그 컴포넌트에 기능을 덧씌워서, 강화된 새 컴포넌트를 만드는 방식

```tsx
function withSomething(Component) {
  return function EnhancedComponent(props) {
    return <Component {...props} />;
  };
}
```

입력 : 컴포넌트
출력 : 더 강해진 컴포넌트..!

```tsx
const numbers = [1, 2, 3];
numbers.map((x) => x * 2);
```

Higher-Order-Function 고차 함수 개념에서 착안된 패턴. 

### 필요

여러 컴포넌트에서 같은 로직이 반복된다
⇒ ex ) 인증 여부, 로딩 처리, 권한 체크, 데이터 처리, 에러… 로깅…. 
여러 사용처가 생기면, 중복이 생김 ⇒ 컴포넌트 자체를 감싸 공통 기능을 추가하자!

```tsx
function ProfilePage() {
  const user = useUser();

  if (!user) {
    return <div>로그인이 필요합니다.</div>;
  }

  return <div>프로필 페이지</div>;
}

function SettingsPage() {
  const user = useUser();

  if (!user) {
    return <div>로그인이 필요합니다.</div>;
  }

  return <div>설정 페이지</div>;
}
```

- 각 페이지마다 로그인 반복

```tsx
function withAuth(Component) {
  return function AuthenticatedComponent(props) {
    const user = useUser();

    if (!user) {
      return <div>로그인이 필요합니다.</div>;
    }

    return <Component {...props} user={user} />;
  };
}

function ProfilePage({ user }) {
  return <div>{user.name}의 프로필</div>;
}

export default withAuth(ProfilePage);
```

- withAuth라는 고차 컴포넌트를 구현, 이후 소비처마다 해당 컴포넌트로 감싸 기능 구현
- 소비처는 인증 로직을 몰라도 됨 ⇒ 관심사 분리

> HOC의 핵심은 공통 관심사를 바깥에서 구현해서 주입하는 것!
> 

+

컴포넌트의 책임

- 화면 렌더, 인터렉션 처리, 단위 로직 표현 …

여기에 인증, 로깅, 권한 체크까지 추가하면 관심사 섞임 ⇒ 결합도 증가

### 구조

```tsx
function withX(WrappedComponent) {
  return function EnhancedComponent(props) {
    // 공통 로직
    return <WrappedComponent {...props} extraProp="something" />;
  };
}
```

일반적인 형태, 원본 컴포넌트를 감싸고 ⇒ 값을 주입한 후 ⇒ 강화 컴포넌트를 반환함

```tsx
// 프롭스 주입
<Component {...props} user={user} />

// 조건부 렌더링
if (!authorized) return <Forbidden />;

// 공통 사이드이펙트 실행
useEffect(() => {
  console.log('page viewed');
}, []);

// 특정 레이아웃/컨테이너 래핑
return (
  <Layout>
    <WrappedComponent {...props} />
  </Layout>
);
```

- 컴포넌트를 직접 수정하지 않는다
- 바깥에서 감싸 공통 로직을 추가한다
- 새 컴포넌트를 반환한다

공부하면서 든 의문 : 대상하는 범위가 컴포넌트 일 뿐, 개념 자체는 그냥 데코레이터 아닌가?

### 특징

- 원본 컴포넌트를 직접 수정하지 않는다 ⇒ 객체 지향 보존 원칙
- 재사용 단위가 ‘컴포넌트’다.
    - 커스텀 훅 = 재사용 단위가 함수, HOC = 재사용 단위가 컴포넌트
- Injection 개념이 강하다
    - user, theme, data, isMobile, permissions 와 같은 값 주입할 때 자주사용

```tsx
const Enhanced = withTheme(withAuth(withLogging(ProfilePage)));
```

체이닝도 가능 .. but 주입된 프롭스 위치 확인하려면 다 뜯어봐야함 가독성 안좋고 디버깅 어려움

```tsx
const enhance = compose(
  withTheme,
  withAuth,
  withLogging
);

const EnhancedProfile = enhance(ProfilePage);
```

이런 방식도 등장! 가독성 개선 

## 트레이드 오프

장점

- 공통 로직 재사용
- 관심사 분리
- 선언적 적용
    - 컴포넌트 외부에서도 어떤 기능이 주입되었는지 식별가능
- 원본 객체 보존

단점

- 프롭스 충돌..!
    - 기존에 있던 프롭스에서 새로 강화되며 주입하는 경우 프롭스가 충돌이남
    - props namespace 오염 가능 ⇒ 근데 추적도 어려움..
- 추적이 어려움
- 래퍼 지옥 ( wrapper hell )
- 타입스크립트와 적합하지 않음
    - 원래 props, 주입 props, 외부 props .. 관계를 타입으로 표현하기 애매함. 설계가 복잡
- CustomHook 대체가능
    - useAuth(), useUserData() ..

### HOC vs

HOC 

- 컴포넌트를 입력받아 새 컴포넌트를 반환
- Props 주입에 강함
- 컴포넌트 외부에서 호출

Custom Hook

- 로직 재사용하는 함수
- 컴포넌트 내부에서 호출

> 외부 래핑, 내부 호출 차이
> 

Render Props

- 함수 형태의 props 받아 제어권 넘김

> 함수로 렌더링 위임, 컴포넌트를 래핑해서 기능 추가
> 

### 설계

좋은 HOC

1. 원본 컴포넌트를 변형하지 말 것
2. 관련 없는 props는 투명히 전달할 것 
3. name space 설계가 될 것
4. display Name을 설정할 것
5. 가능하면 순수하게 유지 할것 

### 정리

대부분은 customHook으로 대체가 가능하지만

- 컴포넌트에 기능을 외부에서 주입하고 싶을 때
- 공통 레이아웃이나 보호막, 트래킹을 선언적으로 붙이고 싶을 때
- 컴포넌트 단위의 강화가 필요할 때

HOC도 고려할만함

### 공부 중 의문

⇒ 외부 래핑이 목적이면 상위 컴포넌트에서 children을 통해 주입하면 안되는건가?

```tsx
export default function AuthVerifyPage() {
  return (
    <NoSSR>
      <VerifyCodeView />
    </NoSSR>
  );
}
```

- 컴포넌트로 함수를 받지 말고, 그냥 부모 래퍼로 만들고 칠드런으로 합성하면 안되나?

```tsx
const ProtectedProfile = withAuth(ProfilePage);
const ProtectedSettings = withAuth(SettingsPage);
const ProtectedAdmin = withAuth(AdminPage);
```

⇒ 단순한 공통사 분리 부분에서는 가능하지만..! 해당하는 방식은 컴포넌트 자체를 강화하는 것이 아님

- 지금 구조는 결국에 공통된 기능을 위해서 매번 래핑을 해야하지만, HOC는 강화된 컴포넌트를 반환하기에, 이 컴포넌트 자체를 재사용할 수 있음
- 칠드런 래핑 ⇒ 이 화면을 지금 강화하자, HOC ⇒ 이 컴포넌트 자체를 적용된 버전으로 바꿔 쓰자

> 사용 시점 조합과, 정의 시점 강화라는 의도 차이가 있음
> 
