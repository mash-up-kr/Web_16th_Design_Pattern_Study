# 옵저버 패턴
- 옵저버 패턴에서는 특정 객체를 구독할 수 있는데, 구독하는 주체를 옵저버라 하고, 구독 가능한 객체를 Observable 이라고 합니다.
- 이벤트가 발생할 때 마다 Observable은 모든 Observer에게 이벤트를 전파합니다.

## Observable
- Observable 객체는 보통 3가지 주요 특징을 포함합니다.
  - observers: 이벤트가 발생할 때마다 전파할 Observer들의 배열
  - subscribe(): Observer를 observers 배열에 추가한다.
  - unsubscribe(): observers 배열에서 Observer를 제거한다.
  - notify(): 등록된 모든 Observer들에게 이벤트를 전파한다.

## 예제
```ts
class Observable {
  constructor() {
    this.observers = []
  }

  subscribe(func) {
    this.observers.push(func)
  }

  unsubscribe(func) {
    this.observers = this.observers.filter(observer => observer !== func)
  }

  notify(data) {
    this.observers.forEach(observer => observer(data))
  }
}
```
- subscribe 메서드를 통해 Observer를 등록하고 반대로 unsubscribe를 통해 등록 해지를 할 수 있습니다.
- 그리고 notify 메서드를 통해 모은 Observer에게 이벤트를 전파할 수 있습니다.
- 구현한 Observable 객체를 이용하여 무언가를 만들어보겠습니다.
  ```tsx
  export default function App() {
    return (
      <div className="App">
        <Button>Click me!</Button>
        <FormControlLabel control={<Switch />} />
      </div>
    )
  }
  ```
  - 앱 내에서 일어나는 사용자 인터렉션을 추적하는 예시입니다.
    - 사용자가 버튼을 클릭하거나
    - 스위치를 토글하거나
    - 이벤트가 발생하는 순간 타임스탬프를 로깅하면서 이벤트 발생시 마다 토스트 알림을 화면에 노출하려고 합니다.
    <iframe src="https://patterns-dev-kr.github.io/design-observer01.mp4" width="100%" height="400" frameborder="0" allowfullscreen></iframe>
    - 사용자가 컴포넌트 핸들러를 호출할 때 마다 핸들러는 Observable의 notify를 호출합니다.
    - notify 메서드는 등록된 모든 Observer에게 핸들러에서 전달된 데이터를 포함한 이벤트를 전파합니다.
      - 각 함수들이 Observer로써 동작하기 위해서는 Observable의 subscribe 메서드를 사용해야 합니다.
  ```tsx
  import { ToastContainer, toast } from 'react-toastify'

  function logger(data) {
    console.log(`${Date.now()} ${data}`)
  }

  function toastify(data) {
    toast(data)
  }

  observable.subscribe(logger)
  observable.subscribe(toastify)

  export default function App() {
    function handleClick() {
      observable.notify('User clicked button!')
    }

    function handleToggle() {
      observable.notify('User toggled switch!')
    }

    return (
      <div className="App">
        <Button>Click me!</Button>
        <FormControlLabel control={<Switch />} />
        <ToastContainer />
      </div>
    )
  }
  ```

## 정리
- Observer 패턴은 다양하게 활용할 수 있지만 비동기 호출 혹은 이벤트 기반 데이터를 처리할 때 매우 유용합니다.
- 만약 어떤 컴포넌트가 특정 데이터의 다운로드 완료 알림을 받기 원하거나, 사용자가 메세지 보드에 새로운 메세지를 게시했을 때 모든 멤버가 알림을 받거나 하는 등의 상황에 사용됩니다.

## 장점
- Observer 패턴을 사용하는 것도 관심사 분리와 단일 책임의 원칙을 강제하기 위한 좋은 방법입니다.
- Observer 객체는 Observable 객체와 강결합 되어있지 않고 언제든지 분리될 수 있습니다. Observable 객체는 이벤트 모니터링의 역할을 갖고, Observer는 받은 데이터를 처리하는 역할을 갖게 됩니다.

## 단점
- Observer가 복잡해지면 모든 Observer들에 알림을 전파하는데 성능 이슈가 발생할 수 있습니다.

---
# 추가 정리
- 객체의 상태 변화를 관찰하는 관찰자들(옵저버) 에게 자동으로 `알림`을 보내는 디자인 패턴 입니다.
  > like 유튜버를 구독하고 알림을 받는 관찰자인 우리
  > 유저에게 좋댓구알을 받는 유튜버

## 프론트엔드에서의 아이디어
1. 상태 변화 감지: 컴포넌트가 특정 상태 변화를 감지하고 반응해야 할 때
2. 비동기 이벤트 처리: API 응답 및 사용자 인터랙션 등의 이벤트 처리
3. 컴포넌트 간 통신: 부모-자식 관계가 아닌 컴포넌트 간의 데이터 동기화
4. 전역 상태 관리: Redux, Context API 등이 내부적으로 옵저버 패턴 사용

