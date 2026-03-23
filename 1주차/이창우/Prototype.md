# 3. 프로토타입 패턴

## 3-1. 정의

기존 객체를 복제해서 새 객체를 만드는 패턴. new + 생성자로 처음부터 만드는 게 아니라, 이미 존재하는 객체를 원형 삼아 복사본을 생성

* 복제 기반 생성 — 기존 인스턴스를 복사해서 새 인스턴스를 만듦
* 원본 보존 — 복제본을 수정해도 원본에 영향 없음

클래스를 정의하고 new로 찍어내는 것이 아니라, 살아있는 객체를 복제해서 변형하는 것이 본질

> JS 프로토타입(언어 메커니즘)과 프로토타입 패턴(설계 전략)은 다른 개념. JS 프로토타입은 객체를 체인으로 연결하는 언어 내부 동작이고, 프로토타입 패턴은 기존 객체를 복제해서 새 객체를 만드는 생성 전략. JS 프로토타입이 프로토타입 패턴을 구현하는 도구 중 하나로 쓰일 수는 있지만, 둘은 별개

## 3-2. 필요성

객체를 처음부터 만드는 비용이 크거나, 동일한 구조의 객체를 반복 생성해야 할 때 유용

ex) 복잡한 설정 객체를 기반으로 여러 변형을 만들어야 하는 경우, DB 쿼리 조건을 공유하면서 분기해야 하는 경우, 상태를 기록해두고 되돌리기를 구현해야 하는 경우

매번 처음부터 구성하면 중복 코드가 생기고, 원본을 직접 수정하면 부작용이 생김 ⇒ 복제를 통해 원본은 보존하면서 변형을 만들어냄

## 3-3. 동작원리

1. 원형(prototype) 객체가 존재함
2. 새 객체가 필요할 때, 원형을 복제함
3. 복제본을 목적에 맞게 수정함
4. 원본은 그대로 남아서 다음 복제에도 재사용 가능

```js
const prototype = {
  type: 'notification',
  priority: 'normal',
  retryCount: 3,
  send() {
    console.log(`[${this.priority}] ${this.message}`);
  }
};

// 원형을 복제해서 변형
const urgent = Object.create(prototype);
urgent.priority = 'urgent';
urgent.message = '서버 다운';

const info = Object.create(prototype);
info.message = '배포 완료';

urgent.send(); // [urgent] 서버 다운
info.send();   // [normal] 배포 완료
```

Object.create(prototype)이 핵심. new로 클래스를 인스턴스화하는 게 아니라, 기존 객체를 원형 삼아 새 객체를 생성. 복제본에서 수정하지 않은 속성(priority, send)은 프로토타입 체인을 통해 원본에서 가져옴

## 3-4. 프로토타입 패턴 실제 활용 ( 스냅샷을 활용한 undo + redo )

Undo/Redo ⇒ 현재 상태를 복제해서 저장해둬야 이전으로 돌아갈 수 있음. 직접 수정(mutation) 하게 되면, 이전 상태가 사라지기 때문에 되돌릴 수 없음. 변경마다 현재 상태를 복제해서 히스토리에 쌓아야하기에 프로토타입 패턴이 필요

```
history: [ state0, state1, state2, state3 ]
                                      ↑
                                   현재 위치 (index: 3)

Undo → index를 2로 이동 → state2를 현재로 사용
Redo → index를 3으로 이동 → state3을 현재로 사용
```

히스토리 배열에 상태의 복제본(스냅샷)이 쌓이고, index를 앞뒤로 움직이는 것

structuredClone은 브라우저, Node.js에 내장된 전역함수 ( ≒ JSON.parse(JSON.stringify())의 상위 호환 )

