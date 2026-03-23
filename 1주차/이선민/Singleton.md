## Singleton 이란?
- 1회만 인스턴스화가 가능함
- 전역에서 접근 가능함

## Singleton 구현
### 클래스 형태
- 1회만 인스턴스화가 가능
- freeze 메서드를 통해 객체 직접 수정이 불가능하도록 함

```javascript
let counter = 0;
let instance;

class Counter {
    constructor(){
        if (instance) {
            throw new Error()
        }
        instance = this;
    }
    getInstance() {
        return this
    }

    getCount() {
        return counter
    }

    increment() {
        return ++counter;
    }

    decrement() {
        return --counter;
    }
}

const singletonCounter = Object.freeze(new Counter())
export default singletonCounter;
```

### 객체 리터럴
- 자바스크립트에서는 클래스보다 더 단순하게 객체 리터럴을 사용해서 동일한 구현을 할 수 있다.

```javascript
let count = 0;

const counter = {
    increment() {
        return ++count;
    }
    decrement() {
        return --count;
    }
}

Object.freeze(counter);
export {counter}
```

## 테스팅
- 싱글톤은 테스트가 좀 까다롭다
- 모든 테스트가 동일한 인스턴스를 사용하기 때문에 하나의 테스트가 끝나면 인스턴스 변경사항을 초기화 해주어야 한다.
- 그렇지 않으면 작은 수정으로 테스트 전체가 실패하는 결과를 초래할 수 있다.
- 또한 순서에 따라 통과/실패가 바뀔 수 있다.

```javascript
// counter.test.js
import counter from './counter';

test('increment 후 count는 1이다', () => {
  counter.increment();
  expect(counter.getCount()).toBe(1); // ✅ 통과
});

test('초기 count는 0이다', () => {
  expect(counter.getCount()).toBe(0); // ❌ 실패! 이전 테스트의 상태가 남아있음
});
```

## 의존성의 불명확
```javascript
// 이 import가 Singleton인지 일반 인스턴스인지 바로 파악하기 어려움
import counter from './superCounter';
```
- import 시 Singleton 여부가 코드만으로 드러나지 않음
- 예상치 못한 상태 변경으로 인해 디버깅하기 어려워질 수 있다.

## 전역 상태
- 여러 컴포넌트가 전역 상태를 직접 수정할 경우 **데이터 흐름 추적**이 어렵다
- 앱 규모가 커질 수록 어디서 값이 바뀌는지 알기 힘들다

## React의 상태 관리
- Redux, React Context
- Singleton과 유사해보이지만 이 도구들은 읽기 전용 상태를 제공함
- Redux를 예시로 들면, 상태 업데이트를 위해서는
  - 디스패쳐를 통해 넘긴 액션을 실행하는 순수함수 reducer에 의해서만 상태를 업데이트할 수 있다.

## Singleton 예시
### Toast, Modal 같은 알림 UI 컴포넌트
- Toast가 여러 인스턴스가 생성되어 버리면 큐가 분산되어서 중복 렌더링 등의 문제가 발생할 수 있다.

```javascript
// toast.js
class ToastManager {
  constructor() {
    if (ToastManager.instance) return ToastManager.instance;
    this.queue = [];
    ToastManager.instance = this;
  }

  show(message) {
    this.queue.push(message);
    this.render();
  }
}

export default new ToastManager();

// 어느 컴포넌트에서 호출해도 같은 큐에 쌓임
import toast from './toast';
toast.show('저장됐어요!');
```

### 웹소켓 연결 등..
```javascript
// socket.js
let instance = null;

export function getSocket() {
  if (!instance) {
    instance = new WebSocket('wss://api.example.com');
  }
  return instance;
}
```

### 모듈
- ES Module이나 CommonJS 모두 모듈 시스템은 기본적으로 module map을 기반으로 한번만 평가하기 때문에, 싱글톤으로 동작한다.
- 실제로 싱글톤으로 구현된건 아니지만, 싱글톤처럼 동작

**예시**
```
import counter from './counter.js'

1. 모듈 레지스트리에 './counter.js' 있나? 확인
        ↓ 없으면
2. 파일 로드 & 실행 → 결과를 레지스트리에 저장
        ↓ 있으면
3. 레지스트리에서 꺼내서 반환 (파일 재실행 안 함)
```