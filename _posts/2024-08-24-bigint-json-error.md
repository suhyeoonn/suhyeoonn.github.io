---
title: TypeError Do not know how to serialize a BigInt 에러 해결
date: 2024-08-24 23:00:00 +/-TTTT
categories: [JavaScript]
tags: [TIL] # TAG names should always be lowercase
description: BigInt 타입 데이터 직렬화 문제 해결 방법 알아보기
---

> 요약: JSON.stringify 사용 시 bigint는 string으로 변환하자
> {: .prompt-info}

## 오류 및 해결 방안

PK 타입을 bigint로 잡았는데, 프리즈마에서 가져온 데이터를 JSON 직렬화하는 과정에서 오류가 발생했다.

```ts
model Transaction {
  id BigInt @default(autoincrement()) @id
  type Int
  date DateTime
  categoryId Int
  content String?
  amount Int
}

const data = await prisma.transaction.findMany({
    where: {
      date: {
        gte: startDate,
        lte: dayjs(startDate).add(1, "M").subtract(1, "d").toDate(),
      },
    },
  });

return res.status(200).json(data); // Error!

```

에러 내용은 다음과 같다.

> Error: TypeError Do not know how to serialize a BigInt
> {: .prompt-danger}

[prisma docs](https://www.prisma.io/docs/orm/prisma-client/special-fields-and-types#serializing-bigint)에서는 다음과 같이 해결하라고 소개되어 있다.

> To work around this issue, use a customized implementation of `JSON.stringify`:
>
> ```js
> JSON.stringify(
>   this,
>   (key, value) => (typeof value === "bigint" ? value.toString() : value) // return everything else unchanged
> );
> ```

그래서 id를 string으로 변환하도록 코드를 추가하여 에러를 해결하였다.

```ts
function getTransaction(d: Transaction) {
  return { ...d, id: d.id.toString() };
}
```

클라이언트에서 Bigint로 다시 전환하려면 `Bigint(id)`와 같이 사용하면 된다.

[여기](https://github.com/GoogleChromeLabs/jsbi/issues/30#issuecomment-953187833)에는 관련 이슈 해결 방법에 대해 여러가지 의견이 나왔는데, 그 중 하나는 몽키패치를 사용하는 방법이다.

```js
BigInt.prototype.toJSON = function () {
  return this.toString();
};
```

한 번만 설정하면, 모든 BigInt를 변환하는데 적용할 수 있다.
타입스크립트에서는 이렇게 사용한다.

```ts
(BigInt.prototype as any).toJSON = function () {
  return this.toString();
};
```

> **몽키패치란** 원래 소스코드를 변경하지 않고 실행 시 코드 기본 동작을 추가, 변경 또는 억제하는 기술이다. 몽키패치는 다른 코드나 라이브러리와의 호환성 문제를 일으킬 수 있으므로, 사용 시 주의가 필요하다.
> {: .prompt-info}

참고로 [MDN 에도 Bigint 직렬화에 대한 가이드](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt#use_within_json)가 나와 있다.

## 두 가지 선택지

그런데 여기서 좀 고민되었던 부분이, id 타입인 bigint는 단순 PK 용도인데 프론트에서 다시 bigint로 변환할 필요가 있는가 하는 점이다.

### A. 프론트에서 BigInt로 변환하여 사용

백엔드에서 id가 bigint 타입이므로, 프론트에서도 데이터 타입의 일관성을 유지하기 위해 백엔드에서 받은 string 타입의 id를 BigInt로 변환하여 사용한다. 이렇게 하면 프론트와 백엔드 사이의 타입 불일치를 방지하고, 개발 시 혼란을 줄일 수 있다.

### B. 프론트에서는 string 타입을 그대로 사용하고, 변환은 백엔드에서 처리

프론트에서는 백엔드에서 받은 string 타입의 id를 그대로 사용하고, 데이터를 백엔드에 전달할 때도 string 타입 그대로 전달한다. 이 경우, id를 BigInt로 변환하는 작업은 백엔드에서만 수행한다. 이렇게 하면 프론트에서의 타입 변환을 최소화하고, 불필요한 연산을 줄일 수 있다.

식별자 용도로만 사용한다면 굳이 프론트에서 bigint로 변환할 필요는 없으므로, A 방식을 택했다.

---

이렇게 string으로 변환하는 과정을 미리 알았다면, bigint 대신 UUID를 사용하는 것이 직렬화 문제를 피하는 더 간단한 해결책이 될 수 있었을 것 같기도... 😅
그래도 덕분에 TIL 했다!
