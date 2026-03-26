## Facade 패턴이란?
- 복잡한 시스템에 **단순화된 인터페이스**를 제공하는 구조 패턴
- 클라이언트와 서브시스템 사이의 의존성을 줄여 코드를 더 모듈화하고 이해하기 쉽게 만든다

## 구조 (3가지 핵심 요소)

```
Client → Facade → Subsystem A
                → Subsystem B
                → Subsystem C
```

- **Facade**: 복잡한 서브시스템에 대한 단순화된 인터페이스. 클라이언트 요청을 적절한 서브시스템으로 위임
- **Subsystem**: 실제 기능을 수행하는 클래스들. Facade를 통해서만 클라이언트에 노출됨
- **Client**: Facade를 통해서만 시스템과 상호작용. 내부 복잡성을 알 필요 없음

## Facade 패턴 구현

### e-커머스 주문 처리 예시
- 주문을 처리하려면 재고 확인, 결제 처리, 알림 발송 등 여러 서브시스템이 필요하다
- Facade 없이는 클라이언트가 이 모든 시스템을 직접 다뤄야 한다

```javascript
// 서브시스템들
class Inventory {
  checkStock(productId) {
    console.log(`상품 ${productId} 재고 확인`);
    return true;
  }
  updateStock(productId, quantity) {
    console.log(`상품 ${productId} 재고 ${quantity}개 변경`);
  }
}

class Payment {
  processPayment(amount) {
    console.log(`${amount}원 결제 처리`);
    return true;
  }
}

class Notification {
  sendConfirmation(orderId) {
    console.log(`주문 ${orderId} 확인 알림 발송`);
  }
}

// Facade — 모든 서브시스템을 하나의 인터페이스로 통합
class OrderFacade {
  constructor() {
    this.inventory = new Inventory();
    this.payment = new Payment();
    this.notification = new Notification();
  }

  placeOrder(productId, quantity, amount) {
    if (!this.inventory.checkStock(productId)) {
      console.log('재고 부족');
      return;
    }
    if (!this.payment.processPayment(amount)) {
      console.log('결제 실패');
      return;
    }
    this.inventory.updateStock(productId, -quantity);
    this.notification.sendConfirmation(productId);
    console.log('주문 완료');
  }
}

// 클라이언트 — 한 줄로 주문 처리
const order = new OrderFacade();
order.placeOrder(1, 1, 50000);
// 상품 1 재고 확인
// 50000원 결제 처리
// 상품 1 재고 -1개 변경
// 주문 1 확인 알림 발송
// 주문 완료
```

### 프론트엔드 예시: 모니터링 Facade

```javascript
// 서브시스템들
const analytics = { track(event, data) { /* GA 호출 */ } };
const logger = { info(msg) { /* 로그 서버 전송 */ } };
const errorReporter = { capture(err) { /* Sentry 전송 */ } };

// Facade — 하나의 인터페이스로 통합
const monitoring = {
  trackClick(buttonName) {
    analytics.track('click', { button: buttonName });
    logger.info(`Button clicked: ${buttonName}`);
  },
  reportError(error) {
    errorReporter.capture(error);
    logger.info(`Error: ${error.message}`);
    analytics.track('error', { message: error.message });
  },
};

// 사용하는 쪽은 analytics, logger, errorReporter를 몰라도 됨
monitoring.trackClick('구매하기');
monitoring.reportError(new Error('결제 실패'));
```

## 장점
- 복잡한 서브시스템을 단순한 인터페이스로 감춰서 사용이 쉬워진다
- 클라이언트와 서브시스템 간 **느슨한 결합** — 서브시스템 내부가 바뀌어도 Facade만 수정하면 됨
- 여러 서브시스템을 조합하는 로직이 Facade에 모여 있어 유지보수가 편하다

## 단점
- 설계가 부실하면 모든 기능을 담은 **"God Object"**가 될 수 있다
- 서브시스템의 세밀한 기능이 필요할 때 Facade가 오히려 제약이 된다
- 이미 단순한 시스템에 적용하면 불필요한 추상화 계층만 추가된다

## 언제 사용하고, 언제 피해야 하는가

### 사용하기 좋은 경우
| 상황 | 예시 |
|---|---|
| 여러 서브시스템의 조합이 필요할 때 | 주문 처리: 재고 + 결제 + 알림 |
| 클라이언트와 서브시스템의 결합을 줄이고 싶을 때 | 외부 API 여러 개를 하나로 통합 |
| 레거시 코드에 현대적 인터페이스를 씌울 때 | 오래된 시스템 위에 새 API 레이어 |

### 피해야 하는 경우
| 상황 | 이유 |
|---|---|
| 이미 단순한 시스템 | 불필요한 추상화로 오히려 복잡해짐 |
| 서브시스템의 모든 기능에 직접 접근해야 할 때 | Facade가 기능을 제한함 |
| 서브시스템이 1~2개뿐일 때 | Facade를 만들 이유가 없음 |

## 참고자료
- https://www.geeksforgeeks.org/python/facade-method-python-design-patterns/