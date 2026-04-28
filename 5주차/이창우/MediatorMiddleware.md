# Mediator / Middleware

둘 다 뭔가 중간에 껴서 흐름을 제어하는 느낌이라 처음 보면 꽤 헷갈림

Mediator는 객체들 사이의 직접 통신을 끊고, 중앙 조정자가 상호작용을 관리하는 패턴

Middleware는 하나의 요청 / 액션 / 이벤트 흐름 중간에 끼어들어 전 / 후 처리를 담당하는 패턴

즉 둘 다 "중간" 에 있다는 공통점은 있지만,

Mediator는 **객체 간 관계를 조정** 하고,
Middleware는 **처리 흐름을 가로채고 확장** 함

> 둘 다 중간에 있지만, 하나는 관계를 정리하고 하나는 흐름을 확장한다

# 1. Mediator 패턴

## 1-1. 정의

여러 객체가 서로 직접 통신하지 않고, 중간의 Mediator를 통해 상호작용하도록 만드는 패턴

객체끼리 서로를 다 알고 직접 호출하기 시작하면 결합이 너무 강해짐

```tsx
input.onChange()
button.enable()
errorMessage.hide()
summary.update()
```

이런 식으로 여기저기 직접 엮이기 시작하면, 하나 바꾸면 연쇄적으로 수정해야 함

⇒ 그래서 객체들끼리 직접 말하지 말고, 중앙 조정자 하나를 두고 거기로 통해서만 소통하자

## 1-2. 필요

객체 수가 늘어날수록 객체 간 연결선도 같이 늘어남

ex)

폼 input 여러 개
버튼
에러 메시지
미리보기 영역

각자가 서로 상태를 알고 직접 반응하면 구조가 금방 꼬임

```tsx
nameInput -> submitButton
nameInput -> preview
emailInput -> submitButton
emailInput -> errorText
passwordInput -> submitButton
passwordInput -> strengthMeter
```

이런 식이면 각 컴포넌트가 다른 컴포넌트를 너무 많이 알아야 함

Mediator 패턴에서는 input이 자기 변경 사실만 mediator에게 알리고, mediator가 누구를 어떻게 반응시킬지 결정

즉 직접 연결을 줄이고 상호작용 규칙을 한 곳에 모으는 구조

## 1-3. 구조

- **Mediator** : 객체 간 상호작용을 조정하는 중앙 객체
- **Colleague** : 실제 기능을 가진 객체들, 직접 통신하지 않고 mediator를 거침

```tsx
class Mediator {
  notify(sender, event) {}
}

class Component {
  constructor(mediator) {
    this.mediator = mediator;
  }
}
```

핵심은 Colleague끼리 직접 참조하지 않는 것

## 1-4. 동작 원리

1. 각 객체는 mediator를 참조
2. 상태 변화가 생기면 mediator에게 알림
3. mediator가 전체 규칙에 따라 다른 객체들의 동작을 조정

```tsx
class FormMediator {
  constructor(nameInput, emailInput, submitButton) {
    this.nameInput = nameInput;
    this.emailInput = emailInput;
    this.submitButton = submitButton;
  }

  notify(sender) {
    const canSubmit =
      this.nameInput.value.length > 0 &&
      this.emailInput.value.includes('@');

    this.submitButton.setDisabled(!canSubmit);
  }
}
```

```tsx
class Input {
  constructor(mediator) {
    this.mediator = mediator;
    this.value = '';
  }

  setValue(value) {
    this.value = value;
    this.mediator.notify(this);
  }
}
```

`Input`은 submitButton이 뭔지 몰라도 됨

그냥 값이 바뀌었다는 사실만 mediator에 전달

## 1-5. 실제 감각

프론트엔드에서는 복잡한 UI 상태 조합에서 이 느낌이 자주 나옴

다이얼로그 열기 / 닫기 조정
폼 상태와 버튼 활성화
여러 위젯 간 동기화
채팅방 / 이벤트 허브

예를 들어 채팅 앱에서 유저끼리 직접 메시지를 보내는게 아니라

```tsx
user.send('안녕')
→ chatRoomMediator.notify(user, 'message')
→ 다른 유저들에게 전달
```

이렇게 채팅방이 중간 조정자가 되는 구조

## 1-6. 트레이드 오프

장점

- 객체 간 직접 결합 감소
- 상호작용 규칙을 한 곳에 모을 수 있음
- 여러 객체 관계를 중앙에서 통제 가능

단점

- mediator가 너무 많은 책임을 가지면 God 객체가 되기 쉬움
- 상호작용이 많아질수록 mediator 내부 조건문이 비대해질 수 있음
- 객체는 단순해지지만 mediator는 복잡해질 수 있음

⇒ 객체 간 연결 복잡성을 줄이는 대신, 복잡성이 mediator 쪽으로 모일 수 있음

# 2. Middleware 패턴

## 2-1. 정의

하나의 요청 / 액션 / 이벤트가 최종 처리되기 전에, 중간 단계에서 끼어들어 부가 작업을 수행하도록 만드는 패턴

즉 핵심 로직 앞뒤에 로깅, 인증 검사, 에러 처리, 데이터 변환, 비동기 처리 같은 공통 관심사를 체인처럼 붙이는 방식

> 핵심 로직을 직접 수정하지 않고, 처리 흐름 중간에 기능을 끼워 넣는 패턴

## 2-2. 필요

