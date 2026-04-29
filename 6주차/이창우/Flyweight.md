# Flyweight 패턴

> 공통 데이터를 공유해서 객체 수를 줄이는 패턴
> 

```tsx
class Tree {
  constructor(
    public x: number,
    public y: number,
    public type: string,
    public color: string,
    public texture: string
  ) {}
}
```

이러한 형식의 나무 객체

```tsx
const forest = [];

for (let i = 0; i < 100000; i++) {
  forest.push(new Tree(randomX, randomY, "oak", "green", "oak-texture"));
}
```

숲을 만들게 된다면 … * 100000 번 중복

⇒ 변하지 않는 건 공유하고, 변하는 것만 따로 만들자!

### 구조

플라이웨이트 패턴의 핵심은 상태 분리

1. 내재 상태 ( type, color, texture )
    1. 공유 가능
    2. 변하지 않음
    3. 여러 객체가 같이 씀
2. 외재 상태 ( x, y (위치) )
    1. 개별 객체마다 다름
    2. 외부에서 주입받음 

```tsx
class TreeType {
  constructor(
    public type: string,
    public color: string,
    public texture: string
  ) {}

  draw(x: number, y: number) {
    console.log(`${this.type} tree at (${x}, ${y})`);
  }
}
```

```tsx
class TreeFactory {
  private static treeTypes = new Map<string, TreeType>();

  static getTreeType(type: string, color: string, texture: string) {
    const key = `${type}-${color}-${texture}`;

    if (!this.treeTypes.has(key)) {
      this.treeTypes.set(key, new TreeType(type, color, texture));
    }

    return this.treeTypes.get(key)!;
  }
}
```

```tsx
class Tree {
  constructor(
    public x: number,
    public y: number,
    public treeType: TreeType
  ) {}

  draw() {
    this.treeType.draw(this.x, this.y);
  }
}
```

```tsx
const forest = [];

for (let i = 0; i < 100000; i++) {
  const type = TreeFactory.getTreeType("oak", "green", "oak-texture");
  forest.push(new Tree(randomX, randomY, type));
}
```

플라이 웨이트는 객체를 줄이는 패턴이 아니라, 중복 데이터를 제거하는 패턴이다!

### 이 방식을 도대체 언제써?

- 객체 개수가 너무 많을 때 ( 수천 ~ 수십만 )
- 내부 데이터 (내재 상태)가 많이 겹칠 때
- 메모리 이슈 발생

⇒ 객체 수 적거나, 데이터가 독립적인 경우 쓰지 말 것.. 정말 복잡함.. 상태 관리 어렵고 디버깅 빡셈

플라이 웨이트 패턴은 공통 상태를 공유하고, 개별 상태를 분리해서 대량 객체의 메모리 사용을 최소화하는 구조적인 패턴!

[patterns.dev](http://patterns.dev) : 자바스크립트에서 [**프로토타입 상속**](https://patterns-dev-kr.github.io/design-patterns/prototype-pattern/)을 통해서도 비슷한 효과를 낼 수 있다 보니 이 패턴은 그리 크게 중요하지 않게 되었다.

```tsx
function User(name) {
  this.name = name;

  // ❌ 모든 인스턴스마다 함수 생성됨
  this.sayHello = function () {
    console.log("Hello " + this.name);
  };
}

const u1 = new User("창우");
const u2 = new User("동훈");
```

sayHello()가 인스턴스 마다 생성.. 많아지는 경우 메모리 부담 상승

```tsx
function User(name) {
  this.name = name;
}

// ✅ 공통 로직을 프로토타입에 둠
User.prototype.sayHello = function () {
  console.log("Hello " + this.name);
};

const u1 = new User("창우");
const u2 = new User("철수");
```

프로토타입에 공통로직을 그냥 둠 ⇒ 하위 인스턴스들 전부 공유 가능

훨씬 간단..! 안 쓸만 하다!
