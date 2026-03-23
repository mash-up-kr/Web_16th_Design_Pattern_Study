# 싱글톤 패턴

- 싱글톤은 1회에 한하여 인스턴스화가 가능하며 전역에서 접근 가능한 클래스를 지칭합니다.
- 만들어진 싱글톤 인스턴스는 앱 전역에서 공유되기 때문에 앱의 전역 상태를 관리하기에 적합합니다.

## 장점과 단점

- 인스턴스를 하나만 만들도록 강제하면 꽤 많은 메모리 공간을 절약할 수 있습니다.
- 매번 새로운 인스턴스를 만들어 메모리 공간을 차지하도록 하는 대신, 앱 전체에서 사용 가능한 하나의 인스턴스를 저장하기 위한 메모리를 사용했습니다.
- 하지만 싱글톤은 안티패턴 혹은 자바스크립트에서 하지 말아야 할 것으로 언급되고는 합니다.
  - Java와 C++ 같은 언어들은 JS처럼 객체를 직접적으로 만들어 낼 수 없습니다.
  - 이런 객체지향 프로그래밍 언어에서는 객체를 만들기 위해 클래스를 꼭 작성해야만 하고, 이렇게 만든 객체는 클래스의 인스턴스가 됩니다.
- 사실 자바스크립트에서는 클래스를 작성하지 않아도 객체를 만들 수 있기 때문에 다양한 방법으로 동일한 구현을 할 수 있습니다.

### 싱글톤은 객체 리터럴로 대체 가능함!!
  ```js
  let count = 0;

  const counter = {
    increment() {
      return ++count;
    },
    decrement() {
      return --count;
    }
  };

  Object.freeze(counter);
  export { counter };
  ```
  - 동일한 객체를 참조하도록 객체의 레퍼런스를 넘겨주기만 하면 서로 다른 위치에서도 하나의 객체를 참조하여 사용할 수 있습니다.

### 테스팅
- 싱글톤 패턴으로 구현된 코드는 인스턴스를 매번 생성할 수 없기 때문에 모든 테스트들은 이전 테스트에서 만들어진 전역 인스턴스를 수정할 수 밖에 없습니다.
- 따라서 테스트들이 실행에 순서가 생기게 되면 작은 수정사항이 전체 테스트의 실패로 이어질 수 있습니다. 하나의 테스트가 끝나면 인스턴스의 변경사항들을 초기화 해 주어야 합니다.

### 명확하지 않은 의존
- 다른 모듈로부터 import 될 때, 싱글톤인지 아닌지 분명히 알 수 없습니다.
  ```js
  import Counter from "./counter";

  export default class SuperCounter {
    constructor() {
      this.count = 0;
    }

    increment() {
      Counter.increment();
      return (this.count += 100);
    }

    decrement() {
      Counter.decrement();
      return (this.count -= 100);
    }
  }
  ```
  - 싱글톤 class를 import해서 super 형태로 사용하는 경우 super된 메서드를 호출해서 싱글톤 객체의 값을 수정하게 된다면 의도하지 않은 예외의 상황으로 이어질 수 있습니다.

### 전역 동작
- 싱글톤 인스턴스를 앱의 전체에서 참조할 수 있어야 합니다.
- 전역 스코프에서 전역 변수를 접근할 수 있는 한, 해당 변수는 앱 전체에서 접근할 수 있기 때문에 전역 변수는 반드시 같은 동작을 구현하는 데 사용해야 합니다.
- 만약 전역 변수가 잘못된 판단으로 올바르지 않게 만들어진 경우, 잘못된 값으로 덮어쓰여질 수 있으며, 이 변수를 참조하는 모든 구현들이 모두 예외를 발생시킬 수 있습니다.

