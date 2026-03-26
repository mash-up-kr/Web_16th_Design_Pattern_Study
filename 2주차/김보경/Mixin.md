# 믹스인 패턴
- 믹스인은 상속 없이 어떤 객체나 클래스에 재사용 가능한 기능을 추가할 수 있는 객체입니다.
- 믹스인은 단독으로 사용할 수는 없고 상속없이 객체나 클래스에 기능을 추가하는 목적으로 사용됩니다.
> 전통적인 프로그래밍 언어에서 믹스인은 서브클래스가 쉽게 상속볻아 기능을 재사용할 수 있는 클래스 라고 합니다.

> 사이트 내용이 너무 없어서 이전에 읽었던 책 내용을 조금 가져와봤습니다.

## 서브클래싱을 먼저 설명해보겠습니다.
- 서브클래싱이란 부모 클래스 객체에서 속성을 상속받아 새로운 객체를 만드는 것을 뜻합니다.
  - 부모 클래스를 확장하는 자식 클래스를 서브클래스라고 합니다.
  - ES2015+에서 도입된 기능을 통해 기존 또는 부모 클래스를 확장활 수도, 부모 클래스의 메서드를 호출할 수도 있게 되었습니다.
- 서브클래스의 메서드는 override된 부모 클래스의 메서드를 호출할 수도 있는데, 이를 **메서드 체이닝** 이라고 부릅니다.
- 마찬가지로 부모 클래스의 생성자를 호출할 수도 있는데, 이를 **생성자 체이닝** 이라고 부릅니다.

### 서브클래싱 사용 예시
- 에러에 대한 서브클래싱은 아주 유우명한 사용 예시입니다.
  ```ts
  class OrderHttpError extends Error {
    private readonly privateResponse: AxiosResponse<ErrorResponse> | undefined;

    constructor(message?: string, response?: AxiosResponse<ErrorResponse>) {
      super(message);
      this.name = "OrderHttpError";
      this.privateResponse = response;
    }

    get response(): AxiosResponse<ErrorResponse> | undefined {
      return this.privateResponse;
    }
  }

  class NetworkError extends Error {
    constructor(message = "") {
      super(message);
      this.name = "NetworkError";
    }
  }

  class UnauthorizedError extends Error {
    constructor(message: string, response?: AxiosResponse<ErrorResponse>) {
      super(message, response);
      this.name = "UnauthorizedError";
    }
  }
  ```
  - 에러를 서브클래싱 함으로써 더욱 명시적으로 사용할 수 있는 내용입니다.


## 자바스크립트에서의 믹스인
- 자바스크립트의 클래스는 부모 클래스를 하나만 가질 수 있지만 여러 클래스의 기능을 섞는 것으로 문제를 해결할 수 있습니다.
  ```js
  // 동적으로 부모 클래스를 받아 확장하는 믹스인 함수 생성
  const MyMixins = (superclass) =>
    class extends superclass {
      moveUp() {
        console.log("move up");
      }

      moveDown() {
        console.log("move down");
      }

      stop() {
        console.log("stop! in the name of love!");
      }
    };

  // ===== 사용처 =====

  // CarAnimator 생성자의 기본 구조
  class CarAnimator {
    moveLeft() {
      console.log("move left");
    }
  }

  // PersonAnimator 생성자의 기본 구조
  class PersonAnimator {
    moveRandomly() {
      /*...*/
    }
  }

  // MyMixins을 사용하여 CarAnimator 확장
  class MyAnimator extends MyMixins(CarAnimator) {}

  // carAnimator의 새 인스턴스 생성
  const myAnimator = new MyAnimator();
  myAnimator.moveLeft();
  myAnimator.moveDown();
  myAnimator.stop();

  // 출력:
  // move left
  // move down
  // stop! in the name of love!
  ```

### 장단점
- 믹스인을 사용하여 함수의 중복을 줄이고 재사용성을 높일 수 있습니다.
- 애플리케이션에서 객체 인스턴스 사이에 공유되는 기능이 있다면 믹스인을 통해 기능을 공유하여 중복을 피하고 고유 기능을 구현하는데 집중할 수 있습니다.
- 단, 믹스인의 단점이 있다면 몇몇의 개발자들은 클래스나 객체의 프로토타입에 기능을 주입하는 것을 나쁜 방법이라고 여깁니다.
  - 프로토타입의 오염과 함수 출처에 대한 불확실성 초래