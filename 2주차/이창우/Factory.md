# 팩토리 패턴

GoF 관점에서 팩토리 패턴은 하나의 개념이 아니다.

- Factory Method
- Abstract Factory

- Simple Factory ⇒ GoF 정식 패턴은 아님

> 객체를 사용하는 코드가, 객체 생성 방식까지 책임지는 것이 과연 좋은 설계인가?

```tsx
const button =
  theme === "dark"
    ? new DarkButton()
    : new LightButton();
```

- 사용하는 쪽이 어떤 버튼을 생성해야 하는지 알고 있어야 함
- 조건 분기가 늘어날수록 가독성을 해침
- 교체, 확장 어려움
- 코드-구현체 강한 결합

⇒ 객체 생성 책임을 분리하여, 사용하는 쪽이 구체 클래스에 덜 의존하게 만드는 것

## 프론트엔드 관점에서의 팩토리 패턴

- 프론트 개발자는 자바 개발자처럼 객체 지향 중심 사고를 자주 하진 않음
- 하지만 생성 책임 분리는 생각보다 자주 겪는 문제
- 테마에 따라 다른 UI 구현, 플랫폼별 구현체, 환경에 따른 API 클라이언트 교체, 차트 종류에 따라 다른 렌더러..

### SimpleFactory

```tsx
class PrimaryButton {
  render() {
    console.log("primary button");
  }
}

class SecondaryButton {
  render() {
    console.log("secondary button");
  }
}

function createButton(type) {
  if (type === "primary") return new PrimaryButton();
  if (type === "secondary") return new SecondaryButton();
  throw new Error("unknown button type");
}

const button = createButton("primary");
button.render();
```

- 생성 로직이 한 곳에 모임, 분기 로직이 단순

### GoF Factory Method 패턴

- 팩토리 메소드는 객체를 생성하기 위한 인터페이스를 정의하되, 어떤 클래스의 인스턴스를 만들지는 서브클래스가 결정하도록 하는 패턴
- 상위 구조는 생성 요청 방식만 정하고, 하위 구조가 어떤 구현체를 만들지 선택하는 패턴

⇒ 생성 결정을 상속 구조에 위임

크게 다음과 같은 역할로서 나눔

- Product : 생성될 객체의 공통 인터페이스
- ConcreteProduct : 실제 생성되는 구체 객체
- **Creator : 팩토리 메서드를 선언하는 상위 클래스**
- ConcreteCreator : 어떤 구체를 만들지 결정하는 하위 클래스

### 프론트엔드 예시: 알림 시스템

```tsx
class Notification {
  show(message) {
    throw new Error("must implement");
  }
}

class ToastNotification extends Notification {
  show(message) {
    console.log(`Toast: ${message}`);
  }
}

class BottomSheetNotification extends Notification {
  show(message) {
    console.log(`BottomSheet: ${message}`);
  }
}
```

```tsx
class NotificationManager {
  createNotification() {
    throw new Error("must implement");
  }

  notify(message) {
    const notification = this.createNotification();
    notification.show(message);
  }
}
```

notify는 공통 흐름, 어떤 알림 객체를 만들지 상위 클래스는 모름!

```tsx
class WebNotificationManager extends NotificationManager {
  createNotification() {
    return new ToastNotification();
  }
}

class MobileNotificationManager extends NotificationManager {
  createNotification() {
    return new BottomSheetNotification();
  }
}

const manager = new WebNotificationManager();
manager.notify("저장되었습니다.");
```

- 상위 클래스가 알림 처리 흐름을 전부 담당하고 있음
- 구체 객체 생성만 하위 클래스가 담당함
- 상위 클래스는 구체 타입을 모른 채 Product 인터페이스에 의존함
- 핵심은 공통 흐름 안에서 "생성 지점"만 하위 클래스에 위임한다는 점

### 그냥 오버라이드 아닌가요

- 넵..!
- 디자인 패턴은 문법이 아닌 설계 의도
- 알고리즘 공통 뼈대는 유지하며, 생성되는 객체 종류만 하위 클래스에서 바꾸게 한다

## Abstract Factory

- 서로 관련되거나 의존적인 객체들의 집합을, 구체 클래스를 명시하지 않고 생성할 수 있도록 인터페이스를 제공하는 패턴

> 객체 하나만 만드는 것이 아니라, 서로 어울리는 모델군 전체를 한 세트로 생성하는 패턴

ex) 디자인 시스템 중에서

- LightThemeButton, LightThemeModal, LightThemeToolTip
- DarkThemeButton, DarkThemeModal, DarkThemeToolTip

테마는 일반적으로 같은 계열끼리 일관되게 생성해야함 ⇒ Abstract Factory

크게 다음과 같은 구조

- AbstractFactory : 제품군 생성 메서드들 선언
- ConcreteFactory : 실제 제품군 생성
- AbstractProduct : 각 제품의 공통 인터페이스
- ConcreteProduct : 실제 제품들

```tsx
// 추상 팩토리
class UIFactory {
  createButton() {
    throw new Error("must implement");
  }

  createModal() {
    throw new Error("must implement");
  }
}
```

```tsx
// 구현 팩토리
class LightUIFactory extends UIFactory {
  createButton() {
    return new LightButton();
  }

  createModal() {
    return new LightModal();
  }
}

class DarkUIFactory extends UIFactory {
  createButton() {
    return new DarkButton();
  }

  createModal() {
    return new DarkModal();
  }
}
```

```tsx
class Page {
  constructor(factory) {
    this.button = factory.createButton();
    this.modal = factory.createModal();
  }

  render() {
    this.button.render();
    this.modal.open();
  }
}

const factory = new DarkUIFactory();
const page = new Page(factory);
page.render();
```

- 페이지는 구체 클래스 이름을 모름
- 버튼과 모달이 같은 제품군에서 생성되며 테마 전체를 한 번에 바꿀 수 있다!

## 팩토리 메소드 vs 추상 팩토리

- 팩토리 메소드 : 객체 하나의 생성에 초점, 상속을 통해 하위 클래스에 위임,

⇒ 어떤 객체를 만들지는 서브클래스가 결정한다
플랫폼별 렌더.. 차트별 렌더.. 상황별 핸들러 ..

- 추상 팩토리 : 관련된 객체 집합 생성에 초점, 팩토리 객체 자체를 주입받아 제품군을 바꿈

⇒ 같은 계열의 객체들을 한 세트로 생성한다
테마별 컴포넌트 세트.. 배포 스테이징별 API, Logger, 조합.. 등

## 트레이드 오프

### 팩토리 메소드

- 구조가 길어질 수 있음
- 단순한건데 과한 추상화.. 클래스 수가 많이 늘어남

### 추상 팩토리

- 제품군 추가는 쉬운데, 제품 종류 추가하는 경우 모든 제품군에 추가해야함
- 버튼, 모달에 툴팁 추가하면 모든 팩토리에 툴팁 추가해야함
- 구조 잘못잡으면 추상계층만 많아짐
