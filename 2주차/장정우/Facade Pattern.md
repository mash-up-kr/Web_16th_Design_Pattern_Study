# Facade Pattern

## [What] Facade Pattern이란?

> "복잡한 시스템을 단순한 인터페이스 뒤에 숨기는 패턴"

> Facade(퍼사드)는 건축 용어로 건물의 **정면(외관)** 을 의미한다.  
> 건물 안의 복잡한 배관, 전기 배선, 구조물은 보이지 않고, 밖에서는 깔끔한 외벽만 보이는 것처럼 — 복잡한 내부 로직을 단순한 인터페이스 하나로 감싸는 것이 Facade Pattern의 핵심이다. _(claude)_

즉, 여러 개의 모듈이나 클래스가 서로 얽혀 있는 복잡한 시스템이 있을 때, 사용자(개발자)가 그 복잡함을 일일이 다루지 않아도 되도록 **하나의 통합된 진입점**을 제공하는 패턴이다.

## [Example] 예시로 알아보기

### Facade 없이 직접 다루는 경우

예를 들어, 프론트엔드에서 "주문하기" 버튼을 눌렀을 때 서버 측에서 처리해야 할 일이 여러 단계라고 해보자.

```js
// 재고 확인
const inventory = new InventoryService();
const isAvailable = inventory.check(productId);

// 결제 처리
const payment = new PaymentService();
const paymentResult = payment.process(cardInfo, amount);

// 배송 등록
const shipping = new ShippingService();
const trackingNumber = shipping.register(address, productId);

// 알림 발송
const notification = new NotificationService();
notification.sendEmail(userEmail, trackingNumber);
notification.sendSMS(userPhone, trackingNumber);
```

주문 로직이 필요한 모든 곳에서 이 4개의 서비스를 **매번 직접 조합**해야 한다.  
순서에 의존성이 생기며 하나의 과정이라도 빠질 경우 에러가 발생한다. 

### Facade를 적용한 경우

```js
class OrderFacade {
  constructor() {
    this.inventory = new InventoryService();
    this.payment = new PaymentService();
    this.shipping = new ShippingService();
    this.notification = new NotificationService();
  }

  async placeOrder({ productId, cardInfo, amount, address, userEmail, userPhone }) {
    // 1. 재고 확인
    const isAvailable = this.inventory.check(productId);
    if (!isAvailable) {
      throw new Error("재고가 부족합니다.");
    }

    // 2. 결제 처리
    const paymentResult = await this.payment.process(cardInfo, amount);
    if (!paymentResult.success) {
      throw new Error("결제에 실패했습니다.");
    }

    // 3. 배송 등록
    const trackingNumber = await this.shipping.register(address, productId);

    // 4. 알림 발송
    await this.notification.sendEmail(userEmail, trackingNumber);
    await this.notification.sendSMS(userPhone, trackingNumber);

    return { trackingNumber, paymentResult };
  }
}
```

이제 외부에서는 이렇게만 호출하면 된다.

```js
const orderFacade = new OrderFacade();

await orderFacade.placeOrder({
  productId: "PROD-001",
  cardInfo: { number: "4242...", expiry: "12/26" },
  amount: 29900,
  address: "서울시 강남구 ...",
  userEmail: "jw@example.com",
  userPhone: "010-1234-5678",
});
```

**4개의 서비스**를 직접 다루던 코드가 **하나의 메서드 호출**로 줄어들었다.  
내부 순서가 바뀌거나 서비스가 추가되어도, 외부 코드는 전혀 수정할 필요가 없다.

### 프론트엔드에서의 Facade 예시

Facade는 백엔드에만 쓰이는 것이 아니다. 프론트엔드에서도 흔하게 사용된다.

```js
// Facade 없이 — fetch를 직접 사용할 때
const response = await fetch("/api/users", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${getToken()}`,
  },
  body: JSON.stringify({ name, email }),
});

if (!response.ok) {
  const error = await response.json();
  throw new Error(error.message);
}

