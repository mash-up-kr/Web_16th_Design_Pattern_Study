# [Module Pattern](https://patterns-dev-kr.github.io/design-patterns/module-pattern/)

## [What] Module Pattern이란?

> **모듈이란?**  
> 특정 기능을 수행하는 독립적인 부품이나 구성 요소 _(google)_

> 코드들을 재사용 가능하면서도 작게 나눌 수 있게 해주고, 내부의 복잡한 로직과 데이터를 숨기고 외부에 필요한 기능만을 노출할 수 있게 해준다 _(patterns.dev)_

## [Patterns.dev] Module Pattern 예시로 알아보기

**ES2015 이전의 JavaScript에는 모듈 시스템 자체가 없었다.**

그래서 `<script>` 태그로 파일을 여러 개 로드하면 모든 변수가 전역 스코프에 올라가기 때문에, 각 파일에서 선언한 이름간 충돌이 허다했다.
```html
<html>
  <script src="/a.js"></script>
  <script src="/b.js"></script>
</html>
```
```js
// a.js
var name = "Alice";

// b.js
var name = "Bob"; // 충돌! a.js의 name을 덮어씀
```

이런 이유로 개발자들은 이름 충돌 문제를 방지하기 위해 `객체 리터럴 방식`, `즉시 실행 함수`, `런타임 모듈 시스템`과 같은 언어 스펙 외적인 부분으로 해결하려고 했다.  

또한 스크립트 로드 시점을 사람이 일일이 관리해줘야 했다.

```html
<!-- 이건 에러남 -->
<html>
  <script src="/index.js"></script>
  <script src="/math.js"></script>
</html>

<!-- 이건 가능함, 하지만 불안정(ex. 번들러, async) -->
<html>
  <script src="/math.js"></script>
  <script src="/index.js"></script>
</html>
```
```js
// index.js
console.log(add(2, 3));

// math.js
function add(x, y) {
  return x + y;
}
```

`math.js`에서 선언된 함수들을 `index.js`에서 사용하려고 할때, 로드 순서가 맞지 않으면 `index.js` 에서 직접 사용하려고 할때 함수가 존재하지 않는다는 에러가 발생한다.

**그래서 ES2015에는 자바스크립트의 빌트인 모듈 기능이 추가되었다.**

ES2015 부터는  
1. `export/import`를 통해서 외부 파일에서 선언된 함수나 변수에 접근하는 것이 편리해졌고 `(외부에 필요한 기능만 드러냄)`
2. 명시적으로 export 하지 않는다면 변수나 함수를 private 하게 관리하는 것도 가능해졌다. `(내부의 로직은 숨김)`

`1`의 경우는 `named export`와 `default export`를 통해서 실현 가능하다.

```js
// math.js
export function add(x, y) {
  return x + y
}

export function multiply(x) {
  return x * 2
}

export function subtract(x, y) {
  return x - y
}

export function square(x) {
  return x * x
}

// index.js
import { add, multiply, subtract, square } from './math.js'

console.log(add(2, 3));
```

다음과 같이 `math.js` 에서 선언된 함수에 `export`를 붙여주면, 외부 파일에서도 `math.js`선언한 함수들에 접근이 가능해진다.  
`named export` 방식의 경우엔 외부 파일에서 선언한 변수/함수의 이름과 동일하게 import 해와야 하며, `as`를 통해서 이름을 수정하여 사용할 수 있기도 하다.
```js
// as 키워드의 사례
import {
  add as addValues,
  multiply as multiplyValues,
  subtract,
  square,
} from './math.js'

console.log((addValues(7, 8));
```

또한 `export default`를 통해서도 변수나 함수를 외부에서 사용할 수 있게 할 수 있다.  
단 `export default`는 폴더에서 딱 하나만 사용가능 하다.

```js
// math.js
export default function add(x, y) {
  return x + y
}

export function multiply(x) {
  return x * 2
}

export function subtract(x, y) {
  return x - y
}

export function square(x) {
  return x * x
}

// index.js
import add, { multiply, subtract, square } from './math.js'

console.log(add(2, 3));
```

