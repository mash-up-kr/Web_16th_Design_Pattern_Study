# [Command Pattern](https://patterns-dev-kr.github.io/design-patterns/command-pattern/)

## [What] Command Pattern이란?

> "요청(작업)을 객체로 캡슐화하여, 실행하는 쪽과 호출하는 쪽을 분리하는 패턴" _(claude)_

> Command 패턴을 사용하면 특정 작업을 실행하는 객체와 메서드를 호출하는 객체를 분리할 수 있다. _(patterns.dev)_

Command Pattern의 핵심 아이디어는 단순하다. **작업을 "실행"이 아닌 "데이터"로 다룬다는 것이다.**

"데이터로 다룬다"는 말은 함수 호출과 비교하면 명확해진다.

함수 호출은 **즉시 실행**이다. 호출하는 순간 끝나고, 되돌릴 방법이 없다.

```js
// 함수 호출 — 이 줄이 실행되는 순간 작업이 완료된다
editor.deleteText(0, 5)
// 끝. 되돌릴 방법이 없다.
```

Command 객체는 **아직 실행하지 않은 작업**을 데이터로 들고 있는 것이다.

```js
// Command — 작업을 "설명하는 객체"를 만든 것. 아직 실행하지 않았다.
const cmd = new DeleteTextCommand(editor, 0, 5)

// 이 객체를 가지고 할 수 있는 것들:
history.push(cmd)      // 저장
queue.enqueue(cmd)     // 큐에 넣기
cmd.execute()          // 지금 실행
cmd.undo()             // 되돌리기
JSON.stringify(cmd)    // 직렬화
```

### [Hum..🤔] Command vs Factory 비교

"구현을 몰라도 사용할 수 있게 캡슐화한다"는 점에서 Factory 패턴과 혼동될 수 있다.  
하지만 둘은 캡슐화하는 대상이 다르다.

- **Factory 패턴**: **생성**을 캡슐화한다. "어떤 객체를 만드는지" 숨긴다.
- **Command 패턴**: **실행**을 캡슐화한다. "어떤 작업을 하는지"를 객체로 만들어 **제어권**을 얻는다.

```js
// Factory — "어떤 버튼인지" 몰라도 만들 수 있다
const button = factory.createButton()  // Windows? Mac? 몰라도 됨

// Command — "무엇을 할지"를 객체로 만들어서 제어할 수 있다
const cmd = new DeleteTextCommand(editor, 0, 5) // 언제 실행할지, 되돌릴지, 기록할지를 내가 결정한다
```

| 패턴 | 캡슐화 대상 | 핵심 가치 |
|:--|:--|:--|
| Factory | **생성** — 어떤 객체를 만드는지 숨김 | 구현을 몰라도 올바른 객체를 받을 수 있다 |
| Command | **실행** — 어떤 작업을 하는지를 객체로 만듦 | 작업을 저장·큐잉·취소·재실행할 수 있다 |

"구현을 몰라도 실행할 수 있다"는 Command의 부수적인 효과이지, 핵심이 아니다. Command의 핵심은 **작업 자체를 객체로 만들어서, 즉시 실행 외의 선택지(저장, 큐잉, 되돌리기)를 얻는 것**이다.

식당 주문을 떠올려 보자. 손님이 웨이터에게 주문서(Command)를 전달하면, 웨이터는 주문서(Command)를 주방에 전달한다. 손님은 주방이 어떻게 요리하는지 몰라도 되고, 주방은 누가 주문했는지 몰라도 된다.

## [Patterns.dev] 예시로 알아보기

### Command 없이 직접 다루는 경우

음식 배달 플랫폼을 만든다고 해보자. `OrderManager` 클래스가 주문 접수, 주문 추적, 주문 취소를 처리한다.

