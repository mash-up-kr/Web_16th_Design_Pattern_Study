_* Mash-UP 16기 웹 팀 내 [디자인패턴 스터디](https://github.com/mash-up-kr/Web_16th_Design_Pattern_Study)에서 진행된 내용으로, [patterns.dev](https://www.patterns.dev/)를 읽고 정리하는 방식으로 진행되었습니다._
_** 정리하다 궁금한 내용(궁금한 내용이 아니더라도 스터디 시간에 얘기하고 싶은)은 ❓ 이모지로 정리해 두었습니다._

> **2주차 자료**
[Factory Pattern](https://www.patterns.dev/vanilla/factory-pattern/)
[Facade Pattern](https://www.patterns.dev/)
[Mixin Pattern](https://www.patterns.dev/vanilla/mixin-pattern/)

---

## Factory Pattern
> 객체 생성 로직을 별도의 클래스나 함수로 분리하여 캡슐화하는 디자인 패턴 

팩로티 패턴을 사용하면 함수를 호출하는 것으로 객체를 만들어낼 수 있다. `new` 키워드를 사용하는 대신 함수 호출의 결과로 객체를 만들 수 있는 것이다. 
👉 객체를 사용하는 쪽(Client)에서 직접 `new` 키워드를 상요해 인스턴스를 생성하지 않고, 팩토리에 요청하여 객체를 전달받는 방식

### 팩토리 패턴의 세 가지 종류

> - **Simple Factory** : 엄밀히 말해 디자인 패턴의 범주보다는 '관용구'에 가까운 방식. 조건문을 통해 상황에 맞는 인스턴스를 반환하는 방식
- **Factory Method Pattern** : 객체를 생성하기 위한 인터페이스를 정의하지만, 어떤 클래스의 인스턴스를 생성할지는 서브클래스가 결정하게 함
- **Abstract Factory Pattern** : 관련성 있는 여러 객체들의 제품군을 생성하기 위한 인터페이스를 제공. 구체적인 클래스에 의존하지 않고 연관된 객체들을 한꺼번에 생성할 때 사용.


patterns.dev에서 설명하고 있는 예제의 경우 심플 팩토리에 가장 가까우므로, 예제 분석과 심플 팩토리에 대해서 설명해보고자 한다.


### 예제와 Simple Factory
```typescript
const createUser = ({ firstName, lastName, email }) => ({
  firstName,
  lastName,
  email,
  fullName() {
    return `${this.firstName} ${this.lastName}`
  },
})
```

위 예제는 `new User(...)` 없이, `createUser({...})`를 호출하는 것만으로 새로운 사용자 객체가 튀어나도록 구현한 팩토리 함수이다. 위처럼 구현할 경우 함수 내부에서 로직을 추가하기 매우 쉽기 때문에, 특정 조건에 따라 객체의 속성을 다르게 부여하거나 초기화 로직을 넣기에 클래스보다 문법적으로 간결하다. 


```typescript
const createObjectFromArray = ([key, value]) => ({
  [key]: value,
})
```
위처럼 사용할 경우, `[key]` 문법을 사용하여 함수에 전달되는 인자에 따라 객체의 키 이름을 동적으로 결정할 수 있다. 즉, 데이터베이스의 colum명이나 사용자의 설정값에 따라 객체의 구조를 실시간으로 바꿔야 할 때 직관적으로 사용할 수 있다.

위 두 개의 예제의 경우 별도의 상속 구조 없이 함수 호출 한 번으로 객체가 즉시 생성되며, 객체를 어떻게 구성할지에 대한 로직이 하나의 함수에 담겨 있기 때문에 팩토리 함수(심플 팩토리)라고 볼 수 있다.

전통적인 디자인 패턴에서는 클래스 기반의 팩토리 메서드와 추상 팩토리를 강조하지만, 현대 JS/React 환경에서는 심플 팩토리를 더 자주 사용한다. 


#### 팩토리 함수 vs 클래스
클래스를 사용할 경우 메서드가 객체마다 생성되지 않고, 프로토타입이라는 공유 공간에 딱 하나만 존재한다.
- 클래스 방식 : user1과 user2가 서로 다른 데이터를 가지고 있어도, `fullName` 메서드는 같은 메모리 주소를 가리킨다.
- 팩토리 함수 방식 : `createUser`를 호출할 때마다 새로움 함수 객체(`fullName`)가 메모리에 계속 생성된다. 👉 만약 사용자가 10000명이면 10000개의 함수 객체가 메모리를 차지하게 된다. 


| 구분 | 팩토리 함수 (Factory Function) | 클래스 (Class) |
| :--- | :--- | :--- |
| **생성 방식** | 함수 호출 (`fn()`) | `new` 키워드 사용 |
| **코드 간결성** | 매우 높음 (화살표 함수 등) | 보통 (생성자, 메서드 정의 필요) |
| **메모리 효율** | 낮음 (메서드가 인스턴스마다 생성) | **높음** (메서드를 프로토타입으로 공유) |
| **적합한 상황** | 작은 객체, 동적인 속성 설정 필요 시 | 대량의 객체 생성, 복잡한 상속 구조 필요 시 |


## Facade Pattern

> 클래스, 인터페이스 및 하위 시스템으로 구성된 복잡한 시스템에 단순화된 인터페이스를 제공하는 구조 디자인 패턴

파사드 패턴을 이용하여 클라이언트는 단일 인터페이스를 통해 시스템과 상호작용할 수 있으며, 시스템 내 개별 구성 요소의 복잡성과 상호 작용을 숨길 수 있다. 
👉 사용하기 복잡한 클래스 라이브러리에 대해 사용하기 편하게 간편한 인터페이스를 구성 가능하다.

Facade는 프랑스어로 **건물의 정면**을 의미한다. 건물의 복잡한 배관, 전기 배선, 내부 구조 등을 알 필요 없이 **문**이라는 인터페이스를 통해 건물에 들어간다고 생각하면 조금 더 이해하기 쉽다! 

### 파사드 패턴의 구조 
![](https://velog.velcdn.com/images/dawnww/post/d7136c1e-6436-4aaa-9d07-e7b2b5a1babe/image.png)
- **Facade** : 서브시스템 기능을 편리하게 사용할 수 있도록 하기 위해 여러 시스템과 상호작용하는 복잡한 로직을 재정리해서 높은 레벨의 인터페이스를 구성한다. Facade의 역할은 서브 시스템의 많은 역할에 대해 '단순한 창구'가 된다.
👉 클라이언트와 서브시스템이 서로 긴밀하게 연결되지 않도록 한다.

- **Additional Facade** : 파사드 클래스는 반드시 한 개만 존재해야 한다는 규칙이 없기 때문에, 연관되지 않은 기능이 있다면 파사드 2세로 분리할 수 있다. 파사드 2세는 다른 파사드에서 사용할 수도 있고 클라이언트에서 직접 접근할 수도 있다.

- **SubSystem** : 수십가지 라이브러리 혹은 클래스들이다. 특정 작업을 수행하고 시스템 기능의 특정 부분을 감당한다. 하위 시스템은 복잡할 수 있으며, 많은 개별 구성 요소와 인터페이스로 구성된다.

- **Client** : 파사드와 상호작용하는 코드이다. 일반적으로 시스템을 사용해야 하며, 그렇지 않으면 하위 시스템과 직접 상호작용해야 하는 코드이다. 


파사드 패턴은 **클래스 구조가 정형화되지 않은 패턴**이다. 단순히 파사드 클래스를 만들어 적절히 기능 집약화만 해 주면 그게 디자인 패턴이 되는 것이다. 

> 일반적인 디자인 패턴들이 설계도에 가깝다면, 파사드 패턴은 포장지에 가깝다. 그냥 사용하기 편하게 깔끔한 껍데기를 하나 씌우는 것.

### JS로 보는 파사드 패턴 예제
집에서 영화를 보기 위해서 전등 조절, 스크린 내리기, 프로젝터 및 사운드 조절하기 등 과정이 필요하다고 가정하자.


#### 서브시스템
시스템의 실제 세부 기능을 담당하는 부품 해당
```javascript
// 전등 제어
class Lights {
  dim(level) { console.log(`전등 밝기를 ${level}%로 조절합니다.`); }
  on() { console.log("전등을 환하게 켭니다."); }
}

// 스크린 제어
class Screen {
  down() { console.log("스크린을 내립니다."); }
  up() { console.log("스크린을 올립니다."); }
}

// 프로젝터 제어
class Projector {
  on() { console.log("프로젝터를 켭니다."); }
  setWideScreen() { console.log("화면 비율을 와이드 스크린으로 설정합니다."); }
  off() { console.log("프로젝터를 끕니다."); }
}

// 음향 시스템 제어
class AudioSystem {
  on() { console.log("오디오 시스템을 켭니다."); }
  setVolume(level) { console.log(`볼륨을 ${level}로 설정합니다.`); }
  off() { console.log("오디오 시스템을 끕니다."); }
}
```

#### 파사드
서브 시스템들을 캡슐화하여 단순한 명령으로 변환
```javascript
class HomeTheaterFacade {
  constructor(lights, screen, projector, audio) {
    this.lights = lights;
    this.screen = screen;
    this.projector = projector;
    this.audio = audio;
  }

  // 복잡한 단계를 하나로 묶은 단순한 메서드
  watchMovie() {
    console.log("--- 영화 감상 준비를 시작합니다 ---");
    this.lights.dim(10);
    this.screen.down();
    this.projector.on();
    this.projector.setWideScreen();
    this.audio.on();
    this.audio.setVolume(25);
    console.log("--- 즐거운 영화 관람 되세요! ---");
  }

  endMovie() {
    console.log("--- 영화가 종료되었습니다. 시스템을 정리합니다 ---");
    this.lights.on();
    this.screen.up();
    this.projector.off();
    this.audio.off();
  }
}
```

#### 클라이언트 코드
파사드를 통해 시스템을 조작하는 사용자 -> 내부가 어떻게 돌아가는지는 신경 쓰지 않음
```javascript
// 클라이언트 코드
// 각 부품들을 생성
const lights = new Lights();
const screen = new Screen();
const projector = new Projector();
const audio = new AudioSystem();

// 파사드(입구)에 부품들을 연결
const homeTheater = new HomeTheaterFacade(lights, screen, projector, audio);

// 복잡한 과정 없이 단 한 줄로 영화 시작!
homeTheater.watchMovie();

// 영화가 끝난 후 정리도 간편하게
homeTheater.endMovie();
```

위 예제는 파사드 패턴에 대해서 간단하게 알기 쉽도록 작성한 예제이고, 일반적으로 프론트엔드에서 파사드 패턴을 가장 많이 사용하는 경우는 API 모듈을 추상화하거나 복잡한 라이브러리를 캡슐화하는 과정이다. 


### API 모듈 추상화 예제
```typescript
import axios from 'axios';

const instance = axios.create({ baseURL: '/api' });

export const UserFacade = {
  async getUserData(userId: string) {
    // 내부적으로 캐싱, 토큰 갱신, 데이터 가공 등을 수행할 수 있음
    const response = await instance.get(`/users/${userId}`);
    return this._transformUser(response.data);
  },

  _transformUser(rawData: any) {
    // 클라이언트에서 쓰기 좋게 데이터 포맷팅
    return {
      name: rawData.user_name,
      email: rawData.user_email,
      joinedAt: new Date(rawData.created_at)
    };
  }
};
```

`axios`나 `fetch`를 컴포넌트에서 직접 사용할 경우 나중에 라이브러리를 변경하는 등의 변경사항이 있을 때 모든 컴포넌트를 수정해야 할 것이다. 그러나 파사드 패턴을 적용하면 서비스 로직을 보호할 수 있다.
또한 컴포넌트는 `UserFacade.getUserData()`만 호출하면 되므로 내부 로직을 알 필요가 없다.


### Refine의 Data Provider
[Refine Data Provider](https://refine.dev/core/docs/data/data-provider/)의 경우 파사드 패턴의 예시라고 볼 수 있다. 

```typescript
const dataProvider = {
  getList: async ({ resource, pagination, filters }) => {...},
  getOne: async ({ resource, id }) => {...},
  create: async ({ resource, variables }) => {...},
  update: async ({ resource, id, variables }) => {...},
  deleteOne: async ({ resource, id }) => {...},
};
```
refine에서 data provider는 위와 같은 구조를 가지고 있지만, 프론트에서는 `const { data } = useList({ resource: "posts" });` 형식으로 사용할 수 있다. 

refine을 기준으로 봤을 때 내부에서는 HTTP 통신, 인증 처리, 에러 핸들링, pagination 변환, filter 변환, 데이터 구조 매핑 등 복잡한 서브시스템이 존재하지만 이를 하나의 인터페이스로 단순화시킴과 동시에 서브시스템과 직접 상호작용할 수 없도록 차단했다.

👉 복잡한 데이터 접근 로직을 숨기고, 단순한 인터페이스를 제공하며 의존성을 추상화에 고정시킬 때 파사드 패턴이 적합하다는 것을 확인할 수 있다.


## Mixin Pattern

> 특정 기능을 포함하는 클래스나 객체를 만들어 다른 클래스에서 이를 혼합하여 사용하는 디자인 패턴
👉 단독으로 사용할 수는 없고 상속 없이 객체나 클래스에 기능을 추가하는 목적으로 사용된다. 

상속과 비슷하다고 볼 수도 있지만, 부모 클래스의 모든 것을 물려받는 게 아니라 필요한 기능만 골라서 내 객체에 붙여넣는 개념이다. 


### TypeScript에서의 Mixins
[TypeScript](https://www.typescriptlang.org/docs/handbook/mixins.html)에서 클래스 계층 구조를 유연하게 가져가기 위해 믹스인 패턴을 공식적으로 권장하는 패턴 중 하나로, 자바스크립트의 단일 상속 제약을 우아하게 해결하는 방법이라고 소개한다.


#### Constructor 타입 정의
타입스크립트에서 믹스인을 만들기 위해서는 어떤 클래스든 받을 수 있는 타입을 필요로 하므로, `Constructor` 타입을 정의한다.

```typescript
// 모든 클래스를 나타내는 제네릭 타입
type Constructor = new (...args: any[]) => {};
```


#### 독립적인 함수 정의
클래스를 인자로 받고, 그 클래스에 기능을 추가하여 새로운 클래스를 내뱉는 `Scale`와 `Alpha`를 정의한다.

```typescript
// Scale Mixin
function Scale<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    _scale = 1;

    setScale(scale: number) {
      this._scale = scale;
    }

    get scale(): number {
      return this._scale;
    }
  };
}

// Alpha Mixin
function Alpha<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    _alpha = 1;

    setAlpha(alpha: number) {
      this._alpha = alpha;
    }

    get alpha(): number {
      return this._alpha;
    }
  };
}
```

#### 기본 클래스에 결합
기본이 되는 `Sprite` 클래스를 만든 후 위에서 만든 기능들을 하나씩 씌워준다. 
```typescript
// 1. 기본 클래스 정의
class Sprite {
  name = "";
  constructor(name: string) {
    this.name = name;
  }
}

// 2. 믹스인 적용 (함수 합성)
// Sprite를 Scale이 감싸고, 그걸 다시 Alpha가 감쌉니다.
const AlphaScalableSprite = Alpha(Scale(Sprite));

// 3. 인스턴스 생성
const player = new AlphaScalableSprite("Hero");

// 4. 결과 확인
player.setScale(2);
player.setAlpha(0.5);
console.log(player.name);  // "Hero" (기본 클래스 속성)
console.log(player.scale); // 2 (Scale 믹스인 속성)
console.log(player.alpha); // 0.5 (Alpha 믹스인 속성)
```


이러한 패턴을 작성할 경우, 세 가지의 장점이 있다.
1. 타입스크립트는 `Alpha(Scale(Sprite))` 과정을 거치면서 생성된 결과물이 `name`, `scale`, `alpha를` 모두 가지고 있다는 것을 알고 있기 때문에 별도의 인터페이스를 수동으로 합칠 필요가 없다.
2. 아래 예제와 같이 특정 조건을 만족하는 클래스에만 믹스인을 적용하고 싶을 때, 제네릭에 제약을 걸 수 있다.
```typescript
// Moveable 인터페이스가 있는 클래스에만 적용 가능
type Moveable = new (...args: any[]) => { setPos: (x: number, y: number) => void };

function Jumpable<TBase extends Moveable>(Base: TBase) {
  return class extends Base {
    jump() { this.setPos(0, 10); } // 부모의 setPos를 안전하게 호출 가능
  };
}
```
3. 상속 계층은 부모-자식 관계로 코드를 경직ㄱ되게 만드는 반면 믹스인은 수평적 확장이 가능하다. 



### Mixin 사용을 중단하세요!
소제목을 조금 강렬하게 쓰긴 했지만....
React 팀은 [Mixins Considered Harmful](https://legacy.reactjs.org/blog/2016/07/13/mixins-considered-harmful.html)이라는 글을 통해 믹스인 사용을 중단할 것을 공식적으로 권고했다.

대표적으로 세 가지 문제를 제시했는데,
> 1. **누가 누구에게 의존하는지 알 수 없다.**
컴포넌트 입장에서는 이 믹스인을 넣는 순간, 내부 코드를 수정하기가 두려워진다. 내가 메서드 이름을 `renderHeader`에서 `drawHeader`로 바꾸면, 믹스인이 소리 소문 없이 깨져버리기 때문이다.
</br>
2. **믹스인은 모든 메서드를 하나의 클래스 네임스페이스에 평면적으로 합친다.**
React는 런타임에 에러를 던지거나, 하나를 무시한다. 만약 3자 라이브러리에서 가져온 믹스인이라면 내가 이름을 바꿀 수도 없다. 결국 두 기능을 동시에 쓸 수 없는 상황에 직면한다.
</br>
3. **믹스인은 처음에는 코드를 줄여주는 것처럼 보이지만, 시간이 흐르면 추적 불가능하다.**
새로 온 개발자가 코드를 볼 때 `this.state.isShowing`이 어디서 정의된 것인지 찾으려면 모든 믹스인 파일을 다 뒤져야 한다. 믹스인이 또 다른 믹스인을 포함하고 있다면 디버깅은 거의 불가능해진다.



React 팀은 이 문제를 해결하기 위해 끊임없이 대안을 제시했고, HOC 이후 Hook을 통해 믹스인의 모든 문제를 해결(명시적이며, 이름 충돌이 없고, 로직을 함수 호출을 통해 자유롭게 조합 가능)할 수 있었다. 