`export default`의 경우 객체 형태로 export해 프로퍼티나 구조분해를 통해 값을 사용할수도 있다.
```js
// a.js
const foo = 1;
const bar = 2;

export default { foo, bar };

// b.js
import obj from './module';
console.log(obj.foo, obj.bar);

// c.js
import obj from './module';
const { foo, bar } = obj;
```

`2`의 경우는 export 하지 않을 경우 외부에서 접근을 방지 할 수 있다.

```js
//math.js
const privateValue = 'This is a value private to the module!'

export function add(x, y) {
  return x + y
}

export function multiply(x) {
  return x * 2
}

export function subtract(x, y) {
  return x - y
}

export function square(x) {
  return x * x
}

// index.js
import { add, multiply, subtract, square } from './math.js'

console.log(privateValue)
/* Error: privateValue is not defined */
```

마지막으로 특정 조건에서만 특정 모듈을 로드해야 할 때가 있는데, `Dynmic import`를 사용하면 필요할 때만 로드할 수 있다.

```js
import('module').then(module => {
  module.default()
  module.namedExport()
})

// Or with async/await
;(async () => {
  const module = await import('module')
  module.default()
  module.namedExport()
})()
```

정리하자면, 모듈 패턴을 사용하면 코드의 일부분을 캡슐화 할 수 있다.  
이는 의도치 않은 전역 변수 할당을 예방할 수 있어 여러 의존 모듈을 사용하거나 네임스페이스를 사용할 때 안전하다.

## [How] Module Pattern은 어떻게 구현하는가?

핵심은 **코드를 캡슐화하여 내부 구현은 숨기고, 외부에는 필요한 기능만 노출하는 것**이다.  
JS를 사용할땐 이를 `ES Module`을 활용해 `export/import`를 적절히 사용하면 된다.

사례는 위의 `[Patterns.dev] Module Pattern 예시로 알아보기`를 참고하자.

## [Why] Module Pattern은 왜 사용하는가?

Q. 언제 사용되면 좋을까?
- 코드를 캡슐화하여 내부 구현은 숨기고, 외부에는 필요한 인터페이스만 노출하고 싶을 때.


Q. 왜 캡슐화가 필요한가?
> 관심사의 분리(Separation of Concerns)를 통해 코드의 유지보수성과 안전성을 높이기 위해
- 이름 충돌 방지
- 의도하지 않은 접근과 수정 방지
- 의존성을 명시적으로 관리 & 트리 셰이킹 용이
- 재사용성과 테스트 용이

## [Question] Module과 관련하여 궁금한 점

Q. 다들 컴포넌트를 선언할때 named export 패턴과 default export 중 어떤 방식을 선호하는지?  
Q. 배럴 파일(Barrel file)에 대해서 어떻게 생각하는지?

상황 1.
```md
Tabs/
├── Tabs.tsx
├── TabsContent.tsx
├── TabsList.tsx
├── TabsTrigger.tsx
└── index.ts
```

```js
// 배럴 있을 때
import { Tabs, TabsList, TabsTrigger, TabsContent } from '@/components/Tabs';

// 배럴 없을 때
import { Tabs } from '@/components/Tabs/Tabs';
import { TabsContent } from '@/components/Tabs/TabsContent';
import { TabsList } from '@/components/Tabs/TabsList';
import { TabsTrigger } from '@/components/Tabs/TabsTrigger';
```

상황 2.
```md
components/
├── Button/
│   └── Button.tsx
├── Input/
│   └── Input.tsx
├── Modal/
│   └── Modal.tsx
└── index.ts 
```
```js
// 배럴 있으면
import { Button, Input, Modal } from '@/components';

// 배럴 없이
import { Button } from '@/components/Button/Button';
import { Input } from '@/components/Input/Input';
import { Modal } from '@/components/Modal/Modal';
```