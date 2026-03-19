# 2. 모듈 패턴

## 2-1. 정의

관련 있는 데이터와 기능을 하나의 단위로 묶고, 외부에 공개할 것과 숨길 것을 구분하여 캡슐화를 구현 하는 패턴

* 정보 은닉 — 내부 상태와 구현을 외부에서 직접 접근하지 못하게 숨김
* 공개 인터페이스 제공 — 필요한 기능만 제한적으로 외부에 노출
* 응집도 향상 — 관련된 상태와 동작을 하나의 단위로 묶음

내부 구현은 감추고, 외부에는 필요한 사용 방법만 제공하는 것이 본질

## 2-2. 필요성

코드 규모가 커질수록 발생되는 문제들

ex ) 상태,함수 분산.. 전역 변수 오염.. 내부 데이터 외부 접근.. 어떤 기능이 어떤 상태를 관리하는지 파악..

모듈 패턴을 사용하면 상태는 숨기고, 조작 가능한 메서드만 공개 가능. 관련 상태와 기능을 안전하게 묶을 수 있음

## 2-3. 동작원리

1. 하나의 독립된 스코프를 만듦
   * 함수 또는 모듈 파일이 범위를 형성
2. 내부 변수와 내부 함수를 해당 스코프 안에 정의
   * 외부에서 직접 접근 불가
3. 외부에 보여줄 기능만 객체 또는 export로 공개 ( Public API )
   * 공개된 함수는 클로저를 통해 내부 상태 접근 가능
4. 외부는 상태를 직접 건드릴 수 없고, 공개 메소드를 통해서만 간접적으로 접근

```js
const counterModule = (() => {
  let count = 0;

  function validate(value) {
    return typeof value === 'number' && value >= 0;
  }

  function log() {
    console.log(`현재 값: ${count}`);
  }

  return {
    increment() {
      count += 1;
      log();
    },
    decrement() {
      if (count > 0) {
        count -= 1;
      }
      log();
    },
    set(value) {
      if (validate(value)) {
        count = value;
      }
      log();
    },
    getCount() {
      return count;
    },
  };
})();

counterModule.increment(); // 현재 값: 1
counterModule.increment(); // 현재 값: 2
console.log(counterModule.getCount()); // 2
console.log(counterModule.count); // undefined
```

- IIFE 함수 가 실행되면서 스코프 생성
- count, validate, log는 스코프 안에만 존재
- 외부는 공개된 increment, decrement, set, getCount만 접근 가능
- 외부에서는 count를 직접 수정할 수 없고, 공개된 메소드를 통해서만 상태 변경 가능

⇒ 스코프로 숨기고, 클로저로 유지

## 2-4. 모듈 패턴 실제 활용 — Redux

Redux는 아키텍처가 모듈 패턴 그 자체…

```js
function createStore(reducer, preloadedState) {
  let state = preloadedState;    // 클로저 안에 갇힘 — 외부 접근 불가
  const listeners = [];           // 클로저 안에 갇힘 — 외부 접근 불가

  function getState() {
    return state;
  }

  function dispatch(action) {
    state = reducer(state, action);        // state를 바꾸는 유일한 경로
    listeners.forEach(listener => listener());  // 변경 후 구독자에게 알림
  }

  function subscribe(listener) {
    listeners.push(listener);
    return function unsubscribe() {
      const index = listeners.indexOf(listener);
      listeners.splice(index, 1);
    };
  }

  dispatch({ type: '@@redux/INIT' });

  return { getState, dispatch, subscribe };
}
```

반환되는건 getState, dispatch, subscribe 세 개 뿐, state listeners는 반환 객체에 없어 아예 접근할 방법이 없음

**getState** 상태를 읽기만 하고, 수정 불가능

```js
const current = store.getState();
console.log(current.count);  // 읽기 OK

store.getState().count = 999; // 객체 참조라 값은 바뀌긴 하지만
                               // dispatch를 안 거쳤으므로
                               // 구독자 알림 안 감, DevTools에 안 잡힘
                               // → Redux가 이걸 "위반"으로 간주
```

