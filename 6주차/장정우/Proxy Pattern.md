# [Proxy Pattern](https://patterns-dev-kr.github.io/design-patterns/proxy-pattern/)

## [What] Proxy Pattern이란?

> "대상 객체와 직접 상호작용하는 대신, 중간에 대리인 객체를 두어 접근을 제어하는 패턴" _(claude)_

Proxy Pattern은 이름 그대로 "대리인" 역할을 하는 객체를 두어, 원본 객체에 대한 접근을 가로채고 제어한다.

현실 세계의 비유를 들면 **비서**와 같다. 사장에게 직접 가지 않고 비서를 거친다. 비서는 일정을 확인하고, 권한을 검증하고, 메모를 남긴다. 외부에서 보면 비서와 사장이 같은 일을 처리하는 것처럼 보이지만, 실제로는 비서가 그 사이에 끼어들어 부수적인 일을 처리한다.

이 패턴은 프로그래밍 전반에 걸쳐 다양한 형태로 등장한다.

- **HTTP Proxy** -- 클라이언트와 서버 사이의 중개자. 캐싱, 인증, 로깅 등을 담당한다.
- **Java RMI / gRPC stub** -- 원격 객체에 대한 로컬 대리인. 네트워크 통신을 추상화한다.
- **iOS의 NSProxy** -- 메시지 전달을 가로채는 추상 클래스.

이름이 다를 뿐, 모두 "원본에 직접 가지 않고 중간 대리인을 거친다"는 동일한 아이디어의 변형이다.

### Proxy의 4가지 종류 (GoF)

GoF는 Proxy를 의도에 따라 네 가지로 분류했다.

1. **Virtual Proxy** -- 무거운 객체의 생성을 지연시킨다. 실제 필요할 때까지 객체를 만들지 않는다.
2. **Protection Proxy** -- 접근 권한을 제어한다. 인증/인가 로직을 프록시에서 처리한다.
3. **Remote Proxy** -- 원격 객체의 로컬 대리인이다. 네트워크 호출을 추상화한다.
4. **Smart Reference Proxy** -- 추가 동작을 끼워넣는다. 참조 카운팅, 락, 캐싱 등.

JavaScript의 빌트인 `Proxy` 객체는 이 네 가지 시나리오를 모두 구현할 수 있는 강력한 메커니즘을 제공한다.   
어떤 종류의 프록시든 `Proxy` 객체와 핸들러(trap) 조합으로 표현할 수 있다.

## [Patterns.dev] Proxy Pattern 예시로 알아보기

### 기본 구조

`person`이라는 객체가 있다고 해보자.

```js
const person = {
  name: 'John Doe',
  age: 42,
  nationality: 'American',
}
```

이 객체에 직접 접근하는 대신, `Proxy`를 통해 접근하도록 만들 수 있다.

```js
const personProxy = new Proxy(person, {})
```

`Proxy` 클래스는 두 개의 인자를 받는다. 첫 번째는 **원본 객체(target)**, 두 번째는 **핸들러(handler)** 이다. 핸들러는 인터렉션 종류에 따른 동작을 정의하는 객체이다.

핸들러를 비워두면 프록시는 단순히 원본 객체를 그대로 전달하는 역할만 한다. 핸들러에 메서드를 정의하면, 그 메서드가 해당 인터렉션을 가로챈다.

대표적인 핸들러 메서드는 다음과 같다.

- **get** -- 프로퍼티에 접근할 때 실행
- **set** -- 프로퍼티 값을 수정할 때 실행

### 로깅 핸들러 적용

프로퍼티 접근/수정 시 로그를 남기는 프록시를 만들어보자.

```js
const personProxy = new Proxy(person, {
  get: (obj, prop) => {
    console.log(`The value of ${prop} is ${obj[prop]}`)
  },
  set: (obj, prop, value) => {
    console.log(`Changed ${prop} from ${obj[prop]} to ${value}`)
    obj[prop] = value
  },
})

personProxy.name        // The value of name is John Doe
personProxy.age = 43    // Changed age from 42 to 43
```

`person` 객체 자체는 변경하지 않았지만, 모든 접근 시점에 로그가 남는다. 이것이 Proxy의 핵심이다.

### 유효성 검사 적용

핸들러에 검증 로직을 넣으면, 잘못된 값의 할당을 막을 수 있다.

```js
const personProxy = new Proxy(person, {
  get: (obj, prop) => {
    if (!obj[prop]) {
      console.log(`Hmm.. this property doesn't seem to exist on the target object`)
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

personProxy.age = 'forty-three' 
personProxy.name = 'A'        
personProxy.name = 'Jane Doe'   
```

`person` 객체에는 검증 로직이 한 줄도 없지만, 프록시를 통해 접근하면 자동으로 검증이 수행된다. 비즈니스 로직과 검증 로직이 분리된다.

### Reflect 활용

`Reflect`는 JavaScript의 빌트인 객체로, 프록시 핸들러와 같은 시그니처의 메서드들을 제공한다. 핸들러 안에서 원본 객체를 다룰 때 `Reflect`를 사용하면 의도가 더 명확해진다.

```js
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