```js
class UndoManager {
  constructor(initialState) {
    this.history = [structuredClone(initialState)];  // 최초 상태 복제해서 저장
    this.index = 0;
  }

  // 상태가 바뀔 때마다 호출
  save(state) {
    // 현재 위치 이후의 히스토리를 버림 (undo 후 새 작업하면 redo 히스토리 삭제)
    this.history = this.history.slice(0, this.index + 1);

    // 현재 상태를 복제해서 히스토리에 추가
    this.history.push(structuredClone(state));
    this.index++;
  }

  undo() {
    if (this.index <= 0) return null;  // 더 이상 되돌릴 게 없음
    this.index--;
    return structuredClone(this.history[this.index]);  // 복제본을 반환
  }

  redo() {
    if (this.index >= this.history.length - 1) return null;  // 더 이상 앞으로 갈 게 없음
    this.index++;
    return structuredClone(this.history[this.index]);
  }
}
```

복제 없이 참조만 저장하면 원본이 바뀔 때 히스토리도 같이 바뀜. 반드시 복제본을 저장해야 이전 상태가 보존됨

이 방식은 상태가 가벼운 앱(폼 상태, 간단한 드로잉, 설정 변경 등)에서 유효. 상태가 크고 복잡한 전문 에디터(ProseMirror, Slate.js 등)는 스냅샷 복제 대신 커맨드 패턴(변경 작업과 역연산을 저장)을 사용

## 3-5. 얕은 복사 vs 깊은 복사

프로토타입 패턴을 적용할때의 문제 ⇒ 어디까지 복제할까?

```js
const config = {
  server: { host: 'localhost', port: 3000 },
  db: { url: 'postgres://localhost/mydb' }
};
```

**얕은 복사** — 1depth만 복제, 내부 객체는 참조 공유

```js
const copy = { ...config };
copy.server.port = 8080;

console.log(config.server.port); // 8080 — 원본도 바뀌어버림
```

copy.server와 config.server가 같은 객체를 가리키고 있어서, 복제본을 수정했는데 원본이 오염됨. 프로토타입 패턴의 전제("원본 보존")가 깨지는 순간

**깊은 복사** — 내부 객체까지 전부 새로 복제

```js
const copy = structuredClone(config);
copy.server.port = 8080;

console.log(config.server.port); // 3000 — 원본 안전
```

안전하지만, 객체가 크고 깊으면 복제 비용이 큼. 바뀌지 않는 부분까지 전부 복제하는 것은 낭비

**구조적 공유** — 수정된 부분만 복제, 나머지는 원본 참조 유지

```js
const copy = {
  ...config,
  server: { ...config.server, port: 8080 }  // server만 새로 복제
};

console.log(config.server.port);  // 3000 — 원본 안전
console.log(copy.db === config.db); // true — db는 복제 안 함, 같은 참조
```

수정이 필요한 server만 새로 만들고, 수정하지 않는 db는 원본 참조를 그대로 사용. 이게 React/Redux 상태 관리에서 쓰이는 불변 업데이트 패턴의 원리

## 3-6. 트레이드오프

**얻는 것**
* 원본 보존 — 복제본을 수정해도 원본에 영향 없음. undo/redo, 상태 비교, 시간여행 디버깅의 전제 조건
* 생성 비용 절감 — 복잡한 초기화를 반복하지 않고 기존 객체를 복제해서 재사용
* 유연한 분기 — 하나의 원형에서 여러 변형을 만들어낼 수 있음

**잃는 것**
* 얕은 복사 vs 깊은 복사 판단 필요 — 어디까지 복제할지 설계 시점에 결정해야 함. 잘못 선택하면 참조 공유로 인한 의도치 않은 원본 오염
* 복제 비용 — 객체가 크고 깊으면 깊은 복사 자체의 성능 비용 존재
* 순환 참조 문제 — 객체가 자기 자신을 참조하는 구조에서는 단순 복제가 무한 루프에 빠질 수 있음

⇒ JS에서는 언어 자체가 프로토타입 기반이라 패턴이 이미 내장되어 있음. Object.create(), spread 연산자, structuredClone(), Immer produce() 등이 전부 프로토타입 패턴의 구현체. 별도로 패턴을 도입하는 게 아니라, JS를 쓰는 것 자체가 프로토타입 패턴을 씀!