**dispatch(action)** 상태를 바꾸는 유일한 공식 경로

```js
store.dispatch({ type: 'INCREMENT' });
// 1. reducer가 호출됨 → 새 state 생성
// 2. listeners 전부 호출됨 → UI 업데이트
// 3. DevTools가 이 action을 기록함
```

이 흐름이 강제되는 이유가 클로저 때문. state에 직접적인 접근이 안되니 dispatch를 거침

**subscribe(listener)** 상태 변경 알림을 받는 등록 함수, unsubscribe 함수를 반환하는 것도 클로저

```js
const unsubscribe = store.subscribe(() => {
  console.log('state changed:', store.getState());
});

// 나중에 구독 해제
unsubscribe();  // listeners 배열에서 제거됨
```

unsubscribe가 클로저로 listener와 listeners 배열을 기억하고 있어서, 정확히 자기 자신만 제거할 수 있음.

### 이게 왜 "아키텍처 자체"인가

Redux가 보장하려는 것 세 가지가 전부 이 클로저 구조에 의존함:

**1) 상태 변경 추적**

```
dispatch({ type: 'INCREMENT' })
  → reducer(현재state, action) 실행
  → 새 state 반환
  → listeners 호출
```

모듈화를 통해 이 흐름이 매번 동일하게 실행. state가 외부에 노출되어 참조한다면 추적이 깨져버림

**2) 미들웨어 체인**

이 흐름이 보장받을 수 있기에 미들웨어 체인 추가 가능. dispatch로만 접근이 가능하니, 앞에 명시적으로 동작을 넣을 수 있음!

그래서 Redux 미들웨어 ( redux-thunk, redux-saga 등 ) 이 적용 가능

```js
// redux-thunk 간략 구조
function thunkMiddleware(store) {
  const originalDispatch = store.dispatch;

  return function(action) {
    if (typeof action === 'function') {
      return action(store.dispatch, store.getState);
    }
    return originalDispatch(action);
  };
}
```

모든 상태 변경이 dispatch를 통하니까 미들웨어가 그 사이에 끼어들 수 있음. state를 직접 바꿔버리면 미들웨어가 개입할 틈이 없음.

**3) 시간여행 디버깅**

Redux DevTools가 "이전 상태로 되돌리기"를 할 수 있는 이유:

```
action 1: { type: 'INCREMENT' } → state: { count: 1 }
action 2: { type: 'INCREMENT' } → state: { count: 2 }
action 3: { type: 'DECREMENT' } → state: { count: 1 }

// "action 2로 되돌리기" → reducer에 action 1, 2만 다시 적용 → { count: 2 }
```

모든 변경이 dispatch(action) ⇒ reducer 경로이기때문에 action 목록만 있으면 어떤 시점의 state 재현 가능 반면에 state를 직접 수정한다면 action으로 기록이 안됨.. 재현 불가능

Redux에서는 모듈 패턴 (클로저) 를 단순한 캡슐화로 사용하지 않음

state를 감춰서 ⇒ dispatch가 유일한 변경 경로가 됨 ⇒ 모든 변경에 reducer 개입 ⇒ 진입 경로가 명확해 미들웨어 낄 수 있음 ⇒ action 로그 보존 ⇒ 시간 여행 디버깅 가능 하도록 사용

⇒ 모듈화가 없다면 아예 성립할 수 없는 구조

Redux 레포지토리 : https://github.com/reduxjs/redux/blob/master/src/createStore.ts

## 2-5. 트레이드오프

**얻는 것**
* 캡슐화 — 내부 상태 구현 숨기고, 필요한 기능만 공개
* 응집도 향상 — 관련 있는 상태와 기능을 한 곳에 묶음
* 유지 보수 향상 — 내부 구현이 바뀌어도 외부 사용 코드는 그대로 유지
* 전역 오염 방지 — 전역 변수 남발을 줄이고 안전한 구조 만들기

**잃는 것**
* 지나친 은닉 — 사용성과 확장성이 떨어질 수 있음
* 내부 상태 확인 어려움 — 디버깅이나 테스트 시 내부 값 보기 어려움
