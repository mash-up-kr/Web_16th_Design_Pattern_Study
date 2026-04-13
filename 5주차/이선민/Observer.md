## Observer 패턴이란?

- **특정 객체(Observable)를 구독(subscribe)하여, 이벤트 발생 시 모든 구독자(Observer)에게 자동으로 알림을 전파하는 패턴**
- 구독하는 주체를 Observer, 구독 가능한 객체를 Observable이라 한다
- Observable과 Observer 간의 느슨한 결합(loose coupling)을 통해 관심사를 분리할 수 있다
- 비동기 이벤트 처리, 상태 변경 알림 등에 널리 사용된다

## Observable의 구성 요소

| 요소 | 역할 |
|---|---|
| `observers` | 이벤트 발생 시 알림을 받을 Observer들의 배열 |
| `subscribe()` | Observer를 배열에 추가하여 등록 |
| `unsubscribe()` | Observer를 배열에서 제거 |
| `notify()` | 등록된 모든 Observer에게 이벤트 전파 |

## 기본 구현

```javascript
class Observable {
  constructor() {
    this.observers = []
  }

  subscribe(func) {
    this.observers.push(func)
  }

  unsubscribe(func) {
    this.observers = this.observers.filter(observer => observer !== func)
  }

  notify(data) {
    this.observers.forEach(observer => observer(data))
  }
}
```

- `subscribe`로 Observer(함수)를 등록하고, `notify`가 호출되면 등록된 모든 Observer에 데이터를 전달한다
- `unsubscribe`로 더 이상 알림이 필요 없는 Observer를 제거할 수 있다

## 실제 사용 예시: 버튼 클릭 이벤트

```javascript
import { ToastContainer, toast } from 'react-toastify'

function logger(data) {
  console.log(`${Date.now()} ${data}`)
}

function toastify(data) {
  toast(data)
}

observable.subscribe(logger)
observable.subscribe(toastify)

export default function App() {
  function handleClick() {
    observable.notify('User clicked button!')
  }

  function handleToggle() {
    observable.notify('User toggled switch!')
  }

  return (
    <div className="App">
      <Button>Click me!</Button>
      <FormControlLabel control={<Switch />} />
      <ToastContainer />
    </div>
  )
}
```

- `handleClick`이나 `handleToggle` 호출 시 Observable의 `notify()`가 실행된다
- 등록된 Observer들(`logger`, `toastify`)이 자동으로 이벤트 데이터를 수신한다
- 각 Observer는 독립적으로 자신의 역할(로깅, 토스트 알림)을 수행한다

## Observer vs Pub/Sub 패턴

Observer 패턴과 유사하지만 다른 패턴으로 Publish/Subscribe(Pub/Sub) 패턴이 있다.

```
Observer 패턴:    Subject ────────────> Observer
                  (직접 참조)

Pub/Sub 패턴:     Publisher ──> Event Channel ──> Subscriber
                  (서로 모름)      (중재자)       (서로 모름)
```

### 핵심 차이: 누가 누구를 아는가

- **Observer 패턴**: Subject(여기서는 Observable)가 Observer 목록을 직접 관리한다. Subject는 자신을 구독하는 Observer가 누구인지 알고 있다.
- **Pub/Sub 패턴**: Publisher와 Subscriber가 서로의 존재를 모른다. 중간에 Event Channel(Message Broker)이 메시지를 중재한다.

### Observer 패턴 코드

Subject가 Observer를 직접 참조하고, 이벤트 발생 시 직접 호출한다.

```javascript
class Subject {
  constructor() {
    this.observers = []
  }

  subscribe(observer) {
    this.observers.push(observer)
  }

  unsubscribe(observer) {
    this.observers = this.observers.filter(o => o !== observer)
  }

  notify(data) {
    // Subject가 Observer를 직접 순회하며 호출
    this.observers.forEach(observer => observer(data))
  }
}

const subject = new Subject()

const observerA = (data) => console.log(`Observer A: ${data}`)
const observerB = (data) => console.log(`Observer B: ${data}`)

subject.subscribe(observerA)
subject.subscribe(observerB)

subject.notify('Hello!')
// Observer A: Hello!
// Observer B: Hello!
```

- Subject가 `observerA`, `observerB`를 직접 알고 있다
- `notify()`를 호출하면 **모든** Observer에게 **동일한** 데이터가 전달된다
- 이벤트의 종류를 구분하지 않는다 (모든 Observer가 모든 알림을 수신)

### Pub/Sub 패턴 코드

Event Channel이 **이벤트 이름(토픽)** 별로 Subscriber를 관리한다.

```javascript
class EventChannel {
  constructor() {
    this.events = {}
  }

  subscribe(event, callback) {
    if (!this.events[event]) {
      this.events[event] = []
    }
    this.events[event].push(callback)
  }

  unsubscribe(event, callback) {
    if (!this.events[event]) return
    this.events[event] = this.events[event].filter(cb => cb !== callback)
  }

  publish(event, data) {
    if (!this.events[event]) return
    // 해당 이벤트를 구독한 Subscriber에게만 전달
    this.events[event].forEach(callback => callback(data))
  }
}

const channel = new EventChannel()

// Subscriber는 관심 있는 이벤트만 구독
channel.subscribe('user:login', (data) => {
  console.log(`로그인 알림: ${data.name}`)
})

channel.subscribe('user:logout', (data) => {
  console.log(`로그아웃 알림: ${data.name}`)
})

channel.subscribe('user:login', (data) => {
  console.log(`로그인 로깅: ${data.name} at ${Date.now()}`)
})

// Publisher는 Subscriber가 누구인지 모른 채 이벤트를 발행
channel.publish('user:login', { name: '선민' })
// 로그인 알림: 선민
// 로그인 로깅: 선민 at 1713004800000

channel.publish('user:logout', { name: '선민' })
// 로그아웃 알림: 선민
```

