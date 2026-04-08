_* Mash-UP 16기 웹 팀 내 [디자인패턴 스터디](https://github.com/mash-up-kr/Web_16th_Design_Pattern_Study)에서 진행된 내용으로, [patterns.dev](https://www.patterns.dev/)를 읽고 정리하는 방식으로 진행되었습니다._
_** 정리하다 궁금한 내용(궁금한 내용이 아니더라도 스터디 시간에 얘기하고 싶은)은 ❓ 이모지로 정리해 두었습니다._

> **2주차 자료**
[Provider Pattern](https://www.patterns.dev/vanilla/provider-pattern/)
[Facade Pattern](https://www.patterns.dev/)
[Render Props Pattern](https://www.patterns.dev/react/render-props-pattern/)

---

## Provider Pattern
> 데이터나 기능을 텀포넌트 트리의 하위 요소들이 접근할 수 있도록 하는 디자인 패턴

중간 단계의 컴포넌트들이 데이터를 직접 사용할 필요가 없더라도, 하위 컴포넌트가 상위 컴포넌트에서 내려주는 데이터를 필요로 한다면 그 데이터를 전달받을 수 있게 하기 위헤 `props drilling`이 일어난다. 때문에 `props drilling` 없이 하위 컴포넌트가 데이터에 접근하기 위해 Provider 패턴이 필요하다.

### 동작 방식
1. 모든 컴포넌트를 `Provider`로 감싼다. Provider는 HOC로 `Context` 객체를 제공한다. React가 제공하는 `createContext` 메서드를 활용하여 Context 객체를 만들어낼 수 있다. 
2. Provider 컴포넌트는 `value`라는 prop으로 하위 컴포넌트들에 내려줄 데이터를 받는다. 이 컴포넌트의 모든 자식 컴포넌트들은 해당 provider를 통해 value prop에 접근할 수 있다. 
3. 각 컴포넌트는 `useContext` 훅을 활용하여 `data`에 접근할 수 있다. 👉 이 훅은 data와 연관된 DataContext를 받아 data를 읽고 쓸 수 있는 context 객체를 제공한다.


### Provider 패턴이 필요한 이유
단순히 생각하면 props-drilling을 막아 필요없는 컴포넌트에 불필요하게 prop을 받을 필요가 없으며, 리팩토링 과정에 prop의 이름을 변경하기 위해 모든 컴포넌트를 찾아다닐 필요가 없다. 단순하게는 props-drilling을 막기 위해서만 필요한 것 같지만, 설계 관점에서 다양한 장점을 가지고 있다. 

1. 의존성 주입
Provider 패턴은 의존성 주입의 UI 버전이다. 컴포넌트가 필요한 데이터를 외부에서 주입받게 함으로써, 컴포넌트 자체는 데이터가 어디서 오는지 몰라도 되게 만든다. 👉 컴포넌트의 **재사용성**을 극대화한다.

2. 단일 진실 공급원
애플리케이션 전반에서 공유되어야 하는 상태를 한 곳에서 관리하고, 이를 하위 트리에 배포함으로써 데이터 불일치 문제를 방지한다. 

3. 지연 초기화
앱이 시작되자마자 메모리를 잡아먹고 초기화 로직을 시작하는 전역 싱글톤 객체와는 달리, 프로바이더는 해당 프로바이더가 마운트 될 때 로직이 가동되기 때문에 컴포넌트 트리가 렌더링되는 시점에 맞춰 자원을 초기화할 수 있다.

4. Composition
프로바이더는 여러 개를 중첩시켜 필요한 기능을 조합할 수 있기 때문에 Mixin의 장점인 기능 조립을 살린 반면 단점인 이름 충돌을 극복한 형태라고 볼 수 있다. 

```javascript
const AppProvider = ({ children }) => {
  return (
    <QueryClientProvider client={queryClient}> {/* 데이터 캐싱 장착 */}
      <ThemeProvider theme={darkTheme}>        {/* 스타일 시스템 장착 */}
        <AuthProvider>                         {/* 인증 로직 장착 */}
          <SocketProvider>                     {/* 실시간 통신 장착 */}
            {children}
          </SocketProvider>
        </AuthProvider>
      </ThemeProvider>
    </QueryClientProvider>
  );
};
```
위처럼 인증, 테마, 캐싱을 별도의 프로바이더로 나누면 각 로직이 완전히 격리된다. 이를 통해 한 가지 로직을 수정하거나 테스트할 때 다른 로직을 건드릴 필요가 없게 된다.


### 성능 주의점
프로바이터 패턴의 가장 큰 단점은 **불필요한 전체 리렌더링**으로 인한 성능 이슈이다. React의 `Context.Provider`는 `value` 프로퍼티가 변경될 때마다 이를 구독하는 모든 하위 컴포넌트를 강제로 다시 렌더링한다.


#### 해결책 1. 값의 메모이제이션
프로바이더 내부 상태가 변경될 때마다 `value`에 전달하는 객체를 새로 생성하면, 실제 데이터가 바뀌지 않더라도 참조값이 달라져 하위 컴포넌트들이 리렌더링된다.

```typescript
const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  
  // ❌ 매 렌더링마다 새로운 객체가 생성됨 👉 성능 저하
  // const value = { user, login: () => {} };

  // ✅ 데이터나 함수가 바뀔 때만 객체를 새로 생성함
  const value = useMemo(() => ({
    user,
    login: async () => { /* ... */ }
  }), [user]); // user가 바뀔 때만 참조값 변경!!

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};
```


#### 해결책 2. 컨텍스트 분리
보통의 컴포넌트는 데이터(상태)를 사용하거나, 그 상태를 변경하는 함수를 사용한다. 즉, 둘을 한 컨텍스트에 둘 경어 상태는 자주 바뀌지만 함수는 거의 바뀌지 않음에도 불구하고 상태만 쓰는 컴포넌트 때문에 함수만 쓰는 컴포넌트까지 리렌더링된다. 때문에 이 둘을 분리하는 방식을 사용함으로써 함수만 쓰는 컴포넌트는 리렌더링 되지 않도록 할 수 있다.
```typescript
const UserStateContext = createContext(null);
const UserDispatchContext = createContext(null);

export const UserProvider = ({ children }) => {
  const [user, setUser] = useState({ name: 'JeongWoo' });

  // 함수는 절대 바뀌지 않도록 useCallback 처리
  const updateName = useCallback((name) => setUser({ name }), []);

  return (
    <UserStateContext.Provider value={user}>
      <UserDispatchContext.Provider value={updateName}>
        {children}
      </UserDispatchContext.Provider>
    </UserStateContext.Provider>
  );
};
```
위의 예제처럼 작성하면, `updateName`만 사용하는 버튼 컴포넌트는 `user.name`이 바뀌어도 리렌더링 되지 않는다.


#### 해결책 3. 컴포넌트 합성
리액트의 특성을 이용한 방식이다. 프로바이더가 관리하는 상태가 바뀌면 프로바이더 자신은 리렌더링되지만, `chlidren`으로 주입된 컴포넌트들은 프로바이더의 자식이 아니라 외부에서 들어온 prop이기 때문에 변경되지 않았다고 판단하고 리렌더링을 건너뛴다. 
```typescript
// ❌ 프로바이더 내부에서 컴포넌트를 직접 호출하면 안 됨!!!
const BadProvider = () => {
  const [count, setCount] = useState(0);
  return (
    <Context.Provider value={count}>
      <HeavyComponent /> {/* count가 바뀔 때마다 HeavyComponent도 리렌더링됨 */}
    </Context.Provider>
  );
};

// ✅ children을 사용하면 HeavyComponent는 보호됨
const GoodProvider = ({ children }) => {
  const [count, setCount] = useState(0);
  return (
    <Context.Provider value={count}>
      {children} 
    </Context.Provider>
  );
};
```

> ❓ 추가로... Provider 패턴을 모든 곳에 적용하는 건 너무 과한가? 에 대한 Reddit 글이 있어서 공유해봅니다!
[Provider 패턴을 모든 곳에 사용하기 — 너무 과한 걸까?](https://www.reddit.com/r/reactjs/comments/1o7yt1r/using_the_provider_pattern_everywhere_is_it_too/?tl=ko)

---

## Decorator Pattern
> 객체를 변경하지 않고도 동적으로 기능을 추가할 수 있게 해주는 구조적 디자인 패턴

클래스나 객체에 새로운 기능을 추가하고 싶을 때, 가장 먼저 떠오르는 방법은 **상속**이다. 하지만 상속은 부모 클래스와 자식 클래스 간의 결합도가 매우 높고, 다양한 기능의 조합이 필요할 때마다 새로운 서브 클래스를 만들어야 하는 '클래스 폭발' 문제를 야기한다.

**Decorator 패턴**은 객체를 '장식자'라고 불리는 특별한 래퍼 객체 안에 넣어서 기능을 확장한다. 이는 **합성**을 활용한 방식으로, 런타임에 유연하게 기능을 갈아끼우거나 조합할 수 있다는 강력한 장점이 있다.

### 동작 방식
> 1. **Component**: 원본 객체와 데코레이터가 공통으로 가져야 할 인터페이스를 정의한다.
2. **ConcreteComponent**: 기능을 추가할 대상이 되는 실제 객체다.
3. **Decorator**: 원본 객체를 참조하며, 원본과 동일한 인터페이스를 유지한다.
4. **ConcreteDecorator**: 실제로 기능을 추가하는 클래스나 함수다.

### JS에서의 기본적인 구현
자바스크립트에서는 클래스뿐만 아니라 고차 함수를 통해서도 데코레이터 패턴을 매우 쉽게 구현할 수 있다.

```javascript
// 1. 원본 클래스
class BasicRequest {
  send(data) {
    console.log(`Sending data: ${data}`);
  }
}

// 2. 로그 기능을 추가하는 데코레이터 함수
function loggingDecorator(request) {
  const originalSend = request.send;

  // 기존 메서드를 가로채서 기능을 추가함
  request.send = function(data) {
    console.log(`[LOG] Attempting to send at ${new Date().toISOString()}`);
    originalSend.call(this, data);
    console.log(`[LOG] Send complete.`);
  };

  return request;
}

// 사용 예시
const req = new BasicRequest();
const decoratedReq = loggingDecorator(req);

decoratedReq.send("Hello World"); 
// [LOG] Attempting to send...
// Sending data: Hello World
// [LOG] Send complete.
```

### 함수형 프로그래밍에서의 데코레이터
함수로 인자를 받아 기능을 확장한 새 함수를 반환하는 방식으로 사용한다. 
```javascript
const multiply = (a, b) => a * b;

// 실행 시간을 측정하는 데코레이터
const withTimer = (fn) => (...args) => {
  console.time('ExecutionTime');
  const result = fn(...args);
  console.timeEnd('ExecutionTime');
  return result;
};

const timedMultiply = withTimer(multiply);
timedMultiply(5, 10); // ExecutionTime: 0.01ms, 50 반환
```

### HOC에서의 데코레이터 패턴
React의 HOC는 컴포넌트 단위의 데코레이터 패턴이ㅏㄷ. 컴포넌트를 감싸서 Props를 주입하거나, 권한 로직을 추가한다.
```javascript
// 권한 체크 데코레이터
const withAdminAuth = (WrappedComponent) => {
  return (props) => {
    const { user } = useAuth(); // 사용자 정보 훅

    if (!user.isAdmin) {
      return <ForbiddenView />;
    }

    // 원본 컴포넌트에 추가 데이터나 기능을 장식하여 반환
    return <WrappedComponent {...props} adminToken={user.token} />;
  };
};

const SettingsPage = ({ adminToken }) => <div>Admin Token: {adminToken}</div>;
export default withAdminAuth(SettingsPage);
```

### 데코레이터 vs 상속
| 비교 항목 | 상속 (Inheritance) | 데코레이터 (Decorator) |
| :--- | :--- | :--- |
| **관계 정의** | **is-a** (자식은 부모의 일종이다) | **has-a** (장식자는 객체를 소유한다) |
| **바인딩 시점** | 정적 (컴파일 타임) | 동적 (런타임) |
| **결합도** | **높음** (부모의 변화에 취약함) | **낮음** (객체 간 독립성 유지) |
| **유연성** | 낮음 (구조 변경이 어려움) | **매우 높음** (자유로운 기능 조합) |
| **복잡도** | 초기 설계는 쉬우나 관리가 어려워짐 | 초기 구조 설계는 복잡하나 확장이 쉬움 |
| **주요 원칙** | 재사용성 위주 | **OCP** (개방-폐쇄 원칙) 준수 |

만약 커피숍 키오스크 시스템을 만든다고 가정했을 때, 

```javascript
class Coffee { cost() { return 5000; } }

// 우유 추가
class MilkCoffee extends Coffee { cost() { return super.cost() + 500; } }

// 설탕 추가
class SugarCoffee extends Coffee { cost() { return super.cost() + 300; } }

// 우유 + 설탕 추가? -> 새로운 클래스 필요
class MilkSugarCoffee extends MilkCoffee { cost() { return super.cost() + 300; } }

// 시럽 추가? -> 또 다른 조합의 클래스들이 기하급수적으로 늘어남 ㅠㅠ
```
위처럼 상속의 경우 재료가 늘어날 때마다 새로운 클래스를 생성해야 한다.

그러나 데코레이터 패턴을 사용했을 경우,
```typescript
// 원본 객체
class SimpleCoffee {
  cost() { return 5000; }
}

// 데코레이터들 (원본과 동일한 인터페이스를 유지하며 감싼다!!)
const withMilk = (coffee) => ({
  cost: () => coffee.cost() + 500
});

const withSugar = (coffee) => ({
  cost: () => coffee.cost() + 300
});

const withSyrup = (coffee) => ({
  cost: () => coffee.cost() + 700
});

// 실행 시점에 자유롭게 조합
let myCoffee = new SimpleCoffee();

// 손님이 우유와 시럽을 선택했다면?
myCoffee = withMilk(myCoffee);
myCoffee = withSyrup(myCoffee);

console.log(myCoffee.cost()); // 6200 (5000 + 500 + 700)
```
위처럼 새로운 재료가 추가되어도 새로운 데코레이터 함수를 추가하면 된다. 


### 장단점
> **장점**
- **단일 책임 원칙** : 복합적인 기능을 가진 클래스를 작은 단위의 데코레이터로 나눌 수 있다.
- **개방-폐쇄 원칙** : 기존 코드를 건드리지 않고도 새로운 기능을 무한히 확장할 수 있다.
- **런타임 유연성** : 상속의 경우 컴파일 타임에 결정되지만, 데코레이터는 코드 실행 중에 특정 조건에 따라 장착하거나 탈착할 수 있다. 


> **단점**
- **디버깅 복잡** : 객체가 여러 겹의 데코레이터로 감싸져 있으면 에러가 발생했을 때 호출 스택을 추적하기 어렵다.
- **순서 의존성** : 데코레이터가 적용되는 순서가 결과에 영향을 줄 수 있다.



## Render Props

> **컴포넌트 간에 로직을 공유하기 위해, 값이 아닌 '함수'를 prop으로 전달하는 디자인 패턴**

리액트에서 컴포넌트는 재사용의 기본 단위다. 하지만 때로는 **"동일한 로직을 가지지만, UI는 완전히 다른"** 상황이 발생한다. 예를 들어, 마우스 위치를 추적하는 로직은 동일하지만 이를 화면에 텍스트로 보여줄지, 이미지의 위치로 사용할지는 컴포넌트마다 다를 수 있다.

Render Props 패턴은 컴포넌트 내부에서 무엇을 렌더링할지 직접 결정하지 않는다. 대신 **"어떻게 렌더링할지 정의된 함수"**를 호출하여 그 결과를 렌더링한다. 이를 통해 '상태 관리 로직'과 'UI 표현'을 완전히 격리할 수 있다.

### 동작 방식과 기본 구현

### 핵심 개념
컴포넌트가 `render`라는 이름의 prop(이름은 자유지만 관례상 `render` 혹은 `children`을 사용)으로 함수를 받고, 내부 상태를 그 함수의 인자로 넘겨주며 호출한다.

### Example
마우스의 위치를 계산하는 로직은 하나로 유지하고, 출력 방식은 외부에서 결정한다.

```javascript
import React, { useState } from 'react';

// 로직을 담당하는 컴포넌트
const MouseTracker = ({ render }) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (event) => {
    setPosition({
      x: event.clientX,
      y: event.clientY
    });
  };

  return (
    <div style={{ height: '100vh' }} onMouseMove={handleMouseMove}>
      {/* 내부 로직(position)을 인자로 넘기며 render 함수 호출 */}
      {render(position)}
    </div>
  );
};

// 3. 사용 측면: 로직은 가져오되 UI는 자유롭게 정의
const App = () => (
  <MouseTracker 
    render={({ x, y }) => (
      <h1>마우스 위치는 현재 ({x}, {y})입니다.</h1>
    )} 
  />
);
```

### 상태 끌어올리기와 비교했을 때의 장점
![](https://velog.velcdn.com/images/dawnww/post/49ffcb8c-dcf1-4546-b1cc-16a0a1168a49/image.png)
위와 같이 최상위 컴포넌트가 있고, 사용자 입력을 상태로 관리하는 `Input` 컴포넌트, 입력을 각각 켈빈과 화씨로 변환하여 보여주는 `kelbin`, `Fahrenheit` 컴포넌트가 있다고 가정하자. 

```javascript
function Input({value, handleChange}) {
  return (<input value={value} onChange={(e) => handleChange(e.target.value}} />);
};

function App() {
  const [value, setValue] = useState("");
  
  return (
    <div className="App">
      <h1>Temperature Converter</h1>
      <Input value={value} handleChange={setValue}/>
      <Kelvin value={value} />
      <Fahreheit value={value} />
    </div>
  );
}
```
일반적으로 상태를 공유하기 위해서는 위와 같은 방식으로 `App`에서 `handleChang`라는 함수 prop을 `Input`에게 내려주고, `Input`은 사용자 입력이 발생하면 `hanleChange`를 사용자 입력과 함께 호출하여 `App`에 사용자 입력을 전달하고, `App`에 도달한 사용자 입력 상태는 캘빈과 화씨 컴포넌트로 전달된다. 

👉 그러나 이때 `App` 컴포넌트가 사용자 입력과 관련 없는 컴포넌트를 하위 컴포넌트로 가지고 있다면, 해당 컴포넌트는 관련이 없음에도 렌더링에 영향을 줄 수 있다. 


그러나 Render Props 패턴으로 구현할 경우, render로 렌더링 로직을 내려주고 Input에서 이를 호출하도록 할 수 있다. 

```javascript
// Input
function Input(props){
  const [value, setValue] = useState("");
  
  return (
    <>
      <input 
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Temp in ℃"
      />
      {props.render(value)}
    </>
  )
}

// App
function App(){
  return (
    <div className="App">
      <h1>Temperature Converter</h1>
      <input 
        render={(value) => (
          <>
            <Kelvin value={value} />
            <Fahrenheit value={value} />
          </>
        )}
      />
    </div>
  )
}
```

위 코드는 Input에서 사용자 입력을 관리하고, 내려온 render prop에 자신의 사용자 입력 상태를 인자로 전달하여 호출한다. App은 Input에게 render prop의 함수를 내려주고, value를 받으면 이를 이용하여 켈빈, 화씨 컴포넌트를 렌더링한다. 이때 render 함수의 value라는 파라미터는 Input이 render 함수를 호출하며 전달된 사용자 입력 값이 된다. 

👉 render props는 무엇을 렌더링할 것인지에 대한 렌더링 로직을 상위 컴포넌트가 갖고 있고, 하위 컴포넌트인 Input은 자신이 관리하고 있는 상태를 인자로 내려온 render prop을 호출하여 특정 데이터/로직이 필요한 컴포넌트들이 데이터/로직을 공유하게 하는 패턴인 것이다. 

### VS Hook

React Hooks가 등장했을 때, 커뮤니티는 "Render Props는 이제 죽었다"라고 선언했다. 하지만 2026년인 지금까지도 수많은 고도화된 라이브러리와 복잡한 디자인 시스템에서 이 패턴은 여전히 핵심적인 역할을 수행하고 있다.
Hooks와 Render Props는 모두 '로직의 재사용'이라는 목적을 공유하지만, 데이터를 컴포넌트에 주입하는 방식과 그 계층 구조에서 결정적인 차이를 보인다.

#### Hooks
Hooks는 로직을 컴포넌트 내부로 '흡수'한다. 호출된 Hook은 해당 컴포넌트의 상태와 생명주기의 일부가 된다.
* **장점**: 들여쓰기가 깊어지지 않고 코드의 가독성이 높다. 로직이 컴포넌트 최상단에 나열되어 흐름 파악이 쉽다.
* **한계**: 로직이 컴포넌트 내부로 들어가기 때문에, 부모가 자식의 렌더링 방식에 개입하기 어렵다. 즉, **제어권의 역전** 강도가 낮다.

#### Render Props
Render Props는 로직을 컴포넌트 사이에 '배치'한다. 로직을 가진 컴포넌트가 '어떻게 그릴지'에 대한 결정권을 사용하는 측에 함수로 넘겨준다.
* **장점**: **렌더링 단계에서 결정을 내릴 수 있다.** 특정 상태값에 따라 아예 다른 컴포넌트 구조를 동적으로 생성해야 할 때 강력하다.
* **핵심 지점**: 로직이 UI의 위계와 밀접하게 연관되어야 할 때 사용한다. 예를 들어, 가상 리스트 라이브러리에서 특정 인덱스의 아이템을 어떻게 그릴지는 Hook보다는 Render Props가 훨씬 선언적이다.


### Render Props의 뚜렷한 장점?... 특징?...
1. **디자인 시스템의 슬롯(Slot) 패턴**: 컴포넌트 내부의 특정 위치에 복잡한 UI를 주입해야 할 때, 단순한 `children`으로는 부족하다. 이때 `renderHeader`, `renderFooter` 같은 Render Props는 강력한 확장성을 제공한다.
2. **동적 스코프 공유**: 렌더링 시점에만 알 수 있는 데이터를 기반으로 UI를 분기 처리해야 할 때, Hook은 컴포넌트 전체를 리렌더링시키지만 Render Props는 해당 함수 스코프 내에서 세밀하게 조절 가능하다.
3. **라이브러리 설계의 유연성**: 사용자가 로직은 그대로 쓰되, UI 프레임워크는 자유롭게 선택하게 하고 싶을 때 최적의 수단이다.


