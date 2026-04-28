# 커맨드 패턴
- 커맨드 패턴을 사용하면 특정 작업을 실행하는 개체와 메서드를 호출하는 개체를 분리할 수 있습니다.

## 예시
- 온라인 음식 배달 플랫폼을 개발한다고 가정해보겠습니다.
  - 사용자는 주문하거나, 주문한 음식이 어디쯤 왔는지 확인하거나, 주문을 취소할 수 있습니다.
  ```ts
  class OrderManager() {
    constructor() {
      this.orders = []
    }

    placeOrder(order, id) {
      this.orders.push(id)
      return `You have successfully ordered ${order} (${id})`;
    }

    trackOrder(id) {
      return `Your order ${id} will arrive in 20 minutes.`
    }

    cancelOrder(id) {
      this.orders = this.orders.filter(order => order.id !== id)
      return `You have canceled your order ${id}`
    }
  }

  // 사용처
  const manager = new OrderManager()

  manager.placeOrder('Pad Thai', '1234')
  manager.trackOrder('1234')
  manager.cancelOrder('1234')
  ```
  - 하지만 이렇게 manager의 메서드를 직접 사용하는 경우 추후에 특정 메서드의 이름을 변경하거나 메서드의 기능을 변경해야 하는 경우가 발생할 수 있습니다.
    - placeOrder 대신에 이름을 addOrder로 변경하면 전체 코드베이스에서 해당 메서드를 호출하지 않도록 코드를 수정해야 합니다.
    - 앱의 규모가 크면 까다로운 작업이 될 것입니다.
    - 이렇게 하는 대신, manager 객체로부터 메서드를 분리하고 각각의 명령을 처리하는 함수를 만들 것 입니다.
  - 먼저 OrderManager 클래스를 리펙터링 해보겠습니다.
    - 각 메서드를 직접 구현하는 대신 execute라는 하나의 메서드만 가지도록 합니다.
    - 이 메서드는 인자로 주어진 어떤 명령이든 실행할 수 있습니다.
    ```ts
    class OrderManager {
      constructor() {
        this.orders = []
      }

      execute(command, ...args) {
        return command.execute(this.orders, ...args)
      }
    }
    ```
  - 아래 예시 코드에 3개의 커맨드를 추가하겠습니다.
    ```ts
    class Command {
      constructor(execute) {
        this.execute = execute
      }
    }

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
    - 이로써 OrderManager가 메서드를 직접 갖는 대신 execute 메서드를 통해 분리된 함수를 사용하도록 코드를 리펙터링 되었습니다.

## 장점
- 커맨드 패턴은 객체와 메서드를 분리할 수 있게 해줍니다.
  - 이렇게 분리하면 수명이 지정된 명령을 만들거나, 명령들을 큐에 담아 특정한 시간대에 처리하는 것도 가능해집니다.

## 단점
- 커맨드 패턴을 쓸만한 상황이 딱히 많지 않고 종종 불필요한 코드가 만들어지곤 합니다.

---
# 추가 보충
- "군대"에 비유가 가능합니다.
  - 요청자: 명령을 하는 장교
  - 수신자: 명령을 받는 병사
- 실행할 명령(command)을 객체로 만들어 요청자(invoker)와 수신자(receiver)를 분리하는 디자인 패턴
  - Command: 실행할 작업의 인터페이스 (execute(), undo())
  - ConcreteCommand: 실제 명령 구현 (Receiver와 연결)
  - Invoker: 명령 실행을 트리거
  - Receiver: 실제 작업 수행자

## 기본 커맨드 패턴
```ts
// 1. Command 인터페이스
interface Command {
  execute(): void;
  undo(): void;
}

// 2. Receiver (실제 작업 수행자)
// 실제 행동들이 여기 구성됨
class Light {
  turnOn() { console.log("전등 켜짐"); }
  turnOff() { console.log("전등 꺼짐"); }
}

// 3. ConcreteCommand
// Receiver를 받으면서 command도 전달받는 역할
// Receiver - Command의 중간 역할
class TurnOnCommand implements Command {
  constructor(private light: Light) {}

  execute() { this.light.turnOn(); }
  undo() { this.light.turnOff(); }
}

class TurnOffCommand implements Command {
  constructor(private light: Light) {}

  execute() { this.light.turnOff(); }
  undo() { this.light.turnOn(); }
}

// 4. Invoker (명령 실행자)
class RemoteControl {
  private command!: Command;

  setCommand(command: Command) {
    this.command = command;
  }

  pressButton() {
    this.command.execute();
  }

  pressUndo() {
    this.command.undo();
  }
}

