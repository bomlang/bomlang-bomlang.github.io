---
layout: post
title: "[유데미x스나이퍼팩토리] 프로젝트 캠프 : Next.js 3기 - 2주차 수업"
date: 2024-10-06
categories:
  - study
  - etc
description: >
  [유데미x스나이퍼팩토리] 프로젝트 캠프 : Next.js 3기 2주차 수업을 수강후 회고
---

# Next.js 3기 - 2주차 수업 정리

## 다이나믹 경로와 Params

Next.js에서 `page.ts` 파일 외에 props를 직접 받을 수 없습니다. 따라서 `useParams`를 사용해 URL의 동적 파라미터를 받아와야 합니다.

🚀 서버에서 렌더링할 때, HTML과 CSS는 먼저 렌더링되고 그 이후에 하이드레이션(hydration)이 발생합니다. 하지만 자바스크립트(JS)가 아예 값을 읽지 못하는 것은 아닙니다.

예를 들어, `useState`에서 초기값을 10으로 설정하고, 이를 변수에 할당하여 렌더링한다고 가정해봅시다. 브라우저에서 JS를 꺼놓고 렌더링하면 `useState`는 JS이기 때문에 값을 못 가져올 것이라고 예상할 수 있지만, 실제로는 초기값인 10이 출력됩니다. 이는 초기 하이드레이션 이전에도 자바스크립트가 값을 참조할 수 있음을 의미합니다.

### 포괄적 경로

동적 경로는 `[id]` 형식으로, 캐치올(catch-all) 세그먼트는 `[...id]` 형식으로 작성합니다. 이 두 가지 경로가 동시에 있을 경우 동적 경로가 우선시됩니다.

## Next.js 캐싱

Next.js에는 4가지 캐싱 메커니즘이 있습니다.

1. **데이터 캐싱**
2. **클라우드 캐싱**
3. **라우터 캐싱** (React 서버의 데이터를 캐싱함)
4. **리퀘스트 메모라이제이션 캐싱**

`Link` 컴포넌트를 사용할 경우 CSR(클라이언트 사이드 렌더링) 캐싱이 30초 동안 유지됩니다. 이 캐싱을 무효화하는 방법은 **새로고침**입니다. 새로고침하면 캐시가 다시 설정되며, 이때부터 30초 동안 캐싱이 다시 적용됩니다.

### 캐싱 흐름

처음 새로고침할 때는 `Miss` 상태로 시작되며, `Full Router 캐싱`을 통과하여 `Hit` 상태가 됩니다. 이후 다시 라우터 캐싱을 설정하고, 30초 동안 유지됩니다.

정적 페이지는 **빌드** 시 `Full Router 캐싱`이 적용됩니다. 빌드된 결과물은 설정된 캐시 시간을 기준으로 유지되며, 예를 들어 11:45에 빌드된 페이지라면 이후에도 그 시점의 데이터가 그대로 표시됩니다.

### 정적 페이지를 동적으로 만드는 방법

1. 상위에 `export const dynamic = "force-dynamic";`를 사용하여 동적 페이지로 전환할 수 있습니다.

   ```js
   export const dynamic = "force-dynamic";
   export default function About() {
     return <h1>About</h1>;
   }
   ```

2. `revalidate` 속성을 0으로 설정하면 항상 최신 데이터를 가져올 수 있습니다.

   ```js
   export const revalidate = 0;
   export default function About() {
     return <h1>About</h1>;
   }
   ```

## 리퀘스트 메모라이제이션

동일한 경로로 들어오는 동일한 요청이 있다면, 이전 요청에서 캐싱된 값을 재사용하는 방식입니다. 예를 들어, 아래 코드에서 `random`과 `random2`는 동일한 값을 갖게 됩니다.

```js
export default async function Home() {
  const random = await (await fetch("http://localhost:3000/random")).json();
  const random2 = await (await fetch("http://localhost:3000/random")).json();
  return (
    <>
      <h1>Home</h1>
      <div>{random}</div>
      <div>{random2}</div>
      {new Date().toLocaleTimeString()}
    </>
  );
}
```

`random2`는 별도의 요청을 보내지 않고 `random`의 값을 그대로 사용합니다. 이는 리퀘스트 메모라이제이션 캐싱 덕분입니다.

## 신선한 데이터를 유지하는 방법: Revalidate

`revalidate`는 Next.js에서 사용하는 속성으로, 특정 시간 간격마다 새로운 데이터를 받아올 수 있게 설정하는 옵션입니다. 캐시된 데이터를 일정 시간 동안 유지하다가, 그 시간이 지나면 새로운 데이터를 요청하게 됩니다.

```js
export const revalidate = 10; // 10초마다 새 데이터 요청
export default async function Home() {
  const random = await (await fetch("http://localhost:3000/random")).json();
  const random2 = await (await fetch("http://localhost:3000/random")).json();
  return (
    <>
      <h1>Home</h1>
      <div>{random}</div>
      <div>{random2}</div>
      {new Date().toLocaleTimeString()}
    </>
  );
}
```

이처럼 `revalidate`를 사용하면, 페이지는 전역적으로 10초마다 새로운 데이터를 요청합니다.

### `cache: no-store` 값 설정

```js
export default async function Home() {
  const random = await (
    await fetch("http://localhost:3000/random", {
      cache: "no-store",
    })
  ).json();
  const random2 = await (await fetch("http://localhost:3000/random")).json();
  return (
    <>
      <h1>Home</h1>
      <div>{random}</div>
      <div>{random2}</div>
      {new Date().toLocaleTimeString()}
    </>
  );
}
```

`cache: "no-store"`를 사용하면 캐싱을 건너뛰고, 항상 새로운 데이터를 받아옵니다. 저장된 데이터가 있더라도 무시하고 새 데이터를 가져옵니다.

## 2주차 수업 후기

이번 수업은 Next.js의 심화된 내용, 특히 **캐싱**과 관련된 부분이 매우 유익했습니다. 캐싱 메커니즘을 정확히 이해함으로써 더 나은 성능을 구현할 수 있다는 점이 인상 깊었습니다. 기회가 된다면 이 강사님의 다른 강의도 들어보고 싶습니다.
