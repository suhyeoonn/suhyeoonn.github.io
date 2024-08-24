---
title: React useRef 타입 정리 (feat. MutableRefObject 에러)
date: 2024-08-23 1:00:00 +/-TTTT
categories: [React]
tags: [react, TIL] # TAG names should always be lowercase
description: 리액트 useRef 훅의 세 가지 타입을 알아보고, MutableRefObject 관련 에러의 원인 파악하기
---

useRef 사용 시 다음과 같은 에러가 발생했다.

> Type `MutableRefObject<HTMLInputElement | undefined>` is not assignable to type `LegacyRef<HTMLInputElement> | undefined`.
{: .prompt-danger }

아래처럼 null로 초기화하면 에러가 사라지던데, 왜 그런지 알아보자.

```ts
const input = useRef<HTMLInputElement>(null);
```

useRef 함수를 타고 들어가보면 세 가지 타입이 정의되어 있는 걸 볼 수 있다.

1. `function useRef<T>(initialValue: T): MutableRefObject<T>;`
2. `function useRef<T>(initialValue: T | null): RefObject<T>;`
3. `function useRef<T = undefined>(): MutableRefObject<T | undefined>;`

## 1. useRef<T>(initialValue: T): MutableRefObject<T>

초기화 시 initialValue를 넣어주면 MutableRefObject 타입이 반환된다.

> mutable: 변할 수 있는, 잘 변하는
> 즉, RefObject이긴 한데 값을 변경할 수 있는 RefObject가 반환되는 것이다.
> 정확히는 제네릭과 초기값의 타입이 일치하면 MutableRefObject가 반환되는데, 이는 밑에서 다시 알아보자.

공식 문서의 [카운터 예제](https://ko.react.dev/reference/react/useRef#click-counter)를 보자.

```tsx
export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    // 👇 ref.current를 수정할 수 있다.
    ref.current = ref.current + 1;
    alert("You clicked " + ref.current + " times!");
  }

  return <button onClick={handleClick}>Click me!</button>;
}
```

useRef의 초기값으로 0을 주었다. 이는 타입 유추가 되어 결과적으로 다음과 같게 된다.

```
let ref = useRef<number>(0);
```

제네릭 타입과 초기값 타입이 일치하여 MutableRefObject가 반환되었기 때문에 `ref.current = ref.current + 1;` 과 같이 ref를 수정할 수 있게 되었다.

## 2. useRef<T>(initialValue: T | null): RefObject<T>

만약 위 코드에서 초기값을 null로 하고 싶어서 null을 넣어주면 어떻게 될까?

```tsx
let ref = useRef<number>(null);

function handleClick() {
  ref.current = ref.current + 1;
  alert("You clicked " + ref.current + " times!");
}
```


Cannot assign to 'current' because it is a read-only property.ts(2540) 에러가 발생한다. 초기값에 null이 들어갔기 때문에 수정할 수 없는 RefObject 타입이 반환되었는데, 이걸 수정하려고 하니 에러가 발생한 것이다.

## 3. useRef<T = undefined>(): MutableRefObject<T | undefined>

useRef로 DOM을 조작하는 대표적인 예시로 [input에 초점 맞추기](https://ko.react.dev/reference/react/useRef#examples-dom)가 있다.

```tsx
import { useRef } from "react";

export default function Form() {
  const inputRef = useRef<HTMLInputElement>(null);

  function handleClick() {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

그런데 초기값이 없으면 어떻게 될까?

```tsx
const inputRef = useRef<HTMLInputElement>();
```

다음과 같은 에러가 발생했다.

> Type 'MutableRefObject<HTMLInputElement | undefined>' is not assignable to type 'LegacyRef<HTMLInputElement> | undefined'.


초기값을 주지 않으면 수정할 수 있는 MutableRefObject 타입이 반환은 되는데 이건 1번 `useRef<T>(initialValue: T): MutableRefObject<T>`과는 다르게 undefined 일 수도 있게 된다.

ref는 다음과 같이 타입이 정의되어있다.

```tsx
interface RefAttributes<T> extends Attributes {
  ref?: LegacyRef<T> | undefined;
}

type LegacyRef<T> = string | Ref<T>;

type Ref<T> = RefCallback<T> | RefObject<T> | null;
```

즉, RefObject를 받지 MutableRefObject는 받을 수 없다. 그래서 에러가 발생하는 것이다.

```tsx
const inputRef = useRef<HTMLInputElement>(null);
```

이렇게 null을 넣어주면 RefObject가 반환되니 ref 속성으로 설정할 수 있다.

## 초기값을 null로 하되 수정도 되게 하려면?

제네릭으로 null을 같이 설정해주면 된다.

```tsx
const inputRef = useRef<HTMLInputElement | null>(null);
```

다시 1번과 2번을 보자.

1. function useRef<T>(initialValue: T): MutableRefObject<T>;
2. function useRef<T>(initialValue: T | null): RefObject<T>;

제네릭에 null 이 포함되어 있고, 초기값으로 null 을 줄 경우 이 null은 제네릭 T 타입과 일치하게 되고 MutableRefObject 가 반환된다. 즉, 제네릭 타입과 초기값 타입이 일치하게 되는 경우 MutableRefObject가 반환되는 것이다.
반면, 제네릭에 null이 없고 초기값으로 null을 넣으면 제네릭 타입으로 지정되지 않은 null 이 초기값으로 들어갔고, 이는 제네릭 타입과 초기값 타입이 일치하지 않게 된다. 그럼 RefObject가 반환된다.

[스톱워치](https://ko.react.dev/reference/react/useRef#referencing-a-value-with-a-ref) 예제를 보자.

```tsx
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}

```

`handleStart`에서 setInterval ID를 intervalRef 에 저장한다. 위 코드에 타입스크립를 적용한다면 useRef는 어떻게 선언해야할까?

값을 변경할 수 있어야 하니 MutableRefObject가 반환되도록 해야 한다.
초기값을 주지 않으면 MutableRefObject가 반환되니 이렇게 할 수도 있다.

```tsx
const intervalRef = useRef<NodeJS.Timeout>();
```

그런데 null로 초기값을 주면서 수정도 되게 하려면 제네릭에 null을 포함시키자.

```tsx
const intervalRef = useRef<NodeJS.Timeout | null>(null);
```

## 정리

> - 제네릭과 초기값 타입이 일치? -> 수정 가능 (MutableRefObject)
> - 초기값 없음? => 수정은 되나 undefined도 가짐 (MutableRefObject \| undefined)
> - 제네릭 타입에 null 없이 null로 초기화? => 수정 불가능 (RefObject)
{: .prompt-info}
