# 옵저버 패턴

어떤 객체의 상태가 바뀌었을 때, 그 객체를 구독하고 있던 다른 객체들에게 자동으로 알림을 보내는 패턴

⇒ 한 객체의 변화가, 여러 객체에게 전파되어야 할 때 사용하는 패턴

### 필요

어떤 시스템에서 하나의 데이터 변화가 여러 곳에 영향을 주는 경우
ex) 사용자 로그인 상태가 바뀌었을 때 헤더 UI 변경, 마이페이지 권한 변경, 알림 시스템 반응 등 ..

```tsx
login()
updateHeader()
updateMyPagePermission()
updateNotificationSystem()
sendAnalytics()
```

상태를 바꾸는 객체는, 누가 반응해야하는지 전부 알고 있어야함 ⇒ 호출하는 대상이 너무 많아짐. 책임을 너무 많이 이해함

옵저버 패턴은 로그인 상태를 관리하는 객체가 상태 변경을 알리면, 그걸 구독한 애들만 알아서 반응하도록 구조 변경

> 변화를 발생시키는 쪽과, 변화하는 쪽을 느슨하게 분리하는 패턴 ( 책임 분리 )
>

### 구조

Subject (= Observable )

- Observer를 등록 할 수 있음 ( register, subscribe )
- Observer를 제거 할 수 있음
- 상태가 바뀌면 Observer들에게 알림

Observer

- 알림을 받는 인터페이스를 가짐

```tsx
class Subject {
  constructor() {
    this.observers = [];
  }

  subscribe(observer) {
    this.observers.push(observer);
  }

  unsubscribe(observer) {
    this.observers = this.observers.filter(obs => obs !== observer);
  }

  notify(data) {
    this.observers.forEach(observer => observer.update(data));
  }
}

class Observer {
  constructor(name) {
    this.name = name;
  }

  update(data) {
    console.log(`${this.name}가 알림을 받음:`, data);
  }
}

const youtubeChannel = new Subject();

const user1 = new Observer('창우');
const user2 = new Observer('민수');

youtubeChannel.subscribe(user1);
youtubeChannel.subscribe(user2);

youtubeChannel.notify('새 영상 업로드!');
```

```tsx
창우가 알림을 받음: 새 영상 업로드!
민수가 알림을 받음: 새 영상 업로드!
```

Subject는 Observer가 누군지 세세하게 모름
단순 subscribe, unsubscribe, notify 이 세 가지 규칙만 이해함

Observer도 Subject 내부 구현을 모름, 알림을 받으면 단순 update를 실행 ⇒ 서로 강하게 엮이지 않음

### 동작 원리

1. Subject가 구독자 목록을 관리
2. 상태가 바뀌는 시점에 notify 실행
3. 등록된 Observer들의 update를 순서대로 호출
4. 각 Observer는 자기 책임에 해당하는 반응만 수행

```tsx
class LoginManager {
  constructor() {
    this.observers = [];
  }

  subscribe(observer) {
    this.observers.push(observer);
  }

  notifyLogin(user) {
    this.observers.forEach(observer => observer.update(user));
  }

  login(user) {
    console.log('로그인 성공');
    this.notifyLogin(user);
  }
}
```

옵저버 패턴은 단순한 알림 보내기 기술인가? ⇒ ㄴㄴ 철학이 있다!

1. 변화의 전파를 표준화 ⇒ 상태 변화가 발생했을 때, 통일된 방식으로 전파
2. 결합도를 낮춤 ⇒ 변화 발생자 <> 반응자 분리
3. 확장이 쉬움 ⇒ 반응 로직 추가 시, Subject를 크게 수정하지 않아도됨

### 비교

```tsx
class LoginManager {
  login() {
    header.update();
    sidebar.update();
    analytics.track();
    popup.show();
  }
}
```

- LoginManager가 알아야 하는 것 : 헤더, 사이드바, 분석 툴, 팝업 ..
- 확장에 닫혀 있고 결합도가 높음

옵저버 패턴 쪽은 위 동작 원리 예시처럼

- LoginManager는 로그인 변화만 알림
- 누가 반응할지는 구독자들이 각자 책임짐
- 반응 로직이 늘어나도 LoginManager는 크게 바뀌지 않음

### EventEmitter / Pub-Sub 와의 차이

공부하다보면 비슷한 개념들이 계속 나옴

**Observer**

- 객체가 자기 구독자 목록을 직접 가짐
- Subject와 Observer가 비교적 직접 연결됨

```tsx
subject.subscribe(observer);
subject.notify(data);
```

**EventEmitter**

- 이벤트 이름 중심으로 동작
- `on`, `emit` 구조로 특정 이벤트를 발행

```tsx
emitter.on('login', handler);
emitter.emit('login', user);
```

EventEmitter도 넓게 보면 옵저버 구조와 닿아있음. 다만 객체 자체를 구독한다기 보다, 이벤트 이름을 구독하는 느낌이 더 강함

**Pub/Sub**

- 발행자와 구독자 사이에 브로커가 존재
- 발행자와 구독자가 서로를 몰라도 됨

```tsx
broker.publish('login', user);
broker.subscribe('login', handler);
```

즉

