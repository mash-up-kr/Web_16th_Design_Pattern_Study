# 컴파운드 패턴

여러 개의 작은 컴포넌트를 조합해서, 하나의 큰 컴포넌트 처럼 만드는 설계 방식

⇒ 하나의 덩어리 컴포넌트가 모든 UI를 독점해서 렌더하는 것이 아닌, 부모가 상태 및 공통 로직 책임, 자식 컴포넌트가 각 역할에 부합한 UI 조각들을 구성하는 것이 특징

Compund Components Pattern

### 필요

```tsx
<Tabs
  tabs={[
    { label: '소개', content: <Intro /> },
    { label: '리뷰', content: <Review /> },
    { label: '문의', content: <QnA /> },
  ]}
/>
```

일반적인 컴포넌트 집약 구조, 구현하기 간단함
but 탭 버튼 구조를 바꾸는데 제약, 특정 탭 별로 아이콘 추가 불가 ⇒ 확장성 떨어짐, 구조가 결합

```tsx
<Tabs defaultValue="intro">
  <Tabs.List>
    <Tabs.Trigger value="intro">소개</Tabs.Trigger>
    <Tabs.Trigger value="review">리뷰</Tabs.Trigger>
    <Tabs.Trigger value="qna">문의</Tabs.Trigger>
  </Tabs.List>

  <Tabs.Content value="intro">
    <Intro />
  </Tabs.Content>
  <Tabs.Content value="review">
    <Review />
  </Tabs.Content>
  <Tabs.Content value="qna">
    <QnA />
  </Tabs.Content>
</Tabs>
```

컴파운드 패턴 방식
구조를 자유롭게 조합할 수 있고, 각 부분의 역할 분리 ⇒ 확장성 확보, 조합의 자유

### 핵심 구조

1. 의미적으로 연결된 컴포넌트들을 하나의 집합으로 묶는다
    1. Tabs 라는 공통 부모를 기반으로 개념을 묶음, Tabs.List, Tabs.Trigger, Tabs.Content 는 따로 보면 독립된 의미 
2. 부모가 상태와 규칙을 소유한다
    1. 공통 상ㅇ태는 중앙에서 관리, 각 자식은 그 상태를 소비하는 형태 
3. 자식은 역할 기반으로 분리된다 
    1. props로 경우의 수를 처리하는 대신, 역활별 컴포넌트로 의미를 분해

⇒ 단순히 컴포넌트를 여러 개 만들어서 결합한다는 컴파운드 패턴의 의도와 맞지 않음! 상태 공유 + 역할 분리 + 선언적 조합, 세 가지가 동시에 존재해야 컴파운드 패턴이라고 볼 수 있음 

### React에서 컴파운드 패턴이 잘 맞는 이유

```tsx
<Modal>
  <Modal.Trigger />
  <Modal.Content>
    <Modal.Header />
    <Modal.Body />
    <Modal.Footer />
  </Modal.Content>
</Modal>
```

리액트는 선언형 UI, 컴파운드는 배치의 구조가 코드로서 아주 잘 표현됨 ⇒ 가독성 굿

## 구현

```tsx
const Tabs = ({ children, defaultValue }) => {
  const [activeValue, setActiveValue] = useState(defaultValue);

  return (
    <TabsContext.Provider value={{ activeValue, setActiveValue }}>
      {children}
    </TabsContext.Provider>
  );
};
```

1. 부모가 상태를 가진다

```tsx
const TabsTrigger = ({ value, children }) => {
  const { activeValue, setActiveValue } = useContext(TabsContext);

  const isActive = activeValue === value;

  return (
    <button
      onClick={() => setActiveValue(value)}
      aria-selected={isActive}
    >
      {children}
    </button>
  );
};
```

1. 자식은 Contex를 읽는다

```tsx
const TabsContent = ({ value, children }) => {
  const { activeValue } = useContext(TabsContext);

  if (activeValue !== value) return null;
  return <div>{children}</div>;
};
```

1. 다른 자식도 같은 상태를 공유한다

