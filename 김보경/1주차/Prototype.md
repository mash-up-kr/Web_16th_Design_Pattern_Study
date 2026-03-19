# 프로토타입 패턴

## 개념
- 프로토타입 패턴은 동일 타입의 여러 객체들이 프로퍼티를 공유할 때 유용하게 사용합니다.
- 프로토타입은 자바스크립트 객체의 기본 속성이므로 Prototype 체인을 활용할 수 있습니다.
  - 어떤 인스턴스던 `__proto__`의 값은 Prototype 객체를 가리킵니다. 객체에 없는 프로퍼티에 접근하려 하는 경우 JS는 해당 프로퍼티가 나타날때 까지 prototype chain을 거슬러 올라갑니다.
  > `__proto__`는 비 표준입니다. `Object.getPrototypeOf`는 표준이니까 이를 널리 알립시다!!


## 활용
- 프로토타입 패턴은 객체들이 같은 프로퍼티를 가져야 하는 경우 유용하게 쓰일 수 있습니다.
  - 중복된 프로퍼티들이 존재하는 객체를 매번 생성하기보다, 프로토타입에 프로퍼티를 추가하면 모든 인스턴스들이 프로토타입 객체를 활용할 수 있습니다.
  - 모든 인스턴스들이 Prototype에 접근 가능하기 때문에 인스턴스를 만든 뒤에도 Prototype에 프로퍼티를 추가할 수 있습니다.

## 정리
- 프로토타입 패턴은 어떤 객체가 다른 객체의 프로퍼티를 상속받을 수 있도록 해줍니다. 프로토타입 체인을 통해 해당 객체에 프로퍼티가 직접 선언되어 있지 않아도 되무로 메서드 중복을 줄일 수 있고 이는 메모리 절약으로 이어집니다.

## 프로토타입 패턴 보충

- "프로토타입 패턴은 쿠키 성형기다."
- 원본 인터페이스에 의존하지 않고 복사할 수 있도록 하는 패턴 입니다.
  1. 복사본을 만들 때 원형(프로토타입)을 가져옵니다.
  2. 복사본에 필요한 요소만 추가하거나 수정합니다.
- 즉, 원본은 추상화된 프로토타입을 가지지만 체인의 하위 요소들은 경우에 따라 다르게 구성되는 경우
  ```ts
  // 프로토타입 인터페이스
  interface 쿠키성형기 extends Cloneable {
    복제(): 쿠키성형기;
    굽기(): void;
  }

  // 구체적인 프로토타입
  class 별모양쿠키 implements 쿠키성형기 {
    private 맛: string = "바닐라";

    constructor(public 크기: number) {}

    복제(): 쿠키성형기 {
      const 복제본 = new 별모양쿠키(this.크기);
      복제본.맛 = this.맛; // 모든 속성 복사
      return 복제본;
    }

    굽기() {
      console.log(`${this.크기}cm ${this.맛} 맛 별모양 쿠키 굽는 중... '+'`);
    }
  }

  // 사용 예시
  const 원본별쿠키 = new 별모양쿠키(5);
  원본별쿠키.굽기(); // "5cm 바닐라 맛 별모양 쿠키 굽는 중... '+'

  // 복제로 쿠키 추가 생성
  const 쿠키1 = 원본별쿠키.복제();
  const 쿠키2 = 원본별쿠키.복제();

  쿠키2.크기 = 3; // 일부 속성 변경 가능
  쿠키2.굽기(); // "3cm 바닐라 맛 별모양 쿠키 굽는 중... '+'
  ```
  - 소금빵이라는 프로토타입으로 초코, 명란, 치즈, 올리브, 소세지 등 다양한 토핑이나 맛으로 변형하는 너낌

- 프로토타입 패턴을 prototype을 이용하여 구현한 예시
  ```ts
  function Car(make, model) {
    this.make = make;
    this.model = model;
  }

  // 프로토타입에 메서드 추가
  Car.prototype.start = function() {
    return `${this.make} ${this.model} engine started!`;
  };

  Car.prototype.stop = function() {
    return `${this.make} ${this.model} engine stopped!`;
  };

  // 새 객체 생성
  const myCar = new Car('Hyundai', 'Sonata');
  console.log(myCar.start()); // "Hyundai Sonata engine started!"
  ```
  - class 문법이 존재하지 않는 시절 this 실행컨텍스트를 사용해서 구현한 예시 코드 입니다.

- 커스텀 모달 훅과 모달 액션을 다루는 프로토타입 패턴
  - 최근의 ts + React에서는 특정 인터페이스(modal)을 기반으로 prototype을 생성하고 사용처에 알맞게 커스텀하는 방식을 이야기하는듯 합니다.
  - [예?시](https://github.com/toss/overlay-kit/blob/main/packages/src/event.ts)
