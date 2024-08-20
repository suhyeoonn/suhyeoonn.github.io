---
title: 메모이제이션과 리액트 memo
date: 2024-08-06 19:00:00 +/-TTTT
categories: [React]
tags: [javascript, react, TIL] # TAG names should always be lowercase
description: 메모이제이션 개념 정리 및 리액트 memo 알아보기
---

## **메모이제이션이 뭐지?**

> **메모이제이션**(memoization)은 컴퓨터 프로그램이 동일한 계산을 반복해야 할 때, 이전에 계산한 값을 메모리에 저장함으로써 동일한 계산의 반복 수행을 제거하여 프로그램 실행 속도를 빠르게 하는 기술이다.
> _메모이제이션을 글자 그대로 해석하면 ‘메모리에 넣기’라는 의미이다_
>
> \- [위키백과](https://ko.wikipedia.org/wiki/%EB%A9%94%EB%AA%A8%EC%9D%B4%EC%A0%9C%EC%9D%B4%EC%85%98) -

메모이제이션은 리액트에서만 사용되는 개념이 아닌, 프로그래밍에 널리 사용되는 기술이다.

**Q. [왜 메모라이제이션이라고 안하고 메모이제이션이라고 하나요?](https://stackoverflow.com/questions/77884747/why-is-it-called-memoization)**

A. "메모이제이션”은 단순히 무언가를 기억하는 “암기”와는 다르며, 특정한 목적(성능 향상)으로 무언가를 기억하는 것을 의미하기 위해 만들어진 용어입니다.

## **캐싱과 비슷한건가?🤔**

내가 이해한 바론, 캐싱과 같은 거라고 볼 수 있다. 다만, 캐싱은 접근 속도를 개선하기 위해 무언가를 저장하는 모든 것에 대한 포괄적인 개념이라면 메모이제이션은 함수와 관련된 캐싱이라고 볼 수 있다.

[여기](https://www.geeksforgeeks.org/what-is-memoization-a-complete-tutorial/) 에서는 이렇게 설명한다.

> 메모이제이션은 실제로 특정 유형의 캐싱입니다. 캐싱이라는 용어는 일반적으로 나중에 사용할 수 있는 모든 저장 기술(예: HTTP 캐싱)을 지칭할 수 있지만 메모이제이션은 값을 반환하는 캐싱 함수를 보다 구체적으로 지칭합니다.

[스택 오버플로우](https://stackoverflow.com/questions/6469437/what-is-the-difference-between-caching-and-memoization)에서도 동일한 질문이 있는데, 답변은 다음과 같다.

> **메모이제이션 은 함수의 매개변수에 따라 함수의 반환 값을 캐싱** 하는 특정 형태의 캐싱입니다 .

따라서, 메모이제이션은 캐싱이긴한데 함수의 반환 값을 캐싱하는 것이다.

### 메모이제이션 구현 예제

어떤 식으로 반환 값을 캐싱한다는 건지 좀 더 이해해보자. 예를 들어, 다음과 같은 `add` 함수가 있다.

```jsx
function add(a, b) {
  return a + b;
}
```

다음처럼 같은 매개변수 1, 2를 전달하여 함수를 여러 번 호출한다고 치자.

```jsx
add(1, 2);
add(1, 2);
add(1, 2);
```

원래라면 매번 1+2 연산을 할 것이다. 그런데 이걸 메모이제이션 한다면?

`add(1+2)` 을 한 번 호출하여 결과(3)를 어딘가에 저장한 후, 이후에 또 다시 `add(1+2)`가 호출될 때 더하기 연산을 하는 것이 아니라 **이미 저장된 결과값인 3을 바로 반환**하는 것이다.

다음은 메모이제이션을 구현하는 간단한 예제이다.

```jsx
function memoize(fn) {
  const cache = new Map(); // 결과를 저장할 Map 객체 생성

  return function (...args) {
    const key = `${a},${b}`; // 인자를 문자열로 변환하여 캐시의 키로 사용

    if (cache.has(key)) {
      return cache.get(key); // 캐시에 결과가 있으면 반환
    }

    const result = fn(...args); // 결과를 계산
    cache.set(key, result); // 결과를 캐시에 저장
    return result; // 계산된 결과 반환
  };
}

// 원본 함수
function add(a, b) {
  return a + b;
}

// 메모이제이션된 함수 생성
const memoizedAdd = memoize(add);

// 사용 예시
console.log(memoizedAdd(1, 2)); // 3
console.log(memoizedAdd(1, 2)); // 캐시된 결과: 3
console.log(memoizedAdd(2, 3)); // 5
```

이렇게 하면 동일한 인자로 함수가 호출될 때마다 계산을 반복하지 않고, 저장된 결과를 효율적으로 재사용할 수 있다.

## **React.memo: 컴포넌트 리렌더링 최적화**

React.memo는 컴포넌트를 메모이제이션하여 불필요한 리렌더링을 방지하는 데 사용된다. 컴포넌트가 동일한 props를 받는 경우, React.memo는 해당 컴포넌트를 다시 렌더링하지 않고, 이전 렌더링 결과를 재사용한다.

말로만 봐서는 무슨 말인지 모르겠다. 아래 예제를 보면서 이해해보자.

### memo: props가 변경될 때만 리렌더링

React는 일반적으로 부모가 리렌더링될 때마다 컴포넌트를 리렌더링한다. `memo`를 사용하면 부모가 리렌더링될 때 새로운 props가 이전 props와 동일하면 리렌더링 되지 않는 컴포넌트를 만들 수 있다. 이러한 컴포넌트를 _memoized_ 상태라고 한다.

다음은 [공식 문서에 있는 예제](https://ko.react.dev/reference/react/memo#skipping-re-rendering-when-props-are-unchanged)이다.

```jsx
import { memo, useState } from "react";

export default function MyApp() {
  const [name, setName] = useState("");
  const [address, setAddress] = useState("");
  return (
    <>
      <label>
        Name{": "}
        <input value={name} onChange={(e) => setName(e.target.value)} />
      </label>
      <label>
        Address{": "}
        <input value={address} onChange={(e) => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log("Greeting was rendered at", new Date().toLocaleTimeString());
  return (
    <h3>
      Hello{name && ", "}
      {name}!
    </h3>
  );
});
```

`Greeting` 컴포넌트는 `name`이 변경될 때만 리렌더링되고, `address`가 변경될 때는 리렌더링 되지 않는다.

### props 비교 방식

주의! **prop가 객체, 배열 또는 함수인 경우 컴포넌트가 리렌더링됩니다.** 
`memo`는 배열, 객체 또는 함수와 같은 props가 전달되는 경우에 완전히 무용지물이다. 이 경우 [`useMemo`](https://ko.react.dev/reference/react/useMemo#skipping-re-rendering-of-components)와 [`useCallback`](https://ko.react.dev/reference/react/useCallback#skipping-re-rendering-of-components) 가 필요할 수 있다.

React는 **얕은 비교**를 기준으로 이전 props와 새로운 props를 비교한다.

```jsx
const obj1 = { name: "clap" };
const obj2 = { name: "clap" };

console.log(obj1 === obj2); // false
```

여기서 `obj1`와 `obj2`의 내용은 동일하지만, 서로 다른 메모리 위치에 저장되어 있기 때문에 `===` 연산자는 두 객체가 동일하지 않다고 판단한다.

즉, 각각의 새로운 prop가 이전 prop와 참조가 동일한지 여부를 고려한다. 부모가 리렌더링 될 때마다 새로운 객체나 배열을 생성하면, 개별 요소들이 모두 동일하더라도 React는 여전히 변경된 것으로 간주한다. 마찬가지로 부모 컴포넌트를 렌더링할 때 새로운 함수를 만들면 React는 함수의 정의가 동일하더라도 변경된 것으로 간주한다.

이 경우 [`useCallback`](https://ko.react.dev/reference/react/useCallback#skipping-re-rendering-of-components) 와 [`useMemo`](https://ko.react.dev/reference/react/useMemo#skipping-re-rendering-of-components) 가 필요할 수 있다.
