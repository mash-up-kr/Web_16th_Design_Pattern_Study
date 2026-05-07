# [Flyweight Pattern](https://patterns-dev-kr.github.io/design-patterns/flyweight-pattern/)

## [What] Flyweight Pattern이란?

> "동일한 데이터를 여러 객체가 공유하여 메모리를 절약하는 패턴" _(claude)_

> 플라이웨이트 패턴은 비슷한 객체를 대량으로 생성해야 할 때 메모리를 절약할 수 있게 해주는 유용한 패턴이다. _(patterns.dev)_

Flyweight Pattern의 핵심 아이디어는 단순하다. **여러 객체가 공통적으로 가지는 무거운 데이터는 한 곳에 두고, 객체별로 다른 가벼운 데이터만 따로 관리하는 것**이다.

게임을 떠올려 보면 직관적이다. 

숲속 장면에 나무 1만 그루를 그려야 한다고 해보자. 각 나무가 메쉬 데이터, 텍스처, 모델 정보를 따로 가진다면 메모리는 금방 폭발한다. 하지만 메쉬와 텍스처는 모든 나무가 같다. 위치(x, y, z)와 크기, 회전값만 나무마다 다를 뿐이다. 공통 데이터(메쉬/텍스처)는 한 번만 만들어 모든 나무가 공유하고, 나무마다 다른 데이터(위치/크기)만 따로 관리하면 메모리를 크게 절약할 수 있다.

이 분리가 Flyweight Pattern의 핵심 개념이다.

- **Intrinsic State (공유 가능한 상태)**: 여러 객체 사이에서 동일하고 변하지 않는 데이터. 한 번만 만들어 공유한다. (예: 책의 제목, 저자, ISBN / 나무의 메쉬, 텍스처)
- **Extrinsic State (객체별 고유 상태)**: 각 객체마다 다른 데이터. 외부에서 주입받거나 객체별로 따로 관리한다. (예: 책의 대출 가능 여부, 판매량 / 나무의 위치, 크기)

모두 "같은 데이터를 한 번만 저장하고 여러 곳에서 참조하자"는 Flyweight 사고방식의 적용이다.

## [Patterns.dev] Flyweight Pattern 예시로 알아보기

### Flyweight 없이 직접 다루는 경우

도서관 앱을 만든다고 해보자. 책에는 `title`, `author`, `isbn`이 있고, 같은 책이 여러 사본으로 존재한다.

```js
class Book {
  constructor(title, author, isbn) {
    this.title = title;
    this.author = author;
    this.isbn = isbn;
  }
}

const books = [
  new Book("Harry Potter", "JK Rowling", "AB123"),
  new Book("Harry Potter", "JK Rowling", "AB123"), // 같은 책의 사본
  new Book("To Kill a Mockingbird", "Harper Lee", "CD345"),
  new Book("To Kill a Mockingbird", "Harper Lee", "CD345"), // 같은 책의 사본
  new Book("The Great Gatsby", "F. Scott Fitzgerald", "EF567"),
];
```

이 코드의 문제는 명확하다. `"Harry Potter"`, `"JK Rowling"`, `"AB123"`이라는 동일한 문자열이 사본 수만큼 중복 저장된다. 책이 1만 권에 사본이 평균 3개라면, 30만 개의 객체가 만들어지고, 그 중 상당수가 동일한 데이터를 들고 있는 셈이다.

### Flyweight를 적용한 경우

ISBN을 키로 사용하여, 같은 책에 대해서는 단 하나의 `Book` 인스턴스만 만들고 재사용한다.

```js
class Book {
  constructor(title, author, isbn) {
    this.title = title;
    this.author = author;
    this.isbn = isbn;
  }
}

const books = new Map();

const createBook = (title, author, isbn) => {
  const existingBook = books.has(isbn);

  if (existingBook) {
    return books.get(isbn); // 이미 만든 인스턴스를 재사용
  }

  const book = new Book(title, author, isbn);
  books.set(isbn, book);

  return book;
};
```

이제 사본별로 다른 정보(`availability`, `sales`)는 따로 관리한다. 공통 정보는 `createBook`에서 받아온 인스턴스를 그대로 활용한다.

```js
const bookList = [];

const addBook = (title, author, isbn, availability, sales) => {
  const book = {
    ...createBook(title, author, isbn), // 공유 데이터 (Intrinsic)
    sales,                               // 사본별 데이터 (Extrinsic)
    availability,
    isbn,
  };

  bookList.push(book);
  return book;
};
```

사용 예시는 다음과 같다.

```js
addBook("Harry Potter", "JK Rowling", "AB123", false, 100);
addBook("Harry Potter", "JK Rowling", "AB123", true, 50);
addBook("To Kill a Mockingbird", "Harper Lee", "CD345", true, 10);
addBook("To Kill a Mockingbird", "Harper Lee", "CD345", false, 20);
addBook("The Great Gatsby", "F. Scott Fitzgerald", "EF567", false, 20);
```

5번의 `addBook` 호출이 있었지만, 실제로 생성된 `Book` 인스턴스는 **3개뿐**이다. 같은 ISBN의 책은 동일한 인스턴스를 공유한다.

### JavaScript에서의 Flyweight

patterns.dev에서는 JavaScript의 **prototype chain** 자체가 이미 Flyweight 비슷한 효과를 제공한다고 언급한다. 동일한 클래스에서 만들어진 인스턴스들은 메서드를 각자 복제하지 않고 prototype을 공유한다. 이 때문에 JavaScript에서는 GoF의 전형적인 Flyweight 구현이 다른 언어만큼 강하게 요구되지는 않는다.

```js
class Dog {
  bark() { return "Woof!"; }
}

const d1 = new Dog();
const d2 = new Dog();
console.log(d1.bark === d2.bark); // true — 메서드는 prototype에서 공유됨
```

다만 `bark`처럼 "메서드"가 아닌 "데이터"를 공유하려면 여전히 명시적인 Flyweight 적용이 필요하다.

## [How] Flyweight Pattern은 어떻게 구현하는가?

핵심은 **여러 객체가 공유할 수 있는 부분(Intrinsic)과 각 객체마다 다른 부분(Extrinsic)을 분리하고, Intrinsic 부분을 캐시하여 재사용하는 것**이다.

### 기본 구조

```ts
// Intrinsic state — 공유될 데이터
interface SharedState {
  mesh: string;
  texture: string;
}

// Extrinsic state — 객체마다 다른 데이터
interface UniqueState {
  x: number;
  y: number;
  z: number;
}

// Flyweight 객체
class Flyweight {
  constructor(private intrinsicState: SharedState) {}

  operation(extrinsicState: UniqueState): void {
    console.log(
      `렌더링: mesh=${this.intrinsicState.mesh}, ` +
      `위치=(${extrinsicState.x}, ${extrinsicState.y}, ${extrinsicState.z})`
    );
  }
}

// Flyweight Factory — 캐시를 통해 인스턴스 재사용
class FlyweightFactory {
  private flyweights: Map<string, Flyweight> = new Map();

  getFlyweight(key: string, sharedState: SharedState): Flyweight {
    if (!this.flyweights.has(key)) {
      this.flyweights.set(key, new Flyweight(sharedState));
    }
    return this.flyweights.get(key)!;
  }

  size(): number {
    return this.flyweights.size;
  }
}
```

---

...^^