# [Prototype Pattern](https://patterns-dev-kr.github.io/design-patterns/prototype-pattern/)

## [What] Prototype Pattern이란?

> 이미 존재하는 객체를 복제(clone)해서 새 객체를 만드는 생성 패턴 _(claude)_

> 동일 타입의 여러 객체들이 프로퍼티를 공유할 때 유용하게 사용한다.   
> 객체들이 같은 프로퍼티를 가져야 하는 경우 유용하게 사용한다. _(patterns.dev)_

## [Question] 상속(Inheritance)과의 다른점
- 상속 (Inheritance): 부모 클래스의 속성과 행위를 자식 클래스가 물려받아 재사용·확장하는 것
- Prototype Pattern: 이미 존재하는 객체를 복사(clone)해서 새 객체를 만드는 것
```ts
// 상속 방식
class Car { ... }
class Benz extends Car { ... }
const myBenz = new Benz();

// 프로토타입 패턴
const baseBenz = new Benz("black", "leather", 19);
// 새 차가 필요하면 기존 객체를 clone
const newBenz = baseBenz.clone();
newBenz.color = "white"; // 일부만 수정
```

## [Patterns.dev] Prototype Pattern 예시로 알아보기

<details>
<summary>Class 와 Prototype에 대한 옛날 얘기</summary>
<div markdown="1">

> JS는 만들어질 당시 Class가 없었다.
>
> ES6 부터 Class가 추가되었지만, 이 또한 타 객체지향 언어에서의 Class와는 다르다.
> Class는 상속이 가능하지만 Prototype은 상속이 불가능하기 때문인데, 즉 JS는 Prototype Chain을 통해 흉내낼 뿐이다.
>
> 그래서 `function foo() {...}` 뭐 이런식으로 정의해도 내부에 prototype이 있으며, `const bar = new foo();` 이렇게 해도 Class 처럼 사용가능하다.

> 또한 설명에서 나온 `instance.__proto__`는 표준이 아니다. (__proto__는 원래 Mozilla가 자체적으로 만든 비표준 기능이었고, 다른 브라우저들이 호환성 때문에 따라 구현)  
> 그러니 혹시 js로 프로토타입 객체에 접근할 일이 있다면, 다음과 같은 방식을 사용하자.
> ```js
> // 읽기
> Object.getPrototypeOf(instance)
> 
> // 쓰기
> Object.setPrototypeOf(instance, prototype)
> ```
</div>
</details>

Dog Class를 통해서 Prototype Pattern에 대해서 알아보자.

```js
class Dog {
  constructor(name) {
    this.name = name
  }

  bark() {
    return `Woof!`
  }
}

const dog1 = new Dog('Daisy')
const dog2 = new Dog('Max')
const dog3 = new Dog('Spot')
```

생성자의 `prototype 프로퍼티` 혹은 생성된 인스턴스의 `__proto__ 프로퍼티`를 통해 Prototype 객체를 확인할 수 있다.
```js
console.log(Dog.prototype)
// constructor: ƒ Dog(name, breed) bark: ƒ bark()

console.log(dog1.__proto__)
// constructor: ƒ Dog(name, breed) bark: ƒ bark()
```

| ![prototype & proto same.png](images/prototype%20%26%20proto%20same.png) |
|:------------------------------------------------------:|
|              동일한 프로토타입 객체를 참조                   |

`dog1`이 Dog 클래스를 상속 받았다면 `Dog.prototype`에 접근해 새로운 매서드를 추가 하는 것도 가능하다.

```js
class Dog {
  constructor(name) {
    this.name = name;
  }

  bark() {
    return `Woof!`;
  }
}

const dog1 = new Dog("Daisy");
const dog2 = new Dog("Max");
const dog3 = new Dog("Spot");

Dog.prototype.play = () => console.log("Playing now!");

dog1.play();
```

이런 식의 Prototype Chain을 통해 부모에 해당 매서드가 없다면 그 부모에 프로토타입 객체에 접근하는 것도 가능하다.

```js
class Dog {
  constructor(name) {
    this.name = name;
  }

  bark() {
    console.log("Woof!");
  }
}

class SuperDog extends Dog {
  constructor(name) {
    super(name);
  }

  fly() {
    console.log(`Flying!`);
  }
}

const dog1 = new SuperDog("Daisy");
dog1.bark();
dog1.fly();
```
| ![prototype chain 2.png](images/prototype%20chain%202.png) |
|:----------------------------------------------------------:|
|                       매서드를 찾기 위한 여정                        |

`Object.create` 메서드는 Prototype으로 쓰일 객체를 인자로 받아 새로운 객체를 만들어낸다.

```js
const dog = {
  bark() {
    return `Woof!`
  },
}

const pet1 = Object.create(dog)
```

`pet1` 자체적으로는 아무런 프로퍼티도 없었지만, dog 객체를 Prototype으로 사용하기 때문에 bark 메서드를 사용할 수 있게 되었다.

## [How] Prototype Pattern은 어떻게 구현하는가?

기존 객체를 복제(clone)하여 새 객체를 생성한다.  
JS에서는 두 가지 방식이 있다.  

### 1. clone 메서드 직접 구현 (원본과 완전히 독립)
```js
const baseCar = {
  brand: 'Benz',
  color: 'black',
  clone() {
    return { ...this, options: JSON.parse(JSON.stringify(this.options)) };
  }
};

const newCar = baseCar.clone();
newCar.color = 'white';
```

### 2. Object.create (프로토타입 체인으로 위임)
```js
const newCar = Object.create(baseCar);
newCar.color = 'white';
```

두 방식의 차이는 원본 변경 시 복제본에 영향이 가느냐의 여부다.  
clone은 독립, Object.create는 연결이 유지된다.

## [Why] Prototype Pattern은 왜 사용하는가?
객체 생성 비용이 클 때, 한 번 만들어둔 객체를 복제하여 비용을 줄이기 위해 사용한다.  

구체적으로는 다음과 같은 상황이다:
- 생성 시 DB 조회, API 호출 등 무거운 초기화가 필요한 객체를 반복 생성해야 할 때
- 속성 대부분이 동일하고 일부만 다른 객체를 여러 개 만들어야 할 때
- 객체의 생성 과정을 외부에 노출하지 않고 복제만으로 제공하고 싶을 때

핵심은 "비용이 큰 생성을 한 번만 하고, 이후에는 복사해서 재사용한다"는 것이다.