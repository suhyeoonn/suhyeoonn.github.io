---
title: 자바스크립트 정렬 sort 메서드 정리
date: 2024-10-01 21:00:00 +/-TTTT
categories: [javascript]
tags: [TIL] # TAG names should always be lowercase
description: JavaScript 정렬 sort 메소드 사용 시 헷갈렸던 부분 정리
---

**`sort()`** 메서드는 배열의 요소를 적절한 위치에 정렬한 후 그 배열을 반환한다. 기본 정렬 순서는 *문자열의 유니코드 코드 포인트*를 따른다.

> [!NOTE]
> 문자열의 유니코드 코드 포인트란 문자열을 구성하는 각 문자의 고유한 숫자 값을 의미한다.

```js
const fruits = ["banana🍌", "apple🍎", "grape🍇"];
fruits.sort();
console.log(fruits); // ['apple🍎', 'banana🍌', 'grape🍇']
```

과일이 사전 순으로 정렬되었다. 그런데 만약 숫자 배열이라면?

```js
const numbers = [1, 2, 10];
numbers.sort();
console.log(numbers); // 1, 10, 2
```

기본 정렬 순서가 유니코드 값 기준이라서 숫자가 문자열로 해석되어버렸다.
숫자를 제대로 정렬하고 싶다면 `sort` 메서드에 비교 함수를 제공해야 한다.

```js
numbers.sort(function (a, b) {
  return a - b;
});
console.log(numbers);
```

이 비교 함수에 대해 좀 더 알아보자.

## 비교 함수

비교 함수 `compareFunction(a, b)`가 제공되면 배열 요소는 함수의 반환 값에 따라 정렬된다. a와 b는 정렬될 배열의 두 요소를 의미하며, 다음 세 가지 결과 중 하나를 반환해야 한다.

- 음수: a가 먼저 온다.
- 양수: b가 먼저 온다.
- 0: 순서를 그대로 유지한다.

```js
const numbers = [1, 2, 3, 4, 5];

numbers.sort(function (a, b) {
  return 1;
});

console.log(numbers); // [1, 2, 3, 4, 5]

numbers.sort(function (a, b) {
  return -1;
});

console.log(numbers); // [5, 4, 3, 2, 1]
```

양수가 반환이 되면 `a, b`를 그 순서대로 유지하고, 음수일 경우 순서가 바뀐다.

**오름차순 정렬**

```js
let numbers = [4, 2, 5, 1, 3];

numbers.sort((a, b) => a - b);
```

sort 메서드는 배열을 정렬할 때 배열 요소들을 서로 비교하면서 그 순서를 결정한다. 비교는 두 요소를 한 쌍씩 뽑아 진행되며, 이를 비교 함수에 넘겨준다.
첫 번째 비교에서 a는 4, b는 2이다.  
4 - 2 = 2로 양수이다. 위에서 비교 함수가 양수를 반환할 시 b가 먼저 온다고 했다. 따라서 b인 2가 4보다 앞으로 가게 된다. 이어서 2(a)와 5(b)를 비교 시 2 - 5 = -3으로 음수이다. 음수일 경우 a가 더 앞으로 가게 되니 2가 앞으로 간다.

그런데 비교 순서는 정해져 있는것일까?
sort 메소드는 내부적으로 비교하는 순서가 고정되어 있지 않고 자바스크립트 엔진에 따라 다르게 동작할 수 있다. 즉, 정렬 과정에서 어떤 요소가 먼저 비교되는지, 어떤 순서로 비교가 진행되는지는 자바스크립트 엔진이 효율적으로 정렬하기 위한 방식에 따라 달라진다.
따라서, 요소들이 비교되는 순서는 딱 정해져있진 않지만 결과적으론 비교 함수의 리턴값에 따라 올바르게 정렬된다.

**내림차순 정렬**
만약 내림차순으로 정렬하고 싶다면 b - a를 사용하면 된다.

```js
numbers.sort((a, b) => b - a);
console.log(numbers); // [5, 4, 3, 2, 1]
```

b - a 결과가 양수일 때 더 큰 수인 b가 앞에 오도록하여 내림차순으로 정렬한다.

### 문자열 정렬

기본적으로 문자열을 비교할 때는 사전순으로 비교가 된다.

```js
const months = ["March", "Jan", "Feb", "Dec"];
months.sort();
console.log(months);
// 결과: ["Dec", "Feb", "Jan", "March"]
```

만약 비교함수를 사용해서 정렬할 경우에는 아래처럼 하면 사전순으로 정렬된다.

```js
const months = ["March", "Jan", "Feb", "Dec"];
months.sort((a, b) => {
  return a > b ? 1 : -1;
});
```

## 다중 조건

이번에는 다음과 같은 배열이 있을 때, 아래 우선 순위를 적용하려고 한다.

```js
const arr = ["apple", "seed", "sand", "append"];
```

1. 알파벳 사전 순으로 배치
2. 단어의 길이가 길수록 앞에 배치

사전 순으로 정렬은 위에서 해봤듯이 `a > b ? 1 : -1` 이다. 여기다 2번 길이 조건을 추가하려면 `||`를 사용하면 된다.

```js
const arr = ["apple", "seed", "sand", "append"];
arr.sort((a, b) => (a > b ? 1 : -1 || b.length - a.length));

console.log(arr); //  ['append', 'apple', 'sand', 'seed']
```

반대로, 아래 조건으로 정렬해보자.

1. 길이가 짧은 것부터 정렬
2. 길이가 같으면 사전 순으로

```js
const arr = ["apple", "seed", "sand", "append"];
arr.sort((a, b) => (a.length - b.length || a > b ? 1 : -1));

console.log(arr); // ['apple', 'sand', 'seed', 'append']
```

원하는 결과와 다르게 출력된다. 그 이유는 연산자 우선순위 때문이다.\
JavaScript에서 `||` 연산자가 `?:` 삼항 연산자보다 **우선순위가 낮기 때문에**, 괄호가 없으면 `a.length - b.length || a > b`가 먼저 평가되고, 이 결과에 따라 삼항 연산자가 적용된다.

```js
const arr = ["apple", "seed", "sand", "append"];
arr.sort((a, b) => a.length - b.length || (a > b ? 1 : -1));

console.log(arr); // ['sand', 'seed', 'apple', 'append']
```

이렇게 삼항연산자에 괄호를 포함하여 `a.length - b.length || (a > b ? 1 : -1)`로 작성해야 **의도한 대로 문자열의 길이 순으로 정렬**하고, **길이가 같을 때 사전순으로 정렬**된다.

## 참고

- [SORT 쉽게 사용하기! 자바스크립트 짤막 팁, 브라우저별 속도 차이까지!](https://www.youtube.com/watch?v=scjyeC74_4k)
