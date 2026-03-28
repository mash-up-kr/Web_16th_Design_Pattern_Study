# 믹스인 패턴

- 객체나 클래스에 재사용 가능한 기능을 조합해서 추가하는 패턴

> 상속을 깊게 만들지 않고, 필요한 기능만 조합해서 재사용 할 수 없을까?

⇒ 객체나 클래스 여러 개가 비슷한 기능을 공유해야 하는 경우
어떤건 log(), 어떤건 draggable(), 어떤거는 hoverable() …..

이런 식으로 가면 조합히 굉장히 많아짐. 상속은 is-a 관계지만 수평정인 기능 조합 문제에는 불편

⇒ 공통 기능을 독립적으로 만들어 두고, 필요한 대상에 가져다 섞어 넣자!

### 상속과의 차이

상속 : 부모의 속성과 메서드를 자식이 물려 받음, 계층 구조, 관계가 강함, A is a B
믹스인 : 어떤 기능 조각을 여러 대상에 조합, 계층보다 기능 재사용 집중, 느슨한 관계, A can do B

### 프론트엔드에서는 어떤 느낌으로 쓰이나

- 순수 디자인 패턴으로서의 믹스인 보다, 기능 조합 방식으로 자주 만남

⇒ UI 기능 조합 ( draggable, resizable, selectable ) , 공통 유틸 기능 ( 이벤트 발행, 로깅, 발리데이션 ), 프레임 워크 재사용 ( Vue2 )

### 트레이드 오프

- 장점

상속보다 유연하다! ( 필요한 만큼 기능 조합 )
코드 재사용성 높다
관심사를 기능 단위로 분리하기 쉽다

하지만! 믹스인은 단점도 뚜렷함. 그래서 패턴을 그대로 따르는것은 좋지 않음

- 단점

출처가 불분명해질 수 있다 ⇒ 조합 주체에서 어떤 메서드가 어디서 왔는지 추적이 어려움

이름 충돌 가능성 ⇒ 서로 다른 믹스인이 같은 메서드 이름 쓰면 덮어 쓰기 일어남

암묵적 의존성 ⇒ 믹스인이 내부적으로 특정 프로퍼티가 있다고 가정하게 되면 유지보수가 어려워짐

구조가 숨겨짐 ⇒ 디버깅이 어려움 ..

이러한 단점들로 인해서 Vue2 에서 공식 기능으로 제공했지만 vue3 에서는 권장되지 않음
 + 리액트 개발팀도 믹스인보다는 custom hook이나 고차 컴포넌트 사용을 권장

### 그러면 어떻게 써야할까?

⇒ 믹스인 패턴을 그대로 따르지 않더라도 그 철학을 이용한 방식 활용
공통 기능을 여러 객체/컴포넌트에서 재사용하고 싶을 때, 상속 대신 기능 단위로 조합하는 방식

React custom hooks ⇒ 하나의 컴포넌트에서 여러가지 훅에서 처리하는 기능 단위들을 결합함

### 믹스인 패턴 활용 - component-emitter ( Socket.IO )

[socket.io](http://socket.io) 는 이벤트를 발생시키고 구독하는 기능이 여러개 있음 Socket, server, client, namespace ..

```tsx
// ❌ 이걸 매번 하면?
class Socket {
  on(event, fn) { /* ... */ }
  emit(event) { /* ... */ }
  off(event, fn) { /* ... */ }
}

class Namespace {
  on(event, fn) { /* 똑같은 코드... */ }
  emit(event) { /* 똑같은 코드... */ }
  off(event, fn) { /* 똑같은 코드... */ }
}
// → 완전 같은 코드를 계속 복붙해야 함
```

매번 하면 중복코드 막 생김

```tsx
function Emitter(obj) {
  // ★ obj가 넘어오면 → Mixin 모드
  if (obj) {
    for (const key in Emitter.prototype) {
      obj[key] = Emitter.prototype[key];
      // obj.on = Emitter.prototype.on
      // obj.emit = Emitter.prototype.emit
      // obj.off = Emitter.prototype.off
      // ... 하나씩 복사
    }
    return obj;
  }
}

// 이벤트 구독
Emitter.prototype.on = function(event, fn) {
  // this._callbacks에 이벤트별로 함수를 배열로 저장
  this._callbacks = this._callbacks || {};
  (this._callbacks[event] = this._callbacks[event] || []).push(fn);
  return this;
};

// 이벤트 발생
Emitter.prototype.emit = function(event, ...args) {
  this._callbacks = this._callbacks || {};
  const callbacks = this._callbacks[event];
  if (callbacks) {
    // 등록된 함수를 하나씩 실행
    callbacks.forEach(cb => cb.apply(this, args));
  }
  return this;
};

// 이벤트 구독 해제
Emitter.prototype.off = function(event, fn) {
  this._callbacks = this._callbacks || {};
  const callbacks = this._callbacks[event];
  if (callbacks) {
    this._callbacks[event] = callbacks.filter(cb => cb !== fn);
  }
  return this;
};
```
그래서 component-emitter 라이브러리를 사용해서
이벤트 기능을 Emitter 라는 덩어리로 만들어 놓고, 필요한 객체에 섞어 넣음 ⇒ 믹스인

```tsx
// 방법 1: 일반 객체에 섞기
const user = { name: 'Kim' };
Emitter(user);
//  이 시점에서 user는:
//  { name: 'Kim', on: fn, emit: fn, off: fn, once: fn }

user.on('login', () => console.log('로그인됨'));
user.emit('login');  // "로그인됨"

// 방법 2: 클래스 프로토타입에 섞기
function Chat() {}
Emitter(Chat.prototype);
//  Chat.prototype에 on, emit, off가 복사됨
//  → 모든 Chat 인스턴스가 이벤트 기능을 가짐

const chat = new Chat();
chat.on('message', (msg) => console.log(msg));
chat.emit('message', '안녕');  // "안녕"

// 방법 3: 직접 인스턴스로 쓰기
const emitter = new Emitter();
emitter.on('ping', () => console.log('pong'));
emitter.emit('ping');  // "pong"
```

---
1번, 2번은 믹스인 예시 3번은 Emitter 인스턴스를 직접 사용하는 방식

### `Emitter(obj)` 호출 전후 비교
```
호출 전: user = { name: 'Kim' }

Emitter(user) 실행 — for...in으로 프로토타입 복사

호출 후: user = {
  name: 'Kim',        ← 원래 있던 것
  on: [Function],      ← Emitter에서 복사됨
  emit: [Function],    ← Emitter에서 복사됨
  off: [Function],     ← Emitter에서 복사됨
  once: [Function],    ← Emitter에서 복사됨
}
```
상속이 아니라, 기존 객체에 이벤트 기능을 직접 주입하는 방식! 

```tsx
// 서버
io.on('connection', (socket) => {
  socket.on('chat', (msg) => { ... });   // ← Emitter에서 온 .on()
  socket.emit('welcome', '안녕');          // ← Emitter에서 온 .emit()
});

// 클라이언트
const socket = io('http://localhost:3000');
socket.on('welcome', (msg) => { ... });   // ← 같은 Emitter
socket.emit('chat', '반가워');              // ← 같은 Emitter
```
