# [Factory Pattern](https://patterns-dev-kr.github.io/design-patterns/factory-pattern/)

## [What] Factory Pattern 이란?

> 객체 지향 디자인 패턴 중 하나로, 객체를 생성하는 로직을 별도의 클래스나 메서드에 위임하여 객체 생성의 구체적인 과정을 숨기는 방식입니다.  
> 쉽게 말해, "어떤 객체를 만들지"를 직접 결정하지 않고, 공장(Factory)에 주문만 하면 공장이 알아서 적절한 객체를 만들어 보내주는 방식이라고 이해하면 쉽습니다. _(Gemini)_

> 팩토리 패턴을 사용하면 함수를 호출하는 것으로 객체를 만들어낼 수 있다.  
> 상대적으로 복잡한 객체 혹은 환경이나 설정에 따라 키와 값을 다양하게 설정해야 하는 객체를 만들어야 할 때 유용하게 사용할 수 있다. _(patterns.dev)_

## [Example] 예시로 알아보기

팩토리 패턴 없이 객체를 직접 만들면, 사용자가 입력하는 데이터의 날것의 상태가 그대로 객체에 반영된다.  
만약 기획적으로 "이름은 앞뒤 공백 제거하고 첫 글자만 대문자로", "이메일은 모두 소문자로 통일" 과 같이 정해두었다면 데이터가 통일 되지 않는 문제가 발생한다.

```js
// 유저 A가 만든 데이터
const user1 = {
  name: "  John Doe  ", // 앞뒤 공백
  email: "JOHN@DOE.COM", // 대문자
  phone: "010-1234-5678" // 하이픈 포함
};

// 유저 B가 만든 데이터
const user2 = {
  name: "jane", // 소문자
  email: "jane@example.com",
  phone: "01098765432" // 하이픈 없음
};
```
하지만 유저를 생성하기 위한 함수를 팩토리 패턴으로 만들어두면, 어떤 데이터가 들어오든 내부에서 정규화(Normalization) 과정을 거쳐 표준화된 규격으로 내보낼 수 있다.

```js
const createCleanUser = ({ name, email, phone }) => {
  // 1. 이름: 앞뒤 공백 제거하고 첫 글자만 대문자로
  const cleanName = name.trim().charAt(0).toUpperCase() + name.trim().slice(1).toLowerCase();

  // 2. 이메일: 모두 소문자로 통일
  const cleanEmail = email.trim().toLowerCase();

  // 3. 전화번호: 숫자만 남기기 (하이픈 제거)
  const cleanPhone = phone.replace(/[^0-9]/g, "");

  // 정제된 데이터로 구성된 객체 반환
  return {
    name: cleanName,
    email: cleanEmail,
    phone: cleanPhone,
    getContactInfo() {
      return `${this.name} (${this.email})`;
    }
  };
};

// 이제 사용자는 대충 넣어도 됩니다.
const user1 = createCleanUser({
  name: "  alice  ",
  email: "ALICE@Service.com",
  phone: "010-1111-2222"
});

console.log(user1.name);  // "Alice" (자동 정제)
console.log(user1.phone); // "01011112222" (자동 정제)
```

이러한 방식이 왜 좋냐?

- **데이터 무결성**: 프로그램 어디에서 유저 객체를 만들든, 이 공장을 거치기만 하면 데이터 형식이 항상 일정함을 보장받습니다. DB에 저장할 때 에러가 날 확률이 획기적으로 줄어들죠.

- **유효성 검사(Validation) 추가 가능**: 만약 이메일 형식이 틀렸다면 객체 생성을 거부하고 에러를 던지는 로직을 팩토리 안에 넣을 수도 있습니다.

- **수정의 용이성**: "오늘부터 전화번호에 국가 코드(+82)를 붙이기로 했습니다"라는 지시가 내려오면? 전국의 소스코드를 뒤질 필요 없이 `createCleanUser` 함수 한 곳만 수정하면 됩니다.

즉, **사용자가 던져주는 날것(Raw)의 데이터를 공장(Factory)에서 예쁘게 가공해서 표준 규격품으로 만들어 내보내는 것** 이 팩토리 패턴의 주된 내용이다.

## [How] Factory Pattern은 어떻게 구현하는가?

