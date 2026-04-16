# [Observer Pattern](https://patterns-dev-kr.github.io/design-patterns/observer-pattern/)

## [What] Observer Pattern이란?

> "특정 객체의 상태 변화를 관찰하는 관찰자(Observer)들에게 자동으로 알림을 보내는 패턴" _(claude)_

> Observable을 이용하면 특정 객체를 구독할 수 있는데, 이벤트가 발생할 때마다 Observable은 모든 구독자에게 이벤트를 전파한다. _(patterns.dev)_

Observer Pattern은 **일대다(one-to-many) 의존 관계**를 정의하는 것이다.  
하나의 객체(Subject 또는 Observable)의 상태가 변경되면, 그 객체에 등록된 모든 의존 객체(Observer)들이 자동으로 알림을 받는다.
(ex. 유투브 구독 알림)

이 패턴은 **이벤트 기반 프로그래밍(Event-Driven Programming)**의 근간이다. React에만 국한되는 개념이 아니다.

- DOM의 `addEventListener` -- 특정 이벤트가 발생하면 등록된 핸들러가 호출된다.
- Node.js의 `EventEmitter` -- 이벤트를 발행하면 등록된 리스너들이 실행된다.
- React의 상태 변화 -> 리렌더링 -- state가 변경되면 해당 state를 구독하는 컴포넌트들이 자동으로 리렌더링된다.

모두 특정 이벤트/상태에 대한 구독이라는 점에서 Observer 패턴의 변형이라고 볼 수 있을 것 같다.

### Observer vs Pub/Sub

Observer 패턴과 자주 혼동되는 개념이 **Publish/Subscribe(Pub/Sub) 패턴**이다.  
둘 다 "이벤트가 발생하면 알림을 보낸다"는 점에서 비슷하지만, 결합도에서 차이가 있다.

- **Observer 패턴**: Subject와 Observer가 **서로를 직접 참조**한다. Subject는 Observer 목록을 직접 관리하고, Observer는 Subject에 직접 등록한다.
- **Pub/Sub 패턴**: Publisher와 Subscriber가 **서로를 모른다**. 중간에 이벤트 버스가 존재하여, Publisher는 이벤트를 발행하기만 하고, Subscriber는 이벤트를 구독하기만 한다. 상대방의 존재를 알 필요가 없다.

## [Patterns.dev] 예시로 알아보기

### Observable 클래스 구현

patterns.dev에서 소개하는 Observable 클래스의 기본 구조이다.

```js
class Observable {
  constructor() {
    this.observers = [];
  }

  subscribe(func) {
    this.observers.push(func);
  }

  unsubscribe(func) {
    this.observers = this.observers.filter((observer) => observer !== func);
  }

  notify(data) {
    this.observers.forEach((observer) => observer(data));
  }
}
```

`observers` 배열에 함수를 등록(`subscribe`)하고, 이벤트가 발생하면 등록된 모든 함수를 호출(`notify`)한다. 더 이상 알림을 받고 싶지 않으면 `unsubscribe`로 제거한다.

### 실제 예시: Button/Switch와 Logger/Toastify

Observer 패턴이 없는 경우, 버튼 클릭 시 로깅과 토스트 알림을 동시에 보여주려면 이벤트 핸들러 안에서 모든 로직을 직접 호출해야 한다.

```js
// Observer 패턴 없이 — 모든 로직이 핸들러에 직접 결합
function handleClick() {
  console.log("사용자가 클릭했습니다");  // 로깅
  showToast("클릭 이벤트 발생!");        // 토스트
  sendAnalytics("click_event");          // 분석
  // 새로운 반응이 필요할 때마다 여기를 수정해야 한다
}
```

Observer 패턴을 적용하면 각 반응을 독립적인 Observer로 분리할 수 있다.

```js
import { Observable } from "./observable";
import { ToastContainer, toast } from "react-toastify";

// Observer 함수들
function logger(data) {
  console.log(`${Date.now()} ${data}`);
}

function toastify(data) {
  toast(data, {
    position: toast.POSITION.BOTTOM_RIGHT,
    closeButton: false,
    autoClose: 2000,
  });
}

// Observable 인스턴스 생성 및 Observer 등록
const observable = new Observable();
observable.subscribe(logger);
observable.subscribe(toastify);
```

```jsx
// 컴포넌트에서는 notify만 호출하면 된다
export default function App() {
  function handleClick() {
    observable.notify("사용자가 클릭했습니다!");
  }

  function handleToggle() {
    observable.notify("토글이 변경되었습니다!");
  }

  return (
    <div>
      <button onClick={handleClick}>클릭</button>
      <Switch onChange={handleToggle} />
      <ToastContainer />
    </div>
  );
}
```

`handleClick`과 `handleToggle`은 `observable.notify()`만 호출할 뿐, 로깅이 어떻게 처리되는지, 토스트가 어떤 옵션으로 표시되는지 알 필요가 없다. 새로운 반응이 필요하면 `observable.subscribe(newObserver)`를 추가하기만 하면 된다. 기존 코드를 수정할 필요가 없다.

### RxJS