// 사용 예시
const light = new Light();
const remote = new RemoteControl();

remote.setCommand(new TurnOnCommand(light));
remote.pressButton();  // "전등 켜짐"
remote.pressUndo();    // "전등 꺼짐"
```

### 실행 취소(Undo) 기능이 있는 텍스트 편집기
> 그럴일은 거의 없겠지만, 디자이너 혹은 기획자 혹은 회사에서 강력하게 기능요청을 했고 그 기능이 라이브러리에서 구현할 수 없는 상태라면 텍스트 에디터를 직접 만들어야 하는 순간이 오기도 합니다.
> 이런 경우 커맨드 패턴으로 로직을 작성하시면 됩니다.
```tsx
// Receiver
class TextEditor {
  private content = "";

  write(text: string) {
    this.content += text;
    console.log(`현재 내용: ${this.content}`);
  }

  deleteLastChar() {
    this.content = this.content.slice(0, -1);
    console.log(`삭제 후: ${this.content}`);
  }

  getContent() { return this.content; }
}

// Command
interface TextCommand {
  execute(): void;
  undo(): void;
}

class WriteCommand implements TextCommand {
  private prevContent = "";

  constructor(private editor: TextEditor, private text: string) {}

  execute() {
    this.prevContent = this.editor.getContent();
    this.editor.write(this.text);
  }

  undo() {
    this.editor["content"] = this.prevContent; // 실제 프로젝트에서는 setter 사용
    console.log(`UNDO: ${this.editor.getContent()}`);
  }
}

// Invoker
class KeyboardShortcut {
  private commands: TextCommand[] = [];

  executeCommand(command: TextCommand) {
    command.execute();
    this.commands.push(command);
  }

  undoLastCommand() {
    const lastCommand = this.commands.pop();
    if (lastCommand) lastCommand.undo();
  }
}

// 사용 예시
const editor = new TextEditor();
const shortcut = new KeyboardShortcut();

shortcut.executeCommand(new WriteCommand(editor, "Hello"));
shortcut.executeCommand(new WriteCommand(editor, " World!"));
shortcut.undoLastCommand();  // "UNDO: Hello"
```

### 작업 큐 시스템 (Redux 미들웨어)
```ts
class CommandQueue {
  private queue: Command[] = [];

  addCommand(command: Command) {
    this.queue.push(command);
  }

  processAll() {
    this.queue.forEach(cmd => cmd.execute());
    this.queue = [];
  }
}

// 사용 예시 (React 훅)
function useCommandQueue() {
  const queueRef = useRef(new CommandQueue());

  const addToQueue = (command: Command) => {
    queueRef.current.addCommand(command);
  };

  const executeAll = () => {
    queueRef.current.processAll();
  };

  return { addToQueue, executeAll };
}
```
  - queue 형태의 데이터를 훅으로 관리하는 경우에는 예시처럼 커맨드 패턴을 주로 사용합니다.

## 커맨드 패턴의 장점
1. 요청자(Invoker)와 수행자(Receiver)의 분리
  - 결합도 감소: 버튼(Invoker)은 구체적인 작업 내용을 몰라도 됨
  - 유연성 향상: 동일한 Invoker로 다양한 Command 실행 가능
2. 실행 취소(Undo)/재실행(Redo)지원
  - 명령 객체가 상태를 보존하므로 쉽게 롤백 가능
  > 특히 롤백같은 구현을 진행할때 아주아주아주 유용함
3. 명령의 조합 및 큐 관리 용이 ⭐️⭐️⭐️⭐️⭐️
  - 매크로 커맨드: 여러 명령을 하나로 묶어 실행
  - 작업 큐: 명령을 저장하여 나중에 처리
  > 큐를 직접 만든다? === 커맨드 패턴으로 만들어야 진짜 맛도리임
4. 비동기 작업 관리에 적합
  - 명령 객체를 스레드 풀, 이벤트 루프 등에서 실행 가능
5. 새로운 명령 추가가 쉬움
  - Command 인터페이스만 구현하면 확장 가능함 (OCP 원칙 준수)

## 커맨드 패턴의 단점
1. 클래스 수 증가
  - 각 명령마다 별도의 클래스가 필요함 -> 작은 기능도 클래스로 분리
2. 복잡도 상승
  - 간단한 작업도 Command 객체로 래핑해야 함
3. 성능 오버헤드
  - 객체 생성 비용이 발생
4. 과도한 설계 가능성
  - 간단한 기능애 페턴 적용시 오히려 복잡도 상승
  > queue 혹은 텍스트편집기 같은 케이스는 커맨드 패턴이 "정답"입니다. 암기 레쓰고