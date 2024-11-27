---
title: Typescript Never 타입 정리
date: 2024-08-27 13:00:00 +/-TTTT
categories: [typescript]
tags: [typescript, TIL] # TAG names should always be lowercase
description: Typescript Never 타입 소개 및 void 타입과의 비교
---

노마드코더 타입스크립트 챌린지 참여 겸, [타입스크립트 강의](https://nomadcoders.co/typescript-for-beginners/lobby) 시청 중인데 `never` 타입에 대한 설명이 나왔다.
아직까지 실무를 하면서 실제로 `never`로 타입을 지정했던 적은 없는지라 자세히 알지 못한 상태에서 그냥 넘어갔었는데, 이번 기회에 정리해 보려고 한다.

---

## Never 타입

### 1. never 타입은 함수가 절대 리턴하지 않을 때 발생한다.

```ts
function hello(): never {
  throw new Error("error");
}

function keepProcessing(): never {
  while (true) {
    console.log("I always does something and never ends.");
  }
}
```

hello 함수는 리턴하지 않고 오류를 발생시키고, keepProcessing는 무한 루프에 빠져 함수가 종료되지 않는다. 이럴 때 `never`를 사용한다.

### 2. 타입이 두 가지 일 수도 있는 상황에 발생한다.

```ts
function hello(name: string | number) {
  if (typeof name === "string") {
    // string
    console.log(name);
  } else if (typeof name === "number") {
    // number
    console.log(name);
  } else {
    // never
    console.log(name);
  }
}
```

name의 타입은 `string` 아니면 `number`만 받을 수 있기에 올바르게 타입을 넘겼다면 else는 절대 실행되지 않는다.
![name-type](/assets/img/posts/2024-08-27/name-type.png)
name에 마우스를 각각 올려보면 다음과 같이 표시된다. else 에서의 name의 타입은 never인 것을 확인할 수 있다.

## Never vs Void

never도 리턴하지 않고, void도 리턴하지 않는데 두 개가 무슨 차이일까?

> - **void** : 반환할 **값**이 없다
> - **never** : 무언가 문제가 있어 **반환할 수 없다**. 즉, 함수가 결코 정상적으로 끝나지 않는다.
>   {: .prompt-info}

void는 함수는 정상적으로 종료가 되는데 반환할 값이 없을 뿐이다. 반면, never은 함수 자체가 정상적으로 끝나지 않기 때문에 값을 반환할 수가 없다.

또한, `never` 타입은 `any` 타입의 값을 포함해 어떤 값도 가질 수 없다.

```ts
let something: void = undefined;
let nothing: never = undefined; // Type 'undefined' is not assignable to type 'never'.
```

never에 `undefined`를 할당하려고 하면 타입 에러가 발생하는 점도 void 타입과의 차이점이다.

---

never 사용 예시에 대해서는 [타입스크립트의 Never 타입 완벽 가이드](https://ui.toast.com/weekly-pick/ko_20220323) 글에 잘 소개되어 있다. 하지만 나는 아직 예시를 봐도 이게 실무에서 어떻게 활용될지 감이 잘 안 잡힌다. never 타입을 좀 더 많이 사용해 본 후 다시 글을 읽어봐야겠다.

## 참고

- [never 타입 소개 강의](https://nomadcoders.co/typescript-for-beginners/lectures/3672)
- [void 와 never 타입 차이점 질문](https://nomadcoders.co/typescript-for-beginners/lectures/3672/comments/129208)
- [https://www.tutorialsteacher.com/typescript/typescript-never](https://www.tutorialsteacher.com/typescript/typescript-never)
- [타입스크립트의 Never 타입 완벽 가이드](https://ui.toast.com/weekly-pick/ko_20220323)
