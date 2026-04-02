# 3. 데코레이터(Decorator) 패턴

## 3-1. 정의

기존 객체를 수정하지 않고, 객체에 새로운 기능을 동적으로 덧붙이는 패턴

* 원본 보존 — 기존 객체 자체는 바꾸지 않음
* 기능 확장 — 객체를 감싸서 새로운 기능을 추가
* 조합 가능 — 기능을 여러 겹으로 붙일 수 있음

핵심은 **원래 객체를 그대로 두고, 같은 인터페이스를 가진 래퍼로 감싸 기능을 더하는 것**

## 3-2. 필요성

기본 기능은 같지만, 부가 기능의 조합이 계속 늘어나는 경우가 있음

ex) 커피 주문

```tsx
Espresso
EspressoWithMilk
EspressoWithWhip
EspressoWithMilkAndWhip
EspressoWithMocha
EspressoWithMilkAndMocha
EspressoWithMilkAndWhipAndMocha
```

옵션이 늘어날수록 조합 수가 폭발하고, 상속이나 클래스 분기로 해결하려 하면 구조가 금방 비대해짐

그래서

```tsx
Mocha(
  Milk(
    Espresso
  )
)
```

처럼 **기본 객체를 만들고, 필요한 기능을 래퍼로 감싸며 추가**하는 방식이 필요  
⇒ 기능이 상속으로 고정되는 게 아니라 **조합으로 유연하게 붙음**

## 3-3. 구조

1. Component
   * 공통 인터페이스
2. ConcreteComponent
   * 기본 기능을 가진 객체
3. Decorator
   * 컴포넌트를 감싸는 공통 래퍼
4. ConcreteDecorator
   * 실제로 추가 기능을 붙이는 데코레이터

즉 **같은 인터페이스를 구현하는 래퍼 객체가 원본 객체를 감싸고, 호출을 위임하면서 자기 기능을 덧붙이는 구조**

## 3-4. 동작 원리

1. 기본 객체를 생성함
2. 같은 인터페이스를 따르는 데코레이터가 그 객체를 감쌈
3. 데코레이터는 원래 동작을 호출한 뒤, 앞뒤로 기능을 추가함
4. 필요한 만큼 여러 데코레이터를 겹쳐 붙일 수 있음

```tsx
interface Coffee {
  getCost(): number;
  getDescription(): string;
}

class Espresso implements Coffee {
  getCost(): number {
    return 3000;
  }

  getDescription(): string {
    return "Espresso";
  }
}

class CoffeeDecorator implements Coffee {
  constructor(protected coffee: Coffee) {}

  getCost(): number {
    return this.coffee.getCost();
  }

  getDescription(): string {
    return this.coffee.getDescription();
  }
}

class MilkDecorator extends CoffeeDecorator {
  getCost(): number {
    return super.getCost() + 500;
  }

  getDescription(): string {
    return `${super.getDescription()}, Milk`;
  }
}

class MochaDecorator extends CoffeeDecorator {
  getCost(): number {
    return super.getCost() + 700;
  }

  getDescription(): string {
    return `${super.getDescription()}, Mocha`;
  }
}
```

```tsx
let coffee: Coffee = new Espresso();
coffee = new MilkDecorator(coffee);
coffee = new MochaDecorator(coffee);

console.log(coffee.getDescription()); // Espresso, Milk, Mocha
console.log(coffee.getCost()); // 4200
```

핵심은 `Espresso` 자체를 수정하지 않고, `MilkDecorator`, `MochaDecorator`를 하나씩 감싸며 기능을 추가했다는 점

## 3-5. 상속과의 차이

* 상속 — 구조가 미리 고정되고, 기능 조합이 클래스 계층으로 표현됨
* 데코레이터 — 실행 시점에 기능을 조합하고, 객체 래핑으로 확장함

상속은 조합 수가 많아질수록 클래스 수도 늘어나지만, 데코레이터는 **필요한 기능만 조합해서 붙일 수 있음**

## 3-6. 실제 활용 예시

객체지향 클래스 구조뿐 아니라, 함수 래핑으로도 같은 의도를 표현할 수 있음

```tsx
function request() {
  console.log("기본 요청 실행");
}

function withLogging(fn: () => void) {
  return function () {
    console.log("요청 시작");
    fn();
    console.log("요청 종료");
  };
}

const loggedRequest = withLogging(request);
loggedRequest();
```

이건 GoF식 클래스 데코레이터와 완전히 똑같은 형태는 아니지만,

* 기존 동작을 감싸고
* 앞뒤에 부가 기능을 추가하며
* 원래 함수 자체는 수정하지 않는다는 점에서

같은 의도를 가진 설계

실무에서는 이런 식으로

* 로깅 추가
* 권한 체크 추가
* 실행 시간 측정
* 캐싱
* 재시도 로직

같은 부가 기능을 감싸는 방식으로 자주 보게 됨

## 3-7. 기대 효과

**얻는 것**

* 유연한 기능 조합 — 경우의 수마다 클래스를 만들 필요가 없음
* 기존 코드 수정 없이 확장 가능 — 원본을 바꾸지 않고 기능을 더할 수 있음
* 기능 분리 — Milk, Mocha처럼 부가 기능을 작은 단위로 나눌 수 있음
* 런타임 조합 가능 — 상황에 따라 붙였다 뗄 수 있음

## 3-8. 트레이드오프

**잃는 것**

* 객체 수 증가 — 래퍼가 계속 쌓일 수 있음
* 구조 복잡도 증가 — 여러 겹 wrapper를 거치면 흐름 파악이 어려움
* 디버깅 불편 — 실제 동작이 어떤 순서로 감싸져 있는지 추적해야 함
* 순서 의존성 — 캐싱 후 로깅과 로깅 후 캐싱은 의미가 달라질 수 있음

## 3-9. 언제 쓰면 좋을까?

* 기존 객체는 건드리고 싶지 않을 때
* 부가 기능을 선택적으로 붙이고 싶을 때
* 기능 조합의 경우의 수가 많아질 때
* 같은 인터페이스를 유지한 채 확장하고 싶을 때
