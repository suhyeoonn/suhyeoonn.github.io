---
title: React 통합테스트 testing-library
categories: [React]
tags: [react, 테스트 자동화, book-rating] # TAG names should always be lowercase
description: testing-library 기본 문법 정리
---

풀스택으로 개인 프로젝트를 작업 중인데, 직접 동작을 테스트하려니 시간이 오래 걸려 테스트를 자동화하려고 한다.

현재 가장 큰 문제는 요구사항을 문서로 정리하지 않고 생각나는 대로 개발을 진행하다 보니 어떤 기능들이 있는지 명확한 정리된 문서가 없다는 점이다. 지금이라도 사양서를 작성할까 고민했지만, 테스트 코드를 작성하면 코드 자체가 일부 사양서의 역할도 할 수 있을 것 같아 통합 테스트를 도입하기로 했다.

통합 테스트는 여러 모듈이 함께 동작할 때 시스템이 기대대로 작동하는지 검증하며, 예상치 못한 상호작용 문제를 발견하는 데 초점을 맞춘다. 이 과정에서 `testing-library`를 활용해 테스트를 자동화할 예정이다. 이를 통해 코드 품질을 높이고, 개발 속도를 유지하면서도 안정성을 확보할 수 있기를 기대한다.

### testing-library

testing-library는 jest를 기반으로 만들어졌으며, DOM을 쿼리하여 웹 페이지를 테스트하는 라이브러리이다. 실제 사용자가 페이지에서 사용하는 방식과 유사하게 DOM에서 노드를 쿼리하여 테스트한다.

공식 문서도 설명이 잘 되어있는 편이나 한국어로 번역되지는 않았다. 어떻게 공부할지 고민했는데, 코딩앙마의 강의가 있었다.

