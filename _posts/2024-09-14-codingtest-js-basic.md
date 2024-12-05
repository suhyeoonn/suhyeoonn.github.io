---
title: 코딩테스트 문제 풀이 시 유용한 자바스크립트 문법 정리
date: 2024-09-14 18:00:00 +/-TTTT
categories: [JavaScript]
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
    1: "w",
    "-1": "s",
    10: "d",
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

## 구조 분해 활용

변수 값을 서로 변경할 때 구조 분해를 활용하여 간단하게 해결할 수 있다.

```js
let a = 1;
let b = 2;

[a, b] = [b, a]; // b: 1, a: 2
```

a에 b 값을 넣고 b에 a 값을 넣었다.

`arr`는 정수 배열, `queries` 는 2차원 정수 배열이다. `queries`의 원소는 `[i, j]` 꼴인데 `arr[i]`의 값과 `arr[j]`의 값을 서로 바꿔야 해서 아래와 같이 작성했다.

```js
function solution(arr, queries) {
  queries.forEach((q) => {
    const [i, j] = q;
    const [a, b] = [arr[i], arr[j]];
    arr[j] = a;
    arr[i] = b;
  });
  return arr;
}
```

이때, `q`를 함수의 인수 부분에서도 바로 구조 분해가 가능하다.
위 코드를 좀 더 간결하게 정리했다.

```js
function solution(arr, queries) {
  queries.forEach(([i, j]) => {
    [arr[j], arr[i]] = [arr[i], arr[j]];
  });
  return arr;
}
```

**관련 문제**

