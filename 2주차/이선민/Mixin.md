## Mixin 패턴이란?
- **상속 없이** 객체나 클래스에 재사용 가능한 기능을 추가하는 패턴
- Mixin 객체는 단독으로 사용되지 않고, 다른 객체나 클래스에 기능을 주입하는 용도로 사용된다

## Mixin 패턴 구현

### 기본 형태
- `Object.assign`을 사용해 클래스의 prototype에 기능을 추가한다

```javascript
class Dog {
  constructor(name) {
    this.name = name;
  }
}

const dogFunctionality = {
  bark: () => console.log('Woof!'),
  wagTail: () => console.log('Wagging my tail!'),
  play: () => console.log('Playing!'),
};

Object.assign(Dog.prototype, dogFunctionality);

const pet1 = new Dog('Daisy');
pet1.bark();    // Woof!
pet1.play();    // Playing!
```

### Mixin 간 조합
- Mixin도 다른 Mixin의 기능을 합칠 수 있다

```javascript
const animalFunctionality = {
  walk: () => console.log('Walking!'),
  sleep: () => console.log('Sleeping!'),
};

const dogFunctionality = {
  bark: () => console.log('Woof!'),
  wagTail: () => console.log('Wagging my tail!'),
  play: () => console.log('Playing!'),
};

// animalFunctionality를 dogFunctionality에 합침
Object.assign(dogFunctionality, animalFunctionality);

// 합쳐진 결과를 Dog에 추가
Object.assign(Dog.prototype, dogFunctionality);

const pet1 = new Dog('Daisy');
pet1.bark();  // Woof!
pet1.walk();  // Walking!
pet1.sleep(); // Sleeping!
```

## 실제 사용 사례
- 브라우저의 **Window 객체**가 대표적인 Mixin 사례
- `setTimeout`, `setInterval`, `indexedDB` 같은 프로퍼티들은 WindowOrWorkerGlobalScope, WindowEventHandler 등의 Mixin으로부터 제공된다

## 장점
- 상속 계층 없이 기능을 추가할 수 있다
- 여러 Mixin을 조합하여 유연하게 기능을 구성할 수 있다
- prototype에 추가하므로 메모리 효율적이다

## 단점
- prototype을 직접 수정하므로 의도하지 않은 부작용이 발생할 수 있다
- 메서드가 어디서 온 건지 코드 출처 추적이 어렵다
- 여러 Mixin이 같은 이름의 메서드를 가지면 **네임스페이스 충돌**이 발생한다

```javascript
const mixinA = { greet: () => 'Hello from A' };
const mixinB = { greet: () => 'Hello from B' };

Object.assign(Dog.prototype, mixinA);
Object.assign(Dog.prototype, mixinB); // mixinA의 greet을 덮어씀

new Dog('Daisy').greet(); // 'Hello from B' — 조용히 덮어써짐
```

## React에서의 Mixin

### 과거: React.createClass에서 Mixin 사용
- ES6 이전 React에서는 컴포넌트에 기능을 추가하기 위해 Mixin을 사용했다

### 현재: 권장하지 않음
- React 팀은 "Mixin이 복잡도를 증가시키고 재사용하기 어렵게 만든다"고 판단
- 대안:
  - **HOC (고차 컴포넌트)** — 컴포넌트를 감싸서 기능 추가
  - **Hooks** — 현재 표준. 로직 재사용의 주요 방법

```javascript
// Mixin이 하던 일을 Hook으로 대체
function useWindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });

  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };
    window.addEventListener('resize', handleResize);
    handleResize();
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// 어떤 컴포넌트에서든 재사용
function Header() {
  const { width } = useWindowSize();
  return <header>{width > 768 ? '데스크톱' : '모바일'}</header>;
}
```