### React의 상태 관리
- React에서는 전역 상태 관리를 위해 싱글톤 객체를 만드는 것 대신 상태관리 API, 라이브러리 등을 사용합니다.
- 싱글톤과 유사해보이지만 싱글톤은 인스턴스의 값을 직접 수정할 수 있는 반면, 언급한 리액트 도구들은 readonly 상태를 제공합니다.
- 전역 동작에서의 단점이 모두 해소되는것은 아니지만, 컴포넌트가 직접 상태를 업데이트하게 두는 것이 아닌 개발자가 의도한대로 수정되도록 하고 있는 것 입니다.

<!--  -->
<!--  -->

## 최근 듣고있는 강의 내용을 기반으로 디자인 패턴에 대해서 간단히 정리해볼게요

### GoF의 객체지향 디자인 패턴
- Gangs of Four 디자인 패턴
- 스스로를 사황(gang)으로 칭하는 `Design Patterns: Elements of Reusable Object-Oriened Software`의 저자들이 있습니다.
  - 객체지향계의 사황으로 이해하셔도 좋을거같습니다.
  - 이분들이 정의한 총 23가지 디자인 패턴이 GoF 디자인패턴 입니다.
- 개인적으로 디자인패턴은 이미 용어와 개념 자체가 널리 사용되고 있으므로 암기과목으로 보아도 좋은 영역이라고 생각해요
  - 이미 앞선 선배님들께서 오랜 답습을 통해 정의하고 연구해온 최적화된 방법이 곧 디자인패턴 입니다.
  - 이름을 알고있으면 어떤 매커니즘으로 동작한다는걸 유추하기 쉬워진다? 로 이어지는것 같아요

#### 생성 패턴
- 객체 생성 방식을 유연하게 제어하거나 캡슐화하는 것.
- 인스턴스화 로직을 분리해여 시스템이 특정 클래스에 의존하지 않도록 합니다.
- 대표 패턴
  - 싱글톤패턴
  - 빌더패턴
  - 프로토타입 패턴
  - 추상 팩토리 패턴
  - 팩토리 메서드 패턴

#### 구조 패턴
- 클래스나 객체를 조합하여 더 큰 구조를 형성하거나, 기존 구조를 유연하게 확장하는데 초점을 둔 패턴
- 시스템의 유연성과 재사용성을 높입니다.
- 대표 패턴
  - 데코레이터 패턴
  - 컴포지트 패턴
  - 플라이웨이트 패턴
  - 어댑터 패턴
  - 브릿지 패턴
  - 퍼사드 패턴
  - 프록시 패턴

#### 행동 패턴
- 객체나 클래스 간의 책임 분산, 커뮤니케이션, 알고리즘의 유연한 교체를 위한 패턴
- 상호작용 방식을 정의하며, 유연하고 확장 가능한 구조를 갖게 됩니다.
- 대표 패턴
  - 책임 연쇄
  - 커맨드
  - 인터프리터
  - 반복자
  - 중재자
  - 옵저버
  - 메멘토
  - 상태
  - 전략
  - 템플릿 메서드
  - 방문자

### 싱글톤 패턴 보충

#### 주요 특징
- 싱글톤 패턴은 주로 공통된 자원을 사용하는 DB연결, 로깅, 설정 등을 중앙에서 관리할 때 사용합니다.
- 사용 예시
  - private 생성자를 통해 외부에서 new로 인스턴스 생성을 제한합니다.
  - static 메서드를 통해 인스턴스에 접근할 수 있는 전역 접근점을 제공합니다.
  - 지연 초기화(Lazy Initialization)을 통해 필요한 시점에 인스턴스를 생성합니다.
    - 사용자가 최초에 class를 호출하는 시점에 인스턴스가 생성됨.

#### 장점과 단점
- 장점
  - 단일 인스턴스 재사용으로 인한 메모리 절약
  - 전역 접근이 쉬움
- 단점
  - 전역 상태 유지로 인한 테스트의 어려움
  - 멀티스레드 환경에서는 복잡성이 증가함
    - 멀티스레드 환경에서는 두 개 이상이 존재할 수 있음
    - 데드락이 발생할 수 있는 원인이 됩니다 - 주로 백엔드 환경에서 발생!
  - 클래스의 인스턴스화를 제어하는 패턴이므로 서브클래싱 확장이 어려운 구조입니다.

