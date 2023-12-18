---
layout: post
title: 로컬스토리지를 이용한 로그인, 세션으로 로그인상태 업데이트.
tag: React, Hook
---

# 로그인을 어떻게 관리할것인가!?

팀원끼리 프로젝트를 하면서 내가 로그인파트를 담당하게 되었다. 여태 다른 잘하는 팀원들이 하다보니 막상 로그인을 담당하게되자 다가오는 엄청난 프레셔... ㅇ으ㅡ,ㄱㄱ....
하지만 할 건 해야겠지?

일단 땅바닥에서 구현하는건 아니고, Supabase를 사용해서 로그인을 구현하기로 했다. 많은 기능을 지원하고, 무엇보다 다양한 OAuth를 지원하는거보니 요즘 대세인게 확실히 느껴졌다.

```jsx
export const enterUserData = async (
  userData: UserData
): Promise<string | undefined> => {
  try {
    const { data, error } = await supabase.auth.signUp({
      email: userData.email,
      password: userData.password,
    });

    if (data) return data?.user?.id;
    else console.log("회원가입 실패:", error);
  } catch (error) {
    console.error(error);
  }
};
```

사실 로그인, 회원가입은 막상 supabase의 공식문서를 뜯어보니 생각보다 쉽게 할 수 있었다.
이제 회원가입까지 구현하니 문뜩 드는 생각....

로그인을 어떻게 유지시킬것인가!!!?!?!

<img src='./../../assets/img/programming/fingerBoy.webp' alt='jest사용시 에러메시지'>

로컬스토리지에 저장해버리면?!

이라는 미친생각을 하게된다.

## 로컬스토리지가 어때서!!

앞에서는 미친생각이라 했지만, 잘못된 생각은 아니다. 왜냐?
로컬스토리지에 저장 안하면 사용자가 페이지 닫았다가 들어오면 어떻게 유지할건데??

'세션스토리지'라는 선택지도 있지만, 세션 스토리지는 창을 닫는순간 모든 기록이 날라간다. 반면 로컬스토리지는 사용자가 직접 제거하지않는이상 게속해서 데이터가 남아있게된다.

그렇다.
계속... always....

<img src='./../../assets/img/programming/always.jpeg' alt='jest사용시 에러메시지'>

아무튼, 브라우저를 닫고 일주일뒤 사용자가 사이트에 들어와도 기록은 남아있는다.

즉 정보가 계속해서 남아있으니 좋을리가. ㅋㅋ

## 그럼 로컬스토리지를 어떻게 이용할까?

일단 로그인을 하는 과정에 대해서 알 필요가 있다.

사용자가 로그인을 시도할 때, 서버에서는 세션 또는 JWT를 사용자에게 주게된다.
간단하게 둘다 입장권이라고 생각해도 전혀 무방하다. 다만 입장권을 서버에 제시했을때 차이점이 있다.

세션과 JWT 둘 모두 로그인을 할 당시에 아이디와 비밀번호를 서버에 전송해 비교하고, 값이 일치한지 확인한다.

1. 여기서 일치하다면 세션방식은 랜덤하게 생성된 일종의 일련번호를 사용자에게 발급해주고, 그 값을 서버에 저장해둔다. 만약 사용자가 얻게된 일련번호 티켓은 사용자가 무언가 인증이 요구되는 어떤 행동을 할 때 서버에 전송하게되는 티켓이라 생각하면 된다.
   만약 사용자가 무언가 하기위해 서버에 티켓을 넘기면서 허락을 요구한다고 할 때, 서버에서는 그 티켓의 값을 서버에서 같은 값을 찾고, 만약 찾을 경우에 인증이 허가된다.

2. JWT는 세션과 다르게 정보량이 제공된다. 단, 정보량이 많은 만큼 서버에 요청하면 인증허가가 바로된다.

그래서 세션을 어떻게 하는가?
세션을 받게된다면 로컬스토리지에 저장한다.
그리고 그 값을 토대로 무언가 사용자가 원하는 값을 요청하게 하면 되는것.
(사실 공부를 덜해서 여기까지밖에 모른다.)

```jsx
useEffect(() => {
  supabase.auth.getSession().then(({ data: { session } }) => {
    setSession(session);
  });

  supabase.auth.onAuthStateChange((_event, session) => {
    setSession(session);
  });
}, []);
```

이 코드는 현재 supabase를 이용해서 세션을 가져오고, 세션의 상태를 업데이트하여 로그인여부를 파악하는 코드이다.

```jsx
useEffect(() => {
  let ignore = false;

  async function getProfile() {
    try {
      setLoading(true);

      if (session && session.user) {
        const { user } = session;

        const { data, error } = await supabase
          .from("users")
          .select(`username, nickname, profile_img`)
          .eq("user_email", user.id)
          .single();

        if (!ignore) {
          if (error) {
            console.warn(error);
          } else if (data) {
            setUsername(data.username);
            setNickname(data.nickname);
            setProfileImg(data.profile_img);
          }
        }
      }
    } catch (error) {
      console.error("Error in getProfile:", error);
    } finally {
      setLoading(false);
    }
  }

  getProfile();

  return () => {
    ignore = true;
  };
}, [session]);
```

로그인은 앞으로 계획한 미니프로젝트에 필요한 필수요소이다. 꼭 공부하도록..