- [수열과 구간 쿼리 3](https://school.programmers.co.kr/learn/courses/30/lessons/181924)

## || 연산자 활용

특정 수 보다 값이 큰 요소들이 있을 경우 그 중 가장 작은 값을 찾고 없으면 -1를 리턴한다.
처음에는 삼항 연산자를 사용하였다.

```js
function solution(arr, queries) {
  return queries.map(([s, e, k]) => {
    const filtered = arr.slice(s, e + 1).filter((n) => n > k);
    return filtered.length > 0 ? Math.min(...filtered) : -1;
  });
}
```

삼항연산자 대신 sort와 `||` 를 사용할 수 있다.

```js
function solution(arr, queries) {
  return queries.map(
    ([s, e, k]) =>
      arr
        .slice(s, e + 1)
        .filter((n) => n > k)
        .sort((a, b) => a - b)[0] || -1
  );
}
```

sort를 사용해서 요소들을 정렬한 후 만약 0번째 요소가 있으면 그 값을 반환하고 없을 경우 -1 을 반환한다.

**관련 문제**

- [수열과 구간 쿼리 2](https://school.programmers.co.kr/learn/courses/30/lessons/181923)

## 배열 만들기 With Array

start부터 end까지의 숫자를 리스트에 담아야 한다.

```js
function solution(start_num, end_num) {
  const answer = [];
  for (let i = start_num; i <= end_num; i++) {
    answer.push(i);
  }
  return answer;
}
```

for문 대신 아래처럼도 가능하다.

```js
function solution(start, end) {
  return Array(end - start + 1)
    .fill(start)
    .map((x, idx) => x + idx);
}
```

- Array(end-start+1): 길이가 end-start+1 인 빈 배열 생성
- fill(start): 배열을 'start' 값으로 채움
- map((x, idx) => x + idx): 각 요소에 인덱스를 더함. x는 fill로 채워진 start 값, idx는 배열 인덱스.

**관련 문제**

- [카운트 업](https://school.programmers.co.kr/learn/courses/30/lessons/181920)

## 재귀 연습

```js
function solution(x, answer = []) {
  answer.push(x);
  if (x === 1) {
    return answer;
  }

  return solution(x % 2 === 0 ? x / 2 : 3 * x + 1, answer);
}
```

**관련 문제**

- [콜라츠 수열 만들기](https://school.programmers.co.kr/learn/courses/30/lessons/181919)

## map vs reduce

배열을 돌면서 특정 값으로 바꿀 때는 map이 더 깔끔해보인다.

```js
function solution(my_string, index_list) {
  return index_list.reduce((str, v) => str + my_string[v], "");
}

function solution(my_string, index_list) {
  return index_list.map((i) => my_string[i]).join("");
}
```

위와 비슷한 문제의 풀이이다.

```js
function solution(my_strings, parts) {
  return parts.map(([s, e], i) => my_strings[i].slice(s, e + 1)).join("");
}
```

**관련 문제**

- [글자 이어 붙여 문자열 만들기](https://school.programmers.co.kr/learn/courses/30/lessons/181915)
- [부분 문자열 이어 붙여 문자열 만들기](https://school.programmers.co.kr/learn/courses/30/lessons/181911)

## 접두사, 접미사

`is_suffix`가 `my_string`의 접미사인지 확인하는 문제이다.
처음에 아래처럼 풀었다.

```js
function solution(my_string, is_suffix) {
  return +[...my_string].map((_, i) => my_string.slice(i)).includes(is_suffix);
}
```

그런데 `endsWith`를 사용하면 접미사인지 간단하게 확인할 수 있다.

```js
function solution(my_string, is_suffix) {
  return +my_string.endsWith(is_suffix);
}
```

**관련 문제**

- [접두사인지 확인하기](https://school.programmers.co.kr/learn/courses/30/lessons/181906)

## 글자 n개씩 자르기

```js
function solution(my_string, m, c) {
  return [...my_string].filter((_, i) => i % m === c - 1).join("");
}
```

조건문 i % m === c - 1은 인덱스 i가 m으로 나누었을 때 나머지가 c - 1인 요소만을 필터링한다.
예를 들어, m = 3이고 c = 2라면, i % 3 === 1인 요소들인 인덱스 1, 4, 7, …에 위치한 문자를 선택한다.

**관련 문제**

- [세로 읽기](https://school.programmers.co.kr/learn/courses/30/lessons/181904)
- [qr code](https://school.programmers.co.kr/learn/courses/30/lessons/181903)

## 특정 인덱스 이후의 값 찾기 - indexOf

배열에서 특정 값의 인덱스를 찾으려면 `indexOf`를 사용하면 된다.

```js
const array = [2, 9, 9];
array.indexOf(2); // 0
array.indexOf(7); // -1 (7은 배열에 없음)
```

이때, 검색은 0번 인덱스부터 시작된다. 만약 시작 인덱스를 변경하고 싶으면 두 번째 파라미터로 인덱스를 설정하면 된다.

```js
const array = [2, 9, 9, 1];
array.indexOf(9); // 1, 0번 인덱스부터 검색
array.indexOf(9, 2); // 2, 2번 인덱스부터 검색
array.indexOf(9, 3); // -1, 3번 인덱스 이후로는 9가 존재하지 않음
```

**관련 문제**

- [가까운 1 찾기](https://school.programmers.co.kr/learn/courses/30/lessons/181898)

## 전위/후위 연산자

```js
let a = 1;
console.log(++a); // 2
console.log(a++); // 2 (2로 출력된 후 3으로 증가)
console.log(a); // 3
```

위 코드를 실행하면 2 -> 2 -> 3 순으로 출력된다.

전위 연산자는 **값 1 증가 -> a에 대입 -> a 값 사용** 순이고,
후위 연산자는 **a 값 사용 -> 값 1 증가 -> a에 대입**이다.

```js
// ++a 전위 연산자 동작 방식
let a = 1;
let temp = a + 1;
a = temp;
console.log(a); // 2

// a++ 후위 연산자 동작 방식
let a = 1;
console.log(a); // 1 (현재 값 사용)
let temp = a + 1; // 그 후에 값 증가
a = temp;
```

while문을 사용해서 arr의 요소의 값을 1씩 증가시켜 보자

```js
const arr = [1, 2, 3];
let i = 0;
while (i < arr.length) {
  arr[i] += 1;
  i++;
}
console.log(arr); // 2,3,4
```

이를 다음과 같이 전위/후위 연산자를 사용할 수도 있다.

```js
const arr = [1, 2, 3];
let i = 0;
while (i < arr.length) {
  arr[i++]++;
}
console.log(arr); // 2,3,4
```

이때, arr 요소의 값에 1을 더할 때는 전위 연산자를 사용해도 동일하나 **i 값을 전위연산자로 사용해버리면** 0번 요소가 아닌 1번 요소부터 변경되고 **`Invalid array length` 에러**가 나니 주의하자.

```js
const arr = [1, 2, 3];
let i = 0;
while (i < arr.length) {
  arr[++i]++; // ❗️ i가 2일 때 arr[3]을 참조하려고 시도
}
```

배열 길이가 3이므로 범위를 벗어나 에러가 발생한다.

**관련 문제**

- [수열과 구간 쿼리 1](https://school.programmers.co.kr/learn/courses/30/lessons/181883)
