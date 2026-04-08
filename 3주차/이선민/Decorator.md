## Decorator 패턴이란?

- **객체를 래퍼(Wrapper) 객체로 감싸 기능을 동적으로 추가**하는 구조 패턴
- 상속 대신 **합성(Composition)** 을 사용하기 때문에, 기존 코드를 수정하지 않고 런타임에 자유롭게 기능을 붙이거나 뗄 수 있다
- 래퍼는 원래 객체와 **동일한 인터페이스**를 구현하므로, 사용하는 쪽에서는 래핑 여부를 알 필요가 없다

## 구조

| 구성요소 | 역할 |
|---|---|
| **Component** | 공통 인터페이스 정의 |
| **Concrete Component** | 기본 동작을 구현한 실제 객체 |
| **Base Decorator** | Component를 구현하면서 내부에 Component 참조를 가짐, 작업을 내부 객체에 위임 |
| **Concrete Decorator** | Base Decorator를 상속받아 추가 동작을 구현 |

```
Component (인터페이스)
├─ Concrete Component   ← 기본 객체
└─ Base Decorator       ← Component를 감싸는 래퍼
   ├─ Concrete Decorator A
   └─ Concrete Decorator B
```

## 기본 구조

```javascript
// Concrete Component
class BasicCoffee {
  cost() { return 2000 }
  description() { return '기본 커피' }
}

// Base Decorator
class CoffeeDecorator {
  constructor(coffee) {
    this._coffee = coffee
  }
  cost() { return this._coffee.cost() }
  description() { return this._coffee.description() }
}

// Concrete Decorators
class MilkDecorator extends CoffeeDecorator {
  cost() { return this._coffee.cost() + 500 }
  description() { return this._coffee.description() + ', 우유' }
}

class ShotDecorator extends CoffeeDecorator {
  cost() { return this._coffee.cost() + 700 }
  description() { return this._coffee.description() + ', 샷 추가' }
}

// 사용 — 여러 Decorator를 자유롭게 중첩
let coffee = new BasicCoffee()
coffee = new MilkDecorator(coffee)
coffee = new ShotDecorator(coffee)
coffee = new ShotDecorator(coffee)

console.log(coffee.description())  // 기본 커피, 우유, 샷 추가, 샷 추가
console.log(coffee.cost())         // 3900
```

- `coffee` 변수를 교체할 뿐 `BasicCoffee` 코드는 전혀 건드리지 않는다
- 동일한 `cost()`, `description()` 인터페이스를 유지하므로 사용 측 코드도 바뀌지 않는다

## 문제 상황: 상속으로 조합하면 클래스가 폭발한다

- 기능이 조합될수록 필요한 서브클래스 수가 기하급수적으로 늘어난다

```javascript
// 알림 기능 예시 — 상속 방식
class Notifier {}
class EmailNotifier extends Notifier {}
class SMSNotifier extends Notifier {}
class SlackNotifier extends Notifier {}

// 조합이 필요해지면?
class EmailAndSMSNotifier extends Notifier {}
class EmailAndSlackNotifier extends Notifier {}
class SMSAndSlackNotifier extends Notifier {}
class EmailAndSMSAndSlackNotifier extends Notifier {}
// 채널이 늘어날수록 클래스 수가 폭발...
```

## Decorator로 해결하기

```javascript
// Concrete Component — 기본 알림
class Notifier {
  send(message) {
    console.log(`기본 알림: ${message}`)
  }
}

// Base Decorator
class NotifierDecorator {
  constructor(notifier) {
    this._notifier = notifier
  }
  send(message) {
    this._notifier.send(message)
  }
}

// Concrete Decorators
class EmailDecorator extends NotifierDecorator {
  send(message) {
    super.send(message)
    console.log(`이메일 발송: ${message}`)
  }
}

class SMSDecorator extends NotifierDecorator {
  send(message) {
    super.send(message)
    console.log(`SMS 발송: ${message}`)
  }
}

class SlackDecorator extends NotifierDecorator {
  send(message) {
    super.send(message)
    console.log(`Slack 발송: ${message}`)
  }
}

// 조합 — 클래스 추가 없이 런타임에 자유롭게 구성
let notifier = new Notifier()
notifier = new EmailDecorator(notifier)
notifier = new SMSDecorator(notifier)
notifier = new SlackDecorator(notifier)

notifier.send('서버 장애 발생!')
// 기본 알림: 서버 장애 발생!
// 이메일 발송: 서버 장애 발생!
// SMS 발송: 서버 장애 발생!
// Slack 발송: 서버 장애 발생!
```

