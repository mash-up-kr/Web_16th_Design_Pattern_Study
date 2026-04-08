# Decorator Pattern

## [What] Decorator Pattern이란?

> "기존 객체를 수정하지 않고, 새로운 기능을 덧붙이는 패턴" _(claude)_
 
> 크리스마스 트리에 전구, 별, 리본 같은 장식을 하나씩 걸어가는 것처럼.  
> 기존 객체의 코드를 건드리지 않고 외부에서 기능을 하나씩 감싸서(wrap) 덧붙이는 것이 Decorator Pattern의 핵심이다. _(gemini)_

즉, 객체의 구조나 클래스 자체를 변경하지 않으면서도, **런타임에 동적으로 새로운 책임(기능)을 추가**할 수 있게 하는 구조적 패턴이다.

## [Example] 예시로 알아보기

### Decorator 없이 직접 다루는 경우

예를 들어, 커피 주문 시스템을 만든다고 해보자.  
아메리카노에 우유를 추가하고, 시럽을 추가하고, 휘핑을 추가할 수 있다.

Decorator 없이 이를 처리하면, 모든 조합에 대해 클래스를 만들어야 한다.

```js
class Americano {
  cost() { return 4000; }
  description() { return "아메리카노"; }
}

class AmericanoWithMilk {
  cost() { return 4500; }
  description() { return "아메리카노 + 우유"; }
}

class AmericanoWithMilkAndSyrup {
  cost() { return 5000; }
  description() { return "아메리카노 + 우유 + 시럽"; }
}

class AmericanoWithWhip {
  cost() { return 5000; }
  description() { return "아메리카노 + 휘핑"; }
}

...

// 조합이 늘어날수록 클래스가 폭발적으로 증가한다...
```

옵션이 3개만 되어도 조합은 2³ = 8가지가 된다.
이를 **클래스 폭발(Class Explosion)** 이라고 한다.

### Decorator를 적용한 경우

```js
// 기본 커피
class Coffee {
  cost() {
    return 4000;
  }

  description() {
    return "아메리카노";
  }
}

// Decorator: 우유 추가
function withMilk(coffee) {
  const originalCost = coffee.cost();
  const originalDesc = coffee.description();

  coffee.cost = () => originalCost + 500;
  coffee.description = () => `${originalDesc} + 우유`;

  return coffee;
}

// Decorator: 시럽 추가
function withSyrup(coffee) {
  const originalCost = coffee.cost();
  const originalDesc = coffee.description();

  coffee.cost = () => originalCost + 500;
  coffee.description = () => `${originalDesc} + 시럽`;

  return coffee;
}

// Decorator: 휘핑 추가
function withWhip(coffee) {
  const originalCost = coffee.cost();
  const originalDesc = coffee.description();

  coffee.cost = () => originalCost + 1000;
  coffee.description = () => `${originalDesc} + 휘핑`;

  return coffee;
}
```

이제 원하는 조합을 자유롭게 만들 수 있다.

```js
let myCoffee = new Coffee();
myCoffee = withMilk(myCoffee);
myCoffee = withSyrup(myCoffee);

console.log(myCoffee.description()); // "아메리카노 + 우유 + 시럽"
console.log(myCoffee.cost());        // 5000
```

클래스를 수십 개 만들 필요 없이, **기본 객체에 Decorator를 하나씩 감싸는 것만으로** 기능이 확장된다.

### 프론트엔드에서의 Decorator 예시

Decorator는 프론트엔드에서도 자주 사용된다.
예를 들어, API 호출 함수에 로깅, 캐싱, 재시도 같은 기능을 덧붙이는 경우를 생각해보자.

