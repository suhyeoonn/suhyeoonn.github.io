---
title: Next.js next/navigation Router 로그인 페이지 리다이렉트
categories: [NextJS]
tags: [nextjs, auth, book-rating] # TAG names should always be lowercase
description: NextJS의 next/navigation Router로 로그인 페이지 리다이렉트 시 발생하는 문제 해결하기
---

## 뒤로가도 로그인 페이지로 되돌아오는 문제

로그인하지 않은 상태에서 마이페이지 진입 시 로그인 페이지로 이동한다. 이때, 뒤로가기를 누르면 다시 로그인 페이지로 돌아오는 문제가 발생했다.

> **문제 상황**
>
> 1. 홈 페이지에서 마이 페이지 진입 시도
> 2. 로그인 페이지로 이동
> 3. 로그인을 하지 않은 채로 뒤로가기
> 4. 다시 로그인 페이지로 이동된다.
>
> **기대**
>
> 뒤로 가기를 누르면 홈 메뉴로 이동해야 한다.

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

![push stack](/assets/img/posts/2024-12/2024-12-19/url-stack-1.png)
_Home -> My (차단됨) -> Login_

이 상태에서 뒤로가기를 누를 경우, 브라우저가 히스토리 스택에 기록된 이전 URL(My)로 이동을 시도한다. 하지만 마이페이지로 이동하자마자 리다이렉트 조건이 다시 실행되어 로그인 페이지로 돌아오는 문제가 발생한다.
`push` 대신 `replace`를 사용하면 문제가 해결된다.

### Replace

`replace`는 현재 URL을 **히스토리 스택에서 대체**한다. `replace`를 사용하면 브라우저 히스토리 스택에서 마이페이지 URL이 대체되어 기록되지 않는다. 따라서, 히스토리 스택은 다음과 같이 된다.

![replace stack](/assets/img/posts/2024-12/2024-12-19/url-stack-2.png)
_Home -> Login_

로그인 페이지에서 뒤로가기를 누르면, 더 이상 마이페이지로 돌아가지 않고 홈 메뉴로 이동하게 된다.

push를 사용하면 로그인 페이지가 히스토리 스택에 추가되어 뒤로가기를 누르면 다시 로그인 페이지로 이동하는 문제가 발생한다. 이를 해결하려면 replace를 사용해 스택을 대체하여, 뒤로가기 버튼으로 원래 페이지(예: Home)로 돌아갈 수 있도록 설정해야 한다.

## 로그인 후 이전 페이지로 돌아가기

이제 뒤로가기 문제는 해결이 되었는데, 한 가지 불편한 점이 생겼다.

> **문제 상황**
>
> 1. 홈 페이지에서 마이 페이지 진입 시도
> 2. 로그인 페이지로 replace
> 3. 로그인 후 / 경로인 홈으로 이동
>
> **기대**
>
> 로그인 후 마이페이지로 바로 진입하는 게 자연스러워 보인다.

현재 코드는 이렇다.

```tsx
const Protected = ({ children }: ProtectedProps) => {
  const { user } = useAuth();
  const router = useRouter();

  if (!user) {
    router.replace("/login");
    return;
  }

  return children;
};
```

로그인을 하지 않은 상태라면 `/login` 경로로 replace 한다. 그리고 로그인이 완료되면 `/` 경로로 replace 시킨다.

```tsx
const login = async (values: LoginInfo) => {
  const data = await loginApi(values);
  if (!data) return;

  setUser(data.user);
  setToken(data.token);

  router.replace("/");
};
```

이전 페이지로 돌아가기 위해선 경로를 기억할 필요가 있다. `usePathname`과 `useSearchParams`를 사용하여 문제를 해결하였다.

```tsx
import { useRouter, usePathname } from "next/navigation";

interface ProtectedProps {
  children: ReactNode;
}
const Protected = ({ children }: ProtectedProps) => {
  const { user } = useAuth();
  const router = useRouter();

  const pathname = usePathname();

  if (!user) {
    router.replace("/login?next=" + pathname);
    return;
  }

  return children;
};
```

쿼리스트링으로 next 값에 현재 경로를 넣는다. 그리고 로그인이 완료되면 next경로로 replace 한다. 만약, 로그인 페이지 경로를 사용자가 직접 입력하여 들어올 경우 next 값이 없을 수 있으니 이땐 `/` 경로로 이동시킨다.

```tsx
const params = useSearchParams();

const login = async (values: LoginInfo) => {
  const data = await loginApi(values);
  if (!data) return;

  setUser(data.user);
  setToken(data.token);

  router.replace(params?.get("next") || "/");
};
```

이젠 로그인을 하면 홈 페이지가 아닌 마이페이지로 바로 이동한다. ([관련 커밋](https://github.com/suhyeoonn/book-rating-frontend/commit/b7e10c6f373d73614c55858dd3f08983f85e2abe))

---

어드민툴, 내부망 사이트를 작업할 땐 모든 페이지는 로그인을 해야지만 사용이 가능했기에 이런 처리를 할 일이 없었는데 이번 기회로 생각보다 고려할 부분이 많다는 걸 알게되었다.
