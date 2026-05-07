# 프록시 패턴

- 프록시 객체를 활용하면 특정 객체와의 인터렉션을 조금 더 컨트롤 할 수 있게 됩니다.
- 프록시 객체는 어떤 객체의 값을 설정하거나 값을 조회할 때 등의 인터렉션을 직접 제어할 수 있습니다.
  - 일반적으로도 "프록시"는 어떤 이의 대리인을 뜻합니다.
    - 그 사람과 직접 이야기하는 대신, 이야기를 원하는 사람의 대리인에게 이야기하는 것 입니다.
  - JavaScript에서도 해당 객체를 직접 다루는 것이 아니고 프록시 객체와 인터렉션 하게 됩니다.

## 예시
```ts
const person = {
  name: 'John Doe',
  age: 42,
  nationality: 'American',
}

const personProxy = new Proxy(person, {})

// personProxy
const personProxy = new Proxy(person, {
  get: (obj, prop) => {
    console.log(`The value of ${prop} is ${obj[prop]}`)
  },
  set: (obj, prop, value) => {
    console.log(`Changed ${prop} from ${obj[prop]} to ${value}`)
    obj[prop] = value
  },
})

```
- 객체와 직접 인터렉션 하는것 대신 personProxy를 통해 객체와 인터렉션 하게됩니다.
- 프록시는 유효성 검사를 구현할 때 유용합니다.
  ```ts
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
  ```
  - 객체를 실수로 수정하는 것을 예방해주어 데이터를 안전하게 관리할 수 있도록 해줍니다.

## Reflect
- JS에서는 Reflect 라는 빌트인 객체를 제공하는데, 프록시와 함께 사용하면 대상 객체를 쉽게 조작할 수 있습니다.
  - 위 예제에서는 프록시 핸들어 내에서 괄호 표기를 통해 직접 프로퍼티를 수정하거나 읽을 수 있었습니다.
  - Reflect 객체를 사용하면 Reflect 객체의 메서드는 핸들어 객체와 동일한 이름의 메서드를 가질 수 있습니다.
  ```ts
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

## 정리
- 프록시는 객체의 동작을 커스터마이징 할 수 있는 유용한 기능합니다.
- 프록시는 유효성 검사, 포메팅, 알림, 디버깅 등 유용하게 사용됩니다.
- 핸들러 객체에서 Proxy를 너무 헤비하게 사용하면 앱의 성능에 부정적인 영향을 줄 수 있습니다.
- Proxy를 사용할 때에는 성능문제가 생기지 않을 만한 코드를 사용합시다!!!

# 추가정리

- 객체에 대한 접근을 제어하기 위한 대리자(프록시)를 제공하는 디자인 패턴
> 사실 모두가 정말 많이 사용되고 있는데, 안쓴다고 착각하는중임

## 대표적인 프록시 예시
- useState
  - 우리는 state 원본에 접근해본 적이 없습니다.
  - 상태를 직접 변경하는것 자체가 리엑트에서는 허용되지 않는 구조입니다.
  - getter, setter에 의해서 수행되는 접근이 제어가 되어있는 형태인것이죠
  
## 핵심 개념
- 대리 객체를 통해 실제 객체에 대한 접근을 제어
- 실제 객체의 수정 없이 추가 기능을 제공
- Lazy initialzation, 접근제어, 로깅, 캐싱 등에 사용

## 장점
1. 성능 최적화: 불필요한 렌더링이나 로딩 방지
2. 보안 강화: 권한 없는 접근 차단
3. 관심사 분리: 핵심 로직과 부가 기능 분리
4. 디버깅 용이: 로깅, 모니터링 추가 용이

## 사용 예시

### Lazy Component
- React.lazy 자체가 프록시 역할입니다.
  ```tsx
  // React.lazy가 프록시 역할을 함
  const LazyComponent = React.lazy(() => import('./HeavyComponent'));

  function MyComponent() {
    return (
      <Suspense fallback={<div>Loading...</div>}>
        <LazyComponent />
      </Suspense>
    );
  }
  ```

### 접근 보호 (Access Control)
```tsx
function ProtectedRoute({ user, requiredRole, children }) {
  if (!user || user.role !== requiredRole) {
    return <Navigate to="/login" />;
  }

  return children; // 실제 컴포넌트에 접근 허용
}

// 사용 예
<ProtectedRoute user={currentUser} requiredRole="admin">
  <AdminDashboard />
</ProtectedRoute>
```

### 로깅 프록시 (HOC)
```tsx
function withLogging(WrappedComponent) {
  return function(props) {
    console.log(`Rendering ${WrappedComponent.name} with props:`, props);
    return <WrappedComponent {...props} />;
  }
}

const EnhancedComponent = withLogging(MyComponent);
```

### 이벤트 모니터링 프록시 (HOC)
```tsx
function withAnalytics(Component) {
  return function({ onClick, ...props }) {
    const handleClick = (e) => {
      console.log('Button clicked at:', new Date());
      if (onClick) onClick(e);
    };

    return <Component {...props} onClick={handleClick} />;
  };
}
```

### API 요청 프록시 (개발환경)
```ts
// setupProxy.js (Create React App)
const { createProxyMiddleware } = require('http-proxy-middleware');

module.exports = function(app) {
  app.use(
    '/api',
    createProxyMiddleware({
      target: 'http://localhost:5000',
      changeOrigin: true,
    })
  );
};
```