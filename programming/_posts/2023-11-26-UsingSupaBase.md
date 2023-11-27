---
layout: post
title: 요즘에 떠오르는 Supa Base를 사용해보면서 느낀점 간단히 정리.
tag: supabase
---

# SupaBase? 알게된 경로

평소에 유튜브로 '노마드코더'를 구독해놓고 최신 개발소식을 들으면서 SupaBase또한 알게된 적이 있다. 프론트엔드를 공부하면서 작은 사이드프로젝트나 포트폴리오를 만드는데 백엔드의 데이터베이스는 무조건적으로 필요하다. 그렇지만 내 주변에는 백엔드가 없어서 이를 대체할 무언가를 찾아야했는데, 앞전에 멋사 6기 프론트엔드스쿨에선 리액트학습을 하면서 `pocketBase`를 사용하였다. 이 또한 v1.0.0도 출시안한 따끈따끈한 데이터베이스를 제공해주는 상품이었고, 무료이며 사용방법도 간단해서 프론트엔드를 공부하는 학생들은 백엔드를 신경쓰는 일이 많이 줄어들었다.

# 사용해보고 느낀점

서론이 길었는데, SupaBase는 멋사 플러스에서 리액트 프로젝트 과제를 진행하면서 백엔드가 필요한 과제였는데, 강사님께서 Supabase를 사용하여 댓글기능을 만들라 하신거다. 물론 사용방법을 간단하게 강사님께서 알려주셨지만, 실제로 사용해보니 테이블을 만들때만 코드를 사용해서 만든 부분만 제외하고는 api문서가 꽤나 친절하고 사용방법도 어렵지 않았다.
FireBase보다 빠르다고 하는데, FireBase를 사용해보진 않았지만 다른 PocketBase나 MongoDB를 사용했을때.. 솔직히 잘 안느껴진다. ㅋㅋㅋㅋㅋ

```ts
import { createClient } from "@supabase/supabase-js";

export const readComment = async () => {
  try {
    const { data, error } = await supabaseAdmin
      .from("video_comment")
      .select("*");

    if (error) {
      console.error(`데이터 통신에 실패하였습니다..😵‍💫 ${error.message}`);
    } else {
      return data;
    }
  } catch (error) {
    console.error(`데이터 통신에 실패하였습니다..😵‍💫 ${error}`);
    throw error;
  }
};
```

이런식으로 데이터를 가져왔다.

일단 기본적인거만 사용해서 잘 모르겠는데, 나머지 부가기능을 더 사용해봐야 자세한 평가를 할 수 있을거같다.