`obj[prop]` 대신 `Reflect.get(obj, prop)`을 사용한다. 동작은 동일하지만, "원본에 위임한다"는 의도가 더 분명히 드러난다.

### 활용 사례와 주의점

patterns.dev에서는 Proxy의 대표 활용 사례로 다음을 든다.

- 유효성 검사 (Validation)
- 포매팅 (Formatting)
- 알림 (Notification)
- 디버깅 (Debugging)

다만 한 가지 주의점이 있다. **핸들러 객체에서 Proxy를 너무 헤비하게 사용하면 앱의 성능에 부정적인 영향을 줄 수 있다.** 모든 프로퍼티 접근에 핸들러가 끼어들기 때문에, 빈번한 접근이 일어나는 객체에 무거운 검증 로직을 넣으면 성능 저하가 발생한다.

### 개인적인 활용 사례 — S3 presigned URL을 위한 API Route Proxy

Next.js로 작업할 때, 클라이언트에서 S3에 파일을 업로드하는 기능을 만든 적이 있다. 이때 **API Route를 Proxy로 활용**했다.

처음 떠올린 구조는 클라이언트가 AWS SDK로 S3와 직접 통신하는 방식이었다. 하지만 이렇게 하면 **AWS 자격증명을 브라우저에 노출**시켜야 한다는 치명적인 문제가 있다.

```
[브라우저] ──(AWS access key)──> [S3]   ❌ 자격증명 노출
```

대신 Next.js의 API Route가 클라이언트와 S3 사이의 **대리인** 역할을 하도록 구현했다.

```
[브라우저] ──> [Next.js API Route] ──> [S3]
                  │
                  ├─ 인증 검사
                  ├─ 권한 검증
                  └─ presigned URL 발급
```

```ts
// app/api/upload/route.ts
export async function POST(req: Request) {
  // 1. Protection Proxy — 인증/권한 검사
  const session = await getSession(req)
  if (!session) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // 2. Remote Proxy — AWS SDK는 서버에서만 사용
  const { fileName, fileType } = await req.json()
  const presignedUrl = await s3.getSignedUrl('putObject', {
    Bucket: process.env.S3_BUCKET,
    Key: `uploads/${session.userId}/${fileName}`,
    ContentType: fileType,
    Expires: 60,
  })

  // 3. 클라이언트는 이 URL로 S3에 직접 업로드
  return Response.json({ uploadUrl: presignedUrl })
}
```

## [How] Proxy Pattern은 어떻게 구현하는가?

핵심은 **원본 객체에 대한 접근을 가로채서, 추가 로직(검증, 로깅, 캐싱 등)을 수행한 후 원본에 위임하는 것**이다.

### 핸들러의 다양한 메서드

`get`, `set` 외에도 다양한 인터렉션을 가로챌 수 있다.

```js
const handler = {
  get(obj, prop) { /* 프로퍼티 접근 */ },
  set(obj, prop, value) { /* 프로퍼티 할당 */ },
  has(obj, prop) { /* in 연산자 */ },
  deleteProperty(obj, prop) { /* delete 연산자 */ },
  apply(target, thisArg, argList) { /* 함수 호출 */ },
  construct(target, argList) { /* new 연산자 */ },
  ownKeys(obj) { /* Object.keys, for...in */ },
}
```

![다운로드 (11).png](%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C%20%2811%29.png)

이 메서드들을 조합하면, 원본 객체에 대한 거의 모든 인터렉션을 통제할 수 있다.

### Vue 3의 반응성 시스템

