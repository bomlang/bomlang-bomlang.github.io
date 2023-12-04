---
layout: post
title: useCallback과 UseMemo의 차이를 알고 최적화를 해보자.
tag: React
---

# 최적화에 대해서

리액트를 시작한지 오래되지 않았지만, 이제 초보운전 면허딱지는 때도 될 정도는 아닌가하는 생각이 스스로 든다. 어느정도 그럴싸한 기능들을 구현가능하고 페이지를 구현하는 능력을 가지게 되었다.
하지만 큰 벽이 기다리고 있었으니...
그것은 '최적화'!

좋은 개발자는 번뜩이는 아이디어가 있으면 먼저 그 기능들에 많은 힘을 쏟아 부어서 만들었다면, 이제 그만큼의 시간을 공들여서 코드를 정리해야한다. 그리고 정리하면서 최적화작업도 같이 해야한다. 자주 사용되는 함수는 따로 빼주고, 리렌더를 막아주는등의 작업은 좋은 웹페이지를 만들고 사용자에게 좋은 경험을 주기 때문이다.

# useCallback & useMemo에 대해서

요새 난 코드를 짜면서 한가지 고민거리가있다.
만약 부모에서 코드가 state변경으로 인하여 리렌더링이 된다고 가정하자.

```jsx
const renderCommentsSection = () => (
  <div className="min-w-[360px] lgpc:mt-3 pc:mt-3">
    <AddComment
      videoId={location.state.item.id}
      setCommentData={setCommentData}
      optionBtnCallback={handleOptionBtnCallback}
    />
    {commentData.map((item, index) => (
      <Comment
        key={`${item.anonymous_user_id}_${index}`}
        commentId={item.anonymous_user_id}
        date={item.created_at}
        text={item.text}
        setCommentData={setCommentData}
        videoId={location.state.item.id}
        optionBtnCallback={handleOptionBtnCallback}
      />
    ))}
  </div>
);
```

그러면 예를들어 위와 같은 코드의 자식컴포넌트를 가지고 있다고 가정할때, 자식컴포넌트도 부모에 의해서 계속해서 리렌더링이 될 것이다.
다행이라면 다행이게도 props를 받는값이 같기때문에 페이지내에서 리렌더링이 되진 않는다. 하지만 그래도 부모에 의해서 리렌더링이되고, 가상돔이 생성되는 과정은 막을 수 없다.
여기에서 비용이 발생하고 고민이 생기게 되었다.
도대체 어떻게해야 이런 고질적인 문제를 해결 할 수있을까? 코드를 처음부터 잘짜야한다..? (말도안되지 ㅋㅋ)

**여기에서 등장하는 useCallback과 useMemo!!**

## Memoization

일단 먼저 메모이제이션이라고 하는 개념에 대해 확실히 알아가야하는데, 메모이제이션은 기존에 수행한 연산의 결과값을 어딘가에 저장해두고 동일한 입력이 들어오면 재활용하는 프로그래밍 기법이다.
이 메모이제이션을 적절하게 활용하면 중복 연산을 피할 수 있기 때문에 메모리를 조금 더 쓰더라도 애플리케이션의 성능을 최적화 할 수 있다.

## useMemo

useMemo는 값을 반환하는 함수이다.

```jsx
useMemo(() => fn, [deps]);
```

deps는 의존성배열로, 만약 없다면 빈공간으로 냅두어도 좋다. 비어있다면 초기에 useMemo는 () => fn 함수를 실행하고, 그 함수의 **반환 값을 반환**한다.

즉 useMemo는 값을 반환한다.
만약 리렌더링이 발생할 경우, useMemo에서 설정한 값이 같을 경우에 리렌더링을 막게된다.

## useCallback

```jsx
useCallback(fn, [deps]);
```

useCallback은 useMemo와 비슷하지만 좀 다르다. useMemo는 함수를 등록하고, 거기에 나온 **값**을 반환했다면, useCallback은 함수 그 자체를 저장한다.

조금 생각해보면, 우리가 자주쓰는 함수를 어딘가에 적어놨다고 생각하자. 함수를 등록한 곳에서 state또는 props의 값이 변경되면서 계속해서 리렌더링이 일어난다.
그러면 선언된 함수는 리렌더링에 의해서 계속해서 함수가 다시 생성되고 지워지고를 반복한다.
함수가 생성되면 아무리 같은 동작과 값을 반환하는 함수라도 참조값이 달라지므로 다른 함수가 된다.

만약 이 함수를 props로 받는 자식이 있다면, 계속해서 다른 참조값의 함수를 받기때문에 react는 props가 다르다고 인식하여 리렌더링을 할 것이다.

이런 부분을 막아주는게 useCallback이다.
