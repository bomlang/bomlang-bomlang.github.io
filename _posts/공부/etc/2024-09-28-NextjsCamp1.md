---
layout: post
title: "[유데미x스나이퍼팩토리] 프로젝트 캠프 : Next.js 3기 - 1주차 수업"
date: 2024-09-28
categories:
  - study
  - etc
description: >
  [유데미x스나이퍼팩토리] 프로젝트 캠프 : Next.js 3기 1주차 수업을 수강후 회고
---

## 프로젝트 캠프 : Next.js 3기 - 1주차 수업 후기

Next.js캠프에 참여하면서 수업에 커리큘럼에 대해서 100% 만족하는 부분은 아니었다. JS와 React의 기본기는 이미 아는부분이라 필요 없었기 때문이다. 수업에 대한 커리큘럼은 마음에 들지않았지만 그럼에도 기대하는 부분도 있었는데, 그건 '수코딩'님께서 직접 강의를 해주신다는 점이 무척이나 기대되는 부분이었다.

<img src='/assets/img/공부/etc/수코딩.png' alt='수코딩의 코딩 자율학습 HTML+CSS+자바스크립트 책'/>

이 책을 코딩을 처음배울때 구매하여 공부했던 경험이 있기에 수코딩님에 대해서 이미 알고있었고, 책의 퀄리티가 매우 좋았어서 실제로 강의를 해주신다기에 많은 기대감을 가지고 있었다.

수업내용 자체는 커리큘럼에 따라서 기본에 충실하게 진행해주셨다. 거기에 추가적으로 수코딩님께서 직접 경험하신 부분이나 주의하면 좋은 부분, 현업에서 실제 쓰이는 방식들을 알려주셔서 수업이 지루하지않게 매우 좋았다.

앞으로 한주 더 수업이 남았는데, Next.js에 관한 수업은 처음듣는만큼 매우 기대되는 부분이다.

<br>

## 수업중 유의깊게 본 부분

### 커스텀 훅

커스텀훅에 대해서 알려주는 수업을 매우 재밌게 보았다. 예시로 로그인이나 회원가입에 많이 사용되는 input에 대한 커스텀훅을 예시로 알려주셨다.

보통은 useState를 각 input마다 상태를 만들거나,

```js

export default const App = () => {
  const [formState, setFormState] = useState({
    name: "",
    email: "",
    password: "",
    passwordConfirm: "",
  });


return (
  <>
  <form action="">
    <input type="text" value={formState.name} onChnage={onChange}/>
    <input type="email" value={formState.email} onChnage={onChange}/>
    <input type="password" value={formState.password} onChnage={onChange}/>
    <input type="password" value={formState.passwordConfirm} onChnage={onChange}/>
  </from>
  </>
)
}

```

위 코드처럼 useState를 객체로 받아서 하나의 onChange함수와 하나의 상태로 관리하도록 하는게 일방적이다.

이 코드들을 간략하게 분리하는 방법이 있는데, 그게 커스텀훅이다.

```js
export default const MyInput = (initialValue: string) => {
  const [value, setValue] = useState(initialValue)
  const onChnageHandler = (e) => {
    setValue(e.target.value)
  }
  return [value, onChnageHandler];
}


export default const App = () => {
  const [name, onChangeName] = MyInput("");
  const [email, onChangeEmail] = MyInput("");

  return (
    <>
      <input type="text" value={name} onChnage={onChangeName}/>
      <input type="email" value={email} onChnage={onChangeEmail}/>
    </>
  )
}

```

이런식으로 커스텀훅을 사용해서 코드를 깔~끔하게 만들 수 있다.

<br>

### Zustand

기존의 zuntand를 많이 사용은 했지만 동작방식에 대해서는 자세히 몰랐다. 단순히 간단하다는 이유로 사용했는데,

```js
import {create} from "zustand";

() = {};

const getUser = () => ({
  name: "HORI",
  age: "20",
})

const getName = () => h0r2";

export const userCountStore = create((state) => ({}));
export const userCountStore2 = create((state) => {});

```

이 코드를 보고 zustand의 함수는 왜이리 뎁스가 깊은지 깨달았다. 단순히 자바스크립트에서 허용된 문법을 해결하다보니 이렇게 뎁스가 깊어진거같다.

추가적으로

```js
export const useCounteStore = () => {
  count: 0,
  increament1: () => setCount({count: 10}), // 이전 상태값을 참조하지 않아도 되는 경우에 사용
  increament1: () => setCount(() => ({count: 10})), // 이전 상태값이 참조해야 하는 경우
}

```

이 부분도 상세히 알지 못했던 부분이라 알려주셨을떄 너무 도움이 되었다.

<br>
