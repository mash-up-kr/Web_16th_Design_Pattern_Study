# 1. 커맨드 패턴

요청 자체를 객체로 캡슐화해서, 실행할 동작을 독립적인 단위로 다루는 패턴

⇒ "무엇을 실행할지" 를 값처럼 들고 다닐 수 있게 만드는 패턴

## 1-1. 필요

어떤 버튼이 눌렸을 때 특정 동작을 실행한다고 가정

```tsx
button.onClick = () => {
  editor.bold();
};
```

처음엔 이 정도면 충분해보임

근데 기능이 늘어나기 시작하면 이야기가 달라짐

- 실행 취소( undo )
- 다시 실행( redo )
- 작업 이력 저장
- 매크로 실행
- 버튼마다 다른 동작 연결

이런 요구가 붙기 시작함

그냥 함수 직접 호출 구조에서는

- 지금 무슨 작업이 실행됐는지 저장하기 애매하고
- 나중에 다시 실행하거나 취소하기 어렵고
- 요청을 데이터처럼 다루기 어려움

⇒ 그래서 "실행할 요청" 자체를 하나의 객체로 감싸서 다루자, 여기서 커맨드 패턴이 등장

## 1-2. 정의

커맨드 패턴은 요청을 객체로 만들어서

- 실행할 수 있게 하고
- 저장할 수 있게 하고
- 전달할 수 있게 하고
- 필요하면 취소 / 재실행도 가능하게

만드는 패턴

핵심은 **메서드 호출을 바로 하지 않고, 실행 요청 자체를 하나의 객체로 분리** 하는 것

> 호출하는 쪽과, 실제 작업을 수행하는 쪽을 분리하고  
> 요청 자체를 독립적인 단위로 다루는 패턴
>

## 1-3. 구조

- **Command** : 실행 인터페이스
- **ConcreteCommand** : 실제 실행 내용을 담은 커맨드 객체
- **Receiver** : 진짜 일을 수행하는 객체
- **Invoker** : 커맨드를 호출하는 객체
- **Client** : 커맨드를 만들고 연결하는 쪽

```tsx
interface Command {
  execute(): void;
}
```

중요한건 Invoker가 Receiver를 직접 모르는 경우가 많다는 점

Invoker는 그냥

```tsx
command.execute()
```

만 호출

실제로 무슨 일이 일어나는지는 Command 내부가 책임짐

## 1-4. 동작 원리

1. Client가 Receiver를 준비
2. Receiver를 감싼 ConcreteCommand 생성
3. Invoker에 그 커맨드를 연결
4. Invoker는 필요할 때 `execute()` 호출
5. 실제 동작은 내부에서 Receiver가 수행

```tsx
class Light {
  on() {
    console.log('불 켜짐');
  }
}

class LightOnCommand {
  constructor(light) {
    this.light = light;
  }

  execute() {
    this.light.on();
  }
}

class Button {
  setCommand(command) {
    this.command = command;
  }

  click() {
    this.command.execute();
  }
}

const light = new Light();
const lightOnCommand = new LightOnCommand(light);

const button = new Button();
button.setCommand(lightOnCommand);
button.click();
```

여기서 보면

- `Button` 은 불을 켜는 방법을 모름
- `Light` 는 버튼의 존재를 모름
- 둘 사이를 `LightOnCommand` 가 연결

즉 버튼은 "무슨 동작인지" 까지는 모르고, 그냥 실행만 함

## 1-5. 그냥 함수 넘기면 되는거 아닌가?

```tsx
button.onClick = () => light.on();
```

맞음

단순 실행만 보면 함수 넘기기로 충분

근데 커맨드 패턴이 필요한 순간은 조금 다름

- 실행 이력을 남기고 싶을 때
- undo / redo가 필요할 때
- 요청을 큐에 저장하고 싶을 때
- 실행 대상을 동적으로 갈아 끼우고 싶을 때

즉 **요청을 단순 콜백이 아니라, 상태 있는 실행 단위로 다루고 싶을 때** 커맨드 패턴이 의미가 생김

## 1-6. 실제 활용

커맨드 패턴은 에디터에서 특히 잘 어울림

예를 들어 텍스트 에디터에서 bold 적용 기능이 있다고 하면

```tsx
class Editor {
  bold() {
    console.log('볼드 적용');
  }

  unbold() {
    console.log('볼드 해제');
  }
}

class BoldCommand {
  constructor(editor) {
    this.editor = editor;
  }

  execute() {
    this.editor.bold();
  }

  undo() {
    this.editor.unbold();
  }
}
```

