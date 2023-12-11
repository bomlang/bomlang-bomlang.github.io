---
layout: post
title: 코드가 더러워? 그럼 커스텀 훅 한입 어때?
tag: React, Hook
---

# 내 더러운 코드에 커스텀 훅을 한스푼 추가해보자.

[자신만의 Hook 만들기 – React](https://ko.legacy.reactjs.org/docs/hooks-custom.html)

## 개요

먼저 커스텀훅을 처음에 몰랐다. 존재 자체만 알고있었지, 도대체 언제 써야 하는지 감이 오지 않았다.
그러던 어느날, 내가 맡은 로그인 및 회원가입 기능을 구현하는데 있어서 궁금점이 생겼다.

1. 회원가입을 하기 위해선 input에 사용자가 입력한 '아이디', '사용자이름', '이메일', '패스워드' 등등 많은 input들이 요구된다.
2. 그럼 이 input들의 값의 상태를 저장해야 하는데, 많은 useState가 필요하다. (onChange함수를 사용해서 만든다는 가정)
3. 하지만 나는 수많은 useState상태값...? 지저분해서 싫다. 상태값은 하나로 관리하고, 객체로 전달해야겠다는 생각을 하게된다.

```jsx
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
    await EnterhUserData(formData);
  } catch (error) {
    console.error(`Error: ${error}`);
  }
};
```

이것은 내가 앞서 작성한 코드다.

이제 궁금한점이 생긴다.

`handleInputChange`함수는 회원가입 말고도, 로그인페이지에서도 똑같이 쓰일거같은데? 라는 생각.
사실 이런 생각이 이번이 처음이 아니다.
항상 프로젝트를 하다보면 분명 비슷한 로직의 이벤트코드가 수없이 많이 존재했고, 나는 그 코드를 일일이 페이지별로 코드를 복붙해서 사용해왔다.

앞전 포스트에서 useCallback, useMeomo를 다뤗듯이, 나는 이제 리팩토링에 어느정도 관심이 생겼고, 관심가져야 할 꿈나무 개발자다.

그래서 강사님께 여줘보았고, 커스텀훅을 사용해야 한다고 말씀해주셨다.

## 그래서 커스텀 훅이 뭔데?

Custom Hooks. 공식문서에선 `개발을 하다 보면 가끔 상태 관련 로직을 컴포넌트 간에 재사용하고 싶은 경우가 생깁니다. 문제를 해결하기 위한 전통적인 방법이 두 가지 있었는데, higher-order components와 render props가 바로 그것입니다.Custom Hook은 이들 둘과는 달리 컴포넌트 트리에 새 컴포넌트를 추가하지 않고도 이것을 가능하게 해줍니다.`라고 말해준다.
즉, 반복되는 로직을 재사용 하기 위함이다.

```jsx
import {useState} from 'react'

const useFetch = () => {
  const [url, setUrl] = useState(url)
  reutrn  [url, setUrl]
}
export defautl useFetch
```

중요한점은 커스텀훅도 다른 훅들처럼 'use...'로 시작해야한다.
또한 이 커스텀훅이 좋은점이 useState와 사용법이 매우 흡사하다.