```js
// 기본 API 호출 함수
async function fetchUser(userId) {
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
}

// Decorator: 로깅 추가
function withLogging(fn) {
  return async function (...args) {
    console.log(`[호출] ${fn.name}(${args.join(", ")})`);
    const result = await fn(...args);
    console.log(`[결과]`, result);
    return result;
  };
}

// Decorator: 캐싱 추가
function withCache(fn) {
  const cache = new Map();

  return async function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      console.log("[캐시 히트]");
      return cache.get(key);
    }
    const result = await fn(...args);
    cache.set(key, result);
    return result;
  };
}

// 조합하기
const enhancedFetchUser = withLogging(withCache(fetchUser));

await enhancedFetchUser(42); // API 호출 + 로깅 + 캐싱
await enhancedFetchUser(42); // 캐시 히트!
```

`fetchUser` 함수 자체는 전혀 수정하지 않았지만, Decorator를 감싸는 것만으로 로깅과 캐싱 기능이 추가되었다.

## [How] Decorator Pattern은 어떻게 구현하는가?

핵심은 **기존 객체(또는 함수)를 감싸는 래퍼(Wrapper)를 만들어, 원래 기능은 그대로 위임하면서 추가 기능만 덧붙이는 것**이다.

### 1. 함수 래핑 방식 (가장 실용적)

```js
function withTimestamp(fn) {
  return function (...args) {
    console.log(`[${new Date().toISOString()}]`);
    return fn(...args);
  };
}

function greet(name) {
  return `안녕, ${name}!`;
}

const timedGreet = withTimestamp(greet);
timedGreet("정우"); // [2026-04-02T...] 안녕, 정우!
```

- **특징**: JS에서 가장 자연스러운 방식. 고차 함수(Higher-Order Function)와 동일한 개념이다.
- **활용**: React의 HOC(Higher-Order Component)가 이 방식의 대표적인 사례.

### 2. 클래스 래핑 방식 (GoF 원형)

```ts
// 공통 인터페이스
interface Component {
  operation(): string;
}

// 기본 객체
class ConcreteComponent implements Component {
  operation(): string {
    return "기본 동작";
  }
}

// Decorator 베이스
class Decorator implements Component {
  constructor(protected component: Component) {}

  operation(): string {
    return this.component.operation();
  }
}

// 구체적인 Decorator들
class LoggingDecorator extends Decorator {
  operation(): string {
    console.log("[LOG] operation 호출됨");
    return this.component.operation();
  }
}

class TimingDecorator extends Decorator {
  operation(): string {
    const start = performance.now();
    const result = this.component.operation();
    console.log(`[TIMING] ${performance.now() - start}ms`);
    return result;
  }
}

// 사용
let component: Component = new ConcreteComponent();
component = new LoggingDecorator(component);
component = new TimingDecorator(component);

component.operation(); // 로깅 → 타이밍 → 기본 동작
```

- **특징**: GoF 디자인 패턴의 원형. 인터페이스를 공유하기 때문에 Decorator가 몇 겹이든 동일하게 사용할 수 있다.

### 3. TC39 Decorator (Stage 3)

TC39에서 JavaScript에 Decorator 문법을 공식으로 추가하는 제안이 Stage 3까지 진행되었다.  
TypeScript 5.0+에서 사용할 수 있으며, 클래스와 클래스 메서드에 `@` 문법으로 Decorator를 적용할 수 있다.

```ts
function log(originalMethod: any, context: ClassMethodDecoratorContext) {
  return function (...args: any[]) {
    console.log(`[LOG] ${String(context.name)} 호출됨`);
    return originalMethod.call(this, ...args);
  };
}

class UserService {
  @log
  getUser(id: number) {
    return { id, name: "정우" };
  }
}

const service = new UserService();
service.getUser(1); // [LOG] getUser 호출됨 → { id: 1, name: "정우" }
```

## [Why] Decorator Pattern은 왜 사용하는가?

Q. 언제 사용하면 좋을까?
- 핵심은 **기존 코드를 수정하지 않고, 기능을 유연하게 추가하거나 조합하고 싶을 때.**

Q. 왜 클래스를 직접 수정하거나 상속하지 않고 Decorator를 쓰는가?