#### 대표적인 사용 예시
1. Firebase 초기화시 사용되는 패턴
  ```ts
  // lib/firebase.ts
  import { initializeApp, getApps, FirebaseApp } from "firebase/app";
  import { getAuth } from "firebase/auth";
  import { getFirestore } from "firebase/firestore";
  import { getStorage } from "firebase/storage";

  const firebaseConfig = {
    apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
    authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
    projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
    storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
    messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
    appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
  };

  let firebaseApp: FirebaseApp;

  if (!getApps().length) {
    firebaseApp = initializeApp(firebaseConfig);
  } else {
    firebaseApp = getApps()[0];
  }

  export const auth = getAuth(firebaseApp);
  export const db = getFirestore(firebaseApp);
  export const storage = getStorage(firebaseApp);
  ```
  - 한번 config를 배경으로 이니쉬에이팅되어 실행된 앱을 또 다시 실행하지 않도록 처리

2. 클로저(Closure)를 이용한 싱글톤 + 즉시 실행함수 이용
```ts
const Singleton = (function() {
  let instance;  // 비공개 인스턴스 변수

  function createInstance() {
    // 실제 싱글톤 객체 생성
    const object = {
      randomNumber: Math.random(),
      getRandom: function() {
        return this.randomNumber;
      }
    };
    return object;
  }

  return {
    getInstance: function() {
      if (!instance) {
        instance = createInstance();  // 최초 1회만 생성
      }
      return instance;
    }
  };
})();

// 사용 예제
const instance1 = Singleton.getInstance();
const instance2 = Singleton.getInstance();

console.log(instance1 === instance2);      // true (동일 인스턴스)
console.log(instance1.getRandom());        // 0.123456789 (같은 값)
console.log(instance2.getRandom());        // 0.123456789 (같은 값)
```
- class가 아닌 클로저를 통해 구조적으로 두 개 이상의 인스턴스를 만들지 못하도록 사용하는 방법도 존재합니다.

3. 유틸리티 함수를 싱글톤으로 관리하기
- 자주 사용되는 유틸 함수의 경우 싱글톤으로 생성하려 관리하면...
- version을 명시하는 경우도 존재합니다.
  - 싱글톤은 전역에서 공통적으로 사용되기 때문에 해당 인스턴스를 참조하고 있는 모든 소스코드가 영향을 받는 경우가 많습니다.
  - 이런 문제로 인해 의도적으로 싱글톤 인스턴스가 변경되는 경우 version 정보를 넣어서 업데이트를 추적하는 경우도 존재합니다.

4. 그 외...
- Google Analytics 등 같은 써드파티
- Kakao Map Init 등 써드파티 앱은 초기화 1번만 수행하고 그것을 전역에서 사용


#### 각 패턴에서의 싱글톤
1. 생성 패턴으로서의 싱글톤
  - 객체의 생성 과정을 제어하여 오직 하나의 인스턴스만 존재하도록 보장합니다.
    - new 연산자를 통한 생성을 차단하고, 정적 메서드(getInstance)로 인스턴스 접근을 통제합니다.
    - 인스턴스 생성 시점을 지연하거나, 초기화 로직을 캡슐화 할 수 있습니다.

2. 행동 패턴으로서의 싱글톤
  - 전역 상태 관리
    - 싱글톤 인스턴스는 앱 전역에서 공유됩니다.
    - 상태(state)와 행동(Behavior)을 중앙 집중화
  - 다른 객체와의 협력
    - 싱글턴 객체는 다른 객체들이 공통으로 접근하는 행동(메서드)을 제공
      - ex) 로깅, 캐싱
  - GoF의 설명
    - 싱글톤은 인스턴스 생성뿐 아니라, 그 인스턴스를 통해 시스템 전체에 서비스를 제공하는 행동을 포함합니다.