const data = await response.json();
```

```js
// Facade 적용 — API 클라이언트로 감싼 경우
class ApiClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }

  async request(endpoint, options = {}) {
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${getToken()}`,
        ...options.headers,
      },
      ...options,
      body: options.body ? JSON.stringify(options.body) : undefined,
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message);
    }

    return response.json();
  }

  get(endpoint) {
    return this.request(endpoint);
  }

  post(endpoint, body) {
    return this.request(endpoint, { method: "POST", body });
  }

  put(endpoint, body) {
    return this.request(endpoint, { method: "PUT", body });
  }

  delete(endpoint) {
    return this.request(endpoint, { method: "DELETE" });
  }
}

const api = new ApiClient("/api");
```

이제 외부에서는 이렇게만 쓰면 된다.

```js
// 인증 헤더, 에러 핸들링, JSON 파싱 — 전부 Facade 내부에서 처리
const user = await api.post("/users", { name: "장정우", email: "jw@example.com" });
const users = await api.get("/users");
```

## [How] Facade Pattern은 어떻게 구현하는가?

Facade Pattern의 구현은 비교적 단순하다. 핵심은 **복잡한 서브시스템들을 하나의 클래스(또는 함수)로 감싸고, 외부에는 단순화된 메서드만 노출하는 것**이다.

### 기본 구조

```ts
// 복잡한 서브시스템들
class SubSystemA {
  operationA() { /* 복잡한 로직 */ }
}

class SubSystemB {
  operationB() { /* 복잡한 로직 */ }
}

class SubSystemC {
  operationC() { /* 복잡한 로직 */ }
}

// Facade: 서브시스템을 조합하여 단순한 인터페이스 제공
class Facade {
  private a = new SubSystemA();
  private b = new SubSystemB();
  private c = new SubSystemC();

  // 외부에서는 이 메서드만 호출하면 된다
  doEverything() {
    this.a.operationA();
    this.b.operationB();
    this.c.operationC();
  }
}
```

### 함수형 스타일의 Facade

클래스가 아니라 함수로도 충분히 구현 가능하다.

```ts
// 서브시스템 함수들
async function validateInput(data: OrderData) { /* ... */ }
async function processPayment(payment: PaymentInfo) { /* ... */ }
async function createShipment(address: string) { /* ... */ }
async function sendConfirmation(email: string) { /* ... */ }

// Facade 함수
export async function placeOrder(orderData: OrderData) {
  await validateInput(orderData);
  const payment = await processPayment(orderData.payment);
  const shipment = await createShipment(orderData.address);
  await sendConfirmation(orderData.email);

  return { payment, shipment };
}
```

이처럼 Facade는 GoF패턴 중에서도 가장 구현이 직관적인 축에 속한다.  
특별한 문법이나 트릭이 필요 없고, "여러 개를 하나로 묶어서 노출한다"는 아이디어 자체가 구현이다.

## [Why] Facade Pattern은 왜 사용하는가?

Q. 언제 사용하면 좋을까?
- 핵심은 **여러 서브시스템을 조합해야 하는 복잡한 작업을, 사용하는 쪽에서는 단 한 줄로 처리하고 싶을 때.**

Q. 왜 직접 서브시스템을 호출하지 않고 Facade를 쓰는가?

1. **복잡성 은닉**
    - 사용하는 쪽은 내부의 5개, 10개 서비스가 어떤 순서로 호출되는지 알 필요가 없다. Facade가 "이건 내가 처리할게"라고 말하는 것이다.

2. **변경의 영향 범위 최소화**
    - 내부 서브시스템의 API가 변경되더라도, Facade 내부만 수정하면 된다. 외부 코드는 Facade의 인터페이스만 바라보고 있으므로 영향을 받지 않는다.
    - 이는 Factory Pattern의 "수정의 용이성"과 비슷한 맥락이지만, Factory는 **객체 생성**의 캡슐화에 초점이 있고, Facade는 **작업 흐름(워크플로)**의 캡슐화에 초점이 있다.

3. **코드 재사용**
    - 동일한 복잡한 작업을 여러 곳에서 수행해야 할 때, 각 곳에서 서브시스템을 직접 조합하는 대신 Facade 하나를 호출하면 된다. 중복 코드가 사라진다.

4. **테스트 용이성**
    - Facade를 모킹하면 내부의 복잡한 서브시스템 전체를 대체할 수 있어, 통합 테스트와 단위 테스트의 경계를 명확히 나눌 수 있다.

Q. Facade의 단점은 무엇인가?
- **거대한 객체가 될 위험**: Facade에 기능을 계속 추가하다 보면, 모든 것을 아는 거대한 객체가 될 수 있다. Facade는 "조합"만 해야 하며, 비즈니스 로직 자체를 담으면 안 된다.
- **과도한 추상화**: 서브시스템이 1~2개뿐이거나 로직이 단순한 경우, 불필요한 레이어만 추가하는 셈이다. Facade가 오히려 코드를 이해하기 어렵게 만들 수 있다.
- **서브시스템 접근 차단 아님**: Facade는 서브시스템에 대한 직접 접근을 **막지는 않는다**. 필요하다면 여전히 서브시스템을 직접 사용할 수 있으며, 이는 Module Pattern의 캡슐화와 다른 점이다.