1. **개방-폐쇄 원칙(OCP)을 지킬 수 있다.**
    - "확장에는 열려 있고, 수정에는 닫혀 있어야 한다"는 원칙. Decorator는 원본 코드를 변경하지 않고 기능을 확장하므로, OCP를 자연스럽게 만족한다.

2. **클래스 폭발을 방지할 수 있다.**
    - 상속으로 기능 조합을 처리하면, 옵션이 N개일 때 최대 2^N개의 서브클래스가 필요하다. Decorator는 기능을 독립적인 단위로 분리하고 런타임에 조합하므로, 클래스 수를 줄일 수 있다.

3. **단일 책임 원칙(SRP)을 지키기 쉽다.**
    - 각 Decorator는 하나의 부가 기능만 담당한다. 로깅은 로깅 Decorator, 캐싱은 캐싱 Decorator. 기능별로 분리되어 있으므로 독립적으로 테스트하고 재사용할 수 있다.

4. **런타임에 동적으로 기능을 추가/제거할 수 있다.**
    - 상속은 컴파일 타임에 구조가 결정되지만, Decorator는 런타임에 자유롭게 조합할 수 있다. 조건에 따라 특정 Decorator만 적용하는 것도 가능하다.

Q. Decorator의 단점은 무엇인가?
- **래퍼가 깊어지면 디버깅이 어렵다**: Decorator를 여러 겹 감싸면, 에러가 발생했을 때 콜 스택을 추적하기 복잡해질 수 있다.
- **순서에 의존적**: Decorator를 적용하는 순서에 따라 결과가 달라질 수 있다. `withLogging(withCache(fn))`과 `withCache(withLogging(fn))`은 동작이 다르다.
- **인터페이스 투명성**: 감싸진 객체가 원본과 동일한 인터페이스를 가져야 하므로, Decorator마다 모든 메서드를 위임(delegate)해야 하는 번거로움이 있다.

## [Question] Decorator Pattern과 관련하여 궁금한 점

### Decorator vs Mixin, 어떤 상황에서 어떤 걸 선택해야 하는가?

둘 다 "기존 객체에 기능을 추가한다"는 점에서 비슷해 보이지만, 접근 방식이 다르다.

| 기준 | Decorator | Mixin |
|:--|:--|:--|
| 적용 방식 | 객체를 감싸서(wrap) 기능 추가 | 객체/프로토타입에 직접 기능 복사 |
| 원본 변경 여부 | 원본 객체를 변경하지 않음 | 프로토타입을 직접 수정함 |
| 조합의 순서 | 순서가 중요함 (래핑 순서 = 실행 순서) | 순서가 중요할 수 있음 (같은 이름이면 덮어씀) |
| 동적 제거 | 래핑을 벗기면 제거 가능 | 한번 복사하면 제거가 어려움 |
| 적합한 경우 | 부가 기능을 조건부로 추가/제거할 때 | 여러 클래스에 공통 기능을 일괄 부여할 때 |

#### Decorator가 적합한 경우 — 조건부로 기능을 덧붙일 때

```js
// 환경에 따라 로깅을 붙이거나 빼거나
let fetchUser = baseFetchUser;

if (isDev) fetchUser = withLogging(fetchUser);
if (useCache) fetchUser = withCache(fetchUser);

await fetchUser(42);
```

원본 함수는 그대로 두고, 필요한 기능만 골라서 감쌀 수 있다.

#### Mixin이 적합한 경우 — 여러 클래스에 공통 기능을 일괄 부여할 때

```js
const Serializable = {
  toJSON() { return JSON.stringify(this); },
  fromJSON(json) { return Object.assign(this, JSON.parse(json)); }
};

// User, Product, Order 등 여러 클래스에 동일한 직렬화 기능을 한 번에 부여
Object.assign(User.prototype, Serializable);
Object.assign(Product.prototype, Serializable);
Object.assign(Order.prototype, Serializable);
```

조건부 추가/제거가 아니라, "이 기능은 모든 곳에 있어야 해"라는 상황에서는 Mixin이 더 직관적이다.