{% include embed/youtube.html id='K1w6WN7q6k8' %}
_[React Testing Library #1 App.test.js, 간단한 테스트 작성](https://youtu.be/K1w6WN7q6k8?si=H6b0cBmq_nY2C3If)_

러닝 커브가 많지 않을까 걱정했는데 정말 설명이 잘되어있어서 이거 보고 나니 괜히 겁만 먹은 것 같았다. 물론 실제로 테스트하다 보면 어려울 때도 생기겠지만 이 영상을 보고 자신감이 많이 붙었다.

## 요소 검사

### getByRole

getByRole은 html role로 조회하는 함수다. role에 대해서는 [codingeverybody-html role](https://codingeverybody.kr/html-role-%EC%86%8D%EC%84%B1%EC%9D%98-%ED%99%9C%EC%9A%A9-%EB%B0%A9%EB%B2%95/#role-attr-3) 에서 잘 설명되어 있다.

```jsx
export default function MyPage({ user }) {
  return (
    <div>
      <h1>hello</h1>
      <h2>world</h2>
    </div>
  );
}
```

두 개의 헤딩이 있다.

```jsx
import MyPage from "./MyPage";
import { render, screen } from "@testing-library/react";

test("제목이 있다", () => {
  render(<MyPage />);
  const titleEl = screen.getByRole("heading");
  expect(titleEl).toBeInTheDocument();
});

// ❗️ error:  TestingLibraryElementError: Found multiple elements with the role "heading"
```

이렇게 작성하면 에러가 난다. **getBy는 하나만** 가져온다. 여러 개일 경우 AllBy를 사용한다.

> toBeInTheDocument: 문서에 존재하는지 확인한다.
{: .prompt-tip }

두 번째 파라미터로 옵션을 전달할 수도 있다. `level: 1` 로 설정하면 h1을 찾는다.
```jsx
const titleEl = screen.getByRole("heading", { level: 1 });
```
다른 예시를 보자.
```jsx
export default function MyPage() {
  return (
    <div>
      <div>
        <label htmlFor="username">이름</label>
        <input type="text" id="username" />
      </div>
      <div>
        <label htmlFor="profile">자기소개</label>
        <textarea type="text" id="profile" />
      </div>
    </div>
  );
}
```

input과 textarea가 있다.

```jsx
import MyPage from "./MyPage";
import { render, screen } from "@testing-library/react";

test("input 요소가 있다.", () => {
  render(<MyPage />);
  const inputEl = screen.getByRole("textbox");
  expect(inputEl).toBeInTheDocument();
});

// ❗️error: TestingLibraryElementError: Found multiple elements with the role "textbox"
```

input과 textarea가 같은 textbox role을 가지기에 에러가 발생한다.

```jsx
const inputEl = screen.getByRole("textbox", {
  name: "자기소개",
});
```

`name` 속성을 사용하여 **label에 해당하는 텍스트**를 입력하면 해당 레이블이 존재하는지 확인한다.
혹은 옵션 대신 getByLabelText를 사용해도 동일하다.

```jsx
const inputEl = screen.getByLabelText("자기소개");
```

> label을 찾는 게 아니라 label과 연결된 text 요소를 찾음에 주의

```jsx
<div>
  <label htmlFor="username">이름</label>
  <input type="text" id="username" value="Tom" readOnly />
</div>
```

### getByTestId

```jsx
export default function MyPage() {
  return (
    <div>
      <div>
        <label htmlFor="username">이름</label>
        <input type="text" id="username" value="Tom" readOnly />
      </div>
      <div /> {/*<- 검사할 요소*/}
    </div>
  );
}
```

`div` 가 있는지 검사하려고 하는데 사용할 메서드가 마땅치 않다. 이럴 땐 `testid` 속성을 사용한다.
`div`에 `data-tsetid` 속성을 추가한다.

```jsx
<div data-testid="my-div" />
```

그리고 `getByTestId` 메서드를 사용하여 `my-div`가 있는지 검사한다.

```jsx
test("input 요소가 있다.", () => {
  render(<MyPage />);
  const inputEl = screen.getByTestId("my-div");
  expect(inputEl).toBeInTheDocument();
});
```

하지만 이 경우 테스트를 위해 기존 코드를 수정해야 하므로 영상에서는 최후의 수단으로 설명한다.

## get, query, find 차이

다음과 같은 리스트를 테스트하려면,

```jsx
export default function UserList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user}>{user}</li>
      ))}
    </ul>
  );
}
```

아래처럼 getAllByRole로 여러 개를 가져오고, toHaveLength로 길이를 검사할 수 있다.

```jsx
const users = ["Alice", "Bob", "Charlie"];
test("li가 3개 있다", () => {
  render(<UserList users={users} />);
  const liEls = screen.getAllByRole("listitem");
  expect(liEls).toHaveLength(3);
});
```

그런데 빈 배열일 때 li가 없는지를 테스트하기 위해 다음과 같이 작성하면 에러가 발생한다.

```jsx
test("li가 없다", () => {
  render(<UserList users={[]} />);
  const liEls = screen.getAllByRole("listitem");
  expect(liEls).toHaveLength(0);
  // expect(liEls).not.toBeInTheDocument(); 이것도 에러가 발생
});
```

`get`은 찾는 요소가 없을 시 에러를 반환한다. 그래서 다른 함수를 써야하는데, `get` 말고도 `query`, `find` 함수가 존재한다.
![Pasted image 20250102231055.png](/assets/img/posts/2025/01/Pasted%20image%2020250102231055.png)
_[testing-library.com](https://testing-library.com/docs/queries/about)_

- get: 없으면 에러 반환
- query: 없으면 null 이나 빈 배열을 반환. 에러 발생 x
- find: 없으면 get처럼 에러를 반환하는데 정한 시간 동안 요소가 나타나는지 기다리기 가능 (async/await)

따라서, `li`가 없는지 확인하려면 아래처럼 `query`를 사용한다.

```jsx
test("li가 없다", () => {
  render(<UserList users={[]} />);
  const liEls = screen.queryByRole("listitem");
  expect(liEls).not.toBeInTheDocument();
});
```

## 이벤트 테스트

간단한 로그인 컴포넌트이다.

```jsx
import { useState } from "react";

export default function Login() {
  const [isLogin, setIsLogin] = useState(false);

  function onClickHandler() {
    setIsLogin(!isLogin);
  }

  return (
    <button onClick={onClickHandler}>{isLogin ? "Logout" : "Login"}</button>
  );
}
```

userEvent.setup()을 한 뒤, click 메서드를 사용한다. 이 때, promise가 반환되기에 async/await을 사용해야 한다.

```js
const user = userEvent.setup();
test("한번 클릭하면 Logout 버튼이 된다.", async () => {
  render(<Login />);
  const buttonEl = screen.getByRole("button");
  await user.click(buttonEl); // promise를 반환
  expect(buttonEl).toHaveTextContent("Logout");
});
```

키보드 이벤트 테스트도 간단하다.

```js
test("tab, space, enter 동작.", async () => {
  render(<Login />);
  const buttonEl = screen.getByRole("button");
  await user.tab();
  await user.keyboard(" ");
  expect(buttonEl).toHaveTextContent("Logout");
});
```
