# Proxy 패턴

## 정의

Proxy 패턴은 **특정 객체와의 상호작용을 더 세밀하게 제어할 수 있게 하는 구조 패턴**이다. 원본 객체를 직접 다루는 대신, 중간에 위치한 **프록시(proxy) 객체**를 거쳐 접근함으로써 객체의 동작을 조정할 수 있다.

> 핵심 개념: 프록시는 "어떤 사람의 대리인" 역할을 한다. 그 사람과 직접 대화하지 않고 중간자를 거쳐 상호작용하는 셈이다.

## JavaScript의 Proxy 객체

### 기본 구조

```javascript
const proxy = new Proxy(targetObject, handler)
```

- **targetObject**: 원본 객체
- **handler**: 인터셉터 메서드(트랩)가 포함된 객체

### 기본 예제

```javascript
const person = {
  name: 'John Doe',
  age: 42,
  nationality: 'American',
}

const personProxy = new Proxy(person, {})
```

`handler`가 비어있으면 일반 객체처럼 동작한다. 핸들러에 트랩을 정의해야 동작을 가로챌 수 있다.

## get과 set 핸들러

### get 핸들러

프로퍼티에 **접근**할 때 실행되는 메서드이다.

```javascript
const personProxy = new Proxy(person, {
  get: (obj, prop) => {
    console.log(`The value of ${prop} is ${obj[prop]}`)
  },
})

personProxy.name
// 출력: "The value of name is John Doe"
```

### set 핸들러

프로퍼티 값을 **수정**할 때 실행되는 메서드이다.

```javascript
const personProxy = new Proxy(person, {
  set: (obj, prop, value) => {
    console.log(`Changed ${prop} from ${obj[prop]} to ${value}`)
    obj[prop] = value
  },
})

personProxy.age = 43
// 출력: "Changed age from 42 to 43"
```

## Validation (유효성 검사)

Proxy는 객체에 **유효성 검사 로직**을 붙일 때 유용하다.

```javascript
const personProxy = new Proxy(person, {
  get: (obj, prop) => {
    if (!obj[prop]) {
      console.log(
        `Hmm.. this property doesn't seem to exist on the target object`
      )
    } else {
      console.log(`The value of ${prop} is ${obj[prop]}`)
    }
  },
  set: (obj, prop, value) => {
    if (prop === 'age' && typeof value !== 'number') {
      console.log(`Sorry, you can only pass numeric values for age.`)
    } else if (prop === 'name' && value.length < 2) {
      console.log(`You need to provide a valid name.`)
    } else {
      console.log(`Changed ${prop} from ${obj[prop]} to ${value}.`)
      obj[prop] = value
    }
  },
})

personProxy.age = 'invalid'  // "Sorry, you can only pass numeric values for age."
personProxy.name = 'A'       // "You need to provide a valid name."
```

이렇게 하면 잘못된 값이 객체에 들어가는 것을 사전에 막을 수 있다.

## Reflect 객체

JavaScript의 `Reflect` 객체는 Proxy와 함께 사용하면 대상 객체를 더 안전하고 효과적으로 조작할 수 있다.

### Reflect.get() / Reflect.set()

```javascript
const personProxy = new Proxy(person, {
  get: (obj, prop) => {
    console.log(`The value of ${prop} is ${Reflect.get(obj, prop)}`)
  },
  set: (obj, prop, value) => {
    console.log(`Changed ${prop} from ${obj[prop]} to ${value}`)
    Reflect.set(obj, prop, value)
  },
})
```

Reflect 메서드는 핸들러 메서드와 동일한 인자를 사용하며, `obj[prop]`처럼 직접 접근하는 방식보다 일관되고 안전하다.

### 왜 `Reflect`가 직접 접근보다 안전한가?

핵심은 **`this` 바인딩(receiver)** 과 **반환값** 두 가지이다.

#### 1. `receiver` 인자 — `this` 바인딩 문제

`get`/`set` 트랩은 사실 네 번째 인자로 `receiver`를 받는다. 이게 **상속/프로토타입 체인**에서 중요하다.

```javascript
const parent = {
  get fullInfo() {
    return `${this.name} (${this.age})`
  },
}

