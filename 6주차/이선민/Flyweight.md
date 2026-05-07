# Flyweight 패턴

## 정의

Flyweight 패턴은 **비슷한 객체를 대량으로 만들어야 할 때 메모리를 절약할 수 있게 해주는 패턴**이다. 동일한 데이터를 가진 객체들의 인스턴스를 재사용하여 메모리 효율성을 높인다.

## 동작 방식

핵심 원리는 다음과 같다.

1. 새로운 객체 생성 전에 동일한 속성의 객체가 이미 존재하는지 확인한다.
2. 존재한다면 **기존 인스턴스를 재사용**한다.
3. 존재하지 않으면 새로운 인스턴스를 생성하고 저장한다.

## 코드 예시

도서관 시스템에서 동일한 책(같은 ISBN)이 여러 권 있는 경우를 생각해보자. 동일한 책 정보를 매번 새로 만들 필요는 없다.

### 기본 클래스 정의

```javascript
class Book {
  constructor(title, author, isbn) {
    this.title = title
    this.author = author
    this.isbn = isbn
  }
}
```

### 책 생성 함수 (Flyweight 구현)

`isbn`을 키로 하는 `Map`에 이미 생성된 책이 있는지 확인하고, 있으면 기존 인스턴스를 반환한다.

```javascript
const books = new Map()

const createBook = (title, author, isbn) => {
  const existingBook = books.has(isbn)

  if (existingBook) {
    return books.get(isbn)
  }

  const book = new Book(title, author, isbn)
  books.set(isbn, book)

  return book
}
```

### 책 추가 함수

각 책의 공통 정보(`title`, `author`, `isbn`)는 `createBook`을 통해 재사용하고, 인스턴스마다 달라지는 정보(`sales`, `availability`)만 별도로 추가한다.

```javascript
const bookList = []

const addBook = (title, author, isbn, availability, sales) => {
  const book = {
    ...createBook(title, author, isbn),
    sales,
    availability,
    isbn,
  }

  bookList.push(book)
  return book
}
```

### 실제 사용 예제

```javascript
addBook('Harry Potter', 'JK Rowling', 'AB123', false, 100)
addBook('Harry Potter', 'JK Rowling', 'AB123', true, 50)
addBook('To Kill a Mockingbird', 'Harper Lee', 'CD345', true, 10)
addBook('To Kill a Mockingbird', 'Harper Lee', 'CD345', false, 20)
addBook('The Great Gatsby', 'F. Scott Fitzgerald', 'EF567', false, 20)
```

총 5권의 책을 추가했지만 실제로 생성된 `Book` 인스턴스는 **3개**뿐이다. 동일한 ISBN을 가진 책은 같은 인스턴스를 공유하기 때문이다.

## 장점

- **메모리 절약**: 동일한 데이터를 가진 객체는 하나의 인스턴스만 유지한다.
- **성능 향상**: 대량의 객체 생성 시 메모리 사용량을 최소화할 수 있다.
- **효율적인 자원 관리**: 중복 데이터 저장을 방지한다.

## 단점

- 구현 복잡도가 증가한다.
- JavaScript에서는 **프로토타입 상속**으로도 비슷한 효과를 낼 수 있어 상대적으로 중요도가 낮다.
- 데이터를 공유하기 때문에 부작용(side effect)이 발생할 수 있다.

## 활용 사례

- 도서관 시스템에서 동일 도서의 여러 사본을 관리
- 게임에서 동일한 텍스처나 모델을 사용하는 다중 객체 생성
- 데이터베이스 커넥션 풀 관리
- 대규모 리스트나 그리드에서 동일 요소를 반복 표시할 때

## 참고

- [patterns-dev-kr - Flyweight Pattern](https://patterns-dev-kr.github.io/design-patterns/flyweight-pattern/)
