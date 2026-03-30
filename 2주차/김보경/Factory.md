# 팩토리 패턴
- 팩토리 패턴을 사용하면 함수를 호출하는 것으로 객체를 만들어낼 수 있습니다.
  - new 키워드를 사용하는 대신 함수 호출의 결과로 객체를 만들 수 있는 것입니다.
  ```ts
  const createUser = ({ firstName, lastName, email }) => ({
    firstName,
    lastName,
    email,
    fullName() {
      return `${this.firstName} ${this.lastName}`;
    }
  });

  const user1 = createUser({
    firstName: "John",
    lastName: "Doe",
    email: "john@doe.com"
  });

  const user2 = createUser({
    firstName: "Jane",
    lastName: "Doe",
    email: "jane@doe.com"
  });
  ```

- 팩토리 패턴은 상대적으로 복잡한 객체 혹은 환경이나 설정에 따라 키와 값을 다양하게 설정해야 하는 객체를 만들어야 할 때 유용하게 사용할 수 있습니다.
  - 팩토리 패턴을 사용해 특정한 키나 값을 가진 객체를 쉽게 만들 수 있습니다.
  ```ts
  const createObjectFromArray = ([key, value]) => ({
  [key]: value,
  })

  createObjectFromArray(['name', 'John']) // { name: "John" }
  ```

## 장점
- 팩토리 패턴은 동일한 프로퍼티를 가진 여러 작은 객체를 만들어 낼 때 유용합니다. 현재의 환경이나 사용자 특징적인 설정을 통해 원하는 객체를 쉽게 만들 수 있습니다.

## 단점
- 자바스크립트에서 팩토리 함수는 new 키워드 없이 객체를 만드는 것에서 크게 벗어나지 않습니다.
- 하지만 대부분의 상황에서는 객체를 일일히 만든는 것 보다 클래스를 활용하는 편이 메모리를 절약하는데 더 효과적입니다.
  ```ts
  class User {
    constructor(firstName, lastName, email) {
      this.firstName = firstName
      this.lastName = lastName
      this.email = email
    }

    fullName() {
      return `${this.firstName} ${this.lastName}`
    }
  }

  const user1 = new User({
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@doe.com',
  })

  const user2 = new User({
    firstName: 'Jane',
    lastName: 'Doe',
    email: 'jane@doe.com',
  })
  ```


# 팩토리 메서드 패턴
- 객체 생성을 서브 클래스로 분리하여 처리하도록 캡슐화 하는 패턴
- 팩토리 메서드는 공장 그 자체로 비유 가능합니다.
  1. 본사 에서는 "차량을 생산하라"는 명령을 전달
     - factoryMethod() 추상화
  2. 각 공장은 자신의 전문 분야대로 차량 생산
     - new SUV(), new Sedan() 등
  3. 클라이언트(구매자)는 어떤 공장을 선택할지만 결정
     - new SUV공장().생산()
  - 인터페이스 및 추상 클래스 정의
  ```ts
  interface Car {
    name: string;
    assemble(): string;
    deliver(): string;
  }

  // Creator 추상 클래스 (자동차 공장)
  abstract class CarFactory {
    public abstract createCar(): Car;

    public produceCar(): string {
      const car = this.createCar();
      return `
        🏭 생산 시작
        ${car.assemble()}
        ${car.deliver()}
        🚗 완성: ${car.name}
      `;
    }
  }
  ```
  - 구체적인 차량 모델
  ```ts
  // Concrete Product A
  class SUV implements Car {
    name = "SUV 모델";

    assemble(): string {
      return "🔧 대형 차체 조립 + 4WD 시스템 장착";
    }

    deliver(): string {
      return "🚛 오프로드 주행 테스트 완료";
    }
  }

  // Concrete Product B
  class Sedan implements Car {
    name = "세단 모델";

    assemble(): string {
      return "🔧 경량 차체 조립 + 공력 설계 적용";
    }

    deliver(): string {
      return "🛣️ 고속도로 주행 테스트 완료";
    }
  }

  // Concrete Product C
  class ElectricCar implements Car {
    name = "전기차 모델";

    assemble(): string {
      return "🔧 배터리 팩 장착 + 전기 모터 설치";
    }

    deliver(): string {
      return "🔌 충전 테스트 + 소음 검사 완료";
    }
  }
  ```
  - 구체적인 공장
  ```ts
  // Concrete Creator A
  class SUVFactory extends CarFactory {
    public createCar(): Car {
      return new SUV();
    }
  }

  // Concrete Creator B
  class SedanFactory extends CarFactory {
    public createCar(): Car {
      return new Sedan();
    }
  }

  // Concrete Creator C
  class ElectricCarFactory extends CarFactory {
    public createCar(): Car {
      return new ElectricCar();
    }
  }
  ```
  - 클라이언트
  ```ts
  function clientApp(factory: CarFactory) {
    console.log(factory.produceCar());
  }

  // SUV 생산
  console.log('=== SUV 공장 가동 ===');
  clientApp(new SUVFactory());

  // 세단 생산
  console.log('\n=== 세단 공장 가동 ===');
  clientApp(new SedanFactory());

  // 전기차 생산
  console.log('\n=== 전기차 공장 가동 ===');
  clientApp(new ElectricCarFactory());
  ```
  - 향후 새로운 팩토리 생성 가능
  ```ts
  class TruckFactory extends CarFactory {
    public createCar(): Car {
      return new Truck();
    }
  }

  clientApp(new TruckFactory());
  ```
