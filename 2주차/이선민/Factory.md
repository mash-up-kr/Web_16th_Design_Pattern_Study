## Factory 패턴이란?
- `new` 키워드를 사용하지 않고, **함수 호출**만으로 객체를 생성하는 패턴
- 동일한 프로퍼티를 가진 여러 객체를 효율적으로 만들어낼 수 있다
- 기본적으로 팩토리 패턴은 심플 팩토리 패턴, 팩토리 메서드 패턴, 추상 팩토리 패턴 세가지가 있다

## Factory 패턴 구현

### 기본 형태 (심플 팩토리 패턴)
- 팩토리 함수는 매번 새로운 객체를 반환한다
- ES6 화살표 함수로 간결하게 구현 가능

```javascript
const createUser = ({ firstName, lastName, email }) => ({
  firstName,
  lastName,
  email,
  fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
});

const user1 = createUser({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@doe.com',
});

const user2 = createUser({
  firstName: 'Jane',
  lastName: 'Doe',
  email: 'jane@doe.com',
});

console.log(user1.fullName()); // "John Doe"
console.log(user2.fullName()); // "Jane Doe"
```

#### 동적 키-값 객체 생성
- 배열이나 동적 데이터를 기반으로 객체를 생성할 때 유용하다

```javascript
const createObjectFromArray = ([key, value]) => ({
  [key]: value,
});

createObjectFromArray(['name', 'John']); // { name: "John" }
```

### 팩토리 메서드 패턴
- 객체 생성을 서브클래스에 위임하는 패턴
- 기반 클래스는 **어떤 객체를 만들지 모르고**, 서브클래스가 결정한다

```javascript
// 기반 클래스 — createTransport()를 서브클래스에 위임
class DeliveryService {
  createTransport() {
    throw new Error('서브클래스에서 구현해야 합니다');
  }

  deliver(item) {
    const transport = this.createTransport(); // 팩토리 메서드 호출
    transport.move(item);
  }
}

// 서브클래스들이 각각 다른 객체를 생성
class TruckDelivery extends DeliveryService {
  createTransport() {
    return { move: (item) => console.log(`트럭으로 ${item} 배송`) };
  }
}

class ShipDelivery extends DeliveryService {
  createTransport() {
    return { move: (item) => console.log(`배로 ${item} 배송`) };
  }
}

// 사용 — deliver() 로직은 동일, 무엇으로 배송할지는 서브클래스가 결정
new TruckDelivery().deliver('TV');  // 트럭으로 TV 배송
new ShipDelivery().deliver('TV');   // 배로 TV 배송
```

## 추상 팩토리 패턴
- 관련된 객체 **세트**를 일관되게 생성하는 패턴
- 추상 팩토리 인터페이스를 정의하고, Implement 팩토리가 이를 구현한다

```javascript
// 추상 팩토리 — JS에는 interface가 없으니 기반 클래스로 대체
class ThemeFactory {
  createButton(label) {
    throw new Error('createButton을 구현해야 합니다');
  }
  createInput(placeholder) {
    throw new Error('createInput을 구현해야 합니다');
  }
}

// 구체 팩토리 1 — 추상 팩토리를 구현
class LightThemeFactory extends ThemeFactory {
  createButton(label) {
    return { render: () => `[Light 버튼: ${label}]` };
  }
  createInput(placeholder) {
    return { render: () => `[Light 인풋: ${placeholder}]` };
  }
}

// 구체 팩토리 2 — 같은 인터페이스를 구현
class DarkThemeFactory extends ThemeFactory {
  createButton(label) {
    return { render: () => `[Dark 버튼: ${label}]` };
  }
  createInput(placeholder) {
    return { render: () => `[Dark 인풋: ${placeholder}]` };
  }
}

// 클라이언트 — ThemeFactory 타입만 알면 됨
function renderUI(factory) {
  const button = factory.createButton('확인');
  const input = factory.createInput('이름');
  console.log(button.render());
  console.log(input.render());
}

renderUI(new DarkThemeFactory());
// [Dark 버튼: 확인]
// [Dark 인풋: 이름]
```

### JavaScript에서의 한계: 덕 타이핑
- JS에서는 `ThemeFactory`를 상속하지 않아도 `createButton`, `createInput`만 있으면 동작한다
- 즉 아래처럼 아무 객체나 넘겨도 에러 없이 실행됨

```javascript
// ThemeFactory를 상속하지 않은 그냥 객체
const fakeFactory = {
  createButton: (label) => ({ render: () => `[Fake: ${label}]` }),
  createInput: (placeholder) => ({ render: () => `[Fake: ${placeholder}]` }),
};

renderUI(fakeFactory); // 정상 동작 — 인터페이스 강제가 안 됨
```

- TypeScript의 interface를 사용하면 이를 컴파일 타임에 강제할 수 있다

```typescript
interface UIComponent {
  render(): string;
}

interface ThemeFactory {
  createButton(label: string): UIComponent;
  createInput(placeholder: string): UIComponent;
}

// 메서드를 빼먹으면 컴파일 에러
class DarkThemeFactory implements ThemeFactory {
  createButton(label: string) {
    return { render: () => `[Dark 버튼: ${label}]` };
  }
  // createInput 빼먹으면 → ❌ 컴파일 에러
}
```



