---
title: 리액트 메모이제이션 - memo, useCallback
date: 2024-08-06 19:00:00 +/-TTTT
categories: [React]
tags: [javascript, react] # TAG names should always be lowercase
description: 리액트 메모이제이션 개념 정리 및 관련 API 알아보기
---

## 메모이제이션이 뭐지?

**메모이제이션**(memoization)은 컴퓨터 프로그램이 동일한 계산을 반복해야 할 때, 이전에 계산한 값을 메모리에 저장함으로써 동일한 계산의 반복 수행을 제거하여 프로그램 실행 속도를 빠르게 하는 기술이다. - [위키백과](https://ko.wikipedia.org/wiki/%EB%A9%94%EB%AA%A8%EC%9D%B4%EC%A0%9C%EC%9D%B4%EC%85%98) -

_메모이제이션은 글자 그대로 해석하면 ‘메모리에 넣기’라는 의미이다_

### 캐싱과 비슷한건가?🤔

메모이제이션(memoization)과 캐싱(caching)은 기본적으로 비슷한 개념입니다.

캐싱은 자주 사용하는 데이터나 결과를 저장하여 접근 속도를 개선하기 위해 사용하는 데 반해, 메모이제이션은 **동일한 입력값에 대해 함수의 결과를 재사용**하여 연산을 줄인다.

예를 들어, 다음과 같은 `add` 함수가 있다.

```jsx
function add(a, b) {
  return a + b;
}
```

이 함수를 메모이제이션 한다면 `add(1+2)` 을 한 번 호출하여 결과(3)를 저장한 후, 이후에 또 다시 `add(1+2)`가 호출될 때 더하기 연산을 하는 것이 아니라 그 결과값인 3을 바로 반환하는 것이다.

### 메모이제이션 구현 예제

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

## 리액트 메모이제이션

React에서는 메모이제이션 개념을 컴포넌트 렌더링에 적용하여, 동일한 props가 전달될 때 컴포넌트가 재렌더링되지 않도록 한다. 이는 컴포넌트가 불필요하게 재렌더링되는 것을 방지하여 앱의 성능을 향상시키는 데 도움을 준다.

메모이제이션과 관련된 3개의 리액트 API를 알아보자.

## memo: props가 변경될 때만 리렌더링

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
`memo`는 배열, 객체 또는 함수와 같은 props가 전달되는 경우에 **완전히 무용지물이**다. 이 경우 [`useMemo`](https://ko.react.dev/reference/react/useMemo#skipping-re-rendering-of-components)와 [`useCallback`](https://ko.react.dev/reference/react/useCallback#skipping-re-rendering-of-components) 가 필요할 수 있다. (밑에서 다시 다룰 예정)

React는 얕은 비교를 기준으로 이전 props와 새로운 props를 비교한다.

```jsx
const obj1 = { name: "clap" };
const obj2 = { name: "clap" };

console.log(obj1 === obj2); // false
```

여기서 `obj1`와 `obj2`의 내용은 동일하지만, 서로 다른 메모리 위치에 저장되어 있기 때문에 `===` 연산자는 두 객체가 동일하지 않다고 판단한다.

즉, 각각의 새로운 prop가 이전 prop와 참조가 동일한지 여부를 고려한다. 부모가 리렌더링 될 때마다 새로운 객체나 배열을 생성하면, 개별 요소들이 모두 동일하더라도 React는 여전히 변경된 것으로 간주한다. 마찬가지로 부모 컴포넌트를 렌더링할 때 새로운 함수를 만들면 React는 함수의 정의가 동일하더라도 변경된 것으로 간주한다.

이 경우 [`useCallback`](https://ko.react.dev/reference/react/useCallback#skipping-re-rendering-of-components) 와 [`useMemo`](https://ko.react.dev/reference/react/useMemo#skipping-re-rendering-of-components) 가 필요할 수 있다.

## useCallback: 함수 정의 캐싱

`useCallback`은 리렌더링 간에 함수 정의를 캐싱해 주는 훅(hook)이다.

[공식 문서에 나오는 예제](https://ko.react.dev/reference/react/useCallback#usage)를 통해 알아보자.

```jsx
function ProductPage({ productId, referrer, theme }) {
  // theme이 바뀔때마다 다른 함수가 될 것입니다...
  function handleSubmit(orderDetails) {
    post("/product/" + productId + "/buy", {
      referrer,
      orderDetails,
    });
  }

  return (
    <div className={theme}>
      {/* ... 그래서 ShippingForm의 props는 같은 값이 아니므로 매번 리렌더링 할 것입니다.*/}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

ShippingForm의 onSubmit props로 전달하는 handleSubmit 함수는 ProductPage가 렌더링될 때마다 handleSubmit 함수가 새로 만들어지기에, ShippingForm을 memo로 감쌌더라도 재렌더링 되는 문제가 있다.

여기서 알아야 할 점은

1. 기본적으로, 컴포넌트가 리렌더링할 때 React는 이것의 모든 자식을 재귀적으로 재렌더링한다.
   1. ProductPage 의 props가 변경되어 재렌더링될 경우 자식 컴포넌트인 ShippingForm은 기본적으로 재린더렝된다. 물론 지금은 memo를 사용해서 ShippingForm의 props가 변경되었을 때만 재렌더링되도록 한 상태이다.
2. 자바스크립트에서 함수는 객체이다. ([참고](https://ko.javascript.info/function-object))

   1. 위의 props 비교 방식의 객체 비교 예시와 마찬가지로 함수도 내부 코드가 동일하더라도 비교해보면 다른 함수로 판단된다.

      ```jsx
      const a = () => {
        console.log("hi");
      };

      const b = () => {
        console.log("hi");
      };

      console.log(a === b); // false
      ```

따라서, `ShippingForm` 의 onSubmit prop은 절대 같아질 수 없고 `memo` 최적화는 동작하지 않는 것을 의미한다. 여기서 `useCallback`이 유용하게 사용된다.

```jsx
function ProductPage({ productId, referrer, theme }) {
  // React에게 리렌더링 간에 함수를 캐싱하도록 요청합니다...
  const handleSubmit = useCallback(
    (orderDetails) => {
      post("/product/" + productId + "/buy", {
        referrer,
        orderDetails,
      });
    },
    [productId, referrer]
  ); // ...이 의존성이 변경되지 않는 한...

  return (
    <div className={theme}>
      {/* ...ShippingForm은 같은 props를 받게 되고 리렌더링을 건너뛸 수 있습니다.*/}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

`useCallback` 사용해서 함수를 캐싱하면 기존에 메모리에 저장된 함수가 props로 전달된다. 이는 동일한 객체(_=같은 메모리 주소를 가지는 객체_)로 판단되기에 의존성이 변경되지 않는 한 ShippingForm은 리렌더링 되지 않는다.