### 여기서 잠깐!
- 상태관리 라이브러리 코드를 직접 살펴보면 subscribe라는 메서드가 자주 등장하게 됩니다.
- 대부분의 상태관리 라이브러리에는 필연적으로 존재합니다.


## 예시 코드
- 다크모드 변경
  ```tsx
  const createObserver = () => {
    let observers = [];

    return {
      subscribe: (callback) => {
        observers.push(callback);
        return () => {
          observers = observers.filter(observer => observer !== callback);
        };
      },
      notify: (data) => {
        observers.forEach(observer => observer(data));
      }
    };
  };

  // 사용 예시: 다크 모드 토글 관찰
  const themeObserver = createObserver();

  const useThemeObserver = () => {
    const [isDarkMode, setIsDarkMode] = useState(false);

    useEffect(() => {
      const unsubscribe = themeObserver.subscribe((newTheme) => {
        setIsDarkMode(newTheme);
      });
      return unsubscribe;
    }, []);

    const toggleTheme = () => {
      const newTheme = !isDarkMode;
      themeObserver.notify(newTheme);
    };

    return { isDarkMode, toggleTheme };
  };

  // 컴포넌트 A: 테마 토글 버튼
  const ThemeToggleButton = () => {
    const { toggleTheme } = useThemeObserver();
    return <button onClick={toggleTheme}>Toggle Theme</button>;
  };

  // 컴포넌트 B: 테마 적용 UI
  const ThemedComponent = () => {
    const { isDarkMode } = useThemeObserver();
    return (
      <div style={{
        background: isDarkMode ? '#333' : '#fff',
        color: isDarkMode ? '#fff' : '#333',
        padding: '20px'
      }}>
        Current Theme: {isDarkMode ? 'Dark' : 'Light'}
      </div>
    );
  };
  ```
  - 사실 다크모드를 변경하는데 이정도의 옵저버 패턴을 사용하는것 자체가 오버엔지니어링이기도 합니다.
  - 하지만 이해를 돕기 위해 가장 쉬운 예시로 가져와보았습니다.

- ContextAPI + useReducer를 조합한 형태

  ```tsx
  // 옵저버 리듀서를 만듭니다. 지이이이잉
  const ObserverContext = createContext();

  const observerReducer = (state, action) => {
    switch (action.type) {
      case 'SUBSCRIBE':
        return {
          ...state,
          observers: [...state.observers, action.payload]
        };
      case 'UNSUBSCRIBE':
        return {
          ...state,
          observers: state.observers.filter(observer => observer.id !== action.payload)
        };
      case 'NOTIFY':
        state.observers.forEach(observer => {
          if (observer.event === action.payload.event) {
            observer.callback(action.payload.data);
          }
        });
        return state;
      default:
        return state;
    }
  };

  // useReducer의 dispatch함수로 옵저버의 subscribe, notify등이 호출될 수 있도록 바인딩을 해줍니다.
  // 그리고 하나의 context로 wrapping을 진행해줍니다.
  const ObserverProvider = ({ children }) => {
    const [state, dispatch] = useReducer(observerReducer, { observers: [] });

    const value = {
      subscribe: (event, callback) => {
        const id = Date.now();
        dispatch({
          type: 'SUBSCRIBE',
          payload: { id, event, callback }
        });
        return () => dispatch({ type: 'UNSUBSCRIBE', payload: id });
      },
      notify: (event, data) => {
        dispatch({ type: 'NOTIFY', payload: { event, data } });
      }
    };

    return (
      <ObserverContext.Provider value={value}>
        {children}
      </ObserverContext.Provider>
    );
  };

  // 사용 예시: 알림 시스템
  const NotificationButton = () => {
    const { notify } = useContext(ObserverContext);

    const handleClick = () => {
      notify('NOTIFICATION', {
        id: Date.now(),
        message: 'Button was clicked!',
        type: 'info'
      });
    };

    return <button onClick={handleClick}>Trigger Notification</button>;
  };


  // 이곳에서는 subscribe를 통해 알림을 확인하기만 하면 됩니다.
  const NotificationDisplay = () => {
    const [notifications, setNotifications] = useState([]);
    const { subscribe } = useContext(ObserverContext);

    useEffect(() => {
      const unsubscribe = subscribe('NOTIFICATION', (data) => {
        setNotifications(prev => [...prev, data]);
        setTimeout(() => {
          setNotifications(prev => prev.filter(n => n.id !== data.id));
        }, 3000);
      });
      return unsubscribe;
    }, [subscribe]);

    return (
      <div className="notification-container">
        {notifications.map(notification => (
          <div key={notification.id} className={`notification ${notification.type}`}>
            {notification.message}
          </div>
        ))}
      </div>
    );
  };
  ```
  - 실제로는 전역 상태관리와 useReducer를 함께 사용하는 패턴이 많이 사용됩니다.
  - 알림을 보내는 곳과 떨어져 있는 경우에 사용하시면 "상당히" 좋습니다.