const child = { name: 'John', age: 42 }
Object.setPrototypeOf(child, parent)

const proxy = new Proxy(child, {
  get: (obj, prop, receiver) => {
    // ❌ 직접 접근: parent.fullInfo()내에서 this가 obj(child)로 바인딩됨 → 프록시 우회
    return obj[prop]

    // ✅ Reflect.get: receiver를 넘겨주면 parent.fullInfo()내에서 this가 proxy로 바인딩됨
    return Reflect.get(obj, prop, receiver)
  },
})
```

`getter` 안의 `this`가 **프록시를 거쳐야** 다른 트랩(예: 로깅, validation)이 다시 동작한다. 직접 `obj[prop]`로 접근하면 프록시를 우회하기 때문에 트랩이 호출되지 않는다.

#### 2. 반환값 — 실패를 감지할 수 있음

```javascript
// ❌ 직접 할당: 실패해도 알 수 없음 (frozen, non-writable 등)
obj[prop] = value

// ✅ Reflect.set: 성공/실패를 boolean으로 반환
const success = Reflect.set(obj, prop, value)
if (!success) {
  // 실패 처리 가능
}
```

특히 `set` 트랩은 **boolean을 반환해야** 한다는 규칙이 있어, strict mode에서 `false`를 반환하면 `TypeError`가 발생한다. `Reflect.set`은 이 값을 그대로 돌려주므로 트랩의 반환값으로 그대로 쓰기 좋다.

```javascript
set: (obj, prop, value, receiver) => {
  return Reflect.set(obj, prop, value, receiver)  // boolean 반환
}
```

#### 3. 일관된 인터페이스

`Reflect`의 메서드들은 Proxy 트랩과 **이름·시그니처가 1:1로 대응**된다 (`get`, `set`, `has`, `deleteProperty`, `ownKeys` ...). 그래서 "트랩에서 기본 동작을 그대로 수행한다"는 의도를 명확하게 표현할 수 있다.

```javascript
// in 연산자에 해당
has: (obj, prop) => Reflect.has(obj, prop)

// delete 연산자에 해당
deleteProperty: (obj, prop) => Reflect.deleteProperty(obj, prop)
```

`delete obj[prop]` 같은 연산자 문법은 트랩 내부에서 쓰기 어색한데, `Reflect`는 함수 형태라 깔끔하다.

#### 정리

| 직접 접근 (`obj[prop]`) | `Reflect.get/set` |
| --- | --- |
| `this`가 항상 원본 객체로 바인딩 | `receiver`로 프록시 전달 가능 |
| 할당 성공 여부 알 수 없음 | boolean 반환 |
| 연산자 문법(`delete`, `in`) 혼합 | 함수 호출로 통일 |

요약하면, **프록시 체인이 단순할 때는 차이가 안 보이지만, 상속/중첩 프록시가 들어오면 `Reflect`를 써야 의도대로 동작**한다. MDN에서도 트랩 안에서는 `Reflect`로 기본 동작을 호출하는 것을 권장한다.

## 활용 사례

- **유효성 검사**: 데이터 타입 및 형식 검증
- **포매팅**: 값의 형태 변환 및 정규화
- **알림(Notification)**: 객체가 변경될 때 알림 발생
- **디버깅**: 객체 접근 추적 및 로깅

## 장점

- 객체의 동작을 **세밀하게 제어**할 수 있다.
- 데이터를 **안전하게 관리**할 수 있다 (validation, 접근 제어).
- 중복 코드를 줄이고, 공통 로직(로깅, 검증)을 한 곳에서 처리할 수 있다.

## 주의사항

- **성능 오버헤드**: 핸들러 객체에서 Proxy를 너무 헤비하게 사용하면 앱의 성능에 부정적인 영향을 줄 수 있다.
- 따라서 정말 필요한 곳에만 **선택적으로** 적용하는 것이 좋다.

## 참고

- [patterns-dev-kr - Proxy Pattern](https://patterns-dev-kr.github.io/design-patterns/proxy-pattern/)
- [MDN - Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