어떤 요청 하나를 처리할 때, 실제 비즈니스 로직 전에 할 일이 자꾸 늘어남

```tsx
checkAuth()
logRequest()
parseBody()
validate()
runHandler()
handleError()
```

이걸 매번 핸들러 안에 직접 쓰면

- 중복 많아짐
- 흐름이 지저분해짐
- 핵심 로직이 묻힘

⇒ 공통 처리는 middleware로 빼고, 실제 핸들러는 자기 일만 하게 하자

## 2-3. 구조

- **Request / Action / Context**
- **Middleware**
- **Next**
- **Final Handler**

각 미들웨어는 현재 흐름을 받고, 필요하면 처리한 뒤 다음 단계로 넘김

```tsx
function middleware(ctx, next) {
  // 전처리
  next();
  // 후처리
}
```

## 2-4. 동작 원리

1. 요청이 들어옴
2. 첫 번째 middleware 실행
3. middleware가 `next()`를 호출하면 다음 middleware로 이동
4. 마지막에 실제 handler 실행
5. 필요하면 되돌아오면서 후처리 수행

```tsx
function logger(ctx, next) {
  console.log('요청 시작');
  next();
  console.log('요청 종료');
}

function auth(ctx, next) {
  if (!ctx.user) {
    throw new Error('인증 필요');
  }
  next();
}
```

```tsx
logger(ctx, () =>
  auth(ctx, () =>
    handler(ctx)
  )
);
```

핵심은 각 middleware가

흐름을 멈출 수도 있고,
수정할 수도 있고,
그냥 통과시킬 수도 있다는 점

## 2-5. 실제 활용

가장 익숙한 예시는 Express

```tsx
app.use(loggerMiddleware);
app.use(authMiddleware);
app.use(errorMiddleware);
```

요청이 들어오면 등록 순서대로 타고 지나가며 처리

Redux도 마찬가지

```tsx
const logger = store => next => action => {
  console.log('dispatch 전', action);
  const result = next(action);
  console.log('dispatch 후', store.getState());
  return result;
};
```

dispatch를 직접 바꾸는게 아니라, dispatch 흐름 중간에 로깅 / 비동기 / 에러 처리를 끼워 넣음

## 2-6. 트레이드 오프

장점

- 공통 관심사 분리
- 체인 구조로 확장 쉬움
- 핵심 로직을 깔끔하게 유지 가능

단점

- 체인이 길어지면 실제 흐름 추적 어려움
- 순서가 중요해짐
- `next()` 호출 누락 같은 실수 발생 가능

⇒ 로직을 분리하는 대신, 파이프라인 흐름을 읽는 부담이 생김

# 3. Mediator vs Middleware

둘 다 중간에 있지만, 해결하는 문제가 다름

## 3-1. Mediator

객체들끼리 너무 많이 직접 연결되어 있을 때
상호작용 규칙을 중앙에서 관리하고 싶을 때
관계 복잡도를 줄이고 싶을 때

즉 **누가 누구와 어떻게 상호작용하느냐** 가 핵심

## 3-2. Middleware

하나의 요청 / 액션 흐름에 공통 처리를 붙이고 싶을 때
로깅, 인증, 변환, 에러 처리 등을 단계적으로 끼워 넣고 싶을 때
파이프라인 확장이 필요할 때

즉 **하나의 흐름이 어떤 단계를 거쳐 처리되느냐** 가 핵심

## 3-3. 한 줄 비교

Mediator는 관계를 조정하는 중앙 허브에 가깝고,
Middleware는 흐름을 통과하며 처리하는 체인에 가깝다

```tsx
// Mediator 느낌
input -> mediator -> button / error / preview

// Middleware 느낌
request -> logger -> auth -> validator -> handler
```

# 4. 같이 보면 좋은 이유

둘 다 "중간 계층을 둬서 복잡성을 직접 드러내지 않는다" 는 공통점이 있음

하지만 복잡성을 다루는 방향이 다름

Mediator는 **객체 간 연결선** 을 줄이고,
Middleware는 **처리 단계의 관심사** 를 분리

그래서 어떤 문제가 보이느냐에 따라 선택이 달라짐

객체끼리 서로 너무 많이 알고 있으면 Mediator 쪽 문제에 가깝고,
요청 처리 앞뒤로 공통 코드가 반복되면 Middleware 쪽 문제에 가깝다

# 5. 공부 중 의문

## 5-1. 둘 다 결국 중간에서 가로채는 거 아닌가?

겉으로 보면 맞음

근데 핵심은 "무엇의 중간인가" 가 다름

Mediator는 **객체 관계의 중간** 이고,
Middleware는 **실행 흐름의 중간** 임

즉 mediator는 네트워크의 허브에 가깝고, middleware는 파이프라인 필터에 가까움

## 5-2. 그럼 둘을 같이 쓸 수도 있나?

가능

예를 들어 채팅 시스템에서

채팅방 자체는 mediator 역할을 하고,
들어오는 메시지 처리 과정의 인증 / 로깅 / 필터링은 middleware 역할을 함

처럼 공존 가능

⇒ 하나는 관계를 정리하고, 하나는 흐름을 정리하기 때문

---

둘 다 "중간에 둔다" 는 점 때문에 비슷해보이지만,

Mediator는 객체들이 직접 얽히는 문제를 푸는 패턴이고,
Middleware는 하나의 처리 흐름에 공통 기능을 확장하는 패턴

즉 비슷한 위치에 있지만, 해결하는 문제 자체가 다르다
