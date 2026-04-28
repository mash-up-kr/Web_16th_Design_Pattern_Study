# [Mediator/Middleware Pattern](https://patterns-dev-kr.github.io/design-patterns/mediator-middleware-pattern/)

## [What] Mediator/Middleware Pattern이란?

> "객체들이 서로 직접 통신하지 않고, 중재자를 통해 간접적으로 소통하는 패턴" _(claude)_

> 중재자 패턴을 사용하면 컴포넌트들이 중재자라는 중앙의 한 지점을 통해 서로 소통할 수 있다. _(patterns.dev)_

이 글에서는 **Mediator**와 **Middleware** 두 가지를 함께 다룬다. 핵심 원리는 같지만 적용 맥락이 다르다.

**Mediator**는 GoF 행위 패턴 중 하나이다. 여러 객체가 서로 직접 통신하는 대신, **중재자 하나를 통해 간접적으로 소통**하게 만든다. 공항 관제탑이 좋은 비유이다. 비행기들이 서로 직접 통신하면 혼란이 생기지만, 관제탑이 모든 통신을 중재하면 안전하게 관리할 수 있다.

Mediator가 없으면 N개 객체 간 연결 수는 **N\*(N-1)/2**개이다. Mediator를 두면 **N**개로 줄어든다.

| ![no mediator.png](images/no%20mediator.png) | ![use mediator.png](images/use%20mediator.png) |
|:--------------------------------------------:|:----------------------------------------------:|
|                Mediator 안썻을 때                |                Mediator 사용했을 때                 |

**Middleware**는 같은 원리를 **요청-응답 파이프라인**에 적용한 것이다. Express.js의 미들웨어가 대표적이다.

## [Patterns.dev] 예시로 알아보기

### 채팅방 예시 (Mediator)

사용자들은 서로 직접 메시지를 보내지 않고, **ChatRoom(Mediator)**을 통해 메시지를 주고받는다.

```js
class ChatRoom {
  logMessage(user, message) {
    const time = new Date()
    const sender = user.getName()
    console.log(`${time} [${sender}]: ${message}`)
  }
}

class User {
  constructor(name, chatroom) {
    this.name = name
    this.chatroom = chatroom
  }

  getName() { return this.name }

  send(message) {
    this.chatroom.logMessage(this, message)
  }
}
```

핵심은 **User 객체들이 서로를 직접 참조하지 않는다**는 점이다. User는 ChatRoom만 알면 되고, 다른 User의 존재를 알 필요가 없다.

### Express.js 미들웨어 (Middleware)

```js
const app = require("express")()

// 미들웨어 1: 커스텀 헤더 추가
app.use((req, res, next) => {
  req.headers["test-header"] = 1234
  next()
})

// 미들웨어 2: 헤더 확인
app.use((req, res, next) => {
  console.log(`test header: ${!!req.headers["test-header"]}`)
  next()
})
```

`next()`를 호출하면 다음 미들웨어로 넘어간다. 호출하지 않으면 체인이 끊어진다. 각 미들웨어는 **요청을 가공하고 다음으로 전달하는 체인**을 형성한다.

## [Why] Mediator/Middleware Pattern은 왜 사용하는가?

Q. 언제 사용하면 좋을까?
- 핵심은 **여러 객체가 서로 복잡하게 통신해야 하는데, 그 관계를 단순화하고 싶을 때 (Mediator)** / **요청 처리에 여러 단계의 가공이 필요할 때 (Middleware)**이다.

Q. 왜 Mediator를 쓰는가?

1. **통신 복잡도 감소** — N\*(N-1)/2 → N
2. **느슨한 결합** — 객체들이 서로를 알 필요 없음
3. **통신 로직의 중앙 집중** — 한 곳에서 관리, 디버깅 용이

Q. 왜 Middleware를 쓰는가?

1. **관심사의 분리** — 인증, 로깅, 파싱 등을 독립적인 미들웨어로 분리
2. **재사용성** — 같은 미들웨어를 여러 라우트에 적용 가능
3. **순서 제어** — 실행 순서를 `app.use()` 호출 순서로 명시적으로 관리

Q. 단점은 무엇인가?

- **Mediator**: God Object 위험 (모든 로직이 Mediator에 집중), 단일 장애 지점
- **Middleware**: `next()` 호출 누락 시 체인이 끊어짐, 미들웨어 순서에 의존하는 버그 디버깅이 까다로움

## [Question] Mediator vs Observer, 뭐가 다른 거야?

둘 다 "객체 간 통신"을 다루지만, 방향과 목적이 다르다.

| 기준 | Mediator | Observer |
|:--|:--|:--|
| 통신 방향 | 양방향 (객체 ↔ 중재자) | 단방향 (Observable → Observer) |
| 목적 | 복잡한 다대다 관계를 단순화 | 상태 변화를 여러 곳에 전파 |
| 결합 방식 | 모든 객체가 Mediator를 참조 | Observer가 Observable을 구독 |

실제로 Mediator 내부에서 Observer 패턴을 사용하는 구현이 흔하다. Mediator가 Publisher 역할을 하고, 각 객체가 Subscriber 역할을 하는 방식이다. 즉, Observer는 Mediator를 구현하는 하나의 수단이 될 수 있다.