팩토리 패턴의 핵심은 **"객체 생성의 구체적인 과정을 숨기고, 완성된 객체만 전달하는 것"** 이다.

1. 팩토리 함수 (Factory Function)  
가장 직관적인 방법으로, 함수 내부에서 객체 리터럴을 생성하여 반환하는 방식이다.
```js
const createCleanUser = ({ name, email, phone }) => {
  // 데이터 정제(Normalization) 로직
  const cleanName = name.trim().charAt(0).toUpperCase() + name.trim().slice(1).toLowerCase();
  const cleanEmail = email.trim().toLowerCase();
  const cleanPhone = phone.replace(/[^0-9]/g, "");

  return {
    name: cleanName,
    email: cleanEmail,
    phone: cleanPhone,
    getContactInfo() {
      return `${this.name} (${this.email})`;
    }
  };
};
```

- **특징**: 구현이 매우 쉽고 논리적으로 명확합니다.  
- **단점**: 객체를 만들 때마다 `getContactInfo` 같은 메서드가 메모리에 새로 생성됩니다. (메모리 낭비 발생), 이미 만들어진 객체들은 예전 방식의 함수를 그대로 들고 있어 업데이트가 불가.

2. `Object.create`를 활용한 최적화

위와 같은 단점을 극복하기 위해, JS의 **프로토타입(Prototype)** 을 이용해 **"데이터는 각자 가지되, 메서드는 한곳에 모아두고 공유하자"** 는 전략을 사용한다.

```js
// 1. 모든 유저가 공유할 '메서드 저장소' (메모리에 딱 하나만 존재)
const userMethods = {
  getContactInfo() {
    return `${this.name} (${this.email})`;
  }
};

const createSmartUser = ({ name, email, phone }) => {
  // 2. Object.create로 userMethods를 부모(프로토타입)로 삼는 새 객체 생성
  const user = Object.create(userMethods);

  // 3. 데이터 정제 로직은 그대로 유지 (각 객체만의 고유 데이터)
  user.name = name.trim();
  user.email = email.trim().toLowerCase();
  user.phone = phone.replace(/[^0-9]/g, "");

  return user;
};

const user1 = createSmartUser({ name: "Alice", ... });
const user2 = createSmartUser({ name: "Bob", ... });

console.log(user1.getContactInfo === user2.getContactInfo); // true
```
왜 `Object.create` 방식이 좋은가?

- **메모리 효율성**: 수만 명의 유저 객체를 만들어도 `getContactInfo` 메서드는 메모리에 딱 하나만 존재한다.

- **실시간 업데이트**: 만약 운영 중에 `userMethods.getContactInfo`의 로직을 수정하면, 이미 생성된 모든 유저 객체에 즉시 수정사항이 빈영된다. (프로토타입 체인 덕분)

- **유연한 상속**: 클래스(class) 문법보다 더 가볍고 동적으로 객체 간의 관계를 설정할 수 있다.

## [Why] Factory Pattern은 왜 사용하는가?

객체를 직접 생성(`new` 또는 `{}`)하지 않고 팩토리 패턴을 사용하는 이유는 코드의 유연성과 유지보수성을 극대화하기 위해서다.

1. 객체 생성 로직의 캡슐화
객체를 생성할 때 데이터 정규화나 유효성 검사 같은 복잡한 과정이 필요할 수 있다.  
팩토리 패턴은 이 로직을 함수 내부에 감춘다. 덕분에 객체를 사용하는 쪽에서는 내부의 복잡한 처리 과정을 몰라도 완벽하게 정제된 객체를 받을 수 있다.


2. 낮은 결합도
코드가 특정 클래스 이름에 강하게 묶여 있으면 내부 구조를 바꿀 때마다 호출하는 모든 코드를 수정해야 한다.  
팩토리를 사용하면 사용자는 객체가 "어떻게 만들어지는지"보다 "어떤 기능을 하는지"에만 집중하게 되므로, 내부 구현이 바뀌어도 외부 코드는 영향을 받지 않는다.  


3. 조건에 따른 동적 생성  
사용자의 권한이나 환경 설정에 따라 서로 다른 속성을 가진 객체를 반환해야 할 때가 있다.  
팩토리 내부에 분기 처리 로직을 두면, 외부에서는 조건에 신경 쓸 필요 없이 상황에 맞는 객체를 즉시 공급받을 수 있다.  