```js
class OrderManager {
  constructor() {
    this.orders = [];
  }

  placeOrder(order, id) {
    this.orders.push(id);
    return `You have successfully ordered ${order} (${id})`;
  }

  trackOrder(id) {
    return `Your order ${id} will arrive in 20 minutes.`;
  }

  cancelOrder(id) {
    this.orders = this.orders.filter((order) => order.id !== id);
    return `You have canceled your order ${id}`;
  }
}
```

사용하는 쪽에서는 이렇게 호출한다.

```js
const manager = new OrderManager();

manager.placeOrder("Pad Thai", "1234");
manager.trackOrder("1234");
manager.cancelOrder("1234");
```

이 구조에서는 `OrderManager`가 `placeOrder`, `trackOrder`, `cancelOrder`라는 구체적인 메서드를 직접 갖고 있다. 작업의 종류가 늘어날수록 `OrderManager`에 메서드가 계속 추가되고, 호출하는 쪽은 각 메서드의 이름과 시그니처를 알아야 한다.

### Command를 적용한 경우

Command Pattern을 적용하면 `OrderManager`는 **하나의 메서드 `execute()`만** 갖게 된다. 구체적인 작업은 Command 객체가 담당한다.

```js
class OrderManager {
  constructor() {
    this.orders = [];
  }

  execute(command, ...args) {
    return command.execute(this.orders, ...args);
  }
}
```

각 작업을 Command 객체로 분리한다.

```js
class Command {
  constructor(execute) {
    this.execute = execute;
  }
}

function PlaceOrderCommand(order, id) {
  return new Command((orders) => {
    orders.push(id);
    return `You have successfully ordered ${order} (${id})`;
  });
}

function TrackOrderCommand(id) {
  return new Command(() => {
    return `Your order ${id} will arrive in 20 minutes.`;
  });
}

function CancelOrderCommand(id) {
  return new Command((orders) => {
    orders = orders.filter((order) => order.id !== id);
    return `You have canceled your order ${id}`;
  });
}
```

이제 사용하는 쪽은 이렇게 호출한다.

```js
const manager = new OrderManager();

manager.execute(PlaceOrderCommand("Pad Thai", "1234"));
manager.execute(TrackOrderCommand("1234"));
manager.execute(CancelOrderCommand("1234"));
```

이 변환으로 얻는 것은 두 가지이다.

**1. 호출 형태가 통일된다.**

Command 적용 전에는 `manager.placeOrder(...)`, `manager.trackOrder(...)`, `manager.cancelOrder(...)` — 작업마다 다른 메서드를 호출해야 했다. Command 적용 후에는 모든 작업이 `manager.execute(command)` 하나의 형태로 통일된다. `OrderManager`는 어떤 작업이 들어오든 `execute()`만 호출하면 되고, 새로운 작업이 추가되더라도 `OrderManager`를 수정할 필요가 없다.

**2. 작업이 객체화되어 관리가 가능해진다.**

이것이 Command 패턴의 진짜 핵심이다. `execute()`라는 단일 통로로 모든 작업이 통과하기 때문에, 그 지점에서 **작업 객체를 들고 있을 수 있다.** 들고 있을 수 있다는 것은 — 이력을 기록하거나, 큐에 넣거나, 되돌리는 것이 가능해진다는 뜻이다.

```js
class OrderManager {
  constructor() {
    this.orders = [];
    this.history = []; // 실행된 작업을 이력으로 보관
  }

  execute(command, ...args) {
    const result = command.execute(this.orders, ...args);
    this.history.push(command); // 작업 객체를 저장해둔다
    return result;
  }

  getHistory() {
    return this.history; // 어떤 작업이 실행되었는지 추적 가능
  }
}
```

메서드를 직접 호출하는 방식(`manager.placeOrder(...)`)에서는 이런 구조가 불가능하다. 호출은 실행되는 순간 사라지기 때문이다.   
작업이 데이터로 존재해야 스택에 담고, 꺼내고, 되돌릴 수 있다.

### Before vs After 비교

