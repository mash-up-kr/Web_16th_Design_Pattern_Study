# 데코레이터 패턴

- 데코레이터 패턴은 코드 재사용을 목표로 하는 구조 패턴입니다. 믹스인과 마찬가지로 객체 서브클래싱의 또 다른 방법이라고 생각하시면 됩니다.

## 데코레이터

- 기본적으로 데코레이터는 기존 클래스에 "동적"으로(Dynamic) 기능을 추가하기 위해 사용합니다.
- 데코레이터에 대한 기능은 데코레이터가 부착되는 클래스에 필수적이지 않을듯한 인상이 있습니다. 만약 클래스 사용에 필수적인 기능이었다면 이미 부모 클래스에 구현되었을 것 입니다.
- 데코레이터를 사용하면 기존 시스템의 내부 코드를 힘겹게 수정하지 않고도 기능을 추가할 수 있게 됩니다. 기능 확장에 조금 더 초점을 둔 패턴이지요
  - 데코레이터 사용의 주된 이유는 애플리케이션의 기능이 다양한 타입의 객체를 필요로 할 수 있기 때문입니다.
    > 다양한 유형과 타입의 사람이 존재하듯이 다양한 타입으로 정희할 기능을 필요로 하는 경우가 존재함!
    > 사실 이 부분은 TS의 식별할 수 있는 유니온으로 처리 가능할거같은 내용이네요
  - 프로토타입 상속에 의지하기 보다는 하나의 베이스가 되는 클래스에 추가 기능을 제공하는 데코레이터 객체를 점진적으로 추가합니다. 이는 서브클래싱 대신 베이스 객체에 속성이나 메서드를 추가하여 간소화 하겠다는 의도입니다.
  ```js
  // Vehicle 생성자
  class Vehicle {
    constructor(vehicleType) {
      // 일부 합리적인 기본값
      this.vehicleType = vehicleType || "car";
      this.model = "default";
      this.license = "00000-000";
    }
  }

  // 기본 Vehicle에 대한 테스트 인스턴스
  const testInstance = new Vehicle("car");
  console.log(testInstance);

  // 출력:
  // vehicle: car, model:default, license: 00000-000

  // 데코레이트할 새로운 차량 인스턴스를 생성합니다.
  const truck = new Vehicle("truck");

  // Vehicle에 추가하는 새로운 기능
  truck.setModel = function (modelName) {
    this.model = modelName;
  };

  truck.setColor = function (color) {
    this.color = color;
  };

  // 같 설정자와 같 함달이 올바르게 작동하는지 테스트
  truck.setModel("CAT");
  truck.setColor("blue");

  console.log(truck);

  // 출력:
  // vehicle:truck, model:CAT, color: blue

  // "vehicle"이 변경되지 않았음을 보여줍니다.
  const secondInstance = new Vehicle("car");
  console.log(secondInstance);

  // 출력:
  // vehicle: car, model:default, license: 00000-000
  ```

## 장단점
- 데코레이터 패턴의 객체는 새로운 기능으로 감싸져 확장되거나 "데코레이트" 될 수 있으며 베이스 객체가 변경될 걱정 없이 사용할 수 있습니다.
- 더 넓은 의미에서 보았을 때 수 많은 서브클래스에 의존할 필요도 없어집니다.
- 그러나 네임스페이스에 작고 비슷한 객체를 추가하기 때문에 잘 관지되지 않는다면 애플리케이션의 구조를 복잡하게 만드는 요인 중 하나가 될 수 있습니다.
  - 데코레이션 패턴에 익숙하지 않은 다른 액터(개발자)가 패턴 사용의 목적을 파악하지 못한다면 유지보수나 관리에 어려움을 겪을 수 있지만, 이 경우 문서화 또는 패턴에 대한 이해도를 높이면 해결이 가능합니다.
  > jsdoc 만세

> 여기까지가 책에서 가져온 내용이었고 아래는 강의 내용을 가져와보겠습니다.

# 데코레이터 패턴 2
- 데코레이터는 포장지다!
  > 프론트에서는 사용할 일이 거의 없지만 nest.js에서는 대놓고 사용하는 경우가 종종 있습니다.
  - 원본 객체를 감싸고있는 선물상자 같은 너낌.. 즉, 원본에는 영향을 미치지 않지만 확장하기엔 좋음
- 객체를 "장식" 하듯이 기능 추가
  - 상속 없이 유연한 확장
- 기존 객체를 수정하지 않고 새로운 기능 부여
  - 이게 매우 큰 강점!
  - OCP 원칙 준수
- 런타임에 동적으로 조합 가능
  - 실행 컨텍스트에 따라서 데코레이터가 부착 여부가 결정될 수 있음

