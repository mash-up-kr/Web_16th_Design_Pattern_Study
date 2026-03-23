디자인 패턴은 단순히 코드를 예쁘게 쓰는 요령이 아닌, 반복해서 등장하는 설계 문제에 대한 검증된 구조적 해법

1. 문제가 무엇인지
2. 문제를 해결하기 위해 어떤 구조를 만드는지
3. 그 구조에서 코드가 어떻게 동작하는지
4. 그 대가로 무엇을 얻고 잃는지 ( 트레이드 오프 )

# 1. 싱글톤 패턴

## 1-1. 정의

클래스의 인스턴스가 오직 하나만 존재하도록 보장하고, 그 인스턴스에 접근할 수 있는 전역적인 접근 지점을 제공하는 패턴

* 유일성 보장 — 인스턴스가 하나만 존재함
* 접근 경로 통제 — 모두가 같은 진입점을 통해 접근함

객체를 여러 개 만들 수 없도록 설계 수준에서 막고, 모두가 같은 객체를 바라보게 만드는 것이 본질

## 1-2. 필요성

시스템 전체에 여러 개 존재하면 곤란한 것들이 있음

ex) 설정 관리, 로그 관리, 캐시 관리, DB 커넥션, 전역 상태…

여러 개 생겼을 때 상태가 분산되거나 충돌할 우려 ⇒ 싱글톤을 통해 "상태의 중심점"을 하나로 고정

## 1-3. 동작원리

1. 클래스가 존재함
   * 아직 객체는 만들어지지 않았을 수 있음
2. 누군가 인스턴스를 요구함
   * getInstance() 같은 진입점을 통해 요청
3. 기존 인스턴스가 있는지 검사
   * 있으면 기존 객체를 반환, 없으면 새로 만든 후 저장
4. 이후 모든 요청은 같은 객체를 돌려줌

```js
class Singleton {
  static instance = null;

  constructor() {
    // JavaScript는 private constructor를 지원하지 않음
    // 그래서 누군가 직접 new Singleton()을 호출하는 것을 막을 수 없음
    // 아래 로직은 그에 대한 방어 장치
    // new를 호출하더라도 기존 인스턴스가 있으면 그것을 반환
    if (Singleton.instance) {
      return Singleton.instance;
    }

    this.createdAt = Date.now();
    Singleton.instance = this;
  }

  // 외부에서는 new Singleton()이 아니라 이 메서드를 통해 접근하는 것이 원칙
  static getInstance() {
    if (!Singleton.instance) {
      Singleton.instance = new Singleton();
    }
    return Singleton.instance;
  }
}

const a = Singleton.getInstance();
const b = Singleton.getInstance();

console.log(a === b); // true
```

## 1-4. 싱글톤 실제 활용 — Prisma + Next.js

Next.js는 dev 모드에서 HMR(Hot Module Replacement) 발생. 코드를 저장할 때마다 모듈이 다시 평가되는데, 이 과정에서 PrismaClient 인스턴스가 중첩 생성됨

```ts
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

export const prisma = new PrismaClient();
```

이 코드가 문제가 되는 과정을 시간순으로 보면:

```
[코드 저장 1회차]  → new PrismaClient() → DB 커넥션 풀 생성 (5개 커넥션)
[코드 저장 2회차]  → new PrismaClient() → DB 커넥션 풀 또 생성 (5개 추가)
[코드 저장 3회차]  → new PrismaClient() → 또 5개 추가
...
[코드 저장 10회차] → 50개 커넥션이 열려있음
```

이전 PrismaClient 인스턴스는 DB 커넥션이 열려있어 GC 대상이 아님. 저장 반복할수록 커넥션 누적 ⇒ DB 커넥션 한도 초과로 앱이 터짐

이를 globalThis와 싱글톤 패턴을 활용하여 해결 가능

```ts
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

Node.js에서 모듈 스코프 변수는 HMR 때마다 리셋됨. 하지만 globalThis는 Node.js 프로세스 전체에서 공유되는 전역 객체로, HMR이 모듈을 다시 평가해도 초기화되지 않음. 이를 싱글톤 인스턴스의 저장소로 활용

```ts
export const prisma = globalForPrisma.prisma ?? new PrismaClient();
```

- globalForPrisma.prisma가 이미 존재하면 → 기존 인스턴스 재사용
- globalForPrisma.prisma가 undefined면 → new PrismaClient() 생성
- if (!instance) instance = new Singleton()과 같은 논리

```ts
if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

production 환경에는 HMR이 없음. 모듈은 한 번만 평가되고, Node.js 모듈 캐싱이 이미 싱글톤 역할을 해줌. 이 상태에서 globalThis에 굳이 저장하면 전역 네임스페이스 오염이므로, 개발 환경에서만 저장

싱글톤 패턴을 통해 커넥션 풀 폭발 방지 + dev 환경 HMR 유지로 개발 생산성까지 확보

Prisma 공식문서 ( singleton best practice ) https://www.prisma.io/docs/orm/more/help-and-troubleshooting/nextjs-help

## 1-5. 트레이드오프

**얻는 것**
* 자원 절약 — 비용이 큰 객체(DB 커넥션)등 하나만 생성하고 재사용
* 상태 일관성 — 앱 전체가 같은 인스턴스를 바라보므로 상태 분산 없음
* 접근 편의성 — 어디서든 동일한 진입점으로 접근 가능

**잃는 것**
* 테스트 격리 어려움 — 전역 상태를 가지므로 테스트 간 상태가 남음. 테스트 A에서 바꾼 상태가 테스트 B에 영향을 줄 수 있음
* 높은 결합도 — 코드 여러 곳에서 싱글톤을 직접 참조하면 의존 모듈이 많아짐. 나중에 교체하거나 분리하기 어려워짐
* 숨겨진 의존성 — 함수 시그니처에 드러나지 않고 내부에서 싱글톤을 호출하면, 해당 함수가 뭘 의존하는지 겉에서 안 보임