| 구분 | Command 없이 | Command 적용 |
|:--|:--|:--|
| OrderManager의 메서드 | `placeOrder`, `trackOrder`, `cancelOrder` | `execute` 하나만 |
| 호출 형태 | 작업마다 다른 메서드 호출 | 모든 작업이 `execute(command)`로 통일 |
| 새 기능 추가 시 | OrderManager에 메서드 추가 필요 | 새 Command 함수만 작성 |
| 이력 관리 | 불가능 (호출은 실행되면 사라짐) | 가능 (Command 객체를 저장) |
| 호출자-실행자 결합도 | 높음 (메서드에 직접 의존) | 낮음 (execute 인터페이스에만 의존) |

## [How] Command Pattern은 어떻게 구현하는가?

핵심은 **작업을 객체(Command)로 캡슐화하고, 실행자(Invoker)는 Command의 `execute()` 메서드만 호출하면 되도록 하는 것**이다.

### 기본 구조

Command Pattern은 네 가지 참여자로 구성된다.

```
Client → Command → Receiver
            ↑
         Invoker
```

| 참여자 | 역할 |
|:--|:--|
| **Command** | 실행할 작업의 인터페이스. `execute()`와 선택적으로 `undo()`를 정의한다 |
| **ConcreteCommand** | Command 인터페이스를 구현하는 구체적인 작업 클래스 |
| **Invoker** | Command를 실행하는 주체. Command의 내부 구현을 모른다 |
| **Receiver** | 실제 작업을 수행하는 객체. Command가 Receiver의 메서드를 호출한다 |

TypeScript로 기본 구조를 표현하면 다음과 같다.

```ts
// Command 인터페이스
interface Command {
  execute(): void;
  undo?(): void;
}

// Receiver: 실제 작업을 수행하는 객체
class Light {
  on() {
    console.log("불을 켰다");
  }

  off() {
    console.log("불을 껐다");
  }
}

// ConcreteCommand: 구체적인 작업
class LightOnCommand implements Command {
  constructor(private light: Light) {}

  execute() {
    this.light.on();
  }

  undo() {
    this.light.off();
  }
}

class LightOffCommand implements Command {
  constructor(private light: Light) {}

  execute() {
    this.light.off();
  }

  undo() {
    this.light.on();
  }
}

// Invoker: Command를 실행하는 주체
class RemoteControl {
  private history: Command[] = [];

  executeCommand(command: Command) {
    command.execute();
    this.history.push(command);
  }

  undoLast() {
    const command = this.history.pop();
    command?.undo?.();
  }
}
```

```ts
// 사용
const light = new Light();
const remote = new RemoteControl();

remote.executeCommand(new LightOnCommand(light));   // "불을 켰다"
remote.executeCommand(new LightOffCommand(light));   // "불을 껐다"
remote.undoLast();                                    // "불을 켰다" (undo)
```

`RemoteControl`(Invoker)은 `Light`(Receiver)의 존재를 모른다. 오직 `Command` 인터페이스의 `execute()`와 `undo()`만 알고 있다. 이것이 Command Pattern의 핵심적인 분리이다.

## [Why] Command Pattern은 왜 사용하는가?

Q. 언제 사용하면 좋을까?

핵심은 **작업의 실행 시점을 제어하거나, 작업을 되돌릴 수 있어야 하거나, 작업의 이력을 관리해야 할 때**이다.

구체적으로는 다음과 같은 상황에서 유용하다.

- 사용자의 작업을 **Undo/Redo**해야 할 때
- 작업을 **큐에 넣어** 순차적으로 실행하거나, 지연 실행해야 할 때
- 실행된 작업의 **이력을 기록**하고 감사(audit)해야 할 때
- **UI 컴포넌트**(버튼, 메뉴)와 **비즈니스 로직**을 분리하고 싶을 때

Q. 왜 메서드를 직접 호출하지 않고 Command로 감싸는가?