⇒ 자식끼리의 관심사 분리. 직접 통신하는것이 아닌 부모가 제공한 Context 기반으로 협력!

보통 Context와 staticProperty 요소를 활용해서 구현

Context : 하위 컴포넌트들이 공통 상태를 공유하기 위해 사용
Static Property : 부모 컴포넌트에서 하위 컴포넌트를 속성처럼 붙여서 사용

```tsx
Tabs.List = TabsList;
Tabs.Trigger = TabsTrigger;
Tabs.Content = TabsContent;

<Tabs>
  <Tabs.List />
  <Tabs.Trigger />
  <Tabs.Content />
</Tabs>
```

### 트레이드 오프

장점

- 컴포넌트 구조가 그대로 드러나서 가독성이 좋음!
- 유연한 조합이 가능
    - 동일한 UI 구조인데 소비처마다 UI 차이가 있는 경우
    - 헤더에 아이콘 추가.. 조건부 렌더링.. 위치 재배치… 레이아웃 추가
    - 일반적인 컴포넌트구조보다 훨씬 유연한 처리가 가능
- 역할 분리가 명확함
    - 부모 = 개념, 하위 컴포넌트 = 역할
- 디자인 시스템과 잘 어울림 ⇒ 재사용 가능성, 구조 명확, 확장성, 접근성 규칙 포함

단점

- 구현 난이도 상승
    - 자식-부모간 역할/책임 분리, 상태 설계, 타입 설계
- 구조를 잘못 쓰면 오용되기 쉬움
    - Tabs.Trigger를 Tabs 밖에서 사용하게 된다면 문제 생김 ⇒ 정의된 Context 없기 때문
- 자율성이 높다 ⇒ 일관성이 적다.
- 학습 비용

### 적합한 사례

1. 하나의 구조를 갖추고, 여러 하위요소와 함께 의미를 이룰 때
- 탭, 아코디언, 셀렉트, 모달, 카드 … 등 재사용되는 컴포넌트 구조
1. 내부구조를 직접 조합할 필요가 있을 때
- 단순 props로 처리하기보다 더 자연스러운 경우나 바리에이션이 잦은 경우
1. 디자인 시스템 만들 때 ( 공통 컴포넌트 )

> 설계 비용이 있는 만큼 이점이 있을때만 쓰자!
> 

### 예제

컴파운드 패턴은 구조가 복잡하면서 자유도가 높음 ⇒ 오용 가능, 타입 설계를 통해 가드레인 필요

ex) Tabs.Trigger 는 value 필요.. Tabs.Content의 value는 Trigger와 연결되는 값, Accordion.Item 안에는 Accordion.Header가 필요 ..

⇒ 타입 스크립트로 프롭스 타입과 context 타입을 정밀하게 만들 수 있음 

```tsx
import React, { createContext, useContext, useState, ReactNode } from 'react';

type TabsContextType = {
  activeValue: string;
  setActiveValue: (value: string) => void;
};

const TabsContext = createContext<TabsContextType | null>(null);

function useTabsContext() {
  const context = useContext(TabsContext);
  if (!context) {
    throw new Error('Tabs components must be used within Tabs');
  }
  return context;
}

type TabsProps = {
  defaultValue: string;
  children: ReactNode;
};

function TabsRoot({ defaultValue, children }: TabsProps) {
  const [activeValue, setActiveValue] = useState(defaultValue);

  return (
    <TabsContext.Provider value={{ activeValue, setActiveValue }}>
      {children}
    </TabsContext.Provider>
  );
}

function TabsList({ children }: { children: ReactNode }) {
  return <div role="tablist">{children}</div>;
}

function TabsTrigger({
  value,
  children,
}: {
  value: string;
  children: ReactNode;
}) {
  const { activeValue, setActiveValue } = useTabsContext();
  const isSelected = activeValue === value;

  return (
    <button
      role="tab"
      aria-selected={isSelected}
      onClick={() => setActiveValue(value)}
    >
      {children}
    </button>
  );
}

function TabsContent({
  value,
  children,
}: {
  value: string;
  children: ReactNode;
}) {
  const { activeValue } = useTabsContext();

  if (activeValue !== value) return null;

  return <div role="tabpanel">{children}</div>;
}

const Tabs = Object.assign(TabsRoot, {
  List: TabsList,
  Trigger: TabsTrigger,
  Content: TabsContent,
});

export default Tabs;

// 사용
<Tabs defaultValue="intro">
  <Tabs.List>
    <Tabs.Trigger value="intro">소개</Tabs.Trigger>
    <Tabs.Trigger value="review">리뷰</Tabs.Trigger>
  </Tabs.List>

  <Tabs.Content value="intro">소개 내용</Tabs.Content>
  <Tabs.Content value="review">리뷰 내용</Tabs.Content>
</Tabs>
```

