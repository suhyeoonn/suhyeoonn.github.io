---
title: NextJS Router push와 replace 차이 및 리다이렉트 문제 해결
categories: [NextJS]
tags: [nextjs, auth] # TAG names should always be lowercase
description: NextJS router의 push와 replace 차이
---

로그인하지 않은 상태에서 마이페이지 진입 시 로그인 페이지로 이동한다. 이때, 뒤로가기를 누르면 다시 로그인 페이지로 돌아오는 문제가 발생했다.
홈 메뉴 > 마이페이지 진입 시도 > 로그인 페이지로 이동 > 뒤로가기 > 로그인 페이지인 상태인데, 뒤로가기를 누르면 홈 메뉴로 이동해야 한다.

리다이렉트 시 push() 를 사용했는데 이게 원인이었다.

```ts
if (!user) {
  router.push("/login");
}
```

`push`와 `replace`는 **히스토리 스택 관리** 방식에 차이가 있다. 둘 다 URL을 변경하는 역할을 하지만, 브라우저의 **히스토리 스택**에 대한 동작이 다르다.

### Push

`push`는 새로운 항목을 **히스토리 스택에 추가**한다. 뒤로가기 버튼을 눌렀을 때 이전 페이지로 돌아갈 수 있다.
하지만 push로 로그인 URL을 스택에 쌓을 경우 히스토리 스택은 다음과 같이 된다.

![push stack](/assets/img/posts/2024-12-19/url-stack-1.png)
_Home -> My (차단됨) -> Login_

이 상태에서 뒤로가기를 누를 경우, 브라우저가 히스토리 스택에 기록된 이전 URL(My)로 이동을 시도한다. 하지만 마이페이지로 이동하자마자 리다이렉트 조건이 다시 실행되어 로그인 페이지로 돌아오는 문제가 발생한다.
`push` 대신 `replace`를 사용하면 문제가 해결된다.

### Replace

`replace`는 현재 URL을 **히스토리 스택에서 대체**한다. `replace`를 사용하면 브라우저 히스토리 스택에서 마이페이지 URL이 대체되어 기록되지 않는다. 따라서, 히스토리 스택은 다음과 같이 된다.

![replace stack](/assets/img/posts/2024-12-19/url-stack-2.png)
_Home -> Login_

로그인 페이지에서 뒤로가기를 누르면, 더 이상 마이페이지로 돌아가지 않고 홈 메뉴로 이동하게 된다.

### 결론

push를 사용하면 로그인 페이지가 히스토리 스택에 추가되어 뒤로가기를 누르면 다시 로그인 페이지로 이동하는 문제가 발생한다. 이를 해결하려면 replace를 사용해 스택을 대체하여, 뒤로가기 버튼으로 원래 페이지(예: Home)로 돌아갈 수 있도록 설정해야 한다.
