# [Mixin Pattern](https://patterns-dev-kr.github.io/design-patterns/mixin-pattern/)

## [What] Mixin Pattern이란?
> "상속 없이, 객체나 클래스에 재사용 가능한 기능을 조합하는 패턴"

> Mixin은 상속 없이 어떤 객체나 클래스에 재사용 가능한 기능을 추가할 수 있는 객체이다. _(patterns.dev)_

Mixin이라는 이름은 미국 매사추세츠의 아이스크림 가게 Steve's Ice Cream Parlor에서 유래했다.  
바닐라, 초콜릿 같은 기본 아이스크림에 견과류, 쿠키, 퍼지 같은 재료를 섞어 넣는 것을 "mix-in"이라 불렀는데, 이 개념이 프로그래밍으로 넘어온 것이다. 기본 클래스(아이스크림)에 원하는 기능(토핑)을 골라서 섞어 넣는다는 비유가 Mixin의 핵심을 잘 드러낸다.

## [Patterns.dev] Mixin Pattern 예시로 알아보기

Patterns.dev에서는 `Dog` 클래스를 통해 Mixin Pattern을 설명하고 있다.

```js
class Dog {
  constructor(name) {
    this.name = name
  }
}
```

강아지는 이름 외에도 짖거나, 꼬리를 흔들거나, 놀 수 있어야 한다. 이 기능들을 `Dog` 클래스에 직접 정의하는 대신, 별도의 Mixin 객체로 분리할 수 있다.

```js
const dogFunctionality = {
  bark: () => console.log('Woof!'),
  wagTail: () => console.log('Wagging my tail!'),
  play: () => console.log('Playing!'),
}
```

`Object.assign`을 사용해 `Dog`의 프로토타입에 이 기능들을 주입하면, 모든 `Dog` 인스턴스가 해당 기능을 사용할 수 있게 된다.

```js
Object.assign(Dog.prototype, dogFunctionality)

const pet1 = new Dog('Daisy')
pet1.bark()  // Woof!
pet1.play()  // Playing!
```

여기서 중요한 점은, `Dog`가 `dogFunctionality`를 **상속**한 것이 아니라는 것이다.  
단지 기능을 **복사해서 넣은 것**뿐이다. 이것이 Mixin의 핵심이다.

```js
const pet1 = new Dog('Daisy')

// 인스턴스 자체에는 name만 있음
console.log(pet1.hasOwnProperty('name'))  // true
console.log(pet1.hasOwnProperty('bark'))  // false

// bark는 프로토타입에 존재

// `pet1.bark()`가 동작하는 이유는 JavaScript의 프로토타입 체인 때문.
// `pet1`에서 `bark`를 찾지 못하면, `pet1.__proto__`(= `Dog.prototype`)에서 찾아 실행함.

pet1 { name: 'Daisy' }
  └─ __proto__ (Dog.prototype) { bark, wagTail, play }  ← 여기에 복사됨
       └─ __proto__ (Object.prototype)
```

다만 이렇게 되면 prototype 오염이 발생할 수 있다. Dog Class의 본체를 건들이는 건 아니지만, prototype을 수정하기 때문에 가능한 것이다.

나아가 Mixin 자체도 다른 Mixin의 기능을 조합할 수 있다.  
예를 들어, 대부분의 포유류가 걷거나 잘 수 있으므로 `animalFunctionality`를 따로 만들고, 이를 `dogFunctionality`에 합칠 수 있다.

```js
const animalFunctionality = {
  walk: () => console.log('Walking!'),
  sleep: () => console.log('Sleeping!'),
}

Object.assign(dogFunctionality, animalFunctionality)
Object.assign(Dog.prototype, dogFunctionality)
```

이제 `Dog` 인스턴스는 `bark`, `wagTail`, `play`뿐 아니라 `walk`, `sleep`까지 사용할 수 있다.

## [How] Mixin Pattern은 어떻게 구현하는가?

Mixin Pattern은 언어마다 구현 방식이 다르다.  
하지만 핵심 개념은 동일하다: **상속 계층을 만들지 않고, 원하는 기능 묶음을 클래스나 객체에 주입하는 것.**

