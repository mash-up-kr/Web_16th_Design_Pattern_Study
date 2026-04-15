## Command 패턴이란?

- **작업을 실행하는 객체와 메서드를 호출하는 객체를 분리하는 패턴**
- 요청(명령)을 객체로 캡슐화하여, 매개변수처럼 전달하거나 큐에 담거나 지연 실행할 수 있게 한다
- 호출자(Invoker)는 명령의 구체적인 구현을 모른 채 `execute()`만 호출한다
- 실행 취소(undo), 작업 큐, 로깅, 예약 실행 등에 활용된다

## 구성 요소

| 요소 | 역할 |
|---|---|
| `Command` | 실행할 작업을 캡슐화한 객체. `execute()` 메서드를 가진다 |
| `Invoker` | 명령을 실행시키는 객체 (예: `OrderManager`) |
| `Receiver` | 실제 작업이 수행되는 대상 (예: `orders` 배열) |
| `Client` | Command 객체를 생성하여 Invoker에게 전달하는 주체 |

## 문제 상황: 커맨드 패턴 없이 구현

`OrderManager`에 모든 작업을 메서드로 직접 정의한 경우를 보자.

```javascript
class OrderManager {
  constructor() {
    this.orders = []
  }

  placeOrder(order, id) {
    this.orders.push(id)
    return `You have successfully ordered ${order} (${id})`
  }

  trackOrder(id) {
    return `Your order ${id} will arrive in 20 minutes.`
  }

  cancelOrder(id) {
    this.orders = this.orders.filter(order => order.id !== id)
    return `You have canceled your order ${id}`
  }
}
```

- `OrderManager`가 모든 작업을 직접 알아야 한다
- 메서드명이 바뀌면(`placeOrder` → `addOrder`) 호출하는 모든 코드를 함께 수정해야 한다
- 새 작업을 추가할 때마다 `OrderManager` 클래스를 수정해야 한다 (OCP 위반)

## 기본 구현

### 1. Invoker와 Command 기본 구조

```javascript
class OrderManager {
  constructor() {
    this.orders = []
  }

  execute(command, ...args) {
    return command.execute(this.orders, ...args)
  }
}

class Command {
  constructor(execute) {
    this.execute = execute
  }
}
```

- `OrderManager`는 `execute()` 하나만 제공하고, 구체적인 작업은 모른다
- `Command`는 실행 함수를 받아 캡슐화한다

### 2. 구체적인 명령 정의

```javascript
function PlaceOrderCommand(order, id) {
  return new Command(orders => {
    orders.push(id)
    return `You have successfully ordered ${order} (${id})`
  })
}

function CancelOrderCommand(id) {
  return new Command(orders => {
    orders = orders.filter(order => order.id !== id)
    return `You have canceled your order ${id}`
  })
}

function TrackOrderCommand(id) {
  return new Command(() => `Your order ${id} will arrive in 20 minutes.`)
}
```

### 3. 사용 예시

```javascript
const manager = new OrderManager()

manager.execute(new PlaceOrderCommand('Pad Thai', '1234'))
manager.execute(new TrackOrderCommand('1234'))
manager.execute(new CancelOrderCommand('1234'))
```

- `OrderManager`는 어떤 명령이 오든 `execute()`만 호출하면 된다
- 새 작업이 필요하면 `XxxCommand`만 추가하면 되고, `OrderManager`는 수정할 필요가 없다

## 활용 예시: 실행 취소(Undo) 구현

명령을 객체로 캡슐화하면 실행 이력을 저장하고 되돌릴 수 있다.

```javascript
class Command {
  constructor(execute, undo) {
    this.execute = execute
    this.undo = undo
  }
}

class OrderManager {
  constructor() {
    this.orders = []
    this.history = []  // 실행 이력
  }

  execute(command, ...args) {
    this.history.push(command)
    return command.execute(this.orders, ...args)
  }

  undo() {
    const command = this.history.pop()
    if (command) return command.undo(this.orders)
  }
}

function PlaceOrderCommand(order, id) {
  return new Command(
    orders => { orders.push(id); return `주문 완료: ${order}` },
    orders => { orders.splice(orders.indexOf(id), 1); return `주문 취소됨` }
  )
}
```

## 활용 예시: 작업 큐

명령을 큐에 쌓아두고 원하는 시점에 순차 실행할 수 있다.

```javascript
class CommandQueue {
  constructor() {
    this.queue = []
  }

  enqueue(command) {
    this.queue.push(command)
  }

  async processAll(context) {
    while (this.queue.length > 0) {
      const command = this.queue.shift()
      await command.execute(context)
    }
  }
}
```

- API 요청 배치 처리, 게임의 매크로/리플레이, 트랜잭션 처리 등에 활용

## 장점

- **느슨한 결합**: Invoker는 Command의 구체적 구현을 알 필요가 없다
- **확장성**: 새 명령을 추가할 때 기존 코드를 수정하지 않는다 (OCP 준수)
- **명령의 객체화**: 명령을 매개변수로 전달하거나 큐에 담거나 지연 실행할 수 있다
- **실행 취소 / 재실행 구현 용이**: 명령 이력을 저장해 역으로 실행할 수 있다
- **로깅·감사(audit)**: 실행된 명령을 기록할 수 있다

## 단점

- **사용 사례가 제한적**: 단순한 작업에는 오히려 복잡도만 늘어난다
- **보일러플레이트 증가**: 각 명령마다 클래스/함수를 만들어야 하므로 코드량이 늘어난다
- **오버엔지니어링**: undo나 큐잉 같은 요구가 없다면 굳이 도입할 필요가 없다

## 적용하면 좋은 상황

- 실행 취소(undo/redo) 기능이 필요한 경우 (텍스트 에디터, 그래픽 툴)
- 작업을 큐에 담아 예약·지연 실행이 필요한 경우
- 명령 실행 이력을 로깅·감사해야 하는 경우
- 매크로/리플레이처럼 명령 시퀀스를 저장·재생해야 하는 경우

## 참고자료

- https://patterns-dev-kr.github.io/design-patterns/command-pattern/