- Observer : 직접 연결된 구독 구조
- EventEmitter : 이벤트 이름 중심 구독 구조
- Pub/Sub : 중간 브로커를 둔 더 느슨한 구조

⇒ 비슷해보여도 결합되는 방식이 다름

### RxJS에서의 옵저버 패턴

프론트엔드에서는 옵저버 패턴을 좀 더 현대적으로 보면 RxJS가 있음

- Observable / Subject : 변화를 발생시키는 쪽
- Observer : 그 변화를 구독하는 쪽
- subscribe() : 구독 등록
- unsubscribe() : 구독 해지

```tsx
import { Subject } from 'rxjs';

const login$ = new Subject();

const headerSubscription = login$.subscribe(user => {
  console.log('헤더 업데이트:', user.name);
});

const analyticsSubscription = login$.subscribe(user => {
  console.log('로그인 분석 전송:', user.name);
});

login$.next({ name: '창우' });

headerSubscription.unsubscribe();
analyticsSubscription.unsubscribe();
```

여기서 `login$` 는 로그인 상태 변화 스트림

- `next()` 가 변화 발생
- `subscribe()` 가 반응 등록

즉 옵저버 패턴 구조가 거의 그대로 들어있음

### RxJS가 단순 옵저버보다 더 강한 점

옵저버 패턴은 보통 “변화를 알린다” 에 집중
RxJS는 여기서 한 단계 더 나아가서, 그 변화의 흐름을 가공할 수 있음

```tsx
import { Subject, filter, map } from 'rxjs';

const login$ = new Subject();

login$
  .pipe(
    filter(user => user.isAdmin),
    map(user => user.name),
  )
  .subscribe(name => {
    console.log('관리자 로그인:', name);
  });

login$.next({ name: '창우', isAdmin: true });
login$.next({ name: '민수', isAdmin: false });
```

이건 그냥 알림을 받는 수준이 아니라

- 특정 조건만 걸러내고
- 필요한 값만 뽑고
- 그 결과에만 반응

하게 만드는 것

⇒ RxJS는 옵저버 패턴을 기반으로, 이벤트 흐름 처리까지 확장한 형태라고 볼 수 있음

### 유의

옵저버 패턴을 사용할 때, 자주 놓치는 것 ⇒ 구독 해지!

- 메모리 누수.. 불필요 콜백.. 죽은 UI 반응
- 컴포넌트가 언마운트 된 시점에서, 리스너가 남아 있으면 버그가 생길 수 있다

```tsx
const unsubscribe = subject.subscribe(observer);

// 나중에
unsubscribe();
```

### 실무에서 더 놓치기 쉬운 것

**1. 순서 의존성**

Observer가 많아지면 누가 먼저 실행되는지에 따라 결과가 달라질 수 있음

- 토큰 저장 먼저?
- UI 갱신 먼저?
- 분석 이벤트 먼저?

반응 순서가 중요해지는 순간부터 구조가 꽤 예민해짐

**2. 연쇄 호출**

어떤 Observer가 반응하면서 또 다른 상태를 바꾸고, 그게 다시 notify를 발생시키면..

```tsx
observer.update(data)
→ 다른 상태 변경
→ 또 notify
→ 또 observer.update(...)
```

이런 식으로 복잡한 연쇄 호출이 발생

⇒ 디버깅 어려움, 사이드 이펙트 증가

**3. 이전 상태 / 현재 상태를 같이 줄 것인가**

단순히 “바뀌었다” 만 알리면 부족한 경우도 많음

```tsx
observer.update({
  prevUser,
  nextUser,
});
```

이렇게 이전값과 다음값을 같이 전달하면 반응 로직이 훨씬 명확해질 수 있음

### 트레이드 오프

장점

- 느슨한 결합, Subject는 Observer 구현을 몰라도 된다
- 확장성, 새로운 Observer 쉽게 추가 가능
- 변화 전파 자동화, 일관된 상태 변화 반응 가능
- 관심사 분리, 상태를 바꾸는 책임 <> 반응하는 책임

단점

- 디버깅.. 이벤트가 어디서 발생했고, 누가 반응했는가에 대한 추적
  - 옵저버가 많아지면 특히 순서에 따른 호출 문제, 구독 관리 파악이 어려워짐
- 사이드 이펙트
  - 옵저버가 상태를 바꾼 호출이 또 notify를 발생시키는 경우.. 복잡한 연쇄 호출 발생
- 메모리 관리 이슈
- 순서 의존성

⇒ 변화 전파가 여러 곳으로 퍼지는 구조일 수록 옵저버 패턴의 가치가 커진다!

### 공부 중 의문

그냥 콜백 배열 만들어서 순회 호출하면 되는거 아닌가?

⇒ 겉으로 보면 맞음. 결국 등록하고 순회해서 호출하는 구조

근데 중요한건 구현 디테일이 아니라 설계 의도

- 상태 변화와 반응 로직이 분리되어 있는가
- 반응자를 쉽게 교체 / 추가할 수 있는가
- 변화 전파가 일관된 규칙으로 관리되는가

이 의도를 만족하면 단순한 콜백 배열 기반 구현이어도 옵저버 패턴이라고 볼 수 있음
