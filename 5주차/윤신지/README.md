_* Mash-UP 16기 웹 팀 내 [디자인패턴 스터디](https://github.com/mash-up-kr/Web_16th_Design_Pattern_Study)에서 진행된 내용으로, [patterns.dev](https://www.patterns.dev/)를 읽고 정리하는 방식으로 진행되었습니다._
_** 정리하다 궁금한 내용(궁금한 내용이 아니더라도 스터디 시간에 얘기하고 싶은)은 ❓ 이모지로 정리해 두었습니다._

> **5주차 자료**
[Observer Pattern](https://www.patterns.dev/vanilla/observer-pattern/)
[Mediator/Middleware Pattern](https://www.patterns.dev/vanilla/mediator-pattern/)
[Command Pattern](https://www.patterns.dev/react/command-pattern/)

---

## Observer Pattern
> 일부 객체들이 다른 객체들에 자신의 상태 변경에 대해 알릴 수 있는 행동 디자인 패턴

**Observer** 패턴은 어떤 객체의 상태를 관찰하고 상태가 변화했을 때 통지를 받아 해당 객체의 상태를 관찰하는 객체들 간에 동기화하기 위해 사용하는 디자인 패턴이다.
👉 관찰자들이 관찰하고 있는 대상자의 상태가 변화가 있을 때마다 대상자는 직접 목록의 각 관찰자들에게 통지하고, 관찰자들은 알림을 받아 조치를 취하는 행동 패턴


### Observer 패턴의 유래
Observer 패턴은 GoF의 디자인 패턴에서 소개되었다. 1980년대 Smalltalk-80이라는 언어의 MVC 구조에서 기원을 찾을 수 있는데, 당시 개발자들은 데이터가 변경될 때 UI가 자동으로 갱신되길 원했다. 그러나 UI를 그리라는 코드를 직접 넣을 경우, ___모델이 뷰의 존재를 알아야 해___ 유지보수 측면에서 아~주 좋지 않은 코드가 된다.

👉 이를 해결하기 위해 **Subject**와 **Observer**라는 역할을 분리하였다. 이를 통해 주체는 누가 자신을 보고 있는지 알 필요가 없으며, 본인의 상태가 변했음만을 알려 관심사를 분리할 수 있다. 

### 핵심 동작 원리

#### Observer 패턴의 구성 요소
> - **Subject(Observable)** : 상태를 보유하고 있으며, 관찰자 목록을 유지한다. 상태가 변경되면 목록에 있는 모든 관찰자에게 알림을 보낸다.
- **Observser**: 주체를 관찰하고 있으며, 주체로부터 알림을 받았을 때 실행될 업데이트 로직을 가지고 있다. 

#### Observer 패턴의 핵심 동작 원리
> - **Registration(subscribe)** : 관찰자가 주체에게 자신을 등록한다.
- **Notification(notify)** : 주체의 상태가 변하면 등록된 모든 관찰자들에게`update`를 실행한다.
- **Unregistrantion(unsubscribe)** : 더 이상 알림이 필요 없는 관찰자는 목록에서 제거한다. 

- Observer 패턴에서는 관찰 대상의 상태가 바뀌면 변경사항을 Observer에게 통보해준다.
- 대상자로부터 통보를 받은 Observer는 값을 바꾸거나 삭제하는 등 적절하게 대응한다.
- 또한 Observer들은 언제든 Subject 그룹에서 추가/삭제될 수 있다. Subject 그룹에 추가되면 Subject로부터 정보를 전달받게 될 것이며, 그룹에서 삭제될 경우 더이상 Subject의 정보를 받을 수 없게 된다. 

![](https://velog.velcdn.com/images/dawnww/post/56a1534c-4791-4065-ad7b-475616241018/image.png)

Observer 패턴은 객체 간에 일대다 관계를 유지하며, 관찰자들은 데이터에 대한 접근 권한이 없기 때문에 정보를 제공하는 주체에 의존한다. 한 객체가 업데이트될 경우 종속 객체들이 자동으로 변화에 대해 통보받는다. 

![](https://velog.velcdn.com/images/dawnww/post/d66ee7f2-768c-4d8f-ae55-e2f8bed7d621/image.png)

### JS 예제
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
위와 같이 `subscribe`메서드를 통해 `Observer`를 등록하고 반대로 `unsubscribe`를 통해 등록 해지할 수 있다. 그리고 `notify`메서드를 통해 모든 `Observer`에게 이벤트를 전파할 수 있다.

### React 예제
`useEffect`는 의존성 배열에 있는 값을 관찰하다가 변화가 생기면 콜백(Observser)를 실행한다. 
```typescript
type EventCallback = (data: any) => void;

class EventBus {
  private static instance: EventBus;
  private listeners: { [key: string]: EventCallback[] } = {};

  private constructor() {}

  public static getInstance(): EventBus {
    if (!EventBus.instance) {
      EventBus.instance = new EventBus();
    }
    return EventBus.instance;
  }

  // 구독 - 특정 이벤트에 대해 관찰자를 등록
  public on(event: string, callback: EventCallback): void {
    // 해당 이벤트에 대한 리스너가 아직 없다면 초기화
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    // 리스트에 콜백 추가 
    this.listeners[event].push(callback);
  }

  // 구독 해제 -> 메모리 누수 방지
  public off(event: string, callback: EventCallback): void {
    if (!this.listeners[event]) return;
    
    // 참조값을 비교하여 리스트에서 해당 콜백만 필터링하여 불변성 유지 
    this.listeners[event] = this.listeners[event].filter(cb => cb !== callback);
  }

  // 알림 
  public emit(event: string, data: any): void {
    if (!this.listeners[event]) return;
    this.listeners[event].forEach(callback => callback(data));
  }
}

// React Hook 형태로 래핑하여 사용
import { useEffect } from 'react';

export const useEventObserver = (eventName: string, handler: EventCallback) => {
  useEffect(() => {
    const bus = EventBus.getInstance();
    bus.on(eventName, handler);

    // 언마운트 시 등록했던 핸들러 해제!!
    return () => {
      bus.off(eventName, handler); // 컴포넌트 언마운트 시 반드시 해제
      // -> 해당 과정이 없으면 컴포넌트가 사라져도 EventBus 메모리에는 핸들러가 남아 좀비 옵저버가 될 수 있음 
    };
  }, [eventName, handler]);	// 이벤트명이나 핸들러가 바뀌면 구독 갱신 
};
```

### 실무 예제
알림 보내기 버튼을 누르면 토스트 메시지가 보내지는 컴포넌트 통신을 가정해보자. 
```typescript
// 알림 보내기 버튼 -> Publisher에 해당

const LogoutButton = () => {
  const handleLogout = () => {
    // ... 로그아웃 로직 수행
    EventBus.getInstance().emit('toast_message', {
      type: 'success',
      text: '로그아웃 되었습니다!'
    });
  };

  return <button onClick={handleLogout}>로그아웃</button>;
};
```

```typescript
// 알림을 보여주는 토스트 컴포넌트 -> Subscriber에 해당 

const ToastContainer = () => {
  const [message, setMessage] = useState('');

  // 훅을 통해 'toast_message' 이벤트를 관찰함
  useEventObserver('toast_message', (data) => {
    setMessage(data.text);
    // 3초 뒤 메시지 삭제 로직 등...
  });

  return message ? <div className="toast">{message}</div> : null;
};
```

위와 같이 구현할 경우 세 개의 장점이 있다.
- **Props Drilling 해결** : ToastContainer가 어디에 위치하든 상관없다. Props를 타고 내려갈 필요가 없다.
- **사이드 이펙트 분리**: 비즈니스 로직(로그아웃)과 UI 피드백(토스트 띄우기)이 완전히 분리되어, 로그아웃 버튼 코드를 수정하지 않고도 알림 방식을 바꿀 수 있다. 
- **확장성**: 나중에 로그아웃 시 "로그 서버로 전송" 기능을 추가하고 싶다면, `EventBus.getInstance().on('toast_message', logHandler)`를 하나 더 추가하기만 하면 된다. 


### Push/Pull 모델 비교
#### Push 모델
주체가 상태 데이터를 관찰자에게 직접 밀어넣어 주는 방식
- 장점 : 관찰자가 데이터를 별도로 요청할 필요가 없음
- 단점 : 주체가 관찰자에게 어떤 데이터를 필요로 하는지 정확히 알아야 하거나, 불필요하게 많은 데이터를 전송할 수 있음

#### Pull 모델
주체가 **상태가 변했다**는 사실만 알리고, 관찰자가 필요한 데이터를 주체로부터 직접 가져오는 방식
- 장점 : 관찰자가 자신에게 필요한 데이터만 선택적으로 가져올 수 있어서 유연
- 단점 : 관찰자가 주체의 내부 구조를 어느정도 알아야 하므로 결합도 상승


### Observer Pattern vs Pub/Sub 패턴 비교

| 구분 | Observer Pattern | Pub/Sub Pattern |
| :--- | :--- | :--- |
| **결합도** | **Subject와 Observer가 서로를 알고 있음** | **중간 매개체(Message Broker) 덕분에 서로 모름** |
| **통신** | **직접 통신 (동기적 성향)** | **Event Bus를 통한 간접 통신 (비동기적 성향)** |
| **구조** | **1:N 관계 중심** | **토픽/채널 기반의 다대다 관계 가능** |

> 옵저버 패턴은 주체가 관찰자를 직접 관리하는 **'객체 지향적'** 모델인 반면, Pub/Sub은 중간에 브로커를 두어 시스템 간의 결합도를 완전히 제거한 **'메시지 지향적'** 아키텍처에 가깝다.
옵저버 패턴은 대상과 결합되어 있기 때문에 같은 공간에서 구현되어야 하지만, 발행/구독 패턴은 애플리케이션 간 통신이기 때문에 이러한 제한이 없다. 


### 장단점
#### 장점
- 변경 없이 정보 또는 데이터 전달을 여러 객체에 허용한다.
- 객체 간에 통신하는 느슨한 결합을 준수한다.
- 개방/폐쇄 원칙을 따르며, Observer는 코드 변경 없이 언제든지 쉽게 추가하거나 제거할 수 있다.

#### 단점
- 복잡성을 증가시킬 수 있다.
- 효율성 문제를 야기할 수 있다.


## Mediator/Middleware Pattern
> 여러개의 객체들이 상호의존적인 상황에서 각 객체들 간의 결합성을 낮추기 위해 사용하는 패턴
👉 중재자 객체를 통해 객체 간의 복잡한 통신 및 제어 로직을 중앙집중화하여 느슨한 결합을 촉진하고 의존성을 줄이는 행동적 디자인 패턴

[4markdown](https://4markdown.com/mediator-pattern-in-typescript/)에 재밌는 예시 문장이 있길래 가져와봤습니다....
> 당신이 배우자와 이혼 절차를 밟고 있다고 상상해보세요 (운이 좋지 않다면). 당신들은 서로를 매우 싫어해서 직접적인 소통이 불가능합니다. 그러나 집, 자녀, 기타 자산과 같은 책임은 공유합니다.
이 시점에서, 당신의 친구를 중재자로 부릅니다. - 네, 그것이 바로 패턴의 이름이자 역할입니다. 이 중재자는 당신과 배우자 사이의 모든 소통과 책임을 처리할 것입니다. 그들은 중립적인 태도로 정보를 전달하고, 서류 작업이나 재정 협상과 같은 필요한 조치를 관리하여, 두 사람이 직접 대면하는 것을 피할 수 있게 합니다. 


### Mediator 패턴의 유래
Observer 패턴과 동일하게 GoF의 디자인 패턴에서 소개되었다. 중재자 패턴은 **디미터의 법칙** 과 궤를 같이하는데, 이 법칙은 객체는 자신이 사용하는 협력 객체의 내부 구조를 몰라야 한다는 원칙이다.
> 디미터의 법칙이란?
디미터의 법칙은 “Object-Oriented Programming: An Objective Sense of Style” 에서 처음으로 소개되었다. Demeter라는 프로젝트를 진행하던 개발자들은 어떤 객체가 다른 객체에 대해 지나치게 많이 알다보니, 결합도가 높아지고 좋지 못한 설계를 야기한다는 것을 발견하였다. 그래서 이를 개선하고자 객체에게 자료를 숨기는 대신 함수를 공개하도록 하였는데, 이것이 바로 디미터의 법칙이다.
즉, 디미터의 법칙은 다른 객체가 어떠한 자료를 갖고 있는지 속사정을 몰라야 한다는 것을 의미하며, 이러한 이유로 Don’t Talk to Strangers(낯선 이에게 말하지 마라) 또는 Principle of least knowledge(최소 지식 원칙) 으로도 알려져 있다.
또는 직관적으로 이해하기 위해 여러 개의 .(도트)을 사용하지 말라는 법칙으로도 많이 알려져 있으며, 디미터의 법칙을 준수함으로써 캡슐화를 높혀 객체의 자율성과 응집도를 높일 수 있다.

중재자를 도입하면 객체 간의 관계가 다대다에서 다대일로 단순화된다. 👉 유지보수 비용을 획기적으로 줄일 수 있다. 

당시 객체 지향 프로그래밍은 모든 것을 객체로 나누라고 가르쳤으나, 기능을 잘게 쪼개어 객체를 많이 만들 수록 그 객체들이 서로 통신하기 위해 서로를 참조해야 하는 상황에 직면했다. 결과적으로 하나를 수정하면 열 개가 고장나는 의존성 지옥에 빠지게 된 것이다.

### 핵심 동작 원리
중재자 패턴은 데이터가 어디로 가는지 쉽게 알 수 있다. 중재자를 사용하면 모든 것이 중앙집중화되며, 확장성이 좋다. 중재자가 모든 상호작용을 처리하고 다른 객체들에게 숨기기 때문에 참여하는 객체가 얼마나 많든 서로 알 필요가 없다. 

기존 방식과 중재자 통신의 차이를 기술적으로 분석하면 아래와 같은 차이점이 있다. 

#### 직접 통신
- 상태 : 모든 객체가 서로의 존재를 알고 있어야 한다.
- 복잡도 : 객체가 n개일 때, 관계의 수는 최대 n(n-1)/2에 달한다.
- 문제점 : 특정 객체를 다른 프로젝트에 재사용하고 싶어도, 그 객체가 참조하고 있는 다른 수많은 객체까지 함께 가져가야 해 사실상 재사용이 불가능하다.
![](https://velog.velcdn.com/images/dawnww/post/f079bca7-8e6d-414b-929f-e88e46243169/image.png)


#### 중재자 통신
- 상태 : 각 객체는 오직 **중재자**만 안다. 다른 동료 객체가 누구인지, 심지어 존재하는 지 조차 몰라도 된다.
- 복잡도 : 관계의 수가 n개로 줄어든다.
- 장점 : 객체들은 오직 자신의 기능에만 집중한다. 누구에게 해당 데이터를 보낼 것인가에 대한 책임은 중재자가 진다. 
![](https://velog.velcdn.com/images/dawnww/post/4b153f22-46fe-4ca8-87c8-d74d7fda06fd/image.png)

![](https://velog.velcdn.com/images/dawnww/post/e7bd47cc-11d6-4a48-a33d-4d56d733653d/image.png)
위 예제의 화살표 모양을 보면 중재자가 다른 사람들로부터 옵션을 숨기고 역방향으로 소통할 수 있음을 알 수 있다.
예를 들어, `Husband` 인스턴스가 `Mediator`에게 무언가를 말하고 싶어하면, `Mediator`는 그것을 `Lawyer`에게 전달할 것이다. 그러나, 만약 `Lawyer`이 `Husband`에게 무언가를 말하고 싶어한다면, `Mediator`는 그것을 허용하지 않는다.


### Typescript 예제
참가자들이 직접 서로 메시지를 보내는 것이 아닌 채팅룸이라는 중재자를 통해 통신하는 예제이다.

#### 인터페이스 정의
```typescript
// 중재자 인터페이스
interface ChatMediator {
  sendMessage(message: string, user: User): void;
  addUser(user: User): void;
}

// 협력 객체(Colleague) 추상 클래스
abstract class User {
  protected mediator: ChatMediator;
  protected name: string;

  constructor(mediator: ChatMediator, name: string) {
    this.mediator = mediator;
    this.name = name;
  }

  abstract send(message: string): void;
  abstract receive(message: string): void;
}
```

#### 클래스 구현
```typescript
// 구체적인 중재자: 채팅방
class ChatRoom implements ChatMediator {
  private users: User[] = [];

  addUser(user: User): void {
    this.users.push(user;
    console.log(`[System] ${user['name']} 님이 입장하셨습니다.`);
  }

  sendMessage(message: string, sender: User): void {
    this.users.forEach((user) => {
      // 메시지를 보낸 본인을 제외하고 모두에게 전달
      if (user !== sender) {
        user.receive(message);
      }
    });
  }
}

// 구체적인 사용자
class ChatUser extends User {
  send(message: string): void {
    console.log(`${this.name} 전송: ${message}`);
    this.mediator.sendMessage(message, this);
  }

  receive(message: string): void {
    console.log(`${this.name} 수신: ${message}`);
  }
}
```

#### 실행 코드
```typescript
const mediator = new ChatRoom();

const sinji = new ChatUser(mediator, "신지");
const jeongwoo = new ChatUser(mediator, "정우");
const changwoo = new ChatUser(mediator, " 창우");

mediator.addUser(sinji);
mediator.addUser(jeongwoo);
mediator.addUser(changwoo);

sinji.send("안녕하세요! 오늘 디자인 패턴 공부하시나요?");
// 출력:
// 신지 전송: 안녕하세요! 오늘 디자인 패턴 공부하시나요?
// 정우 수신: 안녕하세요! 오늘 디자인 패턴 공부하시나요?
// 창우 수신: 안녕하세요! 오늘 디자인 패턴 공부하시나요?
```

위와 같이 개발할 경우 `ChatUser`는 다른 사용자가 누구인지, 몇 명인지 알 필요 없이 `mediator`에게 메시지를 던지기만 하면 된다.
👉 나중에 사용자 타입이 추가되거나 로직이 변경되어도 개별 사용자 코드를 수정할 필요가 없다.


### 미들웨어 패턴 예제
```typescript
type Next = () => Promise<void> | void;
// 미들웨어들이 공유하며 수정할 수 있는 상태 객체
type MiddlewareContext = { data: string };
// 컨텍스트와 다음 실행 권한(next)를 인자로 받는 미들웨어의 형태
type Middleware = (ctx: MiddlewareContext, next: Next) => Promise<void> | void;

class MiddlewareManager {
  // 등록된 미들웨어들을 순서대로 젖앟나느 큐
  private middlewares: Middleware[] = [];

  // 미들웨어 등록 -> 체인 형태 
  use(middleware: Middleware): void {
    this.middlewares.push(middleware);
  }

  // 실행 (재귀적 체이닝)
  async execute(ctx: MiddlewareContext): Promise<void> {
    const run = async (index: number): Promise<void> => {
      // 미들웨어 다 돌면 종료하기
      if (index === this.middlewares.length) return;

      const middleware = this.middlewares[index];
      
      // ** 현재 미들웨어를 실행하면서, 다음 미들웨어를 실행하는 함수를 next 인자로 주입
      await middleware(ctx, () => run(index + 1));
    };

    await run(0);
  }
}

// 데이터 처리 파이프라인
const manager = new MiddlewareManager();

// 1. 로깅 미들웨어
manager.use(async (ctx, next) => {
  console.log(`[Log] 처리 시작: ${ctx.data}`);
  await next();	// 다음 미들웨어 진행
  console.log(`[Log] 처리 완료`);
});

// 2. 변형 미들웨어 (데이터 가공)
manager.use(async (ctx, next) => {
  ctx.data = ctx.data.toUpperCase();
  console.log(`[Transform] 데이터를 대문자로 변환함`);
  await next();
});

// 3. 보안 미들웨어 (민감 단어 필터링)
manager.use(async (ctx, next) => {
  if (ctx.data.includes("BADWORD")) {
    console.log(`[Security] 부적절한 데이터 감지! 중단합니다.`);
    return; // next()를 호출하지 않아 이후 체인 중단
  }
  await next();
});

// 실행
const context = { data: "hello design patterns" };
manager.execute(context).then(() => {
  console.log(`최종 결과: ${context.data}`);
});
```
위 코드는 `next()` 함수를 전달하여 제어권을 다음 단계로 넘기는 것이 핵심이다. 단순히 함수를 순차 실행하는 것보다 강력하다.
미들웨어는 `next()`를 호출하기 전과 후에 로직을 배치할 수 있으며, 심지어 특정 조건에서 `next()`를 호출하지 않음으로써 실행 흐름을 차단할 수 있다.

> 실행 흐름
1. 시작: manager.execute({ data: "hello design patterns" }) 호출.
2. 1번 미들웨어 (Log): [Log] 처리 시작 출력. await next()를 만남.
3. 2번 미들웨어 (Transform): [Transform] 대문자로 변환 출력. ctx.data가 대문자가 됨. await next()를 만남.
4. 3번 미들웨어 (Security): "BADWORD"가 없으므로 통과. await next()를 만남.
5. 종료 조건: run(3)이 호출되나 index === length이므로 즉시 반환(return).
6. 역순 복귀 (Backtracking): * 3번 미들웨어 종료.
  - 2번 미들웨어 종료.
  - 1번 미들웨어로 돌아와서 await next() 다음 줄인 [Log] 처리 완료 출력.
7. 최종 완료: manager.execute().then()이 실행되며 최종 결과: HELLO DESIGN PATTERNS 출력.


### 장단점
#### 장점
- **결합도 감소** : 컴포넌트들이 서로를 몰라도 된다.
- **재사용성 향상** : 독립적인 모듈로 개발하기 쉬워진다.
- **단일 책임 원칙** : 로직을 별도로 관리한다.

#### 단점
- **복잡성 증가** : 중간 단계가 추가되므로 코드 흐름을 추적하기 어려울 수 있다.
- **중재자 비대화** : God Object가 되어 중재자 혼자 모든 로직을 떠안을 수 있다.
- **성능 오버헤드** : 간접적인 호출로 인해 미세한 성능 저하가 발생할 수 있다.


## Command Pattern
> 요청을 객체로 캡슐화하여, 실행자와 수행자를 분리하는 패턴
👉 하고 싶은 일을 하나의 객체로 만들어 필요할 때 실행하는 방식

프로그래밍에서 일반적으로 함수는 '동사'의 역할을 한다. `saveFile()`, `deleteUser()` 처럼 특정 동작을 수행한다. 그러나 이때 소프트웨어가 복잡해질 경우, 다양한 요구사항이 발생한다. (지연 실행, 취소, 한 번에 실행 등)

단순히 함수를 직접 호출하는 방식으로는 호출자와 수신자가 강하게 결합되어 있기 때문에 위 요구사항들을 깔끔하게 해결하기 어렵다.

👉 **요청 그 자체를 객체로 캡슐화**하여 해결. 즉, 동사였던 동작을 명사인 객체로 바꿈으로써 동작은 변수에 저장하고, 전달하고, 조작할 수 있는 데이터처럼 취급하게 된다. 


### Command 패턴의 4가지 핵심 요소
- **Command (커맨드 인터페이스)**: 모든 구체적인 커맨드 클래스가 구현해야 하는 인터페이스. 보통 `execute()` 메서드를 정의하며, 필요에 따라 `undo()`를 포함.
- **Concrete Command (구체적인 커맨드)**: Command 인터페이스를 구현한다. 이 객체는 수신자(Receiver)에 대한 참조를 가지고 있으며, `execute()`가 호출될 때 수신자의 메서드를 실행한다.
- **Receiver (수신자)**: 실제 비즈니스 로직을 수행하는 객체. 커맨드가 '무엇을 할지' 결정한다면, 수신자는 '어떻게 할지'를 알고 있다.
- **Invoker (호출자)**: 커맨드 객체를 보유하고 있으며, 언제 실행할지 결정한다. 호출자는 커맨드가 구체적으로 어떤 일을 하는지, 수신자가 누구인지 알 필요가 없다. 오직 `execute()`만 호출한다.


### 예제

#### Receiver - 작업자
```typescript
class Light {
  on() {
    console.log('불을 켭니다');
  }
  off() {
    console.log('불을 끕니다');
  }
}
```
실제로 불을 끄고 크는 객체

#### Command 인터페이스
```typescript
interface Command {
  execute(): void;
}
```
`excute()`라는 공통 함수 보장

#### ConcreteCommand - 실제 수행 명령
```typescript
class LightCommand implements Command {
  constructor(private light: Light) {}
  
  execute() {
    this.light.on();
  }
}

class LightOffCommand implements Command {
  constructor(private light: Light) {}
  
  execute() {
    this.light.off();
  }
}
```
`excute()`라는 공통 함수를 보장하고, 내부에 `Receiver`를 가진다. 

#### Invoker
```typescript
class RemoteControl {
  constructor(private command: Command) {}
  
  press() {
    this.command.execute();
  }
}
```
무엇을 실행하는지 모르고 `command.excute()`를 호출한다. 

#### 사용
```typescript
const light = new Light();

const onCommand = new LightOnCommand(light);
const offCommand = new LightOffCommand(light);

const remote = new RemoteControl(onCommand);
remote.press(); // 불을 켭니다

const remote2 = new RemoteControl(offCommand);
remote2.press(); // 불을 끕니다
```

### 텍스트 에디터 예제

#### Receiver
```typescript
class TextEditor {
  private content: string = "";

  public type(text: string): void {
    this.content += text;
  }

  public delete(length: number): string {
    const deletedText = this.content.substring(this.content.length - length);
    this.content = this.content.substring(0, this.content.length - length);
    return deletedText;
  }

  public getContent(): string {
    return this.content;
  }

  public setContent(text: string): void {
    this.content = text;
  }
}
```
- 실제 에디터의 상태와 로직 담당

#### Command Interface
```typescript
interface Command {
  execute(): void;
  undo(): void;
}

//  텍스트 입력 
class TypeCommand implements Command {
  private prevText: string = "";

  constructor(
    private editor: TextEditor,
    private textToType: string
  ) {}

  execute(): void {
    this.editor.type(this.textToType);
    console.log(`Typed: "${this.textToType}"`);
  }

  undo(): void {
    this.editor.delete(this.textToType.length);
    console.log(`Undo Typed: "${this.textToType}"`);
  }
}

// 텍스트 삭제
class DeleteCommand implements Command {
  private deletedText: string = "";

  constructor(
    private editor: TextEditor,
    private length: number
  ) {}

  execute(): void {
    this.deletedText = this.editor.delete(this.length);
    console.log(`Deleted: "${this.deletedText}"`);
  }

  undo(): void {
    this.editor.type(this.deletedText);
    console.log(`Undo Deleted: "${this.deletedText}"`);
  }
}
```

#### Invoker
```typescript
class CommandManager {
  private history: Command[] = [];
  private redoStack: Command[] = [];

  executeCommand(command: Command): void {
    command.execute();
    this.history.push(command);
    this.redoStack = []; // 새로운 동작 시 redo 스택 초기화
  }

  undo(): void {
    const command = this.history.pop();
    if (command) {
      command.undo();
      this.redoStack.push(command);
    } else {
      console.warn("No more commands to undo.");
    }
  }

  redo(): void {
    const command = this.redoStack.pop();
    if (command) {
      command.execute();
      this.history.push(command);
    } else {
      console.warn("No more commands to redo.");
    }
  }
}
```
- 커멘드 실행 및 히스토리 관리. Unde/Redo 스택을 유지한다.

#### 사용
```typescript
// 1. 초기화
const myEditor = new TextEditor();
const manager = new CommandManager();

// 2. 사용자 동작 시뮬레이션
console.log("--- Initial State ---");
manager.executeCommand(new TypeCommand(myEditor, "Hello "));
manager.executeCommand(new TypeCommand(myEditor, "World!"));
console.log("Current Content:", myEditor.getContent()); // "Hello World!"

// 3. 삭제 동작
manager.executeCommand(new DeleteCommand(myEditor, 6));
console.log("After Delete:", myEditor.getContent()); // "Hello "

// 4. 취소 (Undo) 실행
manager.undo();
console.log("After Undo:", myEditor.getContent()); // "Hello World!"

// 5. 다시 실행 (Redo)
manager.redo();
console.log("After Redo:", myEditor.getContent()); // "Hello "
```

### 장단점
#### 장점 
- **결합도 해제** : 버튼을 실행한다고 할 때, 자신이 실행하는 것이 무엇인지 알 필요가 없다.
- **Undo/Redo** : 상태를 통째로 저장하는 스냅샷 방식보다 메모리 효율적이다. 
- **작업 예약 및 큐잉** : 커맨드 객체들을 큐에 쌓아두고, 브라우저의 메인 스레드가 한가할 때나 네트워크가 복구되었을 때 하나씩 호출할 수 있다. 혹은 낙관적 업데이트를 구현할 때, 실패 시 `undo()`를 호출하여 이전 상태로 되돌리는 로직을 깔끔하게 유지할 수 있다.

#### 단점
- **코드 복잡도 및 보일러플레이트 증가** : 단순히 함수 하나 호출하면 끝날 일을 위해 `interface`, `concrete class`, `invoker`를 정의해야 한다.
- **추적의 어려움** : 동작이 객체 뒤로 숨어버리기 때문에, 코드 에디터에서 실제 로직이 아닌 추상 클래스로 이동할 수 있다.
- **메모리 관리 이슈** : Undo 기능을 위해 스택에 커맨드 객체를 쌓아두면 메모리 누수의 원인이 된다. 