```tsx
<Card>
  <Card.Header>
    <CustomTitle />
  </Card.Header>
  <Card.Footer>
    <Button />
  </Card.Footer>
</Card>
```

### 정리

- UI가 구조를 가지고 있는가?
- 각 부분이 같은 상태를 공유할 필요가 있는가?
- 추후 구조의 확장/축소가 일어날 가능성이 있는가?
- Props로 API 구현이 자연스럽지 않은가? (지저분한가)

⇒ 컴파운드 고려!

- 적용기

    # **Card Compound Pattern 적용기**

    ## **1. 왜 도입했는가**

    ### **기존 구조의 문제**

    결과 모달의 `Item` 컴포넌트는 하나의 컴포넌트가 여러 유형(객관식, OX, 주관식)의 렌더링을 모두 담당했다. `Card.Content`에 `choiceComponent`, `submittedAnswerComponent` 등을 props로 전달하는 구조였다.

    ```tsx
    // Before: Item.tsx
    <Card>
      <Card.Header
        number={props.number}
        type={props.type}
        status={status}
      />
      <Card.Content
        question={props.question}
        imageUrl={props.imageUrl}
        choiceComponent={
          // 유형별 분기를 Content 내부에서 처리
          props.type === 'multiple' ? <MultipleChoice items={props.items} /> :
          props.type === 'ox'       ? <OXChoice answer={props.answer} /> :
                                      <SubjectiveAnswer answers={props.answers} />
        }
        submittedAnswerComponent={
          props.type === 'multiple' ? <SubmittedAnswer.Multiple {...} /> :
          props.type === 'ox'       ? <SubmittedAnswer.OX {...} /> :
                                      <SubmittedAnswer.Subjective {...} />
        }
      />
    </Card>
    ```

    **문제점:**

    1. **Props 폭발** - `Card.Content`가 question, imageUrl, choiceComponent, submittedAnswerComponent 등 많은 props를 받음
    2. **유연성 부족** - 레이아웃을 바꾸려면 새로운 prop을 추가해야 함
    3. **관심사 혼재** - 데이터 변환 로직과 UI 렌더링이 한 컴포넌트에 섞여 있음
    4. **바리에이션 대응 어려움** - 정답/오답 상태에 따른 스타일 분기가 props 조합으로만 가능

    ---

    ## **2. 컴파운드 패턴 적용**

    ### **Card 네임스페이스 구조**

    `Object.assign` 패턴으로 하나의 네임스페이스 아래 모든 서브 컴포넌트를 구성했다.

    ```tsx
    // index.ts
    export const Card = Object.assign(CardRoot, {
      Header: CardHeader,
      Content: CardContent,
      Footer: CardFooter,
      QuestionText: QuestionText,
      Image: CardImage,
      MultipleChoice: MultipleChoice,
      OXChoice: OXChoice,
      SubjectiveAnswer: SubjectiveAnswer,
    });
    ```

    사용하는 쪽에서 `Card.Header`, `Card.MultipleChoice.Item` 처럼 dot notation으로 접근할 수 있다.

    ### **MultipleChoice - Props API와 Compound API 공존**

    기존 `Item`은 props로 모든 걸 받는 고정 구조였다. 여기에 `ItemCompound`를 추가해서 레이아웃을 자유롭게 조합할 수 있도록 했다.

    ```tsx
    const ItemCompound = Object.assign(ItemRoot, {
      Row: ItemRow,
      Indicator: { Container: IndicatorContainer },
      Content: ItemContent,
    });

    export const MultipleChoice = {
      Container,
      Item,           // Props 기반 API (단순 케이스)
      ItemCompound,   // Compound API (복잡 케이스)
    };
    ```

    `forwardRef + ComponentPropsWithoutRef<'div'>`를 모든 서브 컴포넌트에 적용해서 HTML 네이티브 속성(className, onClick 등)을 그대로 전달할 수 있게 했고, React DevTools 식별을 위해 `displayName`도 설정했다.

    ---

    ## **3. Dual API 전략: Props API + Compound API**

    모든 유형별 컴포넌트에 두 가지 API를 공존시켰다.

    ### **Props 기반 API (단순 케이스)**

    정해진 레이아웃에서 사용:

    ```tsx
    <Card.MultipleChoice.Container>
      <Card.MultipleChoice.Item
        text="선택지 텍스트"
        isAnswer={true}
        number={1}
      />
    </Card.MultipleChoice.Container>
    ```

    ### **Compound API (복잡 케이스)**

    결과 화면처럼 정답/오답 상태에 따라 다른 스타일이 필요한 경우:

    ```tsx
    <Card.MultipleChoice.ItemCompound>
      <Card.MultipleChoice.ItemCompound.Row>
        {/* 정답을 골랐을 때 */}
        {isSelected && isCorrect && (
          <Card.MultipleChoice.ItemCompound.Indicator.Container className="bg-green-500">
            ✓
          </Card.MultipleChoice.ItemCompound.Indicator.Container>
        )}
        {/* 오답을 골랐을 때 */}
        {isSelected && !isCorrect && (
          <Card.MultipleChoice.ItemCompound.Indicator.Container className="bg-red-500">
            ✕
          </Card.MultipleChoice.ItemCompound.Indicator.Container>
        )}
        {/* 선택하지 않았을 때 */}
        {!isSelected && (
          <Card.MultipleChoice.ItemCompound.Indicator.Container className="bg-gray-200" />
        )}
        <Card.MultipleChoice.ItemCompound.Content>
          <span>{choice.text}</span>
        </Card.MultipleChoice.ItemCompound.Content>
      </Card.MultipleChoice.ItemCompound.Row>
    </Card.MultipleChoice.ItemCompound>
    ```

    **왜 두 API를 공존시켰나?**

    - Props API: 대부분의 사용처에서 충분하고 코드가 간결함
    - Compound API: 결과 화면처럼 세밀한 제어가 필요한 곳에서만 사용
    - 기존 코드를 깨뜨리지 않으면서 점진적으로 마이그레이션 가능

    리팩토링 후 `Item`은 유형별로 어떤 카드 컴포넌트를 그릴지 디스패치하는 역할만 담당하게 되었다.

    ```tsx
    // After: Item.tsx - 디스패치만 담당
    export const Item = (props: Props) => {
      switch (props.type) {
        case 'multiple':   return <ResultCard.Multiple {...baseProps} items={props.items} />;
        case 'ox':         return <ResultCard.OX {...baseProps} answer={props.answer} />;
        case 'subjective': return <ResultCard.Subjective {...baseProps} answers={props.answers} />;
      }
    };
    ```

    ---

    ## **4. 개선 효과 요약**

    | 관점 | Before | After |
    | --- | --- | --- |
    | **Props 수** | `Card.Content`에 6개+ props | `children`으로 조합 |
    | **레이아웃 유연성** | props 추가 필요 | 서브 컴포넌트 조합으로 자유 배치 |
    | **바리에이션 대응** | props 조합 + 조건부 렌더링 | className 전달로 스타일 제어 |
    | **유형별 분리** | 하나의 Item에 모든 로직 | 유형별 독립 카드 컴포넌트 |
    | **재사용성** | 결과 화면 전용 | 목록/상세/결과 화면 공용 |
    | **확장성** | 새 유형 추가 시 기존 컴포넌트 수정 | 새 카드 컴포넌트만 추가 |
