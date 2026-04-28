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

## 장점

- **관심사의 분리**: Observable은 이벤트 발생만, Observer는 수신한 데이터 처리만 담당한다
- **단일 책임 원칙**: 각 Observer가 자신만의 역할을 수행하므로 책임이 명확하다
- **느슨한 결합**: Observable과 Observer가 독립적이므로 언제든지 구독/해제할 수 있다
- **유연한 확장**: 새로운 Observer를 추가해도 기존 코드를 수정할 필요가 없다

## 단점

- **성능 이슈**: Observer가 많아질수록 `notify()` 호출 시 모든 Observer에 알림을 전파하므로 성능이 저하될 수 있다
- **디버깅 어려움**: Observer 간 실행 순서가 보장되지 않아 복잡한 이벤트 체인에서 추적이 어려울 수 있다
- **메모리 누수**: `unsubscribe`를 빠트리면 Observer가 해제되지 않아 메모리 누수가 발생할 수 있다

## 실제 사용 예시
### 1. 버튼 클릭 이벤트

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

### 2. addEventListener는 어디에 해당할까?

`addEventListener`는 이벤트 종류(토픽)별로 구독하지만, 특정 DOM 요소(Subject)를 직접 참조해야 한다는 점에서 **Observer패턴과 Pub/Sub 패턴의 중간 지점**에 위치한다.

```javascript
// addEventListener — 특정 요소(Subject)를 직접 참조해야 구독 가능
button.addEventListener('click', handler)

// Pub/Sub — 발행자가 누구인지 몰라도 구독 가능
channel.subscribe('click', handler)
```

- **Observer적 특성**: `button`이라는 Subject를 직접 알아야 구독할 수 있다 (결합)
- **Pub/Sub적 특성**: `'click'`, `'scroll'` 등 이벤트 종류(토픽)별로 구독자를 따로 관리한다

순수한 Observer도, 순수한 Pub/Sub도 아닌 **토픽 필터링이 있는 Observer 패턴**에 가깝다.

### 3. RxJS (ReactiveX)

**Observer 패턴 + 이터레이터 패턴 + 함수형 프로그래밍**을 조합한 반응형 프로그래밍 라이브러리. 시간에 따라 흐르는 이벤트의 **스트림(stream)** 을 다룬다.

**핵심 개념**

| 요소 | 역할 |
|---|---|
| `Observable` | 시간에 따라 발생하는 값의 스트림 (클릭, fetch 응답, 타이머 등) |
| `Observer` | 스트림의 값을 소비하는 구독자 (`next`, `error`, `complete` 핸들러) |
| `Subscription` | 구독 연결. `unsubscribe()`로 해제 가능 |
| `Operator` | 스트림을 변환·결합하는 순수 함수 (`map`, `filter`, `debounceTime` 등) |
| `Subject` | Observable + Observer를 겸하는 객체. 멀티캐스트 가능 |

**기본 사용 예시**

```javascript
import { fromEvent } from 'rxjs'
import { map, filter, debounceTime } from 'rxjs/operators'

// 1. input의 입력 이벤트를 스트림으로 변환 (Observable 생성)
const input$ = fromEvent(document.querySelector('#search'), 'input')

// 2. 파이프라인으로 스트림 변환
// search$: Observable
const search$ = input$.pipe(
  debounceTime(300),                       // 300ms 동안 입력 멈추면 통과 (Operator)
  map(event => event.target.value),        // 입력값만 추출 (Operator)
  filter(text => text.length >= 2)         // 2글자 이상만 통과 (Operator)
)

// 3. 구독
const subscription = search$.subscribe({
  next: (value) => console.log('검색:', value),
  error: (err) => console.error(err),
  complete: () => console.log('완료'),
})

// 구독 해제
subscription.unsubscribe()
```

- 기존 Observer 패턴이 "이벤트 발생 → 알림" 수준이라면, RxJS는 **스트림을 데이터 파이프라인처럼 가공**할 수 있다
- `debounceTime`, `throttleTime`, `switchMap` 등 시간·비동기를 다루는 연산자가 풍부하다

## 참고자료

- https://patterns-dev-kr.github.io/design-patterns/observer-pattern/
- https://www.sitepoint.com/javascript-design-patterns-observer-pattern/