이렇게 되면

- 실행할 때는 `execute()`
- 취소할 때는 `undo()`

로 다룰 수 있음

```tsx
class History {
  constructor() {
    this.stack = [];
  }

  push(command) {
    this.stack.push(command);
  }

  undo() {
    const command = this.stack.pop();
    command?.undo();
  }
}
```

핵심은 "볼드 적용" 이 그냥 메서드 호출로 끝나는게 아니라, 하나의 작업 단위로 기록된다는 것

⇒ 그래서 Undo / Redo, 매크로, 작업 기록 같은 기능이 가능해짐

## 1-7. 프론트엔드에서의 감각

프론트엔드에서 커맨드 패턴은 "사용자 액션을 실행 가능한 단위로 모델링" 하는 느낌으로 볼 수 있음

- 에디터 툴바 명령
- 단축키 명령
- 메뉴 액션
- 드로잉 앱 조작
- 실행 취소 / 다시 실행

예를 들어

```tsx
saveCommand.execute()
deleteSelectionCommand.execute()
duplicateLayerCommand.execute()
```

이런 식으로 액션을 개별 객체 / 단위로 다루는 구조

특히 undo / redo가 들어가는 순간, 커맨드 패턴의 장점이 확실히 드러남

## 1-8. 트레이드 오프

장점

- 호출자와 실행자 분리
- 요청을 객체로 저장 / 전달 가능
- undo / redo 구현에 유리
- 작업 큐, 매크로, 히스토리 관리에 적합

단점

- 클래스 / 객체 수가 늘어남
- 단순한 동작에도 구조가 무거워질 수 있음
- 작은 프로젝트에서는 오버엔지니어링이 되기 쉬움

⇒ 실행 요청을 독립된 단위로 다뤄야 할 때는 강력하지만, 단순 클릭 처리 수준에서는 과할 수 있음

## 1-9. 커맨드 패턴이 빛나는 순간

1. 작업 이력을 남겨야 할 때
2. 실행 취소 / 다시 실행이 필요할 때
3. 요청을 큐잉하거나 지연 실행해야 할 때
4. 같은 Invoker에 여러 동작을 갈아 끼워야 할 때

반대로

- 그냥 한 번 실행하고 끝
- 이력 없음
- undo 없음
- 요청을 저장할 필요 없음

이면 그냥 함수 호출이 더 단순함

## 1-10. 커맨드 vs 콜백

콜백

- 그냥 실행할 함수를 넘김
- 가볍고 단순함
- 일회성 처리에 적합

커맨드

- 실행 요청 자체를 객체화
- 필요한 데이터와 실행 / 취소 로직을 함께 가질 수 있음
- 저장, 기록, 재실행에 적합

즉

- "지금 그냥 실행" 이 목적이면 콜백
- "실행 요청을 관리" 해야 하면 커맨드

## 1-11. 커맨드 vs 전략 패턴

둘 다 뭔가 바꿔 끼우는 느낌이 있어서 헷갈림

전략 패턴은 알고리즘 / 동작 방식을 교체하는 쪽에 가깝고,
커맨드 패턴은 실행 요청 자체를 객체화하는 쪽에 가까움

즉 전략은 "어떻게 할까" 에 가깝고,
커맨드는 "무엇을 실행할까" 에 더 가까움

## 1-12. 공부 중 의문

함수형으로도 커맨드 패턴이 가능한가?

⇒ 가능

꼭 class가 아니어도 됨

```tsx
const boldCommand = {
  execute() {
    editor.bold();
  },
  undo() {
    editor.unbold();
  },
};
```

핵심은 문법이 아니라

- 요청을 하나의 단위로 묶고
- 실행 / 취소 / 저장 가능하게 만들고
- 호출자와 실행자를 분리

하는 설계 의도

그래서 클래스 문법을 쓰지 않아도, 이 의도를 만족하면 커맨드 패턴으로 볼 수 있음

---

커맨드 패턴은 단순히 메서드를 감싸는게 아니라,

- 요청을 실행 가능한 객체로 만들고
- 그 요청을 기록 / 전달 / 취소할 수 있게 하며
- 호출자와 실제 작업 수행자를 분리

하는 패턴

즉 "동작을 바로 호출" 하는 구조에서  
"동작 자체를 다룰 수 있는 구조" 로 바꾸는 패턴이라고 볼 수 있다
