## Module 패턴이란?
- 코드를 재사용 가능하고 독립적인 단위로 분리하는 패턴
- 명시적으로 export한 값만 외부에 노출되어 **캡슐화**가 가능
- 전역 스코프 오염과 네임스페이스 충돌을 방지할 수 있다

## ES2015 모듈

### Named Export
- 여러 값을 이름 기반으로 내보낼 수 있다

```javascript
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
```

```javascript
import { add, multiply, subtract, square } from './math.js'
```

### Default Export
- 모듈당 하나의 기본 내보내기
- import 시 중괄호 없이 가져올 수 있고, 이름을 자유롭게 지정할 수 있다

```javascript
export default function add(x, y) {
  return x + y
}

export function multiply(x) {
  return x * 2
}
```

```javascript
import add, { multiply } from './math.js'

// 이름 변경도 가능
import addValues, { multiply } from './math.js'
```

### Private 변수
- export하지 않은 변수는 모듈 내에서만 접근 가능
- 전역 스코프에 의도치 않게 변수를 추가하는 것을 방지할 수 있다

```javascript
const privateValue = 'This is a value private to the module!'

export function add(x, y) {
  return x + y
}
```

### 이름 충돌 해결 (as 키워드)
- 기존 변수명과 충돌할 경우 `as` 키워드로 이름을 변경할 수 있다

```javascript
import {
  add as addValues,
  multiply as multiplyValues,
  subtract,
  square,
} from './math.js'

// 로컬에 동일한 이름의 함수가 있어도 충돌하지 않음
function add(...args) {
  return args.reduce((acc, cur) => cur + acc)
}
```

### 와일드카드 Import
- `*`를 사용해 모듈의 모든 export를 가져올 수 있다

```javascript
import * as math from './math.js'

math.default(7, 8)    // default export 호출
math.multiply(8, 9)
math.subtract(10, 3)
math.square(3)
```

## Dynamic Import (동적 모듈 로딩)
- 파일 상단에서 import하면 다른 코드보다 먼저 로드됨
- 동적 import는 **필요할 때만** 모듈을 로드하여 페이지 로딩 타임을 줄일 수 있다

```javascript
// Promise 방식
import('module').then(module => {
  module.default()
  module.namedExport()
})

// async/await 방식
const module = await import('module')
module.default()
module.namedExport()
```

### 사용자 이벤트 기반 로드
- 버튼 클릭 등 특정 이벤트 시점에 모듈을 로드할 수 있다

```javascript
button.addEventListener('click', async () => {
  const module = await import('module')
  module.someFunction()
})
```

### 템플릿 리터럴을 활용한 동적 경로
```javascript
const res = await import(`../assets/dog${num}.png`)
```

## 장점
- **캡슐화**: export한 값만 외부에 노출되므로 내부 구현을 숨길 수 있다
- **이름 충돌 방지**: 모듈 스코프 덕분에 전역 스코프 오염이 없다
- **재사용성**: 독립적인 단위로 분리되어 어디서든 재사용 가능
- **성능 최적화**: 동적 import를 통해 코드 분할(Code Splitting) 가능
- **유지보수성**: 코드를 논리적 단위로 관리할 수 있다

## 주의사항
- 와일드카드(`*`) import 시 불필요한 코드까지 포함될 수 있다 (Tree Shaking에 불리)
- 모든 런타임에서 ES2015 모듈을 지원하려면 Babel 같은 트랜스파일러가 필요하다