1. **Undo/Redo**
    - 각 Command에 `execute()`와 `undo()`를 구현하면, 작업 이력을 스택으로 관리할 수 있다. 메서드 직접 호출로는 이 구조를 만들 수 없다. 작업이 "객체"여야 스택에 담을 수 있기 때문이다.

2. **실행 지연/큐잉**
    - Command 객체를 큐에 담아 나중에 실행하거나, 다른 스레드/프로세스에서 실행할 수 있다. 메서드 호출은 즉시 실행되지만, Command 객체는 "아직 실행하지 않은 작업"으로 보관할 수 있다.

3. **로깅/감사**
    - 실행된 Command를 기록하면 작업 이력이 자동으로 남는다. "누가 언제 무엇을 했는가"를 추적하는 데 있어 Command 객체는 자기 설명적인(self-descriptive) 로그가 된다.

4. **호출자와 실행자의 분리**
    - UI 버튼(Invoker)은 비즈니스 로직(Receiver)을 알 필요 없다. 버튼은 그저 `command.execute()`를 호출할 뿐이다. 같은 Command를 메뉴, 단축키, 스크립트에서 재사용할 수 있다.

Q. Command Pattern의 단점은 무엇인가?

- **코드 복잡도 증가**: 간단한 작업도 Command 클래스를 만들어야 하므로 보일러플레이트가 늘어난다. "불 켜기" 하나에 `LightOnCommand` 클래스를 만들어야 하는 것은 과도해 보일 수 있다.

- **과도한 추상화**: patterns.dev에서도 "쓸만한 상황이 딱히 많지 않고, 종종 불필요한 코드가 만들어진다"고 언급한다. Command Pattern은 Undo/Redo, 큐잉 같은 명확한 요구사항이 있을 때 도입해야지, "나중에 필요할 수도 있으니까"라는 이유로 도입하면 안 된다.

- **Undo 구현의 어려움**: 모든 작업이 되돌릴 수 있는 것은 아니다. 외부 API 호출, 파일 삭제, 이메일 발송 등은 undo가 불가능하거나 매우 복잡하다. Undo를 지원하려면 각 Command가 복원에 필요한 상태를 보관해야 하는데, 이 상태 관리 자체가 새로운 복잡성을 만들어낸다.

| 장점 | 단점 |
|:--|:--|
| Undo/Redo 자연스럽게 구현 가능 | 클래스/객체 수 증가로 보일러플레이트 발생 |
| 작업의 지연 실행, 큐잉 가능 | 단순한 작업에는 과도한 추상화 |
| 실행 이력 로깅/감사 용이 | 모든 작업의 Undo가 가능한 것은 아님 |
| 호출자-실행자 완전 분리 | 상태 복원 로직의 복잡성 |

## [Hum..🤔] Redux의 Action/Dispatch

```js
// Command 패턴
const addTodoCommand = new AddTodoCommand({ text: '공부하기' });
invoker.execute(addTodoCommand);

// Redux
dispatch({ type: 'ADD_TODO', payload: { text: '공부하기' } });
```

Redux의 action은 **작업을 데이터로 표현한 것**이다. `{ type: 'ADD_TODO', payload: {...} }`는 "할 일을 추가하라"는 명령을 객체로 캡슐화한 것이며, 이는 Command Pattern의 정의와 정확히 일치한다.

| Command Pattern 용어 | Redux 대응 |
|:--|:--|
| Command | Action (`{ type, payload }`) |
| Invoker | `dispatch()` |
| Receiver | Reducer |
| execute() | Reducer가 action을 처리하는 것 |
| 이력 관리 | Redux DevTools의 time-travel debugging |

Redux DevTools에서 과거 상태로 돌아갈 수 있는 **Time Travel Debugging**도 결국 "모든 작업이 Command(Action) 객체로 기록되어 있기 때문에" 가능한 것이다.