Vue 3의 반응성 시스템은 사실상 Proxy 패턴의 대표적인 활용 사례이다. [예전에 정리한 거](https://so-tired.tistory.com/317)

```ts
import { ref } from 'vue'

const state = ref({ count: 0 })

// state.count++ 가 일어나면
// → Proxy의 set trap이 가로채서
// → 변경을 추적하고
// → 의존하는 컴포넌트에 리렌더링을 트리거
```

Vue 2에서는 `Object.defineProperty`로 반응성을 구현했지만, 새로 추가되는 프로퍼티를 감지할 수 없다는 한계가 있었다.   
Vue 3는 `Proxy`로 대체하여 이 문제를 해결했다. 모든 프로퍼티 접근/수정을 trap이 가로채기 때문에, 동적으로 추가되는 프로퍼티도 자연스럽게 반응성을 가진다.

## [Why] Proxy Pattern은 왜 사용하는가?

Q. 언제 사용하면 좋을까?
- 핵심은 **객체에 대한 접근을 제어하거나, 접근 시점에 부수 작업(로깅, 캐싱, 검증)을 수행하고 싶을 때.**

Q. 왜 직접 객체를 다루지 않고 Proxy를 쓰는가?

1. **관심사의 분리**
    - 비즈니스 로직(객체)과 횡단 관심사(검증, 로깅, 캐싱)를 분리할 수 있다. 객체는 자기 본연의 역할에 집중하고, 프록시가 부수적인 일을 처리한다.

2. **원본 코드 무수정 (OCP)**
    - 원본 객체를 수정하지 않고도 새로운 동작을 추가할 수 있다. "확장에는 열려 있고, 수정에는 닫혀 있다"는 개방-폐쇄 원칙을 자연스럽게 만족한다.

    ```js
    // 원본은 그대로 두고, 프록시로 검증을 추가
    const validatedUser = new Proxy(user, validationHandler)
    // 원본 user 객체에는 검증 로직이 들어가지 않는다
    ```

3. **지연 초기화**
    - 무거운 객체의 생성을 실제 필요한 시점까지 미룰 수 있다. 사용하지 않는 객체에 대한 초기화 비용을 절약한다.

4. **반응성 구현**
    - Vue 3의 반응성 시스템처럼, 객체의 변화를 감지하여 자동으로 후속 동작을 트리거할 수 있다. 이는 단순한 프록시 활용을 넘어 프레임워크의 근간이 되기도 한다.

Q. Proxy의 단점은 무엇인가?

- **성능 오버헤드**: 모든 프로퍼티 접근에 핸들러가 끼어들기 때문에, 직접 접근보다 느리다. 빈번한 접근이 일어나는 핫 패스(hot path)에 사용하면 성능 저하가 발생한다.
- **디버깅 복잡도**:  실제 객체와 프록시의 동작이 분리되어 있어, "왜 이 값이 바뀌었지?"를 추적할 때 핸들러까지 따라가야 한다. 디버거의 콜 스택도 복잡해진다.
- **호환성 제한**: 일부 빌트인 동작과 충돌할 수 있다. `instanceof` 체크, 일부 내장 객체(Date, Map 등)의 내부 슬롯 접근 등은 프록시로 감싸면 동작이 깨질 수 있다.
- **암묵적 동작**:  코드만 봐서는 프록시가 끼어 있는지 알기 어렵다. `user.name = 'A'`가 단순 할당인지, 검증을 거치는지, 어딘가에 알림을 보내는지 호출부에서 알 수 없다.

## [Hum..🤔] Proxy, Decorator, Adapter - 다 비슷한 거 아닌가?

이 셋은 모두 **"원본 객체를 감싸는(wrapping)" 구조 패턴**이다. 코드 형태도 비슷하다 보니 헷갈리기 쉽다. 하지만 의도가 다르다.

| 기준 | Proxy | Decorator | Adapter |
|:--|:--|:--|:--|
| 의도 | 접근 제어 | 기능 추가/확장 | 인터페이스 변환 |
| 인터페이스 변경 | 동일 | 동일 (확장됨) | 다른 인터페이스로 변환 |
| 원본 노출 | 본인 행세 (대체) | 본인 + 추가 기능 | 다른 형태로 변환 |
| 사용 시점 | 컴파일/런타임 | 런타임 | 런타임 |
| 예시 | 가상 프록시, 보호 프록시, Vue reactive | 로깅 데코레이터, 캐싱 데코레이터 | 외부 라이브러리 통합, 레거시 API 매핑 |

### 의도의 차이

- **Proxy**: "본인 행세를 하면서 접근을 통제한다."  
  외부에서는 원본인지 프록시인지 구분할 수 없다. 원본과 동일한 인터페이스를 유지하면서, 그 안에서 검증/로깅/캐싱 등을 처리한다.

- **Decorator**: "본인에 추가 기능을 덧붙이고 본인을 그대로 노출한다."  
  원본의 동작을 보존하면서, 그 위에 새로운 기능을 쌓는다. 여러 데코레이터를 중첩해서 적용할 수 있다.

- **Adapter**: "다른 인터페이스로 변환한다."  
  원본과 호출자의 인터페이스가 다를 때, 그 둘을 연결하는 변환 계층이다. 클라이언트는 원본의 형태를 모른다.

### 코드 형태로 비교

```ts
// Proxy: 동일 인터페이스, 접근 제어
const protectedUser = new Proxy(user, {
  set(obj, prop, value) {
    if (!hasPermission()) throw new Error('Access denied')
    return Reflect.set(obj, prop, value)
  },
})

// Decorator: 동일 인터페이스, 기능 추가
function withLogging<T extends Function>(fn: T): T {
  return ((...args: unknown[]) => {
    console.log('Called with', args)
    return fn(...args)
  }) as unknown as T
}

// Adapter: 다른 인터페이스로 변환
class LegacyAPI {
  fetchData(callback: (data: string) => void) { /* ... */ }
}

class ModernAPIAdapter {
  constructor(private legacy: LegacyAPI) {}

  // Promise 기반 인터페이스로 변환
  fetchData(): Promise<string> {
    return new Promise((resolve) => this.legacy.fetchData(resolve))
  }
}
```
