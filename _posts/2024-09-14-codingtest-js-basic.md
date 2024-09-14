---
title: 코딩테스트 문제 풀이 시 유용한 자바스크립트 문법 정리
date: 2024-09-14 18:00:00 +/-TTTT
categories: [javascript]
tags: [TIL, 코딩테스트] # TAG names should always be lowercase
description: 프로그래머스 코딩테스트 기초 문제 풀이에 도움이 되는 자바스크립트 문법 정리
---

프로그래머스 코딩테스트 기초 문제를 풀며, 다른 사람의 풀이와 비교해 깨달은 내용을 정리

## 거듭 제곱 연산자

거듭 제곱 시 `Math.pow` 메서드를 사용해도 되지만 [거듭 제곱 연산자](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Exponentiation) `**` 도 있다.

```js
Math.pow(2, 2); // 4
2 ** 2; // 4
```

`Math.pow()`는 `Number` 타입의 인수만 받을 수 있다. `BigInt` 타입의 거듭제곱을 수행하려면 `**` 를 사용해야 한다.

**관련 문제**

- [원소들의 곱과 합](https://school.programmers.co.kr/learn/courses/30/lessons/181929)

##  input과 output이 명확히 매핑될 경우 객체를 활용

numLog 배열에서 i 요소의 값과 다음 요소의 값 차이에 따라 각기 다른 문자를 리턴하는 문제이다.

```js
function solution(numLog) {
  let answer = "";
  for (let i = 0; i < numLog.length - 1; i++) {
    switch (true) {
      case numLog[i] + 1 === numLog[i + 1]:
        answer += "w";
        break;
      case numLog[i] - 1 === numLog[i + 1]:
        answer += "s";
        break;
      case numLog[i] + 10 === numLog[i + 1]:
        answer += "d";
        break;
      case numLog[i] - 10 === numLog[i + 1]:
        answer += "a";
        break;
    }
  }
  return answer;
}
```

처음에는 switch 문을 사용해서 문제를 풀었는데, 아래처럼 객체를 활용하면 더욱 간결하게 작성할 수 있고 성능면에서도 좋다.
객체는 해시 맵 구조를 사용해 값에 바로 접근할 수 있으나, switch문은 조건을 순차적으로 검사한다.

```js
function solution(numLog) {
  const convert = {
    "1": "w",
    "-1": "s",
    "10": "d",
    "-10": "a",
  };
  return numLog
    .slice(1)
    .map((n, i) => convert[n - numLog[i]])
    .join("");
}
```

**관련 문제**

- [수 조작하기1](https://school.programmers.co.kr/learn/courses/30/lessons/181926)
- [수 조작하기2](https://school.programmers.co.kr/learn/courses/30/lessons/181925)

<!-- TODO: ## 구조 분해 활용
### 변수 값 서로 변경하기

쿼리 값 구조분해

sort 활용
 -->
