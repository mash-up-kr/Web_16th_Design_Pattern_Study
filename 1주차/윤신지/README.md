![](https://velog.velcdn.com/images/dawnww/post/4b2e1a1a-330c-44d2-bd90-e3f0bb5d72a4/image.png)

_* Mash-UP 16기 웹 팀 내 [디자인패턴 스터디](https://github.com/mash-up-kr/Web_16th_Design_Pattern_Study)에서 진행된 내용으로, [patterns.dev](https://www.patterns.dev/)를 읽고 정리하는 방식으로 진행되었습니다._
_** 정리하다 궁금한 내용(궁금한 내용이 아니더라도 스터디 시간에 얘기하고 싶은)은 ❓ 이모지로 정리해 두었습니다._

> **1주차 자료**
[Signleton Pattern](https://www.patterns.dev/vanilla/singleton-pattern/)
[Module Pattern](https://www.patterns.dev/vanilla/module-pattern/)
[Prototype Pattern](https://www.patterns.dev/vanilla/prototype-pattern/)

---

## Singleton Pattern 	

> 1회에 한하여 인스턴스화가 가능하며 해당 인스턴스에 대한 전역 접근 지점을 제공하는 생성 디자인 패턴 

Singleton 패턴이란 특정 인스턴스가 오직 **하나만 존재**하도록 보장하여 단 하나의 유일한 객체를 만들기 위한 코드 패턴이다.
👉 시스템 전체에서 딱 하나의 인스턴스만 공유해야 하는 상황에 사용 


### 특징
>
✅ 메모리 절약을 위해 사용된다.
✅ 전역 변수를 사용하지 않고 객체를 하나만 생성하도록 한다.
✅ 생성된 객체를 어디서든지 참조할 수 있도록 하여 같은 똑같은 인스턴스를 새로 만들지 않고 기존의 인스턴스를 가져와 활용하는 기법이다. 


#### 💡 동작 원리 
> 1. 인스턴스 존재 여부 확인: 클래스가 호출될 때, 이미 생성된 인스턴스가 있는지 확인한다.
2. 생성 또는 반환:
    - 인스턴스가 없다면?새로 생성하여 내부 변수에 저장하고 반환한다.
    - 인스턴스가 있다면? 새로 만들지 않고 기존에 저장된 인스턴스를 그대로 반환한다. 
3. 전역 접근: 앱 어디서든 이 동일한 인스턴스에 접근할 수 있는 통로를 제공한다. 



### JavaScript에서 싱글톤 패턴을 구현하는 방법
테마를 관리하는 상황이라고 할 때, JS에서 싱글톤 패턴을 구현하는 방법은 5가지가 있다. 

#### 🙌 객체 리터럴
객체 리터럴을 사용하면 자바스크립트 엔진이 시작될 때 단 한 번만 생성된다. 👉 엔진이 로드될 때 즉시 생성되므로 가장 빠르다.
그러나 은닉화가 불가능해서 외부로 노출된다. 
```js
const SettingsManager = {
  theme: "dark",
  setTheme(newTheme) {
    this.theme = newTheme;
  }
};

Object.freeze(SettingsManager); 
export default SettingsManager;
```
싱글톤 패턴을 사용할 때 객체가 생성된 후 누군가 속성을 바꾸거나 삭제하면 시스템 전체에 문제가 생길 수 있기 때문에 `Object.freeze()`를 통해 객체를 얼려버린다. `Object.freeze()`를 사용하면 새로운 속성 추가나 삭제가 불가능하고, 기존 속성 값이 변경 불가하며 프로토타입 변경이 불가하다. 


#### 🙌 클래스 기반 
전통적인 객체지향 방식으로, new 키워드를 사용하더라도 항상 같은 인스턴스를 반환한다. 그러나 JS에서는 클래스를 작성하지 않아도 객체를 만들 수 있기 때문에 오버 엔지니어링으로 볼 수 있다. 
```js
class SettingsManager {
  static instance;

  constructor() {
    if (SettingsManager.instance) {
      return SettingsManager.instance;
    }
    this.theme = "dark";
    SettingsManager.instance = this;
  }

  setTheme(newTheme) {
    this.theme = newTheme;
  }
}

const s1 = new SettingsManager();
const s2 = new SettingsManager();
// s1 === s2 true
```

#### 🙌 클로저 & IIFE
내부 변수를 외부에서 절대 건드릴 수 없도록 한다. 👉 외부에서 내부 변수에 직접 접근할 수 없기 때문에 완벽한 정보 은닉 가능
`getInstance()`를 호출할 때 Lazy Initialization이 가능해 메모리를 효율적으로 사용할 수 있다. 

👉 보안이 중요하거나 객체 생성 비용이 커서 실제 사용 시점까지 생성을 미뤄야 할 떄 사용! 
```js
const SettingsManager = (function () {
  let instance;
  let theme = "dark"; 

  function createInstance() {
    return {
      getTheme: () => theme,
      setTheme: (newTheme) => { theme = newTheme; }
    };
  }

  return {
    getInstance: function () {
      if (!instance) instance = createInstance();
      return instance;
    }
  };
})();

const s1 = SettingsManager.getInstance();
```

#### 🙌 ES 모듈
자바스크립트 모듈 시스템(ESM)의 캐싱 특성을 이용한다. 👉 가장 자바스크립트스러운 최신 방식 
파일 단위 모듈 시스템의 캐싱 기능을 활용한다. 한 번 import 된 모듈은 메모리에 유지된다. 
🚨 테스트 시 싱글톤을 초기화하고 싶을 때 모듈 캐시를 강제로 날려야 한다.
```js
class SettingsManager {
  constructor() {
    this.apiKey = "https://api.myapp.com";
    this.theme = "dark";
  }
  setTheme(newTheme) { this.theme = newTheme; }
}

export default new SettingsManager(); // 👉 클래스가 아닌 인스턴스를 내보냄
```

사용 시 
```js
// main.js
import settings from './SettingsManager.js'; // 어디서 import하든 항상 같은 객체
```


#### 🙌 Proxy 
클래스 외부에서 `new` 연산자의 동작을 가로채서 제어한다. 때문에 관심사의 분리가 완벽하고, 클래스를 수정하지 않고도 싱글톤으로 만들 수 있다. 

```js
class Settings {
  constructor() {
    this.theme = "dark";
  }
}

const instance = new Settings();

const SettingsManager = new Proxy(Settings, {
  construct(target, args) {
    return instance; // 어떤 시도에도 기존 인스턴스만 반환
  }
});

const s1 = new SettingsManager();
const s2 = new SettingsManager();
```

### 정리
#### 💡 장점
**메모리 효율성**: 인스턴스를 매번 `new`로 생성하지 않고 하나만 공유하므로, 메모리 낭비를 줄이고 가비지 컬렉션의 부담을 덜어준다.
**데이터 공유 용이**: 전역 인스턴스이기 때문에, 여러 모듈이나 컴포넌트 간에 상태를 공유하고 전달하기가 매우 쉽다.
**일관성 보장**: 시스템 전체에서 특정 자원에 대해 일관성을 보장할 수 있다.


#### 💡 단점
**테스트의 어려움**: 싱글톤은 상태를 유지한다.. 단위 테스트 시 각 테스트는 독립적이어야 하는데, 싱글톤의 상태가 이전 테스트의 영향을 받아 테스트 결과가 오염될 수 있다.
**강한 결합도**: 앱의 너무 많은 곳에서 싱글톤을 직접 참조하면, 나중에 해당 코드를 수정하거나 교체하기가 매우 까다로워진다.
**유연성 부족**: 무조건 하나만 존재해야 한다는 제약 때문에, 나중에 멀티 인스턴스가 필요한 상황으로 변경하기 어렵다.
**동시성 문제**: (멀티스레드 환경의 경우) 여러 스레드가 동시에 인스턴스를 생성하려고 할 때 자칫 두 개가 생성될 위험이 있어 처리가 복잡해진다. 👉 비동기 로직에서 주의 필요! 


#### 💡 언제 사용할까?
- **전역 상태 관리**: 애플리케이션 전체에서 공유되어야 하는 환경 설정, 사용자 인증 정보를 관리할 떄 사용한다.
- **리소스 공유 제어**: 데이터베이스 연결 풀이나 로깅 서비스처럼 리소스를 과도하게 생성하지 않고 하나만 써야 할 때 사용한다.
- **브라우저 객체 래핑**: window나 document 객체를 직접 다루는 대신, 이를 래핑한 단일 인터페이스가 필요할 때 사용한다. 

</br>

## Module Pattern
> 클로저를 활용해 관련 있는 변수와 함수를 하나의 외부 객체로 묶고, 불필요한 내부 데이터는 숨겨 캡슐화하는 디자인 패턴 

👉 전역 스코프 오염을 막기 위해 데이터를 비공개로 보호하고 필요한 기능만 선택적으로 노출하는 방식 
코드베이스가 커질수록 코드를 잘 쪼개고 유지보수하기 좋게 만들어야 하기 때문에, 코드를 재사용 가능한 단위로 나누는 동시에 내부 데이터를 안전하게 보관하기 위해 사용된다. 

#### 💡 동작 원리
> 1. IIFE: 함수를 선언하자마자 실행하여 독립적인 스코프를 만든다. 이 내부에서 선언된 변수는 외부에서 절대 접근할 수 없다.
2. 클로저 : IIFE가 실행을 마친 뒤에도, 내부에서 반환된 객체의 메서드들이 IIFE 내부의 지역 변수를 계속해서 참조할 수 있게 메모리에 유지시키는 역할을 한다.
3. 공개 API 반환: 외부에서 사용하길 원하는 기능만 객체에 담아 return한다.


### 💡 그렇다면 클로저란 뭘까?
> 함수가 선언될 때, 자신이 **선언된 시점의 스코프에 정의된 변수를 기억**하는 기능 👉 함수를 호출할 때 해당 범위에서 변수를 참조할 수 있게 한다. 

함수가 외부 범위의 변수에 접근할 수 있게 하는 기능으로, 외부에서는 접근할 수 없는 변수와 메소드를 만들 수 있다.


#### 코드로 클로저 알아보기
```js
function createCounter() {
  let count = 0; // 외부 함수의 변수 

  return function() { 
    count++; // 외부 변수에 접근
    return count;
  };
}

const myCounter = createCounter(); 

console.log(myCounter()); // 1
console.log(myCounter()); // 2
// count 변수는 직접 접근할 수 없지만, myCounter 함수를 통해서만 제어된다. 
```
위 코드의 경우에 count는 전역 변수로 만들어야 하지만, 누구나 count를 수정할 수 있어서는 안 된다. 때문에 클로저를 이용하여 현재 상태를 안전하게 기억하고, 외부에는 필요한 기능만 주고 속 내용을 숨겨 캡슐화한다. 


### 💡 그러면 클로저와 모듈은 무슨 관계일까?
클로저는 전용 범위를 만들고 내부 상태와 동작을 비공개로 만들기 위해 모듈의 일부로 사용될 수 있다. 때문에 클로저를 사용하여 모듈의 상태 및 동작에 대한 전용 범위를 생성한다. 
 

### JavaScript에서 모듈 패턴을 구현하는 방법

#### 🙌 클로저를 이용한 노출 모듈
클로저를 통해 변수를 숨기는 방식이 있다. 호출할 때마다 새로운 실행 컨텍스트가 만들어진다. 
```js
const createStudent = () => {
  let name = "Sinji";	// private 변수: 외부에서 직접 접근 불가
  let age = 20;

  const getName = () => name;
  const getAge = () => age;
  const setName = (val) => (name = val);

  // 외부로 노출하고 싶은 것만 객체에 담아 반환
  return {
    getName,
    getAge,
    setName
  };
};

const s1 = createStudent();
const s2 = createStudent(); // s1과 별개의 독립된 인스턴스
console.log(s1.getName()); // "Sinji"
console.log(s1.name);      // undefined 👉 캡슐화 
```

#### 🙌 IIFE 모듈 
정의와 동시에 실행되어 단 하나의 결과값만 반환한다. 클로저를 적용하여 전용 범위를 만들고 IIFE 표현식을 사용하여 모듈을 생성하는 패턴으로, 외부에서의 접은을 막고 이름 충돌을 방지하며 재사용성을 높일 수 있다. 
추가로, IIFE를 이용하면 싱글톤 패턴을 만들 수 있다. `🚨 IIFE을 사용한 모든 모듈 패턴이 싱글톤 패턴인 것은 아님!` 다시 인스턴스를 만들 수 없는 단일 인스턴스를 반환하는 경우에는 싱글톤 패턴이지만, 생성자 함수를 반환하는 경우에는 싱글톤 패턴이 아니다.
```js
const student = (() => {
  let name = "Sinji";
  let age = 20;

  return {
    getName: () => name,
    setName: (val) => (name = val)
  };
})(); 

student.getName() // Sinji
student.name // undefined
```


#### 🙌 ES6 모듈 패턴
JavaScript에서 표준화된 모듈 시스템으로, 파일 단위 모듈화를 위해 등장했다. `import`와 `export`를 사용하며 트리 쉐이킹에 최적화되어 있다.

여러 기능을 하나의 파일에 담고 싶을 때 사용하며, `export` 키워드를 선언 앞에 붙여 사용한다. 

```js
// student.js
let name = "Sinji";

export const getName = () => name; // 개별 내보내기
export default { name, getName };   // 기본 내보내기


// index.js
import { getName } from "./student.js";	// 가져올 때는 중괄호를 사용! 
import student from "./student.js";	// default import의 경우에는 중괄호 사용 X 

console.log(getName()); // "Sinji"
```

그 외에도 `as` 키워드를 사용하여 이름을 변경하거나, `*`을 사용하여 모듈이 내보내는 모든 기능을 한번에 가져올 수 있다. 

</br>

### 모듈 패턴의 Dynamic Import
일반적인 import 구문은 파일의 맨 윗부분에서 사용된다. 이 방식은 정적 로딩으로, 파일 내의 다른 코드들이 실행되기도 전에 해당 모듈을 먼저 로드한다. 만약 특정 조건에서만 특정 모듈을 로드해야 한다면? 

`import` 함수를 사용하여 특정 조건이 충족되었을 때만 모듈을 로드할 수 있다. 
```js
// Promise 방식
import('./module.js').then(module => {
  module.default();
  module.namedExport();
});

// async/await 방식
(async () => {
  const module = await import('./module.js');
  module.default();
})();
```

👉 초기 접속 시 모든 코드를 다운로드 하지 않고, 필요한 기능만 먼저 로드하여 더 좋은 사용자 경험을 제공한다.
👉 필요한 시점에만 모듈을 로드, 파싱, 컴파일하므로 브라우저의 부담이 줄어든다.


#### 🙌 사용자 액션에 따라 로드할 경우
```js
button.addEventListener('click', async () => {
  const moment = await import('moment'); // 클릭 시점에 동적으로 로드
  console.log(moment.default().format('MMMM Do YYYY'));
});
```

#### 🙌 템플릿 리터럴을 활용해 동적으로 불러와야 할 경우
```js
const loadDogImage = async (num) => {
  const res = await import(`../assets/dog${num}.png`);
  document.getElementById('dog-img').src = res.default;
};
```

### 정리
#### 💡 장점
**캡슐화**: private 키워드가 없는 JS에서 데이터를 완벽하게 보호한다.
**전역 오염 방지**: 단일 객체만 전역에 노출하여 이름 충돌을 막는다.
**가독성**: 관련 로직이 한곳에 모여 있어 코드 파악이 쉽다.


#### 💡 단점
**메모리 효율**: 모든 인스턴스가 함수 사본을 가지거나 클로저를 유지해야 한다.
**확장성 제약**: 나중에 외부에서 추가한 메서드는 비공개 변수에 접근할 수 없다.
**테스트 어려움** : private 멤버를 직접 단위 테스트하기 까다롭다.

#### 💡 언제 사용할까?
- 싱글톤(Singleton) 객체가 필요할 때: 애플리케이션 전체에서 단 하나만 존재해야 하는 설정 관리자나 상태 관리 라이브러리를 만들 때 적합하다. 

> ❓ 왜 싱글톤 객체가 필요할 때 모듈 패턴이 적합할까?
IIFE 기반의 모듈 패턴은 정의되자마자 즉시 실행되며 그 결과값을 반환한다. 👉 스크립트가 로드될 때 딱 한 번 실행되므로, 그 과정에서 생성된 객체는 자연스럽게 유일한 인스턴스가 된다. 
또한 모듈 패턴은 클로저를 사용하여 내부 변수를 보호한다. 이 내부 변수들은 메모리의 특정 구역에 딱 한 번 할당되며, 반환된 객체의 메서드들은 항상 동일한 메모리 공간의 변수를 참조한다.

- 라이브러리/프레임워크 제작 시: 전역 네임스페이스를 단 하나(예: $ 또는 _)만 사용하여 다른 라이브러리와의 충돌을 최소화해야 할 때 사용한다. 
- 민감한 데이터 보호: 외부에서 직접 수정하면 안 되는 핵심 로직이나 상태값이 있을 때 캡슐화를 위해 사용한다. 

</br>

## Prototype Pattern
> 코드를 그들의 클래스에 의존시키지 않고 기존 객체들을 복사할 수 있도록 하는 생성 디자인 패턴

동일한 타입의 여러 객체들이 프로퍼티나 메서드를 공유할 때 사용하는 전략이다. 기존에 존재하는 객체를 복사 혹은 참조하여 새로운 객체를 생성하는 것이 핵심이다. 

#### 💡 동작 원리
> 1. 객체 생성: 클래스나 생성자 함수를 통해 객체를 만든다.
2. 속성 검색: 객체의 특정 프로퍼티에 접근하면, 자바스크립트 엔진은 먼저 해당 객체 본인에게 그 프로퍼티가 있는지 확인한다.
3. 체인 탐색: 본인에게 없다면 프로토타입을 타고 상위 객체로 올라가며 검색한다.
4.  최상단 도달: `Object.prototype`까지 올라갔는데도 없다면 `undefined`를 반환한다.


### ES6 클래스로 Prototype 구현하기
```js
class Dog {
  constructor(name) {
    this.name = name;
  }

  bark() {
    return `Woof! 내 이름은 ${this.name}이야!`;
  }
}

const dog1 = new Dog('Daisy');
const dog2 = new Dog('Max');
const dog3 = new Dog('Spot');

console.log(dog1.bark()); // "Woof! 내 이름은 Daisy이야!"
console.log(dog2.bark()); // "Woof! 내 이름은 Max이야!"
```
위 코드에서 `name`은 각 인스턴스마다 고유하게 할당되지만, `bark` 메서드는 자동으로 `Dog.prototyp`에 추가되어 모든 강아지 인스턴스는 하나의 `bark` 함수를 공유한다. 

![](https://velog.velcdn.com/images/dawnww/post/26745620-cda7-4373-bfbb-5a0dab9497ac/image.png)
위 그림처럼 객체들이 같은 프로퍼티를 가져야 하는 경우 Prototype 패턴을 사용할 수 있다. 


```js
// 클래스의 prototype 확인
console.log(Dog.prototype); 
// { constructor: f Dog(), bark: f bark() }

// 인스턴스를 통한 접근 (비표준: __proto__)
console.log(dog1.__proto__); 
// { constructor: f Dog(), bark: f bark() }

// 표준적인 접근 방법 (권장: Object.getPrototypeOf)
console.log(Object.getPrototypeOf(dog1) === Dog.prototype); // true
```


### 프로퍼티 추가하기
프로토타입 패턴은 **객체를 생성한 후에도 기능을 확장할 수 있다.** 
```js
Dog.prototype.play = function() {
  return `${this.name}가 공놀이를 시작합니다!`;
};

console.log(dog1.play()); // "Daisy가 공놀이를 시작합니다!"
console.log(dog3.play()); // "Spot가 공놀이를 시작합니다!"
```
위 코드가 이미 인스턴스가 생성된 후라고 가정했을 때, 프로토타입 객체에 메서드를 추가하면 이미 생성된 모든 인스턴스가 즉시 그 메서드를 사용할 수 있다.

#### 🙌 Prototype Chain
위 기능이 가능한 이유는 Prototype Chain 덕분이다. 자바스크립트는 특정 프로토타입에 접근하려 할 때, 해당 객체에 그 값이 없으면 같은 이름의 프로퍼티를 찾을 때까지 `__proto__`를 타고 부모 프로토타입을 거슬러 올라간다.

```js
class SuperDog extends Dog {
  constructor(name) {
    super(name);
  }

  fly() {
    return `${this.name}가 하늘을 납니다!`;
  }
}

const superDaisy = new SuperDog('Daisy');

console.log(superDaisy.bark()); // (Dog의 prototype에서 찾음) "Woof! 내 이름은 Daisy이야!"
console.log(superDaisy.fly());  // (SuperDog의 prototype에서 찾음) "Daisy가 하늘을 납니다!"
```
위 코드처럼 fly 메서드를 구현했을 때, SuperDog 는 Dog 를 상속했다. 따라서 인스턴스 dog1 은 bark 메서드도 호출 할 수 있다. SuperDog 의 Prototype 객체의 proto 는 Dot.prototype을 가리키고 있다.

![](https://velog.velcdn.com/images/dawnww/post/08e65956-49c7-4858-8b73-6f6d3a6221df/image.png)
체인의 흐름은 위 이미지와 같다. 말로 풀어서 설명할 경우 
> 1. superDaisy.bark() 호출
2. superDaisy 본인에게 bark가 있나? -> No
3. SuperDog.prototype에 있나? -> No
4. Dog.prototype에 있나? -> Yes (실행)

이런 흐름을 통해 실행된다. 


### Object.create
`Object.create` 메서드는 더 직접적으로 프로토타입 패턴을 구현할 수 있다.
```js
const dogRef = {
  bark() {
    return `Woof!`;
  },
};

// dogRef를 프로토타입으로 가지는 새로운 객체 생성
const pet1 = Object.create(dogRef);

console.log(pet1.bark()); // Woof!
console.log(Object.getPrototypeOf(pet1) === dogRef); // true
```

`Object.create`는 인자로 전달된 객체를 프로토타입으로 삼는 새로운 객체를 반환한다. 


### 정리
#### 💡 장점
**메모리 효율성**: 모든 인스턴스가 동일한 메서드를 개별적으로 갖지 않고, 프로토타입 객체에 있는 하나의 메서드를 공유한다. 1,000개의 객체를 만들어도 메서드는 메모리에 딱 하나만 존재한다.
**실시간 확장성**: 이미 생성된 객체들이라도 프로토타입에 새로운 기능을 추가하면 즉시 모든 인스턴스에서 사용할 수 있다. 
**성능**: 복잡한 초기화 과정이 필요한 객체의 경우, 처음부터 새로 만드는 것보다 이미 만들어진 프로토타입을 복제하거나 참조하는 방식이 더 빠를 수 있다.

#### 💡 단점
**디버깅의 복잡성**: 프로퍼티가 객체 본인에게 있는지, 아니면 프로토타입 체인 어딘가에 있는지 파악하기 어려울 때가 있다.
**조회 비용**: 프로토타입 체인이 너무 깊으면, 상위 객체의 프로퍼티를 찾기 위해 체인을 거슬러 올라가는 과정에서 성능 저하가 발생할 수 있다.
**수정의 위험성**: 프로토타입 객체를 수정하면 이를 공유하는 모든 인스턴스에 영향을 미치므로, 의도치 않은 사이드 이펙트가 발생할 수 있다.

#### 💡 언제 사용할까?
- 대량의 객체 생성 시:
동일한 기능을 가진 객체를 수백, 수천 개 생성해야 하는 상황에서 메모리 낭비를 줄이기 위해 반드시 사용한다.
- 공통 메서드가 많을 때:
객체마다 데이터는 다르지만 수행하는 동작은 완벽히 일치할 때 프로토타입에 메서드를 정의하는 것이 좋다. 
- 상속 구조가 필요할 때:
기본 기능을 가진 '부모' 객체를 바탕으로 기능을 추가하거나 변형한 '자식' 객체를 만들고 싶을 때 유용하다.
- 라이브러리/프레임워크 개발 시:
플러그인 구조를 만들거나 사용자가 기능을 확장할 수 있도록 설계할 때 프로토타입 패턴은 유연한 도구가 된다. 