- 새로운 채널이 생겨도 `Decorator` 클래스 하나만 추가하면 된다
- 기존 `Notifier` 코드는 변경되지 않는다

## 함수형 Decorator

- 클래스 없이 **함수를 감싸는 방식**으로도 동일한 패턴을 구현할 수 있다

```javascript
// 기본 함수
function fetchData(url) {
  return fetch(url).then(res => res.json())
}

// 로깅 Decorator
function withLogging(fn) {
  return function(...args) {
    console.log(`호출: ${fn.name}(${args})`)
    const result = fn(...args)
    console.log(`완료: ${fn.name}`)
    return result
  }
}

// 캐싱 Decorator
function withCache(fn) {
  const cache = new Map()
  return function(...args) {
    const key = JSON.stringify(args)
    if (cache.has(key)) {
      console.log('캐시 히트')
      return Promise.resolve(cache.get(key))
    }
    return fn(...args).then(result => {
      cache.set(key, result)
      return result
    })
  }
}

// 재시도 Decorator
function withRetry(fn, maxRetry = 3) {
  return async function(...args) {
    for (let i = 0; i < maxRetry; i++) {
      try {
        return await fn(...args)
      } catch (e) {
        if (i === maxRetry - 1) throw e
        console.log(`재시도 ${i + 1}/${maxRetry}`)
      }
    }
  }
}

// 여러 Decorator 조합
let enhancedFetch = fetchData
enhancedFetch = withLogging(enhancedFetch)
enhancedFetch = withCache(enhancedFetch)
enhancedFetch = withRetry(enhancedFetch)

enhancedFetch('/api/data')
```

- 각 Decorator는 함수를 받아 함수를 반환하는 순수한 형태로, 클래스 기반보다 훨씬 간결하다
- 이 방식은 Express의 미들웨어, React의 HOC 등과 동일한 아이디어다

## 다른 패턴과의 비교

| | Decorator | Adapter | Proxy |
|---|---|---|---|
| 목적 | 기능 추가 | 인터페이스 변환 | 접근 제어 |
| 인터페이스 | 동일하게 유지 | 다른 인터페이스로 변환 | 동일하게 유지 |
| 중첩 | 여러 개 중첩 가능 | 보통 하나 | 보통 하나 |

- **Proxy**와 구조가 거의 같지만, Proxy는 접근을 제어하는 목적이고 Decorator는 기능을 추가하는 목적이다
- **Composite** 패턴과 함께 쓰이기도 한다 — Composite는 여러 자식을 가지지만, Decorator는 항상 단일 자식만 갖는다

## 장점

- **OCP 준수**: 기존 코드를 수정하지 않고 새로운 기능을 추가할 수 있다
- **유연한 조합**: 여러 Decorator를 자유롭게 중첩하여 복합적인 행동을 구성할 수 있다
- **단일 책임 원칙**: 각 Decorator가 하나의 기능만 담당하므로 역할이 명확하다
- **런타임 확장**: 상속은 컴파일 타임에 결정되지만, Decorator는 런타임에 동적으로 기능을 붙이거나 뗄 수 있다

## 단점

- **순서 의존성**: Decorator를 중첩하는 순서에 따라 동작이 달라질 수 있어 주의가 필요하다
- **특정 래퍼 제거 어려움**: 중간에 특정 Decorator만 제거하기가 어렵다
- **디버깅 복잡**: 여러 겹으로 중첩되면 어떤 계층에서 문제가 생겼는지 추적하기 어렵다
- **단순한 경우엔 과도함**: 기능 하나만 추가할 때는 오히려 코드가 복잡해질 수 있다

## 참고자료

- https://refactoring.guru/design-patterns/decorator
- https://medium.com/@OSACHIL/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4-%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0-%ED%8C%A8%ED%84%B4-decorator-pattern-e9821185eb9e
