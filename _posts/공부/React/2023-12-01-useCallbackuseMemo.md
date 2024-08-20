---
layout: post
title: "useCallback과 useMemo"
date: 2023-12-01
categories:
  - study
  - React
description: >
  'React의 Hooks, useCallback과 useMemo에 대해서 공부하고 기록합니다.
---

# React 최적화: useCallback과 useMemo

## 개요

리액트를 시작한 지 오래되지 않았지만, 이제 초보운전 면허 딱지는 때도 될 정도는 아닌가 하는 생각이 듭니다.

어느 정도 그럴싸한 기능들을 구현하고 페이지를 구현하는 능력을 가지게 되었습니다.

하지만 큰 벽이 기다리고 있었으니, 그것은 바로 '최적화'입니다!

좋은 개발자는 번뜩이는 아이디어가 있으면 먼저 그 기능들에 많은 힘을 쏟아서 만들지만, 이제 그만큼의 시간을 공들여서 코드를 정리하고 최적화 작업을 함께 해야 합니다.

자주 사용되는 함수를 따로 빼고, 리렌더를 막아주는 등의 작업은 좋은 웹페이지를 만들고 사용자에게 좋은 경험을 주기 위해 필수적입니다.

<br>

## 최적화의 필요성

리액트 애플리케이션에서 최적화는 성능 향상과 사용자 경험 개선을 위해 필수적입니다.

예를 들어, 부모 컴포넌트가 리렌더링되면 자식 컴포넌트들도 함께 리렌더링될 수 있습니다.

이로 인해 불필요한 리렌더링이 발생하고, 이는 성능 저하를 초래할 수 있습니다.

<br>

## useCallback & useMemo에 대해

리액트에서 최적화를 위해 자주 사용되는 두 가지 훅은 useCallback과 useMemo입니다.

이들 훅을 적절히 사용하면 불필요한 렌더링을 방지하고 성능을 개선할 수 있습니다.

<br>

### Memoization

먼저 메모이제이션(Memoization)이라는 개념에 대해 이해할 필요가 있습니다.

메모이제이션은 기존에 수행한 연산의 결과를 저장해두고 동일한 입력이 들어오면 저장된 결과를 재활용하는 프로그래밍 기법입니다.

이를 통해 중복 연산을 피하고 애플리케이션의 성능을 최적화할 수 있습니다.

<br>

### useMemo

`useMemo`는 메모이제이션을 사용하여 값을 반환하는 훅입니다. 사용법은 다음과 같습니다:

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

여기서 `computeExpensiveValue`는 비싼 계산을 수행하는 함수입니다. a와 b가 변경되지 않는 한, `memoizedValue`는 이전의 계산 결과를 재사용합니다.

**주요 포인트:**

- `useMemo`는 주어진 의존성 배열([a, b])이 변경되지 않는 한, 메모이제이션된 값을 반환합니다.
- 이 훅은 렌더링 성능을 최적화하는 데 도움이 되지만, 너무 많이 사용하면 오히려 성능이 저하될 수 있습니다. 필요한 경우에만 사용하는 것이 좋습니다.

<br>

### useCallback

`useCallback`은 메모이제이션을 사용하여 함수를 반환하는 훅입니다. 사용법은 다음과 같습니다:

```js
const memoizedCallback = useCallback(() => {
  handleSomething(a, b);
}, [a, b]);
```

여기서 `handleSomething` 함수는 a와 b가 변경될 때만 새로운 함수를 생성합니다.

**주요 포인트:**

- `useCallback`은 주어진 의존성 배열([a, b])이 변경되지 않는 한, 같은 함수 인스턴스를 반환합니다.
- 함수가 자식 컴포넌트에 props로 전달될 때, `useCallback`을 사용하면 자식 컴포넌트의 불필요한 리렌더링을 방지할 수 있습니다.

<br>

#### 예제

다음은 `useMemo`와 `useCallback`을 사용하는 간단한 예제입니다:

```js
import React, { useState, useMemo, useCallback } from "react";

const ExpensiveComponent = ({ computeValue }) => {
  return <div>Computed Value: {computeValue()}</div>;
};

const ParentComponent = () => {
  const [count, setCount] = useState(0);

  const computeExpensiveValue = useCallback(() => {
    // Expensive computation
    return count * 2;
  }, [count]);

  const memoizedValue = useMemo(
    () => computeExpensiveValue(),
    [computeExpensiveValue]
  );

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <ExpensiveComponent computeValue={computeExpensiveValue} />
      <div>Memoized Value: {memoizedValue}</div>
    </div>
  );
};

export default ParentComponent;
```

이 예제에서 `computeExpensiveValue`는 count가 변경될 때만 재계산됩니다.

`ExpensiveComponent`는 동일한 함수 참조를 받기 때문에, 자식 컴포넌트의 불필요한 리렌더링이 방지됩니다.

<br>

### 결론

`useCallback`과 `useMemo`는 리액트 애플리케이션에서 성능 최적화를 위해 유용한 도구입니다.

`useCallback`은 함수의 메모이제이션을, `useMemo`는 값의 메모이제이션을 도와줍니다. 이 훅들을 적절하게 활용하면 성능을 개선하고 사용자 경험을 향상시킬 수 있습니다.

하지만, 이러한 최적화는 남용하지 않도록 주의하며 필요에 따라 적절히 사용하는 것이 중요합니다.