## 자바스크립트에서 함수로 구현한 팩토리 패턴의 장단점
### 1. 클로저를 활용해 진짜 Private 변수를 만들 수 있다
![alt text](image.png)
```javascript
// 클래스: #private은 문법적 private
class BankAccount {
  #balance = 0;
  deposit(amount) { this.#balance += amount; }
  getBalance() { return this.#balance; }
}

const acc = new BankAccount();
acc.#balance; // ❌ SyntaxError (접근 불가)
// 하지만 devtools에서 보임.

// 팩토리: 클로저 private은 진짜로 외부에서 접근할 방법이 없음
const createBankAccount = () => {
  let balance = 0; // 스코프 밖에서는 절대 접근 불가

  return {
    deposit: (amount) => { balance += amount; },
    getBalance: () => balance,
  };
};

const acc2 = createBankAccount();
acc2.balance; // undefined — 프로퍼티 자체가 없음
// devtools에서도 안 보임, 우회 방법 없음
```

### 2. 클래스와 다르게 this 바인딩 문제가 없다

#### 클래스의 this 문제
- 클래스 메서드를 변수에 꺼내거나 콜백으로 넘기면 `this`가 사라진다

```javascript
class User {
  constructor(name) {
    this.name = name;
  }
  greet() {
    return `안녕, ${this.name}`;
  }
}

const user = new User('선민');

// 메서드를 변수에 꺼내면 this가 사라짐
const greetFn = user.greet;
greetFn(); // ❌ this가 undefined가 되어 TypeError발생

// 콜백으로 넘겨도 마찬가지
setTimeout(user.greet, 100); // ❌ this가 undefined가 되어 TypeError발생

// bind로 해결해야 함
setTimeout(user.greet.bind(user), 100); // ✅
```

#### 팩토리는 this가 필요 없음
- 클로저가 `name` 값을 직접 잡고 있기 때문에, `this`가 뭐든 상관없다

```javascript
const createUser = ({ firstName, lastName, email }) => ({
  firstName,
  lastName,
  email,
  fullName() {
    return `${firstName} ${lastName}`;
  },
});

const user1 = createUser({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@doe.com',
});

const userFn = user1.fullName;
console.log(userFn()); // ✅ 'John Doe'
```

### 3. 클래스를 사용하는 것이 **메모리 효율성** 측면에서 더 낫다
객체를 1000개 만들면:
- 클래스: 프로퍼티 1000개 + 메서드 1개 (prototype에 공유)
- 팩토리: 프로퍼티 1000개 + 메서드 1000개 (각각 새로 생성)
```javascript
// 클래스: prototype으로 메서드 공유 (메모리 효율적)
class User {
  constructor(firstName, lastName, email) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.email = email;
  }

  fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}

// 팩토리: 매번 새로운 fullName 함수가 생성됨
const createUser = ({ firstName, lastName, email }) => ({
  firstName,
  lastName,
  email,
  fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
});
```

## Factory 패턴 예시

### React 커스텀 훅
- 커스텀 훅은 호출할 때마다 상태와 로직이 담긴 새로운 객체를 반환한다

```javascript
function useForm(initialValues) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    setValues({ ...values, [e.target.name]: e.target.value });
  };

  const reset = () => setValues(initialValues);

  // 팩토리처럼 매번 새로운 상태 + 메서드 세트를 반환
  return { values, errors, handleChange, reset, setErrors };
}

// 사용 — 각 컴포넌트가 독립적인 폼 상태를 가짐
function LoginForm() {
  const { values, handleChange, reset } = useForm({ email: '', password: '' });
  return <input name="email" value={values.email} onChange={handleChange} />;
}

function SignupForm() {
  const { values, handleChange } = useForm({ name: '', email: '' });
  return <input name="name" value={values.name} onChange={handleChange} />;
}
```

### 설정에 따른 객체 생성
- 환경(개발/운영)이나 역할(관리자/일반)에 따라 다른 설정의 객체를 생성
- ex. Logger

```javascript
const createLogger = (env) => {
  const isDev = env === 'development';
  return {
    log(message) {
      if (isDev) console.log(`[DEV] ${message}`);
    },
    error(message) {
      console.error(`[ERROR] ${message}`);
    },
    getLevel() {
      return isDev ? 'debug' : 'error';
    },
  };
};

const logger = createLogger(process.env.NODE_ENV);
logger.log('디버그 메시지'); // 개발 환경에서만 출력
```

### 복잡한 초기화 로직 캡슐화

```javascript
const createFetcher = ({ baseURL, timeout = 5000, headers = {} }) => {
  // 복잡한 초기화 로직을 숨김
  const defaultHeaders = {
    'Content-Type': 'application/json',
    ...headers,
  };

  const controller = new AbortController();

  return {
    async get(path) {
      const res = await fetch(`${baseURL}${path}`, {
        headers: defaultHeaders,
        signal: controller.signal,
        timeout,
      });
      return res.json();
    },
    async post(path, body) {
      const res = await fetch(`${baseURL}${path}`, {
        method: 'POST',
        headers: defaultHeaders,
        body: JSON.stringify(body),
        signal: controller.signal,
      });
      return res.json();
    },
    cancel() {
      controller.abort();
    },
  };
};

// 사용하는 쪽은 내부 설정을 몰라도 됨
const api = createFetcher({
  baseURL: 'https://api.example.com',
  headers: { Authorization: 'Bearer token123' },
});

api.get('/users');
api.post('/users', { name: '선민' });
```

## 참고 자료
- https://coor.tistory.com/51
- [테코톡-팩토리 패턴](https://www.youtube.com/watch?v=bxlNOb5PDS8&t=324s)