Observer 패턴을 대규모로 활용할 때 자주 언급되는 라이브러리가 **RxJS**이다. RxJS는 **Observer 패턴 + 이터레이터 패턴 + 함수형 프로그래밍의 조합**이다.

```js
import { fromEvent, merge } from "rxjs";
import { map, filter, throttleTime } from "rxjs/operators";

// DOM 이벤트를 Observable 스트림으로 변환
const click$ = fromEvent(document, "click");
const scroll$ = fromEvent(document, "scroll");

// 스트림을 합성하고 변환
merge(click$, scroll$)
  .pipe(
    throttleTime(300),
    map((event) => ({
      type: event.type,
      timestamp: Date.now(),
    })),
    filter((data) => data.type === "click")
  )
  .subscribe((data) => {
    console.log("필터링된 이벤트:", data);
  });
```

단순한 Observer 패턴과 달리, RxJS는 **이벤트 스트림을 변환, 필터링, 합성**할 수 있는 풍부한 연산자를 제공한다. Angular에서는 HTTP 요청, 폼 입력, 라우팅 이벤트 등 거의 모든 비동기 처리에 RxJS를 사용한다.
> 만약 배워 보겠다면 이글을 한번.. https://velog.io/@teo/rxjs

### 개인적인 활용 사례,,😂

예전에 html의 데이터 속성 기반 기능(우클릭 방지, 드래그 방지 등)을 제공하는 라이브러리를 만든적이 있는데, 내부에서 Observer 패턴을 활용하고 있다.

`lifecycleManager`가 Observable 역할을 하고, 각 보안 기능 모듈이 Observer로 구독하는 구조이다.

```ts
// lifecycleManager.ts — Observable
const lifecycleEvents: { [key in LifecycleEvent]?: Array<() => void> } = {
  DOMContentLoaded: [],
  load: [],
};

const onEvent = (event: LifecycleEvent, callback: lifecycleHandler) => {
  if (eventFired[event]) { callback(); return; }
  lifecycleEvents[event]?.push(callback); // subscribe
};

document.addEventListener('DOMContentLoaded', () => {
  lifecycleEvents.DOMContentLoaded?.forEach((cb) => cb()); // notify
});
```

```ts
// 각 보안 모듈 — Observer
onEvent('DOMContentLoaded', preventContextMenu);
onEvent('DOMContentLoaded', preventDrag);
onEvent('DOMContentLoaded', preventCopy);
```

`DOMContentLoaded` 하나의 이벤트에 여러 Observer가 독립적으로 구독하고, 이벤트 발생 시 모두에게 알림이 전파된다. 새로운 보안 기능을 추가할 때도 `onEvent('DOMContentLoaded', newFeature)` 한 줄이면 되고, 기존 코드를 수정할 필요가 없다.

## [How] Observer Pattern은 어떻게 구현하는가?

핵심은 **상태 변화가 발생하는 객체(Observable)가 등록된 관찰자(Observer)들에게 자동으로 알림을 보내는 구조를 만드는 것**이다.

### 기본 구조 (TypeScript)

```ts
type Observer<T> = (data: T) => void;

class Observable<T> {
  private observers: Observer<T>[] = [];

  subscribe(observer: Observer<T>): void {
    this.observers.push(observer);
  }

  unsubscribe(observer: Observer<T>): void {
    this.observers = this.observers.filter((obs) => obs !== observer);
  }

  notify(data: T): void {
    this.observers.forEach((observer) => observer(data));
  }
}
```

### 사용 패턴: subscribe -> notify -> unsubscribe

```ts
const userEvents = new Observable<string>();

// 1. Observer 등록
const logger = (message: string) => console.log(`[LOG] ${message}`);
const analytics = (message: string) => sendToServer(message);

userEvents.subscribe(logger);
userEvents.subscribe(analytics);

// 2. 이벤트 발생 시 모든 Observer에게 알림
userEvents.notify("사용자 로그인");
// [LOG] 사용자 로그인
// (서버로 전송)

// 3. 더 이상 필요 없는 Observer 해제
userEvents.unsubscribe(analytics);

userEvents.notify("페이지 이동");
// [LOG] 페이지 이동
// analytics는 더 이상 호출되지 않는다
```

### 이벤트 이름을 지원하는 확장 구현

기본 Observable은 하나의 이벤트 타입만 다룰 수 있다. 여러 종류의 이벤트를 구분하려면 이벤트 이름을 키로 사용하는 구조가 필요하다. Node.js의 `EventEmitter`가 이 방식을 사용한다.

```ts
type Listener = (...args: any[]) => void;

class EventEmitter {
  private events: Map<string, Listener[]> = new Map();

  on(event: string, listener: Listener): void {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event)!.push(listener);
  }

  off(event: string, listener: Listener): void {
    const listeners = this.events.get(event);
    if (listeners) {
      this.events.set(
        event,
        listeners.filter((l) => l !== listener)
      );
    }
  }

  emit(event: string, ...args: any[]): void {
    const listeners = this.events.get(event);
    if (listeners) {
      listeners.forEach((listener) => listener(...args));
    }
  }

  once(event: string, listener: Listener): void {
    const wrapper = (...args: any[]) => {
      listener(...args);
      this.off(event, wrapper);
    };
    this.on(event, wrapper);
  }
}
```