- 수정 없이 동일한 형태로 생성 가능

> 팩토리 패턴 예제
> - [참고 블로그 - [tyler's blog] React Hook Factory](https://blog.tyler.fun/3mheonekark2a)

# 추상 팩토리 패턴
- 추상 팩토리는 동일한 성격을 가진 것들을 추상화하여 상위 팩토리로 둔 형태를 띄고 있습니다.

```ts
// --- 추상 제품: 스킬 (각 슬롯 타입을 나누거나, 하나의 Skill로 통일해도 됨) ---
interface Skill {
  readonly name: string;
  cast(): string;
}

// --- 구체 제품: 챔 A의 스킬들 (한 세트) ---
class YasuoQ implements Skill {
  readonly name = "핀토 스탑";
  cast() {
    return "Q: 선풍";
  }
}
class YasuoW implements Skill {
  readonly name = "바람 장막";
  cast() {
    return "W: 투사체 막기";
  }
}
// E, R도 같은 식으로…

// --- 구체 제품: 챔 B의 스킬들 (다른 세트) ---
class ZedQ implements Skill {
  readonly name = "예리한 표창";
  cast() {
    return "Q: 표창";
  }
}
class ZedW implements Skill {
  readonly name = "그림자 분신";
  cast() {
    return "W: 분신";
  }
}
// E, R…

// --- 추상 팩토리: "이 챔의 풀 키트를 만든다" ---
abstract class ChampionKitFactory {
  abstract createQ(): Skill;
  abstract createW(): Skill;
  abstract createE(): Skill;
  abstract createR(): Skill;

  /** 클라이언트는 구체 클래스 이름을 몰라도 한 세트를 받을 수 있음 */
  loadout() {
    return {
      q: this.createQ(),
      w: this.createW(),
      e: this.createE(),
      r: this.createR(),
    };
  }
}

// --- 구체 팩토리: 챔별로 "가족"이 바뀜 ---
class YasuoKitFactory extends ChampionKitFactory {
  createQ() {
    return new YasuoQ();
  }
  createW() {
    return new YasuoW();
  }
  createE() {
    return new YasuoE(); // 예시: 실제로는 클래스 정의 필요
  }
  createR() {
    return new YasuoR();
  }
}

class ZedKitFactory extends ChampionKitFactory {
  createQ() {
    return new ZedQ();
  }
  createW() {
    return new ZedW();
  }
  createE() {
    return new ZedE();
  }
  createR() {
    return new ZedR();
  }
}

// --- 클라이언트: 팩토리만 바꾸면 전투 스타일이 통째로 바뀜 ---
function playChampion(factory: ChampionKitFactory) {
  const kit = factory.loadout();
  console.log(kit.q.cast(), kit.w.cast());
}

playChampion(new YasuoKitFactory());
playChampion(new ZedKitFactory());
```
- 클래스형 컴포넌트 사용 시절에는 자주 사용되었지만 최근에는 사용 빈도가 많이 줄어든 패턴이라고 하네요