> #### 잠깐! OCP 원칙이 뭔가요 🤔
> - 개방 / 폐쇄 원칙 Open/Close Principle
> - 소프트웨어 요소는 확장에는 열려 있으나, 변경에는 닫혀 있어야 한다..!
> 
> ##### 핵심 개넘
> 1. 확장에 열려 있다는 것은? (Open for Extension)
>    - 새로운 기능이 추가되거나 요구사항이 변경될 때, 기존 코드를 수정하지 않고도 확장할 수 있어야 합니다.
>    - (예: 플러그인 아키텍처, 데코레이터 패턴)
> 2. 변경에 닫혀 있다는 것은? (Closed for Modification)
>    - 기존 코드는 최대한 건드리지 않도록 설계
>    - 버그 발생 시 리스크는 줄이고 유지보수성은 올린다
>
> - OCP 위반 (나쁜 사례)
> ```js
> class PaymentProcessor {
>   processPayment(type: "credit" | "paypal", amount: number) {
>     if (type === "credit") {
>       // 신용카드 처리 로직 (수정 필요 시 이 부분을 변경해야 함)
>     } else if (type === "paypal") {
>       // 페이팔 처리 로직 (새 결제 수단 추가될 때마다 코드 수정 필요)
>     }
>   }
> }
> ```
> - OCP 준수 (좋은 사례)
> ```js
> // 1. 기본 결제 인터페이스 (Component)
> interface Payment {
>   pay(amount: number): void;
> }
> 
> // 2. 기본 결제 구현 (Concrete Component)
> class BasicPayment implements Payment {
>   pay(amount: number) {
>     // 여기에는 메인 결제 로직이 들어오게 됩니다.
>     console.log(`기본 결제: ${amount}원 처리`);
>   }
> }
> 
> // 3. 데코레이터 추상 클래스 (Decorator)
> abstract class PaymentDecorator implements Payment {
>   constructor(protected payment: Payment) {}
> 
>   pay(amount: number) {
>     this.payment.pay(amount);
>   }
> }
> 
> // 4-1. 로깅 기능 데코레이터 (Concrete Decorator A)
> class LoggingDecorator extends PaymentDecorator {
>   pay(amount: number) {
>     console.log(` [로그] 결제 시작: ${amount}원`);
>     super.pay(amount);
>     console.log("[로그] 결제 완료");
>   }
> }
> 
> // 4-2. 보안 검사 데코레이터 (Concrete Decorator B)
> class SecurityDecorator extends PaymentDecorator {
>   pay(amount: number) {
>     if (this.checkFraud(amount)) {
>       super.pay(amount);
>     } else {
>       console.error("보안 위협 감지: 결제 차단");
>     }
>   }
> 
>   private checkFraud(amount: number): boolean {
>     console.log(`[보안] ${amount}원 결제 검증 중...`);
>     return amount <= 1000000; // 100만원 이상 시 차단
>   }
> }
> 
> // 4-3. 환율 변환 데코레이터 (Concrete Decorator C)
> class CurrencyDecorator extends PaymentDecorator {
>   constructor(payment: Payment, private exchangeRate: number) {
>     super(payment);
>   }
> 
>   pay(amount: number) {
>     const convertedAmount = amount * this.exchangeRate;
>     console.log(`[환율] ${amount}USD -> ${convertedAmount}원`);
>     super.pay(convertedAmount);
>   }
> }
> 
> // 5. 클라이언트 코드
> const payment: Payment = new BasicPayment();
> 
> // 기능 조립 (런타임에 동적 구성)
> // 경우에 따라서 "포장"을 통해 추가 구성
> const decoratedPayment = new CurrencyDecorator(
>   new SecurityDecorator(new LoggingDecorator(payment)),
>   1300 // USD -> KRW 환율
> );
> 
> // 실행
> decoratedPayment.pay(100); // 100USD 결제
> ```

## 실생활 예시
1. 커피 주문
   - 기본 객체 : 아메리카노
   - 데코레이터: 시럽추가(+500원), 휘핑크림(+1_000원), 얼음(+0원)
2. 캐릭터 장비
   - 기본: 검
   - 데코레이터: 불 속성 부여(+공격력), 날카로운 연마(+치명타)
3. 텍스트 편집기
   - 기본: 기본 텍스트
   - 데코레이터: 볼드체, 밑줄, 색상변경

> 개발자라면 앞으로 커피 주문시에는 데코레이터 패턴을 응용해서 주문하시면 됩니다.
> "아이스 아메리카노 한 잔에 얼음 많이 추가해주세요" ❌
> "아이스 아메리카노 한 잔에 얼음 많이 데코레이트 해주세요" ✅

## `@`를 사용한 타입스크립트에서의 활용법
- 리빙포인트: 다른 언어에서도 데코레이터는 @를 통해 사용합니다.
- tsconfig.json 파일에서 컴파일러 옵션을 추가하면 사용하실 수 있습니다.
  > ts가 곧 7.0이 릴리즈될거같네요 Go lang으로 tsc가 새로 작성되었다고 합니다.

```ts
// 클래스 데코레이터: 생성자 함수(또는 클래스)를 받아서 꾸밈
function LogClass(constructor: new (...args: unknown[]) => object) {
  // 원래 생성자를 감싼 새 생성자 반환 (인스턴스 생성 시 한 번 실행)
  return class extends constructor {
    constructor(...args: unknown[]) {
      super(...args);
      console.log(`[LogClass] ${constructor.name} 인스턴스 생성됨`);
    }
  };
}

@LogClass
class User {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

new User("Kim"); // 콘솔: [LogClass] User 인스턴스 생성됨
```

```ts
// 메서드 데코레이터: (프로토타입, 메서드 이름, 설명자) 를 받음
function LogMethod(
  _target: object,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const original = descriptor.value as (...args: unknown[]) => unknown;

  // 메서드를 새 함수로 교체
  descriptor.value = function (this: unknown, ...args: unknown[]) {
    console.log(`[LogMethod] ${propertyKey} 호출, 인자:`, args);
    const result = original.apply(this, args);
    console.log(`[LogMethod] ${propertyKey} 반환:`, result);
    return result;
  };
}

class Calculator {
  @LogMethod
  add(a: number, b: number): number {
    return a + b;
  }
}

new Calculator().add(2, 3);
```
- OCP에서 언급했던 new 키워드를 통한 사용 방식보다는 해당 ts방식이 더 좋은 사용 예시라고 합니다.

## 데코레이터 패턴의 장점
- 조립식 확장 가능: 레고 블록 혹은 포장지처럼 기능을 쌓아 올리기 가넝
- 단일 책임 원칙: 각 데코레이터는 한 가지 기능만 담당
- 유지보수성: 기존 코드 건드리지 않고 새 기능 추가