### TypeScript — 함수 기반 Mixin

TypeScript에서는 클래스를 인자로 받아 확장된 클래스를 반환하는 함수 형태로 Mixin을 구현할 수 있다.

```ts
type Constructor<T = {}> = new (...args: any[]) => T

// Mixin 1: 말하기 초능력
function CanSay<TBase extends Constructor>(Base: TBase) {
   return class extends Base {
      say(message: string) {
         console.log(`[말하기]: "${message}"라고 외칩니다.`);
      }
   }
}

// Mixin 2: 날기 초능력
function CanFly<TBase extends Constructor>(Base: TBase) {
   return class extends Base {
      isFlying = false;

      fly() {
         this.isFlying = true;
         console.log("슝~ 하늘을 날기 시작합니다!");
      }
   }
}

// 1. 기본 클래스 (그냥 평범한 사람)
class Person {
   constructor(public name: string) {}
}

// 2. 능력 조합하기: 사람 + 말하기 + 날기
const SuperHero = CanFly(CanSay(Person));

// 3. 사용하기
const hero = new SuperHero('아이언맨');

hero.say('자, 가자!'); // [말하기]: "자, 가자!"라고 외칩니다.
hero.fly();           // 슝~ 하늘을 날기 시작합니다!
console.log(hero.isFlying); // true
```

## [Why] Mixin Pattern은 왜 사용하는가?

Q. 언제 사용하면 좋을까?
- 핵심은 **여러 클래스에 걸쳐 공통 기능을 재사용하고 싶지만, 상속 계층을 만들고 싶지 않은 경우.**

Q. 왜 상속 대신 Mixin을 쓰는가?

1. **다이아몬드 문제(Diamond Problem)를 회피할 수 있다.**
    - 다중 상속 구조는 두 부모 클래스가 같은 메서드를 갖고 있을 때 어느 것을 사용할 지 구조적 문제가 생기지만, Mixin은 맨 마지막에 등록한 매서드의 기능을 사용한다.

2. **"is-a" 관계가 아닌 "has-a (기능)" 관계를 표현한다.**
   - Mixin은 제약이 아닌 **조합의 유연성**에 초점을 맞추고 있다.
   - "이 유저는 관리자이면서(is-a), 게시글 삭제 능력도 있고(has-a), 로그 기록 능력도 있어야(has-a) 해!" 같은 상황에서 도움이 됨.

3. **단일 책임 원칙(SRP)을 지키기 쉽다.**
    - 각 Mixin은 하나의 기능만 담당하므로, 기능별로 코드를 분리하고 독립적으로 테스트할 수 있다.

Q. Mixin의 단점은 무엇인가?
- **프로토타입 오염**: JavaScript에서 `Object.assign`으로 프로토타입을 직접 수정하면, 의도치 않은 프로퍼티 충돌이나 덮어쓰기가 발생할 수 있다.
- **추적이 어려움**: 어떤 메서드가 어디서 온 건지 파악하기 어려울 수 있다. 특히 여러 Mixin이 동일한 이름의 메서드를 정의하면 마지막에 적용된 것이 이전 것을 덮어쓰게 된다.
- **React에서의 퇴출**: React 팀은 Mixin이 컴포넌트의 복잡도를 증가시키고 재사용을 어렵게 만든다고 판단하여 사용을 공식적으로 지양하고, 고차 컴포넌트(HOC)와 이후 Hooks로 대체했다고 하네용.

## [Question] Mixin Pattern과 관련하여 궁금한 점

### Mixin vs 상속, 언제 어떤 걸 써야 하는가?

| 기준 | 상속 | Mixin |
|:--|:--|:--|
| 관계 | "A는 B이다" (is-a) | "A는 B의 기능을 갖는다" (has-a) |
| 계층 | 수직적 계층 구조 | 수평적 기능 조합 |
| 유연성 | 부모 변경 시 자식에 영향 | 기능 단위로 독립적 추가/제거 |
| 적합한 경우 | 명확한 분류 체계가 있을 때 | 여러 클래스에 공통 기능을 뿌릴 때 |
