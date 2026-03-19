# [Singleton Pattern](https://patterns-dev-kr.github.io/design-patterns/singleton-pattern/)

## [What] Singleton Pattern이란?
> "인스턴스가 딱 하나만 존재해야 할 때" 쓰는 패턴  _(claude)_

> Singleton은 1회에 한하여 인스턴스화가 가능하며 전역에서 접근 가능한 클래스 _(patterns.dev)_

## [Patterns.dev] Singleton Pattern 예시로 알아보기

Patterns.dev에서는 Singleton Pattern을 `Counter` 클래스를 통해 설명하고 있다.

```ts
let counter = 0

class Counter {
  getInstance() {
    return this
  }

  getCount() {
    return counter
  }

  increment() {
    return ++counter
  }

  decrement() {
    return --counter
  }
}
```

해당 `Counter` 클래스는 Singleton Pattern 이라고 할 수 없다.  
왜냐면 외부에서 `new Counter()`를 통해 여러 개의 인스턴스를 만들 수 있기 때문이다.

```ts
const counter1 = new Counter()
const counter2 = new Counter()

console.log(counter1.getInstance() === counter2.getInstance()) // false
```

이렇게 되면 각 인스턴스의 `getInstance()`를 호출해 반환되는 레퍼러스는 같지 않다.

| ![prototype_chain.png](images/prototype%20chain.png) |
|:--------------------------------------------------:|
|            인스턴스의 각각의 this는 다른 메모리를 가리킴             |

`Counter` 클래스가 한 번만 만들어질 수 있도록 코드를 수정해 보자.

그 방법은 `instance`라는 변수를 만들어서, 인스턴스가 이미 생성되어 할당 되었는지 체크하는 것이다.

```ts
let instance
let counter = 0

class Counter {
    constructor() {
        if (instance) {
            throw new Error('You can only create one instance!')
        }
        instance = this
    }

    getInstance() {
        return this
    }

    getCount() {
        return counter
    }

    increment() {
        return ++counter
    }

    decrement() {
        return --counter
    }
}

const counter1 = new Counter()
const counter2 = new Counter()
// Error: You can only create one instance!
```

여기에 `Object.freeze`를 통해 외부에서 프로퍼티가 writable 되지 않게 해주면 금상첨화이다.

```ts
...(동일)

const singletonCounter = Object.freeze(new Counter())
export default singletonCounter
```

이를 그림으로 보면 다음과 같다.

| ![singleton pattern compare.png](images/singleton%20pattern%20compare.png) |
|:--------------------------------------------------------------------------:|
|                      기존 Class와 Singleton Pattern의 차이                       |

이렇게 해두면 외부에서 `singletonCounter.increment()`를 호출하면, Singleton 인스턴스의 counter 값은 양쪽 파일에서 모두 증가된 동일한 값을 공유받을 수 있다.

| ![samsame instance.png](images/samsame%20instance.png) |
|:------------------------------------------------------:|
|                각 파일에서 동일한 인스턴스를 참조하게 됨                 |

## [How] Singleton Pattern은 어떻게 구현하는가?

Singleton Pattern은 인스턴스를 **하나만 만들어야 하는 경우**에 사용한다.

아래는 Singleton Pattern의 기본적인 구조이다.
(하나의 예시일 뿐이며 구현체 보단 개념이 중요하다. 즉, 하나의 인스턴스만 허용한다는 점이 핵심이다.)

```ts
class Singleton {
    // 유일한 인스턴스를 저장할 static 변수
    // 클래스 레벨에서의 인스턴스 관리 & getInstance시 동일한 인스턴스 반환
    private static instance: Singleton;
    
    // constructor를 숨겨 외부에서 인스턴스를 생성하지 못하도록 막음
    private constructor() {}
    
    public static getInstance(): Singleton {
        // 인스턴스가 존재하지 않으면 새로 만들고
        if (!Singleton.instance) {
            Singleton.instance = new Singleton();
        }
        // 존재하면 기존의 인스턴스를 반환
        return Singleton.instance;
    }
}

export default Singleton;
```

## [Why] Singleton Pattern은 왜 사용하는가?

Q. 언제 사용되면 좋을까?  
- 핵심은 **인스턴스가 하나만 있어야 하는 경우.**

Q. 왜 인스턴스가 하나만 있어야 하는가?
> 하나의 진실(Single Source of Truth)을 보장하기 위해
- 데이터 중복 생성을 방지하기 위해
  - 한 번의 데이터 요청으로 가져온 데이터를 애플리케이션 전반에 걸쳐 사용해야 할 때.
  - ex) 한 번 불러온 api를 다시 불러오지 않아도 되는데, 각각의 컴포넌트에서 불러오는 경우.
- 애플리케이션의 전체 생명주기에서 한 곳에서 데이터를 관리하기 위해
  - 컴포넌트나 라우트를 이동해도 초기화되지 않고 유지될 수 있게끔 하는 처리가 필요할 때.
  - ex) 장바구니(각 페이지에 접속할 때마다 장바구니 품목이 초기화 된다면?), 전역 상태관리