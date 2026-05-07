# Container / Presentational 패턴

> 로직과 UI를 분리하는 패턴
> 

컨테이너 / 프레젠테이셔널 패턴은 컴포넌트를 두 종류로 나눠서 생각하는 패턴이다.

```tsx
Container → 데이터 / 상태 / 이벤트 로직
Presentational → 화면 렌더링
```

즉, 하나의 컴포넌트 안에서 데이터 요청, 상태 관리, 로딩 처리, UI 렌더링을 전부 하지 말고

**데이터를 다루는 쪽과 화면을 그리는 쪽을 분리하자** 는 사고방식이다.

### 철학

UI는 최대한 순수하게 두고, 로직은 바깥으로 밀어낸다

## 왜 등장했을까?

초기 React 컴포넌트는 이런 형태가 많았다.

```tsx
function UserPage() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch("/api/user")
      .then(res => res.json())
      .then(setUser);
  }, []);

  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

처음엔 괜찮아 보이지만 이 컴포넌트는 너무 많은 일을 한다.

- 데이터 요청
- 상태 관리
- 로딩 처리
- UI 렌더링

⇒ 관심사가 섞여 있음

그래서 “데이터 처리하는 놈 따로, 화면 그리는 놈 따로 두자” 라는 방식이 필요해졌다.

## 구조

### 1. Container Component

컨테이너 컴포넌트는 어떤 데이터를 보여줄지 결정한다.

```tsx
function UserContainer() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch("/api/user")
      .then(res => res.json())
      .then(setUser);
  }, []);

  return <UserPresenter user={user} />;
}
```

여기서 `UserContainer`는 데이터를 가져오고, 상태를 만들고, 그 결과를 프레젠테이셔널 컴포넌트에 넘긴다.

### 2. Presentational Component

프레젠테이셔널 컴포넌트는 데이터를 어떻게 보여줄지 결정한다.

```tsx
function UserPresenter({ user }) {
  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

여기서 `UserPresenter`는 데이터를 어디서 가져왔는지 모른다.

그냥 props로 받은 데이터를 화면에 그릴 뿐이다.

## 역할 분리

| 역할 | 책임 |
| --- | --- |
| Container | 데이터 가져오기, 상태 관리, 비즈니스 로직 처리 |
| Presenter | props로 받은 데이터를 UI로 렌더링 |

핵심은 Presenter가 최대한 순수해야 한다는 점이다.

```tsx
<UserPresenter user={user} />
```

이렇게만 보면 데이터를 넣으면 UI가 나오는 컴포넌트처럼 사용할 수 있다.

## 이 패턴의 장점

### 재사용성 증가

Presenter는 데이터 출처를 모르기 때문에 다른 곳에서도 재사용하기 쉽다.

```tsx
<UserPresenter user={apiUser} />
<UserPresenter user={mockUser} />
<UserPresenter user={cachedUser} />
```

props의 형태만 맞으면 동일한 UI를 사용할 수 있다.

### 테스트가 쉬워짐

프레젠테이셔널 컴포넌트는 보통 순수 함수에 가깝다.

```tsx
render(<UserPresenter user={mockUser} />);
```

API 요청이나 전역 상태 없이도 테스트할 수 있다.

### 가독성 상승

```tsx
UserContainer → 무슨 데이터를 준비할까?
UserPresenter → 그 데이터를 어떻게 보여줄까?
```

역할이 나뉘어 있으니까 코드를 읽는 관점이 명확해진다.

## 그런데 지금은 왜 덜 쓰일까?

이 패턴은 개념은 중요하지만, 지금 기준에서는 약간 예전 방식에 가깝다.

React Hooks가 등장하면서 로직을 꼭 Container 컴포넌트로 분리하지 않아도 되기 때문이다.

```tsx
function useUser() {
  return useQuery(["user"], fetchUser);
}
```

그리고 UI 컴포넌트에서는 그 hook을 사용한다.

```tsx
function UserPage() {
  const { data: user, isLoading } = useUser();

  if (isLoading) return <div>Loading...</div>;

  return <UserView user={user} />;
}
```

즉, 과거의 Container 역할이 Custom Hook으로 이동한 것에 가깝다.

```tsx
useUser() → 데이터 / 상태 / 비즈니스 로직
UserView → UI 렌더링
```

## 실무에서 자주 보이는 형태

지금 프론트엔드에서 자주 쓰는 것들도 사실 역할로 보면 비슷하다.

```tsx
useQuery()
useMutation()
useAuth()
useForm()
```

이런 것들은 Container 역할에 가깝다.

```tsx
<UserCard />
<Button />
<List />
```

이런 것들은 Presenter 역할에 가깝다.

그래서 지금은 “Container 컴포넌트를 만들어야지” 라기보다는,

```tsx
이 로직을 UI 안에 둬도 되나?
아니면 hook으로 분리해야 하나?
```

이 질문을 하는 게 더 중요하다.

## 트레이드 오프

장점

- 관심사 분리가 명확해짐
- Presenter 재사용이 쉬워짐
- UI 테스트가 쉬워짐

단점

- 파일 수가 늘어남
- props 전달이 많아질 수 있음
- 작은 컴포넌트까지 억지로 나누면 오히려 복잡해짐
- Hooks를 쓰면 Container 컴포넌트가 불필요한 경우가 많음

```tsx
<Presenter a={a} b={b} c={c} d={d} />
```

이런 식으로 props가 너무 많아지면 분리의 장점보다 불편함이 커질 수도 있다.

⇒ 분리할 이유가 명확할 때 사용하는 것이 좋다

## 요약

컨테이너 / 프레젠테이셔널 패턴은 로직과 UI를 분리하는 패턴이다.

> Container는 어떤 데이터를 준비할지 담당하고,

> Presenter는 그 데이터를 어떻게 보여줄지 담당한다.

하지만 현대 React에서는 이 구조가 Custom Hook + UI Component 형태로 많이 바뀌었다.

즉, 지금 중요한 질문은 이것이다.

```tsx
이 로직은 UI 컴포넌트 안에 있어야 하는가?
아니면 Hook으로 분리해야 하는가?
```

컨테이너 / 프레젠테이셔널 패턴은 예전 방식처럼 보일 수 있지만,

**UI는 순수하게, 로직은 외부로** 라는 사고방식은 여전히 중요하다.
