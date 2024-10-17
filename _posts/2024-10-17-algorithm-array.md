---
title: 배열과 구간 합
date: 2024-10-17 21:00:00 +/-TTTT
categories: [algorithm]
tags: [TIL] # TAG names should always be lowercase
description: 배열과 구간 합 내용 정리
---

## 배열

배열은 메모리의 연속 공간에 값이 채워져 있는 형태의 자료구조이다. 인덱스를 사용하여 값에 바로 접근할 수 있고, 새로운 값을 삽입하거나 삭제하려면 해당 인덱스 주변에 있는 값을 이동시키는 과정이 필요하다.

## 구간 합

구간 합은 합 배열을 이용하여 **시간 복잡도를 더 줄이기 위해** 사용하는 목적의 알고리즘이다.

### 합 배열

구간 합 알고리즘을 활용하려면 먼저 합 배열을 구해야 한다. 리스트 A가 있을 때 합 배열 S는 다음과 같이 정의한다.

> **A[0]부터 A[i] 까지의 합**  <br>
> S[i] = A[0] + A[1] + ... + A[i-1] + A[i] 
{: .prompt-info }

합 배열을 미리 구해 놓으면 기존 리스트의 일정 범위의 합을 구하는 시간 복잡도가 O(N)에서 O(1)로 감소한다.

```
A = [1, 2, 3, 4, 5]
S = [1, 3, 6, 10, 15] # [1, (1 + 2), (1 + 2 + 3), ...]
```

A[i]부터 A[j]까지의 리스트 합을 합 배열 없이 구하는 경우, 최악의 경우는 i가 0이고 j가 N인 경우로 시간 복잡도는 O(N)이다. 이런 경우 앞에서 알아본 합 배열을 사용하면 O(1) 안에 답을 구할 수 있다.
예를 들어, A[3]까지의 합은 S[3]인 11로 바로 값을 알 수 있다.

> **합 배열 S를 만드는 공식** <br>
> S[i] = S[i-1] + A[i]
{: .prompt-info }

자바스크립트로 다음과 같이 구현할 수 있다.

```js
const A = [1, 2, 3, 4, 5];
const S = [A[0]];
for (let i = 1; i < A.length; i++) {
  S[i] = S[i - 1] + A[i];
}
console.log(S); // [1, 3, 6, 10, 15]
```

### 구간 합

이렇게 구현된 합 배열을 이용하여 구간 합 역시 쉽게 구할 수 있다.

> **i에서 j까지 구간 합 구하기** <br>
> S[j] - S[i-1]
{: .prompt-info }

예를 들어, A[2]부터 A[4]까지의 구간 합을 구해야 한다.

```
A = [1, 2, 3, 4, 5]
S = [1, 3, 6, 10, 15]
```

A[0] + ... + A[4] 에서 A[0] + A[1]을 빼면 구간 합 A[2] + ... A[4]가 나오므로, S[4]에서 S[1]을 빼면 구간 합을 쉽게 구할 수 있다.

```
S[4] = A[0] + A[1] + A[2] + A[3] + A[4] // 1 + 2 + 3 + 4 + 5
S[1] = A[0] + A[1] // 1 + 2
// j = 4, i = 2
S[4] - S[1] = A[2] + A[3] + A[4] // 3 + 4 + 5
```

A[2] 부터 A[4]까지니 j = 4, i = 2 이다. 위 공식을 적용했을 때 S[4] - S[2 - 1], 따라서 S[4] - S[1]가 된다.

### 문제

다음은 구간 합을 구하는 백준 문제이다. [구간 합 구하기 4](https://www.acmicpc.net/problem/11659)
![문제](/assets/img/posts/2024-10-17/Pasted image 20241017073308.png)

자바스크립트로 풀었는데, 합 배열을 사용하지 않을 경우 시간 초과가 발생했다.
![시간초과](/assets/img/posts/2024-10-17/Pasted image 20241017210211.png)
시간 초과가 발생했던 코드이다.

```js
/* input
5 3
5 4 3 2 1
1 3
2 4
5 5
*/
const [_, nums, ...input] = require("fs")
  .readFileSync(process.platform === "linux" ? "/dev/stdin" : "./input.txt")
  .toString()
  .trim()
  .split("\n");

const A = nums.split(" ").map(Number);
const result = [];

for (let l of input) {
  // 시작점과 끝점 s, e를 공백 기준으로 분리하고 숫자형으로 변환
  const [s, e] = l.split(" ").map(Number);
  // A 배열에서 s번째부터 e번째까지의 합을 구해서 result 배열에 추가
  let sum = 0;
  for (let i = s - 1; i < e; i++) {
    sum += A[i];
  }
  result.push(sum);
}
console.log(result.join("\n"));
```

합 배열을 이용하니 통과했다.
![통과](/assets/img/posts/2024-10-17/Pasted image 20241017211414.png)

```js
// A: 5 4 3 2 1
// S: [ 0, 5, 9, 12, 14, 15 ]
// 합을 구해야 하는 구간
// 1 3 -> 5 + 4 + 3 = 12, S[3] - S[0] = 12
// 2 4 -> 4 + 3 + 2 = 9, S[4] - S[1] = 9
// 5 5 -> 1, S[5] - S[4] = 1

const [_, nums, ...input] = require("fs")
  .readFileSync(process.platform === "linux" ? "/dev/stdin" : "./input.txt")
  .toString()
  .trim()
  .split("\n");

const A = nums.split(" ").map(Number);
const S = [0];
for (let i = 0; i < A.length; i++) {
  S[i + 1] = S[i] + A[i];
}

const result = [];

for (let l of input) {
  const [i, j] = l.split(" ");
  result.push(S[j] - S[i - 1]);
}
console.log(result.join("\n"));
```

## 출처

- 알고리즘 코딩테스트 책 (이지스퍼블리싱)