```ts
const emitter = new EventEmitter();

// 이벤트 이름으로 구분하여 구독
emitter.on("login", (user) => console.log(`${user} 로그인`));
emitter.on("logout", (user) => console.log(`${user} 로그아웃`));

emitter.emit("login", "정우");   // 정우 로그인
emitter.emit("logout", "정우");  // 정우 로그아웃

// once: 한 번만 실행
emitter.once("error", (err) => console.error("에러 발생:", err));
emitter.emit("error", "네트워크 오류");  // 에러 발생: 네트워크 오류
emitter.emit("error", "다른 오류");      // (아무 일도 일어나지 않는다)
```

기본 Observable과 EventEmitter 방식의 차이를 정리하면 다음과 같다.

| 기준 | Observable (기본) | EventEmitter |
|:--|:--|:--|
| 이벤트 구분 | 단일 이벤트만 처리 | 이벤트 이름으로 구분 |
| 등록 방식 | `subscribe(fn)` | `on(eventName, fn)` |
| 알림 방식 | `notify(data)` | `emit(eventName, ...args)` |
| 사용 예시 | 단일 상태 변화 추적 | 여러 종류의 이벤트 처리 |

## [Why] Observer Pattern은 왜 사용하는가?

Q. 언제 사용하면 좋을까?

핵심은 **하나의 상태 변화가 여러 곳에 영향을 미쳐야 하지만, 그 "여러 곳"이 서로를 알 필요가 없을 때**이다.

예를 들어, 사용자가 로그인하면 네비게이션 바의 프로필이 바뀌고, 알림 센터가 새 알림을 로드하고, 분석 서비스가 로그인 이벤트를 기록해야 한다. 이 세 가지 반응은 서로 무관하지만, 모두 "로그인"이라는 동일한 이벤트에 반응한다. 이때 Observer 패턴이 적합하다.

Q. 왜 직접 함수를 호출하지 않고 Observer 패턴을 쓰는가?

1. **느슨한 결합**
    - Observable은 Observer가 누구인지 알 필요가 없다. Observer를 추가하거나 제거해도 Observable 코드에 변경이 없다. 직접 호출 방식에서는 새로운 반응이 추가될 때마다 `login` 메서드를 수정해야 한다.

2. **개방-폐쇄 원칙(OCP)**
    - 새로운 Observer를 추가해도 Observable 코드를 수정할 필요가 없다. `auth.subscribe(newObserver)`만 호출하면 된다. 확장에는 열려 있고, 수정에는 닫혀 있다.

3. **관심사의 분리**
    - 상태 변화의 **"발생"**과 **"반응"**이 분리된다. AuthService는 인증만 담당하고, 각 Observer는 자신의 반응만 담당한다. 각각을 독립적으로 테스트할 수 있다.

Q. Observer 패턴의 단점은 무엇인가?

1. **성능 이슈**
    - Observer가 많아지면 `notify` 시 모든 Observer를 순회하므로 성능 저하가 발생할 수 있다. 특히 Observer의 콜백이 무거운 연산을 수행하는 경우, 하나의 `notify` 호출이 전체 시스템을 느리게 만들 수 있다.

    ```ts
    // Observer가 1000개 등록된 경우
    observable.notify(data);
    // -> 1000개의 콜백이 동기적으로 순차 실행된다
    ```

2. **메모리 누수**
    - `unsubscribe`를 하지 않으면 Observer가 GC(Garbage Collection)되지 않는다. 컴포넌트가 언마운트되었지만 구독을 해제하지 않으면, 해당 Observer 함수와 그것이 참조하는 모든 변수가 메모리에 남는다.

    ```ts
    // React에서 흔히 발생하는 메모리 누수
    useEffect(() => {
      const handler = (data) => setState(data);
      observable.subscribe(handler);

      // cleanup을 빠뜨리면 컴포넌트가 언마운트되어도
      // handler가 계속 호출되고, 메모리에 남는다
      return () => observable.unsubscribe(handler); // 반드시 해제!
    }, []);
    ```

3. **디버깅 어려움**
    - 누가 언제 `subscribe`했는지 추적하기 어렵다. 이벤트 흐름이 코드에 명시적으로 드러나지 않기 때문에, 버그가 발생했을 때 "이 Observer는 어디서 등록된 거지?"를 찾아야 한다. 직접 함수 호출 방식에서는 콜 스택을 따라가면 되지만, Observer 패턴에서는 `notify`에서 출발하여 등록 시점을 역추적해야 한다.

4. **실행 순서 불확실**
    - Observer들의 실행 순서는 일반적으로 등록 순서를 따르지만, 이를 **보장**하기 어렵다. 순서에 의존하는 로직을 Observer로 구현하면 예측하기 어려운 버그가 발생할 수 있다. Observer 간에 실행 순서가 중요하다면, 그것은 Observer 패턴이 적합하지 않다는 신호일 수 있다.