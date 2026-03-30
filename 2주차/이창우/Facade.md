# 퍼사드(Facade) 패턴

- 복잡한 하위 시스템을, 단순한 하나의 창구로 감싸서 쉽게 사용하게 만드는 패턴

어떤 기능 하나를 수행하기 위해 여러 객체나 모듈을 순서대로 조합해야하는 경우

ex) 로그인 처리.. 입력값 검증 ⇒ API 요청 ⇒ 토큰 저장 ⇒ 사용자 정보 업데이트 ⇒ 로깅 ⇒ 성공 알림…

```tsx
validateEmail(email);
validatePassword(password);

const token = await authApi.login(email, password);
tokenStorage.save(token);

const user = await userApi.getMe();
authStore.setUser(user);

logger.log("login success");
toast.show("로그인 성공");
router.push("/home");
```

- 사용하는 쪽에서 전부 호출하게 되는 경우..
- 알아야할 객체가 너무 많다! 호출 순서도 매번 알아야함! 내부 구현이 바뀌면 코드 또 수정해야함!

```tsx
await authFacade.login(email, password);
```

⇒ 복잡한 여러 단계의 작업을, 하나의 단순한 인터페이스로서 제공한다

Facade : 정면 외관 == 건물 내부는 복잡하지만 외부는 하나의 입구만 있다

### 구조

- Facade : 고수준 인터페이스
- Subsystem classes : 실제 일을 처리하는 여러 하위 시스템
- 퍼사드는 직접 모든 일을 하는게 아니라, 하위 시스템을 적절히 조합해서 호출하는 역할

### 프론트엔드

- 퍼사드는 프론트엔드에서 자주 사용하는 패턴
- 결제 처리 로직이 있다고 가정

장바구니 유효성 검사, 재고 확인, 결제 요청, 주문 생성, 로그 전송, 라우터 이동.

```tsx
// none facade
cartValidator.validate(cartItems);
await inventoryService.checkStock(cartItems);
const paymentResult = await paymentService.pay(paymentInfo);
await orderService.createOrder(paymentResult);
analytics.track("purchase");
router.push("/order/complete");

// inner facade
class CheckoutFacade {
  constructor(
    private cartValidator,
    private inventoryService,
    private paymentService,
    private orderService,
    private analytics,
    private router
  ) {}

  async checkout(cartItems, paymentInfo) {
    this.cartValidator.validate(cartItems);
    await this.inventoryService.checkStock(cartItems);
    const paymentResult = await this.paymentService.pay(paymentInfo);
    await this.orderService.createOrder(paymentResult);
    this.analytics.track("purchase");
    this.router.push("/order/complete");
  }
}

// client
await checkoutFacade.checkout(cartItems, paymentInfo);
```

- 여러 API 호출과 상태 반영을 하나의 메서드로 감쌀 때..
- API 묶음 서비스, 복잡한 UI 흐름 캡슐화, 서드 파티 라이브러리 감싸기

⇒ 공부하면서 든 의문 : 호출하는 쪽만 깔끔하고, 로직들은 퍼사드 내부로 그대로 이동하는건데 그러면 근본적인 문제 해결이 아니지 않나? 퍼사드가 필요한 경우는 정말 그 내부 로직을 몰라도 되는 라이브러리를 제공하는 정도로 한정되는거 아닐까?

> 퍼사드는 최적화나 모델 개선의 역할이 아니다. 소프트웨어 설계에서는 복잡성이 존재하느냐 뿐 아니라, 그 복잡성이 어디에 퍼져있느냐도 매우 중요하기 때문

퍼사드는 만능 객체가 아니다. 진입점 단순화와 복잡성 숨기기가 그 의도가 되어야한다.

하나의 퍼사드에 로그인, 결제, 검색, 알림.. 다 넣으면 좋은 퍼사드가 아님

하나의 유즈케이스나 하나의 하위 시스템에 대한 명확한 입구. 많이 감싼 퍼사드가 좋은게 아니라, 경계가 명확한 퍼사드가 좋은 것이다.

good

- `AuthFacade`
- `CheckoutFacade`
- `EditorFacade`
- `PaymentFacade`

bad

- `AppFacade`
- `CommonFacade`
- `MainFacade`
- `SystemFacade`

### 퍼사드와 어댑터의 차이

Facade : 복잡한 시스템을 쉽게 사용하게하는 단순한 진입점

Adaptor : 호환 되지 않은 인터페이스 맞춰줌, 인터페이스 변환 목적

지도 SDK 넘 복잡해서 mapFacade.showMaraker() ⇒ Facade
외부 라이브러리 메서드명이 우리 코드와 잘 안맞아 renderMap() ⇒ Adapter

⇒ 퍼사드는 쉽게 쓰게 하는 것, 어댑터는 맞지 않는 걸 맞추는 것!

### 퍼사드 예시 Axios

**Axios** — 가장 교과서적인 사례. `axios.get(url)` 하나로 XMLHttpRequest/fetch/Node http 3개 어댑터 + 인터셉터 체인 + 데이터 변환을 전부 숨김. `lib/axios.js`의 `createInstance()`가 Facade 진입점

```
사용자: axios.get('/users')
          │
          ▼
┌─────────────────────────────────────────┐
│  lib/axios.js                           │  ← 1층: Facade 진입점
│  createInstance() → 함수처럼 쓸 수 있는  │
│  인스턴스 생성                            │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  lib/core/Axios.js                      │  ← 2층: 파이프라인 오케스트레이터
│  request() → 설정 병합 → 인터셉터 체인   │
│  구성 → dispatchRequest 호출             │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  lib/core/dispatchRequest.js            │  ← 3층: 어댑터 위임
│  데이터 변환 → 어댑터 선택 → 요청 실행   │
└────────────────┬────────────────────────┘
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
   ┌────────┐ ┌───────┐ ┌───────┐         ← 4층: 실제 구현체
   │ xhr.js │ │fetch.js│ │http.js│
   │브라우저 │ │브라우저 │ │Node.js│
   │레거시   │ │모던    │ │서버   │
   └────────┘ └───────┘ └───────┘
```

**Webpack** — `webpack(config)` 한 줄 뒤에서 30개+ 플러그인 자동 등록, 모듈 resolve, 파싱, 코드 생성이 일어남. `Compiler`가 Tapable 훅으로 확장점을 열어두면서도 사용자에겐 `run()` 하나만 노출.

**Next.js `<Image />`** — 컴포넌트 레벨 Facade. srcSet 생성, lazy loading, blur placeholder, CDN 포맷 최적화를 `<Image src="..." />` 하나로 끝냄.

**NestJS** — `NestFactory.create(AppModule)` 한 줄로 모듈 스캐닝, DI 컨테이너 구축, 라우터 설정, 미들웨어 체인까지 전부 처리.