- Publisher(`publish` 호출부)는 누가 구독하고 있는지 전혀 모른다
- Subscriber는 누가 이벤트를 발행했는지 전혀 모른다
- **이벤트 이름(토픽)** 을 기준으로 메시지가 필터링된다

### 비교 정리

| | Observer | Pub/Sub |
|---|---|---|
| 결합도 | Subject가 Observer를 직접 알고 있음 | Publisher와 Subscriber가 서로 모름 |
| 중간 계층 | 없음 (직접 통신) | Event Channel이 중재 |
| 이벤트 필터링 | 없음 (모든 Observer가 모든 알림 수신) | 토픽 기반으로 관심 이벤트만 수신 |
| 동기/비동기 | 주로 동기적 | 동기/비동기 모두 가능 |
| 적합한 상황 | 컴포넌트 간 직접적인 상태 공유 | 모듈/시스템 간 느슨한 이벤트 전달 |
| 실제 예시 | RxJS | React 상태 구독, Node.js `EventEmitter` |

### addEventListener는 어디에 해당할까?

`addEventListener`는 이벤트 종류(토픽)별로 구독하지만, 특정 DOM 요소(Subject)를 직접 참조해야 한다는 점에서 **두 패턴의 중간 지점**에 위치한다.

```javascript
// addEventListener — 특정 요소(Subject)를 직접 참조해야 구독 가능
button.addEventListener('click', handler)

// Pub/Sub — 발행자가 누구인지 몰라도 구독 가능
channel.subscribe('click', handler)
```

- **Observer적 특성**: `button`이라는 Subject를 직접 알아야 구독할 수 있다 (결합)
- **Pub/Sub적 특성**: `'click'`, `'scroll'` 등 이벤트 종류(토픽)별로 구독자를 따로 관리한다

순수한 Observer도, 순수한 Pub/Sub도 아닌 **토픽 필터링이 있는 Observer 패턴**에 가깝다.

### 상태 관리 라이브러리에서의 패턴

실제 상태 관리 라이브러리들도 순수한 한 패턴이 아니라 여러 패턴이 혼합된 형태다.

**Redux — Provider + Pub/Sub**

```javascript
// Provider 패턴 — Context로 store를 주입
<Provider store={store}>
  <App />
</Provider>

// Pub/Sub — action(토픽)을 dispatch하면, 해당 상태를 구독한 컴포넌트만 리렌더링
dispatch({ type: 'counter/increment' })  // publish
useSelector(state => state.counter)       // subscribe (특정 상태 토픽 구독)
```

- `dispatch`하는 쪽은 누가 구독하는지 모르고, `useSelector`하는 쪽은 누가 dispatch하는지 모른다
- 중간에 store가 중재자 역할을 하므로 Pub/Sub에 해당한다

**Recoil / Jotai — Provider + Observer**

```javascript
// Recoil — atom을 직접 참조하여 구독
const counterAtom = atom({ key: 'counter', default: 0 })
const [count, setCount] = useRecoilState(counterAtom)

// Jotai — 마찬가지로 atom을 직접 참조
const counterAtom = atom(0)
const [count, setCount] = useAtom(counterAtom)
```

- 컴포넌트가 atom(Subject)을 직접 참조해야 구독할 수 있다 (Observer적 결합)
- action/dispatch 같은 중재자 없이 atom에 직접 읽기/쓰기한다
- 다만 atom별로 구독자를 따로 관리하므로 토픽 필터링 특성은 있다
- Recoil은 `RecoilRoot`(Provider)가 필수이고, Jotai는 Provider 없이도 동작 가능하다

**Zustand — Pub/Sub만**

```javascript
// Provider 불필요 — 모듈 레벨에서 store 생성
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}))

// selector로 관심 있는 상태만 구독
const count = useStore((state) => state.count)
```

- Provider 없이 모듈에서 직접 store를 생성하고, selector를 통해 필요한 상태만 구독한다
- store가 중재자 역할을 하며, 컴포넌트는 store의 내부 구현을 알 필요 없다

**비교 정리**

| 라이브러리 | 패턴 구성 | 상태 구독 방식 | Provider 필요 여부 |
|---|---|---|---|
| **Redux** | Provider + Pub/Sub | action(토픽) dispatch로 디커플링 | O |
| **Recoil** | Provider + Observer | atom을 직접 참조하여 구독 | O (`RecoilRoot`) |
| **Jotai** | Observer | atom을 직접 참조하여 구독 | X (선택적) |
| **Zustand** | Pub/Sub | selector로 관심 상태만 구독 | X |

## 장점

- **관심사의 분리**: Observable은 이벤트 발생만, Observer는 수신한 데이터 처리만 담당한다
- **단일 책임 원칙**: 각 Observer가 자신만의 역할을 수행하므로 책임이 명확하다
- **느슨한 결합**: Observable과 Observer가 독립적이므로 언제든지 구독/해제할 수 있다
- **유연한 확장**: 새로운 Observer를 추가해도 기존 코드를 수정할 필요가 없다

## 단점

- **성능 이슈**: Observer가 많아질수록 `notify()` 호출 시 모든 Observer에 알림을 전파하므로 성능이 저하될 수 있다
- **디버깅 어려움**: Observer 간 실행 순서가 보장되지 않아 복잡한 이벤트 체인에서 추적이 어려울 수 있다
- **메모리 누수**: `unsubscribe`를 빠트리면 Observer가 해제되지 않아 메모리 누수가 발생할 수 있다

## 참고자료

- https://patterns-dev-kr.github.io/design-patterns/observer-pattern/
- https://www.sitepoint.com/javascript-design-patterns-observer-pattern/