- API 데이터 관찰 예제
```tsx
const createDataObserver = () => {
  const observers = new Map(); // { queryKey: [callbacks] }

  const fetchData = async (queryKey) => {
    const response = await fetch(`https://api.example.com/data?key=${queryKey}`);
    const data = await response.json();
    notify(queryKey, data);
    return data;
  };

  const notify = (queryKey, data) => {
    if (observers.has(queryKey)) {
      observers.get(queryKey).forEach(callback => callback(data));
    }
  };

  return {
    subscribe: (queryKey, callback) => {
      if (!observers.has(queryKey)) {
        observers.set(queryKey, []);
        // 최초 구독 시 데이터 가져오기
        fetchData(queryKey);
      }
      observers.get(queryKey).push(callback);

      return () => { // 구독 해제
        const callbacks = observers.get(queryKey).filter(cb => cb !== callback);
        if (callbacks.length === 0) {
          observers.delete(queryKey);
        } else {
          observers.set(queryKey, callbacks);
        }
      };
    },
    refetch: (queryKey) => {
      return fetchData(queryKey);
    }
  };
};

// 전역 데이터 옵저버
const dataObserver = createDataObserver();

const useDataObserver = (queryKey) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    const unsubscribe = dataObserver.subscribe(queryKey, (newData) => {
      setData(newData);
      setLoading(false);
    });
    return unsubscribe;
  }, [queryKey]);

  const refetch = () => {
    setLoading(true);
    return dataObserver.refetch(queryKey);
  };

  return { data, loading, refetch };
};

// 사용 예시
const UserProfile = ({ userId }) => {
  const { data: user, loading } = useDataObserver(`user_${userId}`);

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
    </div>
  );
};

const UserStats = ({ userId }) => {
  const { data: stats, refetch } = useDataObserver(`stats_${userId}`);

  return (
    <div>
      {stats && (
        <div>
          <p>Posts: {stats.postCount}</p>
          <p>Followers: {stats.followerCount}</p>
        </div>
      )}
      <button onClick={refetch}>Refresh Stats</button>
    </div>
  );
};
```

### 프론트엔드에서 자주 하는 실수
- 프로젝트의 규모가 커질수록 전역 상태관리 스토어가 오염되는 상황이 자주 발생하곤 합니다.
  - 개발이 고도화되면서 전역 상태에 너무 많은 state들이 전역 상태로 흘러들어오게 되는 것이지요.
  - 이때 개선이 될 수 있는 부분이 전역 상태에서 setter함수 혹은 update 함수를 두는것이 아닌, 각 컴포넌트 레벨에서 처리 가능한 구조로 개선이 가능합니다.
- 옵저버 패턴을 사용할 때에는 오버 엔지니어링을 조심해야 합니다.
  - 작은 프로젝트에 과도하게 넣는것이 아닌 플랫폼이 커져가면서 고도화하는 패턴중의 하나입니다.
  - 정말로 대규모 프로젝트의 전역 상태관리 경험을 이야기하고 싶다면 "옵저버 패턴"을 구현할 수 있느냐를 이야기할 역량을 뜻하는 것이기도 합니다.
    - zustand와 같은 전역 상태관리는 누구나 다 쉽게 학습하고 활용할 수 있기 때문에 이야기하는것 자체가 모순인 상황인거죠.
  - 이벤트가 발생하는곳과 이벤트를 수신하는곳 그리고 수신하는곳에서의 비즈니스 로직 처리와 어떤 중앙화된 전역 상태에 과밀하게 주입되는 오염되는 시나리오를 방지할 수 있는지에 대해서 이야기를 할 수 있어야 하는것 입니다.


## 옵저버 패턴의 강점
1. 느슨한 결합: 주체와 옵저버가 서로를 몰라도 됨
2. 동적 관계: 런타임에 옵저버 추가/제거 가능
3. 효울적인 상태 관리: 상태 변경 시 여러 컴포넌트에 자동 반영
4. 확장성: 새로운 옵저버 추가가 용이해짐

### 효율적인 상태관리
- 개발 혹은 고도화중인 어플리케이션의 규모가 커졌을때 옵저버 패턴을 도입을 고민해야 하는 특정 시점이 다가오게 됩니다.
- 옵저버 패턴을 사용하지 않는다면 코드간 결합도가 높아지며 각각의 컴포넌트와 전역 상태의 관심사 분리가 진행되지 않아 전역상태 로직이 무거워질 수 있습니다.

## 주의사항
1. 메모리 누수: 절대 구독해제를 잊지 말 것 (useEffect의 cleanup 함수 사용)
2. 성능 문제: 너무 많은 옵저버가 있을 경우 성능 저하 가능성
3. 무한 루프: 옵저버에서 상태 변경 시 주의 (조건 없이 상태를 변경하면 무한 루프 발생 가능)
4. 디버깅 어려움: 간접적인 호출로 인해 실행 흐름 추적이 어려울 수 있음
