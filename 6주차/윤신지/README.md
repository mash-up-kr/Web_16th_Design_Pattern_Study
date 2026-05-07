_* Mash-UP 16기 웹 팀 내 [디자인패턴 스터디](https://github.com/mash-up-kr/Web_16th_Design_Pattern_Study)에서 진행된 내용으로, [patterns.dev](https://www.patterns.dev/)를 읽고 정리하는 방식으로 진행되었습니다._
_** 정리하다 궁금한 내용(궁금한 내용이 아니더라도 스터디 시간에 얘기하고 싶은)은 ❓ 이모지로 정리해 두었습니다._

> **5주차 자료**
[Flyweight Pattern](https://www.patterns.dev/vanilla/flyweight-pattern/)
[Proxy Pattern](https://www.patterns.dev/vanilla/proxy-pattern/)
[Presentational/Container Pattern](https://www.patterns.dev/react/presentational-container-pattern/)

---

## Flyweight Pattern
> 재사용 가능한 객체를 공유시켜 메모리 사용량을 최소화하는 구조 패턴
간단히 말해서 캐시 개념을 코드로 패턴화 한 것으로 볼 수 있다.


### Flyweight 패턴의 유래
플라이웨이트 패턴은 복싱의 체급 중 하나인 '플라이급(가장 가벼운 체급)'에서 유래했다. 이름 그대로 객체를 가볍게 만들어 메모리 부담을 줄이는 것이다.
GoF에서 처음 정립되었으며, 당시의 하드웨어는 메모리가 많이 부족했기 때문에 초창기 워드 프로세서를 개발하던 엔지니어들은 문서 안의 모든 글자 하나하나를 폰트, 크기, 색상 정보를 담은 독립적인 '객체'로 만들고자 했다. 이때 10만 자의 글자가 있는 문서의 경우 10만 개의 객체를 생성해야 하는 문제가 있었다.
👉 수만 개의 'A' 글자가 문서에 존재하지만, 'A'라는 글자의 고유한 형태(폰트, 모양)는 모두 똑같다. 글자의 위치(X, Y 좌표)만 다를 뿐이다!! 라는 생각에서 시작되었다.
'A'라는 글자의 고유 정보를 메모리에 단 하나만 생성하여 캐싱하고, 문서 곳곳에 있는 10만 개의 'A'는 이 단 하나의 객체를 참하도록 구조를 바꾸는 것이 플라이웨이트 패턴의 탄생이다. 


### 핵심 동작 원리
| 구분 | Intrinsic State (본질적/내부 상태) | Extrinsic State (부가적/외부 상태) |
| :--- | :--- | :--- |
| **의미** | 객체 자체가 가지고 있는 고유한 성질 | 상황이나 문맥에 따라 변화하는 성질 |
| **특징** | 어떤 상황에서도 변하지 않음 (Immutable) | 언제든 변할 수 있음 (Mutable) |
| **공유 여부** | 여러 객체에서 공유됨 (캐싱 대상) | 공유되지 않음 (클라이언트가 관리) |

플라이웨이트 패턴의 핵심 로직은 Intinsic 상태만을 가지는 순수한 객체를 팩토리를 통해 단 하나만 생성하고 공유하는 것이다. Extrinsic 상태는 객체 외부에 두고, 필요할 때마다 Flyweight 객체에 주입해서 사용한다.

예를 들어서 똑같은 이미지를 사용하는 마커 1,000개를 만들면 메모리에는 1,000개의 이미지 데이터가 올라간다. 하지만 플라이웨이트 패턴을 사용할 경우 이미지 데이터가 하나만 존재하고, 1,000개의 좌표 데이터는 이미지의 메모리 주소만 가리킨다. 

#### 동작의 3대 요소
> -  Flyweight Factory (관리자): 기존 객체가 있는지 확인하고, 없으면 새로 만들어 저장소에 보관한다. 객체의 생명주기를 제어하는 관제탑 역할을 한다.
- Flyweight (공유 객체): 모든 인스턴스가 공통으로 가지는 본질적 상태(Intrinsic State)만 들고 있는 가벼운 객체
- Context/Client (사용자): 각 상황마다 달라지는 부가적 상태(Extrinsic State)를 관리하며, 필요할 때 공유 객체에 이 데이터를 전달하여 동작을 완성시킨다. 


### 예제
도서관에 책이 수만 권 있을 때, 동일한 종류의 책이 여러 권 중복해서 존재하는 경우를 예제로 들 수 있다.

만약 Flyweight 패턴이 적용되지 않은 경우 아래와 같이 구현할 수 있다ㅏ.
```typescript
class HeavyBook {
  constructor(
    public title: string,
    public author: string,
    public isbn: string,
    public availability: boolean,
    public sales: number
  ) {}
}

const heavyLibrary: HeavyBook[] = [];

// 해리포터를 10,000권 입고한다고 가정
for (let i = 0; i < 10000; i++) {
  // 매번 똑같은 문자열(title, author, isbn)을 가진 객체가 10,000개 생성
  heavyLibrary.push(new HeavyBook("Harry Potter", "JK Rowling", "AB123", true, 0));
}
```

만약 위와 같이 구현될 경우, 문자열 데이터가 메모리 힙에 10,000번 할당되는 방식으로 저장될 것이다.

이를 Flyweight 패턴을 적용하여 구현할 경우
```typescript
// Intrinsic State: 절대 변하지 않고 공유될 정보
interface IBookInfo {
  readonly title: string;
  readonly author: string;
  readonly isbn: string;
}

// Flyweight 객체: 생성 후 상태가 변경되지 않도록 처리
class BookFlyweight implements IBookInfo {
  constructor(
    public readonly title: string,
    public readonly author: string,
    public readonly isbn: string
  ) {}
}

// Flyweight 팩토리: 동일한 ISBN을 가진 책의 객체는 단 하나만 유지
class BookFactory {
  // 캐시 메모리 역할을 하는 Map (ISBN -> Flyweight)
  private static booksCache = new Map<string, BookFlyweight>();

  public static createBook(title: string, author: string, isbn: string): BookFlyweight {
    if (!this.booksCache.has(isbn)) {
      // 캐시에 없으면 새로 생성하여 캐싱
      this.booksCache.set(isbn, new BookFlyweight(title, author, isbn));
      console.log(`[Factory] 새로운 책 정보 객체 생성: ${title}`);
    }
    // 이미 있다면 기존에 생성된 객체의 메모리 참조를 반환
    return this.booksCache.get(isbn)!;
  }

  public static getCacheSize(): number {
    return this.booksCache.size;
  }
}

// Extrinsic State와 조합된 클라이언트 데이터
// 객체의 참조(bookInfo)를 저장하여 메모리를 최소화
class BookRecord {
  constructor(
    public bookInfo: BookFlyweight, // 메모리 주소만 참조!
    public availability: boolean,
    public sales: number,
    public id: string // 각 책의 고유 바코드
  ) {}
}

const bookList: BookRecord[] = [];

const addBook = (title: string, author: string, isbn: string, availability: boolean, sales: number, id: string) => {
  // 팩토리를 통해 공통 객체를 얻음
  const sharedBookInfo = BookFactory.createBook(title, author, isbn);
  
  // 개별 상태와 공통 객체 참조를 묶어서 저장
  const record = new BookRecord(sharedBookInfo, availability, sales, id);
  bookList.push(record);
};

// ===============================

addBook("Harry Potter", "JK Rowling", "AB123", false, 100, "ID-001");
addBook("Harry Potter", "JK Rowling", "AB123", true, 50, "ID-002");
addBook("To Kill a Mockingbird", "Harper Lee", "CD345", true, 10, "ID-003");
addBook("To Kill a Mockingbird", "Harper Lee", "CD345", false, 20, "ID-004");
addBook("The Great Gatsby", "F. Scott Fitzgerald", "EF567", false, 20, "ID-005");

console.log(`총 도서 대여 기록 수: ${bookList.length}`); // 5
console.log(`메모리에 생성된 실제 책 객체 수: ${BookFactory.getCacheSize()}`); // 3
```
위와 같은 구조를 통해 책의 대여 기록이 100만개로 늘어나더라도, `BookFlyweitgh` 객체는 세상에 존재하는 책의 종류만큼만 생성된다. 


### 실전 예제 - 마커
지도 마커를 한 번도 다루어보지 못했는데, 마커가 여러 개 있을 때 어떻게 최적화 할 것인지가 자주 언급되는 문제인 것 같아서 + 흥미로운 예제인 것 같아서 가져와보았어요! 

```typescript
// Intrinsic State: 마커의 시각적 형태 (공유됨)
interface MarkerStyleFlyweight {
  iconImage: HTMLImageElement; // 메모리를 많이 차지하는 이미지 객체
  shadowColor: string;
  render(context: CanvasRenderingContext2D, x: number, y: number): void;
}

// 공유될 마커 스타일 구체 클래스
class HospitalMarkerStyle implements MarkerStyleFlyweight {
  iconImage: HTMLImageElement;
  shadowColor = 'rgba(255, 0, 0, 0.5)';

  constructor() {
    this.iconImage = new Image();
    this.iconImage.src = '/assets/hospital-icon.png';
  }

  // 외부 상태인 x, y를 매개변수로 받아서 처리!!! -> Flyweight의 핵심
  render(context: CanvasRenderingContext2D, x: number, y: number): void {
    context.shadowColor = this.shadowColor;
    context.drawImage(this.iconImage, x, y, 32, 32);
  }
}

class SchoolMarkerStyle implements MarkerStyleFlyweight {
  // ... School 전용 이미지와 스타일 설정
  iconImage = new Image();
  shadowColor = 'rgba(0, 255, 0, 0.5)';
  render(context: CanvasRenderingContext2D, x: number, y: number) { /* 렌더링 로직 */ }
}

// Flyweight Factory: 마커 스타일을 카테고리별로 하나씩만 생성
class MarkerStyleFactory {
  private static styles: Record<string, MarkerStyleFlyweight> = {};

  public static getStyle(type: 'hospital' | 'school'): MarkerStyleFlyweight {
    if (!this.styles[type]) {
      if (type === 'hospital') this.styles[type] = new HospitalMarkerStyle();
      if (type === 'school') this.styles[type] = new SchoolMarkerStyle();
    }
    return this.styles[type];
  }
}

// Extrinsic State: 마커가 그려질 좌표계 데이터 (클라이언트에서 관리)
interface MapMarkerData {
  id: string;
  type: 'hospital' | 'school';
  lat: number;  
  lng: number;  
  x: number;    
  y: number;    
}

// 렌더링 엔진 (Client)
class CanvasMapRenderer {
  constructor(private context: CanvasRenderingContext2D) {}

  public renderMarkers(markers: MapMarkerData[]) {
    // 5,000개의 마커를 순회하지만 메모리에 존재하는 'Image' 객체는 단 2개뿐!
    markers.forEach(marker => {
      // 팩토리에서 공유 객체를 가져옴
      const style = MarkerStyleFactory.getStyle(marker.type);
      
      // 공유 객체에게 고유 상태(좌표)를 넘겨주며 렌더링을 위임
      style.render(this.context, marker.x, marker.y);
    });
  }
}
```


### 프론트엔드에서의 Flyweight
#### 프로토타입 상속과 플라이웨이트
JavaScript의 `prototype` 자체가 이미 언어 레벨에서 구현된 플라이웨이트이다. 클래스의 모든 메서드는 인스턴스가 생성될 때마다 복제되지 않고, `prototype` 객체 하나에 캐싱되어 모든 인스턴스가 이를 참조한다. (함수라는 '상태'를 공유하는 셈!!)


#### React와 Synthetic Event Pooling
React 16까지는 `SyntheticEvent` 시스템에 Object Pooling이 하드코딩 되어 있었다. 그러나 유저가 마우스를 움직일 때마다 수백 개의 이벤트 객체가 생성되고 버려지면, 가비지 컬렉터가 이를 청소하느라 브라우저에 버벅임이 발생한다. React 팀은 이벤트 객체를 하나만 만들어 두고, 이벤트가 끝날 때 속성값(Extrinsic State)을 null로 초기화한 뒤 다음 이벤트에서 동일한 객체 껍데기를 재사용했다. (React 17부터는 플라이웨이트 패턴을 너무 엄격하게 적용한 나머지 비동기 처리에서 문제를 일으키기도 하고, 브라우저 GC 성능 향상을 위해서 객체 풀링이 폐지되었다 ㅠㅠ )

#### Virtual DOM과 조정
Virtual DOM은 실제 브라우저의 DOM 노드를 Intrinsic State(본질적 상태)로 간주하고 이를 최대한 재사용하도록 설계되었다. 실제 DOM 노드는 생성 및 메모리 점유 비용이 매우 비싼 자원이기 때문에, React는 이를 직접 조작하는 대신 가벼운 자바스크립트 객체인 Virtual DOM(Flyweight)을 매번 생성하여 이전 상태와 비교한다. 조정 과정에서 `type`과 `key`가 일치하면 React는 기존의 실제 DOM 인스턴스를 파괴하지 않고 유지하며, 오직 변경된 속성값(Extrinsic State)만 해당 노드에 다시 주입한다. 결과적으로 수만 개의 컴포넌트가 렌더링되더라도 실제 물리적인 DOM 노드의 개수는 최소한으로 유지되어 메모리 효율을 극대화한다.


### 한계 
- **코드의 복잡성 증가**: 단일 클래스로 끝날 구조가 Flyweight, Factory, Context(Extrinsic 관리자)로 강제로 쪼개지기 때문에 유지보수 비용이 증가한다.
- **시간 복잡도와의 교환**: 메모리를 절약하는 대신, 매번 렌더링이나 접근 시 팩토리를 통해 인스턴스를 찾아야 하고 상태를 조합해야 하므로 CPU 연산이 미세하게 증가한다.
- **가비지 컬렉터와의 충돌**: 팩토리에 한 번 캐싱된 Flyweight 객체는 명시적으로 제거해주지 않으면 메모리에 영구히 남아있게 된다. 따라서 Map 대신 WeakMap을 사용하여 캐싱 로직을 고도화해 주어야 한다! 


## Proxy Pattern
> 대상 원본 객체를 대리하여 대신 처리하게 함으로써 흐름을 제어하는 행동 패턴

프록시(Proxy)의 사전적인 의미는 '대리인'이라는 뜻이다. 즉, 누군가에게 어떤 일을 대신 시키는 것을 의미하는데, 이를 객체 지향 프로그래밍에 접목해보면 클라이언트가 대상 객체를 직접 쓰는게 아니라 중간에 프록시(대리인)을 거쳐서 쓰는 코드 패턴이라고 보면 된다. 따라서 대상 객체(Subject)의 메소드를 직접 실행하는 것이 아닌, 대상 객체에 접근하기 전에 프록시(Proxy) 객체의 메서드를 접근한 후 추가적인 로직을 처리한뒤 접근하게 된다.

> - **가상 프록시 (Virtual Proxy):** 생성 비용이 많이 드는 객체(예: 고해상도 이미지)를 실제로 사용할 때까지 인스턴스화를 지연시킨다. 
2.  **보호 프록시 (Protection Proxy):** 객체에 대한 접근 권한을 제한다. 
3.  **원격 프록시 (Remote Proxy):** 다른 주소 공간에 있는 객체를 로컬 객체처럼 다룰 수 있게 해 준다.


```typescript
// 기본적인 사용자 정보를 담는 예제

interface User {
  name: string;
  age: number;
}

const user: User = {
  name: "윤신지",
  age: 24,
};

const userProxy = new Proxy<User>(user, {
  get(target, prop: keyof User) {
    console.log(`[Log]: ${prop} 속성에 접근했습니다.`);
    return target[prop];
  },
  set(target, prop: keyof User, value) {
    console.log(`[Log]: ${prop} 속성이 ${target[prop]}에서 ${value}로 변경됩니다.`);
    target[prop] = value as never;
    return true; // 할당이 성공했음을 알림
  },
});

console.log(userProxy.name);
// [Log]: name 속성에 접근했습니다.
// "윤신지"

userProxy.age = 25;
// [Log]: age 속성이 24에서 25로 변경됩니다.
```


### Reflect
Proxy를 다룰 때 `Reflect` 객체를 함께 사용하는 것을 권장한다. 

만약 단순히 `tatget[prop]`으로 접근하거나 할당할 경우, 아래와 같은 문제가 생긴다.
```typescript
const parent = {
  _name: "Parent",
  get name() {
    return this._name;
  }
};

const parentProxy = new Proxy(parent, {
  get(target, prop, receiver) {
    console.log(`트랩 실행됨: ${String(prop)} 접근`);
    return target[prop as keyof typeof target]; // Reflect를 사용하지 않음
  }
});

const child = {
  __proto__: parentProxy,
  _name: "Child"
};

// 프로토타입 체인에 의해 child에는 name이 없으므로 parentProxy의 get 트랩이 호출된다. 
console.log(child.name);
```
`child.name`을 호출했으므로, `this`는 `child`를 가리켜서 "Child"가 출력될 것 같다. 하지만 트랩은 `name`에 대해서만 한 번 실행되고, 결과는 "Parent"가 나온다. 

이유는 `target[prop]`을 반환할 때, 원본 객체(parent)의 `getter`가 실행되면서 내부의 `this`가 프록시나 호출자(child)가 아닌 원본 객체(parent)로 고정되어 버리기 때문이다. 따라서 `this._name`을 찾을 때 트랩을 거치지 않고 바로 원본 객체의 `_name`을 반환한다. 

이를 해결하기 위해서는 아래와 같이 `Reflect`로 해결할 수 있다. 

```typescript
const parentProxyReflect = new Proxy(parent, {
  get(target, prop, receiver) {
    console.log(`트랩 실행됨: ${String(prop)} 접근`);
    // receiver를 통해 올바른 this 컨텍스트를 원본 getter에 전달
    return Reflect.get(target, prop, receiver);
  }
});

const childReflect = {
  __proto__: parentProxyReflect,
  _name: "Child"
};

console.log(childReflect.name);
```
위 코드의 경우 아래의 실행 순서를 거친다.
> 1. 트랩 실행됨: `name` 접근
2. 트랩 실행됨: `_name` 접근 (getter 내부에서 this._name을 호출할 때 다시 트랩을 거침)
3. "Child" (정확한 컨텍스트 반영)


### 실전 예제
#### 런타임 타입 검사 + 유효성 검증 
타입스크립트의 타입 시스템은 컴파일 타입에 국한되기 때문에, API 응답이나 폼 입력 등 런타임에 동적으로 들어오는 데이터의 유효성을 검사할 때 프록시를 사용할 수 있다. 

```typescript
interface Product {
  id: string;
  price: number;
  stock: number;
}

/**
 * 런타임 유효성 검증 프록시 생성 함수
 */
function createValidatorProxy<T extends object>(
  target: T,
  validators: { [K in keyof T]?: (value: any) => boolean }
): T {
  return new Proxy(target, {
    set(obj, prop, value, receiver) {
      const validator = validators[prop as keyof T];
      
      if (validator && !validator(value)) {
        throw new Error(`[Validation Error]: ${String(prop)}에 잘못된 값이 할당되었습니다: ${value}`);
      }
      
      return Reflect.set(obj, prop, value, receiver);
    }
  });
}

const rawProduct: Product = { id: "p1", price: 1000, stock: 10 };

const product = createValidatorProxy(rawProduct, {
  price: (val) => typeof val === 'number' && val > 0,
  stock: (val) => Number.isInteger(val) && val >= 0,
});

product.price = 2000; // 성공
console.log(product.price); // 2000

try {
  product.price = -500; // Error 발생!
} catch (e) {
  console.error((e as Error).message); // [Validation Error]: price에 잘못된 값이 할당되었습니다: -500
}
```

#### 반응형 상태 관리
```typescript
type Observer = () => void;
let currentObserver: Observer | null = null;

/**
 * 간단한 반응형 객체 생성기
 */
function reactive<T extends object>(target: T): T {
  const deps = new Map<keyof T, Set<Observer>>();

  return new Proxy(target, {
    get(obj, prop: keyof T, receiver) {
      // 의존성 수집 (Track)
      if (currentObserver) {
        let propDeps = deps.get(prop);
        if (!propDeps) {
          propDeps = new Set();
          deps.set(prop, propDeps);
        }
        propDeps.add(currentObserver);
      }
      return Reflect.get(obj, prop, receiver);
    },
    set(obj, prop: keyof T, value, receiver) {
      // 값 업데이트
      const result = Reflect.set(obj, prop, value, receiver);
      
      // 의존성 트리거 (Trigger)
      const propDeps = deps.get(prop);
      if (propDeps) {
        propDeps.forEach(observer => observer());
      }
      return result;
    }
  });
}

/**
 * 상태 변화를 감지하여 실행할 함수를 등록
 */
function autorun(fn: Observer) {
  currentObserver = fn;
  fn(); // 최초 실행을 통해 get 트랩을 발생시켜 의존성 수집
  currentObserver = null;
}

// 사용 예시
const state = reactive({ count: 0, text: "Hello" });

autorun(() => {
  console.log(`[DOM Render]: count가 ${state.count}로 변경되었습니다.`);
});

state.count = 1; // [DOM Render]: count가 1로 변경되었습니다. 출력
state.count = 2; // [DOM Render]: count가 2로 변경되었습니다. 출력
state.text = "World"; // count에 의존하지 않으므로 아무것도 출력되지 않음
```

### 한계
- **성능 오버헤드** : 프록시를 거치는 모든 작업은 원본 객체에 직접 접근하는 것보다 미세하게 느리다. JS 엔진은 일반 객체에 대해 고도로 최적화된 인라인 캐싱을 수행하지만, Proxy 객체의 경우 동적인 트랩 호출로 인해 엔진 단의 최적화를 일부 방해받을 수 있다. 
- **디버깅의 복잡성** : 코드가 명시적인 함수 호출 형태가 아니라 `obj.prop = value` 형태의 할당만으로도 수많은 내부 로직이 실행될 수 있으며, 사이드 이펙트를 추적하기 어렵게 만들어 유지보수 난이도가 높아진다.
- **깊은 복사/중첩 객체의 한계** : `Proxy`는 객체의 1-Depth 프로퍼티만을 감지한다. 만약 객체 내부의 중첩된 객체까지 반응형으로 만들거나 가로채기 위해서는 `get` 트랩 내부에서 반환값이 객체일 경우 재귀적으로 Proxy를 다시 씌워주는 작업이 필요하다. 


## Presentational/Container Pattern

> 컴포넌트에서 로직과 프리젠테이션을 분리하여 조금 더 관리하기 쉽고 재사용하기 용이하게 만드는 패턴
- Container : 컴포넌트의 로직에 관한 대부분의 것들을 다루며 API 호출, 데이터 조작, 이벤트 처리 작업 등을 수행
- Presentation : UI에 표시할 요소를 만드는 부분, UI가 정의되는 곳이며 컨테이너에서 `props`형태로 데이터를 받음

단순히 코드를 두 조각으로 나누는 것이 아니라 변하는 것(UI)과 변하지 않는 것(도메인 로직)을 구분하는 방법이다. 


### 유래 
리덕스의 창시자인 Dan Abramov가 작성한 블로그 포스트를 통해 널리 알려졌다. 당시 리액트 생태계는 클래스 컴포넌트 중심이었고, 하나의 컴포넌트가 API 호출, 상태 관리, 복잡한 마크업 출력을 모두 담당하는 '뚱뚱한 컴포넌트' 문제가 심각했다.

초기에는 Smart & Dumb Components라는 이름으로 불리기도 했다.
- Smart (Container): 데이터를 가져오고 상태를 관리하는 영리한 컴포넌트
- Dumb (Presentational): 전달받은 데이터를 화면에 그리기만 하는 수동적인 컴포넌트


### 핵심 동작 원리

해당 패턴을 적용하면, 컴포넌트는 두 개의 명확한 책임이 있는 전보다 작은 컴포넌트로 분리된다. 컨테이너 컴포넌트는 로직에 대해 알고 있으며 위에서 언급한 API 호출, 데이터 조작, 이벤트 처리 등을 담당하고, 프리젠테이션 컴포넌트는 UI를 정의하며 컨테이터로 부터 props로 데이터를 받기 때문에 내부에는 로직이 없고 상태가 없는(stateless, 물론 최소한의 컴포넌트 동작을 위한 상태는 존재할 수도 있다) 컴포넌트가 된다.

| 구분 | Presentational Component | Container Component |
| :--- | :--- | :--- |
| **주요 역할** | UI 렌더링 (어떻게 보일 것인가) | 데이터 및 로직 처리 (어떻게 동작할 것인가) |
| **상태 소유** | 거의 없음 (UI 상태 정도만 소유) | 적극적 소유 (데이터 fetching, 에러 상태 등) |
| **데이터 소스** | Props를 통해 주입받음 | API, Store(Redux/Zustand), Context 등 직접 참조 |
| **의존성** | 외부 로직에 대한 의존성 낮음 | 데이터 소스나 비즈니스 로직에 의존적 |
| **장점** | 높은 재사용성, Storybook 등 UI 테스트 용이 | 로직 집중화로 인한 비즈니스 흐름 파악 용이 |

> 1. Container가 API를 호출하거나 글로벌 스토어에서 데이터를 가져온다.
2. Container는 가공된 데이터와 기능을 수행할 함수를 props로 준비한다.
3. Presentational 컴포넌트에 이들을 주입한다.
4. Presentational은 받은 데이터를 화면에 출력하고, 유저 인터랙션이 발생하면 주입받은 함수를 실행한다.


### 예제 
사용자 정보를 API에서 가져와서 목록을 보여주고, 검색 기능을 포함하는 시스템을 예시로 들었을 때, 모든 로직이 한 곳에 뭉쳐 있는 경우 아래와 같다. 

```typescript
import React, { useState, useEffect } from 'react';

const UserListAllInOne = () => {
  const [users, setUsers] = useState<any[]>([]);
  const [loading, setLoading] = useState(true);
  const [search, setSearch] = useState("");

  useEffect(() => {
    fetch('https://jsonplaceholder.typicode.com/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);

  const filteredUsers = users.filter(user => 
    user.name.toLowerCase().includes(search.toLowerCase())
  );

  if (loading) return <div>로딩 중...</div>;

  return (
    <div className="user-container">
      <input 
        type="text" 
        placeholder="사용자 검색" 
        onChange={(e) => setSearch(e.target.value)} 
      />
      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>
            <strong>{user.name}</strong> - {user.email}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

그런데 Presentational/Container 패턴을 적용할 경우, UI는 순수하게 데이터만 받고 컨테이너는 로직에 집중하도록 작성할 수 있다. 

#### Presentational 컴포넌트 (UI 담당)
- types.ts
```typescript
export interface User {
  id: number;
  name: string;
  email: string;
}

interface UserListProps {
  users: User[];
  loading: boolean;
  onSearch: (keyword: string) => void;
}
```

- UserList.tsx <- 여기가 프리젠테이션 컴포넌트
```typescript
export const UserList: React.FC<UserListProps> = ({ users, loading, onSearch }) => {
  if (loading) return <div className="spinner">데이터를 불러오는 중입니다...</div>;

  return (
    <section className="user-view">
      <header>
        <h2>사용자 디렉토리</h2>
        <input 
          type="search" 
          placeholder="이름으로 검색하세요" 
          onChange={(e) => onSearch(e.target.value)} 
        />
      </header>
      <ul className="user-list">
        {users.map(user => (
          <li key={user.id} className="user-card">
            <h4>{user.name}</h4>
            <p>{user.email}</p>
          </li>
        ))}
      </ul>
    </section>
  );
};
```

#### Container 컴포넌트 (로직 담당)
- UserListContainer.tsx
```typescript
import React, { useState, useEffect } from 'react';
import { UserList } from './UserList';
import { User } from './types';

export const UserListContainer = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [filteredUsers, setFilteredUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState<boolean>(true);

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/users');
        const data = await response.json();
        setUsers(data);
        setFilteredUsers(data);
      } catch (error) {
        console.error("Failed to fetch users", error);
      } finally {
        setLoading(false);
      }
    };
    fetchUsers();
  }, []);

  const handleSearch = (keyword: string) => {
    const result = users.filter(user => 
      user.name.toLowerCase().includes(keyword.toLowerCase())
    );
    setFilteredUsers(result);
  };

  // 로직을 수행한 후 Presentational 컴포넌트에 데이터를 주입하여 반환
  return (
    <UserList 
      users={filteredUsers} 
      loading={loading} 
      onSearch={handleSearch} 
    />
  );
};
```

### 컨테이너... 와 커스텀 훅
리액트에 Hook이 없던 시절에는 컴포넌트 사이에서 상태 로직을 재사용하는 방법이 제한적이었다. HOC 방식과 Render Props 방법이 있었으나, 각각 Wrapper Hell을 유발하고 가독성을 저해시켰다. 이때 가장 깔끔한 대안이 프리젠테이션/컨테이너 패턴이다. 
👉 로직을 격리할 수 있는 유일한 단위가 컴포넌트였기 때문에!! 

2019년 Hook이 등장하면서 컴포넌트가 아닌 Hook으로 상태와 사이드 이펙트를 뺄 수 있었고, 컨테이너 역할을 수행하는 것이 파일 단위의 컴포넌트가 아닌 커스텀 훅이 되었다. 


| 구분 | 과거: Container Component | 현대: Custom Hook |
| :--- | :--- | :--- |
| **격리 단위** | 별도의 `.tsx` 파일 (컴포넌트) | 별도의 `.ts` 파일 (함수) |
| **데이터 전달** | Props Drilling 발생 가능 | 필요한 곳에서 직접 호출 |
| **코드 구조** | 부모-자식 계층 구조 강제 | 평면적인 로직 주입 |
| **추상화 수준** | UI와 결합된 상태로 분리 | 순수 비즈니스 로직만 추출 가능 |


그럼에도 불구하고 컨테이너 컴포넌트가 필요한 순간들이 있다! 

> - Context API / Provider 주입: 특정 섹션 전체에 데이터를 공급해야 하는 진입점 역할이 필요할 때
- Error Boundary / Suspense: UI 전체를 감싸서 에러나 로딩 상태를 선언적으로 관리해야 할 때
- Recursive Components: 자기 자신을 호출하는 복잡한 UI 구조에서 로직과 뷰를 섞으면 디버깅이 지옥이 될 때
