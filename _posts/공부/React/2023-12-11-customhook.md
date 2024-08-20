---
layout: post
title: "내 더러운 코드에 커스텀 훅을 한 스푼 추가해보자."
date: 2023-12-11
categories:
  - study
  - React
description: >
  Custom Hook에 대해서 알아보고 기록합니다.
---

# 내 더러운 코드에 커스텀 훅을 한 스푼 추가해보자.

## 개요

처음에는 커스텀 훅이라는 개념을 잘 몰랐습니다. 존재는 알았지만, 도대체 언제, 어떻게 써야 할지 감이 오지 않았죠.

그러던 어느 날, 제가 맡은 로그인 및 회원가입 기능을 구현하면서 궁금증이 생겼습니다.

1. 회원가입을 위해서는 사용자가 입력한 '아이디', '사용자 이름', '이메일', '패스워드' 등 많은 `input` 값이 필요합니다.
2. 이 input 값들을 관리하기 위해 여러 개의 `useState`를 사용해야 할까요? (예: `onChange` 함수를 사용해서)
3. 하지만 수많은 `useState`... 지저분하고 비효율적으로 느껴졌습니다. 상태값을 하나로 관리하고, 객체로 전달해야겠다는 생각을 하게 되었습니다.

```js
const [formData, setFormData] = useState({
  email: "",
  username: "",
  nickname: "",
  password: "",
});

const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  const { name, value } = e.target;
  setFormData((prevData) => ({
    ...prevData,
    [name]: value,
  }));
};

const handleEnterUserData = async (event: React.MouseEvent) => {
  event.preventDefault();
  try {
    await EnterUserData(formData);
  } catch (error) {
    console.error(`Error: ${error}`);
  }
};
```

이것은 제가 작성한 코드입니다.

그런데 문득 생각이 들었습니다. `handleInputChange` 함수는 회원가입뿐만 아니라 로그인 페이지에서도 똑같이 쓰일 수 있지 않을까? 사실 이런 생각은 이번이 처음이 아닙니다.

프로젝트를 하다 보면 비슷한 로직의 이벤트 코드가 반복되는 경우가 많습니다. 저 역시 그런 코드를 페이지별로 복붙해서 사용하곤 했죠.

앞서 작성한 포스트에서 `useCallback`과 `useMemo`를 다루면서 리팩토링에 관심이 생기기 시작했습니다.

이제는 코드의 재사용성과 유지보수성을 높이기 위해 커스텀 훅을 활용해야겠다는 생각을 하게 되었습니다.

<br/>

## 그래서 커스텀 훅이 뭔데?

Custom Hooks는 React에서 반복되는 로직을 재사용하기 위해 사용하는 기능입니다. 공식 문서에서는 이를 다음과 같이 설명합니다:

```markdown
"개발을 하다 보면 가끔 상태 관련 로직을 컴포넌트 간에 재사용하고 싶은 경우가 생깁니다.

기존의 방법으로는 higher-order components와 render props가 있었으나, Custom Hook은 컴포넌트 트리에 새로운 컴포넌트를 추가하지 않고도 이 문제를 해결할 수 있습니다."
```

즉, 커스텀 훅은 반복되는 로직을 쉽게 재사용할 수 있게 해줍니다.

예를 들어, 앞서 언급한 `handleInputChange` 함수처럼 여러 컴포넌트에서 동일한 입력 처리를 해야 한다면, 이를 커스텀 훅으로 추출하여 재사용할 수 있습니다.

<br/>

## 커스텀 훅 만들기

간단한 예제로 시작해봅시다. 아래는 입력 필드를 처리하는 로직을 커스텀 훅으로 만든 코드입니다.

```js
import { useState } from "react";

const useFormData = (initialState = {}) => {
  const [formData, setFormData] = useState(initialState);

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData((prevData) => ({
      ...prevData,
      [name]: value,
    }));
  };

  return [formData, handleInputChange];
};

export default useFormData;
```

이 커스텀 훅 `useFormData`는 `useState`와 `handleInputChange` 함수를 한 번에 제공합니다. 이제 이 훅을 사용하여 코드를 간결하게 만들 수 있습니다.

<br/>

### 사용 예시

```js
import React from "react";
import useFormData from "./useFormData";

const SignUpForm = () => {
  const [formData, handleInputChange] = useFormData({
    email: "",
    username: "",
    nickname: "",
    password: "",
  });

  const handleSubmit = async (event) => {
    event.preventDefault();
    try {
      await enterUserData(formData);
    } catch (error) {
      console.error(`Error: ${error}`);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" value={formData.email} onChange={handleInputChange} />
      <input
        name="username"
        value={formData.username}
        onChange={handleInputChange}
      />
      <input
        name="nickname"
        value={formData.nickname}
        onChange={handleInputChange}
      />
      <input
        name="password"
        value={formData.password}
        onChange={handleInputChange}
      />
      <button type="submit">Sign Up</button>
    </form>
  );
};

export default SignUpForm;
```

이제 `SignUpForm`과 같은 다른 컴포넌트에서도 동일한 입력 처리 로직을 재사용할 수 있습니다. 이처럼 커스텀 훅을 사용하면 코드가 훨씬 더 깔끔해지고 유지보수가 쉬워집니다.

<br/>

### 커스텀 훅의 장점

- 코드 재사용성: 동일한 로직을 여러 컴포넌트에서 쉽게 재사용할 수 있습니다.
- 유지보수성: 로직이 한 곳에 모여 있으므로, 변경 사항을 쉽게 관리할 수 있습니다.
- 가독성: 복잡한 로직을 훅으로 분리하면 컴포넌트 코드가 훨씬 더 간결해집니다.

### 주의사항

- 의존성 관리: 커스텀 훅 내에서 `useEffect` 또는 `useCallback`을 사용할 때는 의존성 배열을 정확하게 관리해야 합니다. 그렇지 않으면 예기치 않은 버그가 발생할 수 있습니다.
- 성능: 커스텀 훅이 너무 많아지면 오히려 성능에 부정적인 영향을 줄 수 있으므로, 적절히 사용해야 합니다.
