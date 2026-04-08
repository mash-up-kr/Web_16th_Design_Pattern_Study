# 2. 프로바이더(Provider) 패턴

## 2-1. 정의

어떤 데이터나 상태를 여러 하위 컴포넌트에게 공통으로 제공하기 위해, 상위에서 값을 공급하는 패턴

* 공통 값 공급 — 트리 아래 여러 컴포넌트가 같은 값을 공유
* 중간 전달 제거 — props를 여러 단계에 걸쳐 전달하지 않아도 됨
* 접근 지점 통일 — 필요한 컴포넌트가 직접 값을 꺼내 쓸 수 있음

핵심은 **상위에서 공통 상태나 기능을 제공하고, 하위 컴포넌트가 중간 props 전달 없이 직접 접근하게 만드는 구조**라는 점

## 2-2. 필요성

앱 전반에서 공통으로 쓰는 데이터가 있음

ex) 다크모드, 언어, 유저 정보, 토스트, 모달 제어, 인증 상태

이 값을 props로만 전달하면 다음처럼 중간 컴포넌트들이 단순 전달자 역할만 하게 됨

```tsx
function App() {
  const user = { name: "창우" };

  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {
  return <UserProfile user={user} />;
}

function UserProfile({ user }) {
  return <div>{user.name}</div>;
}
```

이게 바로 **props drilling**

필요한 건 `UserProfile`인데, `Layout`, `Sidebar`도 그 값을 계속 받아서 넘겨야 함  
⇒ 이런 문제를 줄이기 위해 프로바이더 패턴이 필요

## 2-3. Context와의 관계

React에서 Provider 패턴은 보통 **Context를 이용해 구현**

하지만 둘은 완전히 같은 말은 아님

* Context — React가 제공하는 상태 전달 메커니즘
* Provider 패턴 — 그 메커니즘을 활용해서 값을 만들고, 묶고, 공급하는 설계 방식

즉 **Context는 도구**, **Provider 패턴은 그 도구를 이용한 구조적 설계**

## 2-4. 동작 원리

1. Context를 생성함
2. Provider 컴포넌트 안에서 상태와 동작 함수를 만듦
3. `value`로 하위 트리에 공급함
4. 하위 컴포넌트는 `useContext` 같은 방식으로 값을 직접 꺼내 씀

```tsx
function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (nextUser) => setUser(nextUser);
  const logout = () => setUser(null);

  return (
    <UserContext.Provider value={{ user, login, logout }}>
      {children}
    </UserContext.Provider>
  );
}
```

중요한 점은 공급하는 값이 단순 상수 하나가 아니라, **상태 + 상태 변경 함수 + 관련 로직**까지 묶인다는 것  
그래서 Provider는 단순 전달자를 넘어서 **공통 상태 공유의 진입점** 역할을 함

## 2-5. 실제 활용 예시 — ThemeProvider

```tsx
const ThemeContext = createContext(null);

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");

  const toggleTheme = () => {
    setTheme((prev) => (prev === "light" ? "dark" : "light"));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  const context = useContext(ThemeContext);

  if (!context) {
    throw new Error("useTheme must be used within ThemeProvider");
  }

  return context;
}
```

이 구조를 사용하면 트리 깊숙한 곳의 컴포넌트도 중간 부모를 거치지 않고 `theme`, `toggleTheme`에 직접 접근 가능

## 2-6. 기대 효과

**얻는 것**

* props drilling 제거 — 중간 전달 컴포넌트가 줄어듦
* 공통 상태 집중 관리 — 관련 상태와 기능을 한 곳에서 묶어 다룸
* 하위 컴포넌트 재사용성 증가 — 중간 부모 구조에 덜 의존함
* 앱 전역 API처럼 사용 가능 — `AuthProvider`, `ThemeProvider`, `ModalProvider`처럼 역할을 명확히 나눌 수 있음

## 2-7. 트레이드오프

**잃는 것**

* 데이터 흐름 가시성 저하 — props처럼 부모-자식 흐름이 눈에 바로 보이지 않음
* Provider 비대화 위험 — 너무 많은 책임이 한 Provider에 몰리면 God 객체가 됨
* 불필요한 리렌더 가능성 — `value` 객체가 자주 바뀌면 구독 중인 하위 컴포넌트가 함께 다시 렌더링될 수 있음

그래서 Provider는 **역할별로 작게 나누고 이름을 명확하게 짓는 것**이 중요

good

* `AuthProvider`
* `ThemeProvider`
* `ModalProvider`

bad

* `AppProvider`
* `CommonProvider`
* `GlobalProvider` 하나에 너무 많은 책임 넣기

## 2-8. Context는 상태 관리 도구인가?

공부하면서 든 의문은:

> 프로바이더 패턴은 그냥 일반적인 Context 사용과 같은 개념인가?  
> Context는 상태 관리 도구인가?

정리해보면, Context/Provider는 **상태를 전달하는 방법**에 더 가깝고, 상태 자체를 어떻게 설계하고 바꿀지는 `useState`, `useReducer`, 혹은 외부 상태 관리 라이브러리가 담당할 수 있음

즉

* Context/Provider — 전달 메커니즘
* 상태 관리 — 상태를 만들고 바꾸는 로직

전역적으로 자주 읽히는 값이나 공통 기능을 내려주는 데는 매우 적합하지만, **복잡한 비즈니스 상태 전체를 무조건 Context 하나로 해결하려고 하면 오히려 관리가 어려워질 수 있음**

## 2-9. 언제 쓰면 좋을까?

* 깊은 자식들이 같은 값을 써야 할 때
* props drilling이 반복될 때
* 앱 전역 기능을 공통 인터페이스로 제공하고 싶을 때
* 인증, 테마, 언어, 모달, 토스트처럼 여러 화면에서 함께 쓰는 상태/기능이 있을 때
