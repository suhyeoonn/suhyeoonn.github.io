---
title: SSR에서 마주한 쿠키 기반 인증 문제
date: 2024-12-02 19:00:00 +/-TTTT
categories: [NextJS]
tags: [nextjs, nestjs, auth, jwt, book-rating] # TAG names should always be lowercase
description: CSR에서 SSR로 전환 시 마주한 쿠키 기반 인증 문제와 헤더 기반 인증 전환
---

## 문제 상황

토큰을 쿠키에 저장 방식으로 사용하다가 특정 페이지를 초기 렌더링이 빠르게 되게 하려고 CSR에서 SSR로 방식을 바꿨다.
데이터 조회 API를 클라이언트에서 호출하는게 아닌 서버에서 API를 호출하여 페이지를 미리 렌더링해서 내려주는 방식으로 변경한 것이다.

```tsx
interface Props {
  id: number;
}
export const MyBookDetailPage = async ({ id }: Props) => {
  const myBook = await getBook(id);
  const miniReview = await getMyReviewByBookId(myBook.book.id);

  return (
    <div className="container md:w-3/4 lg:w-1/2 mx-auto py-8 px-4 md:px-6 flex flex-col  gap-6">
      <Breadcrumb
        links={[{ href: "/my-books", label: "내 책장" }]}
        pathName={myBook.book.title}
      />
      <BookInfo book={myBook} />
      <hr />
      <ReviewForm myBook={myBook} />
      <hr />
      <MiniReviewForm miniReview={miniReview} />
    </div>
  );
};
```

그런데 CSR에서는 분명 API 호출 시 응답이 잘 오는걸 확인했는데 SSR로 변경하니 에러가 발생했다.

```
⨯ AxiosError: Request failed with status code 401
```

사용자 인증 오류 시 401이 전달되는데, 왜 갑자기 잘 되던게 401로 내려오는지 의아해서 계속 디버깅을 하였다.
그러다 문득, '잠깐... **SSR은 API를 브라우저에서 호출하는게 아닌데** 쿠키가 당연히 전달 안되는 거 아니야?🤔' 라는 생각이 들었다.

### SSR에서의 쿠키 문제

사용자가 화면에서 로그인 시 로그인 API는 브라우저에서 호출된다. 그리고 NestJS 서버는 토큰을 쿠키로 만들어 응답을 내려보낸다.

```ts
@Post('/login')
@HttpCode(200)
async login(@Body() userDto: UserDto, @Res() res: Response) {
	const result = await this.authService.validateUser(userDto);
	res.cookie('jwt', result.accessToken, {
	  httpOnly: true,
	  maxAge: 24 * 60 * 60 * 1000, // 1d
	});
	return res.send({ user: result.user });
}
```

이후 브라우저에 저장된 쿠키는 클라이언트가 다른 API를 호출할 때 자동으로 전달되어 토큰이 유효한지 확인한다.

```ts
async validate(payload: Payload, done: VerifiedCallback) {
    const user = await this.authService.tokenValidateUser(payload);
    if (!user) {
      return done(new UnauthorizedException());
    }
    return done(null, user);
}
```

하지만 SSR로 전환 시 API 호출은 브라우저가 아닌 서버, Node.js에서 호출된다. 이 과정에서 쿠키는 기본적으로 포함되지 않으므로, 백엔드 서버에서 인증 정보를 확인할 수 없어 401 에러를 계속 반환했던 것이다.

처음에 쿠키를 적용했던 이유는 구현 방식이 간단해서이다. 쿠키를 만들어주면 그 이후로는 프론트에서 크게 신경쓸 필요 없이 자동으로 전달되니깐 빨리 만들 수 있을거라 생각했다. SSR에서의 호출도 고려했어야 했는데 CSR에 아직 익숙해서 그런지 SSR에서의 쿠키 전달 문제를 미리 생각하지 못했다.

## 문제 해결 방안

### CSR 방식으로 API 요청 전환

기존처럼 브라우저에서 API를 호출하도록 전환하면 쿠키가 자동으로 전달되므로 문제가 해결된다.
제일 간단하지만 렌더링 속도를 개선하려는 본래 목적과 맞지 않아 패스

### 쿠키를 가져와 헤더에 포함하여 API 요청을 수행

getServerSideProps 로 쿠키를 직접 심는 방식이다. GPT가 알려준 방식대로 코드를 적용해봤다.

```tsx
import { getBook, getMyReviewByBookId } from "@/shared/api"; // API 호출 함수 import
import { Breadcrumb, BookInfo, ReviewForm, MiniReviewForm } from "@/components";

interface Props {
  id: number;
  myBook: any;
  miniReview: any;
}

export const MyBookDetailPage = ({ id, myBook, miniReview }: Props) => {
  return (
    <div className="container md:w-3/4 lg:w-1/2 mx-auto py-8 px-4 md:px-6 flex flex-col gap-6">
      <Breadcrumb
        links={[{ href: "/my-books", label: "내 책장" }]}
        pathName={myBook.book.title}
      />
      <BookInfo book={myBook} />
      <hr />
      <ReviewForm myBook={myBook} />
      <hr />
      <MiniReviewForm miniReview={miniReview} />
    </div>
  );
};

// getServerSideProps 함수 추가
export async function getServerSideProps(context: any) {
  const { id } = context.params; // URL 파라미터에서 id 가져오기
  const cookies = context.req.headers.cookie || ""; // 브라우저에서 전달된 쿠키 가져오기

  try {
    // API 호출 시 쿠키를 헤더에 포함
    const myBook = await getBook(id, cookies); // 쿠키를 전달
    const miniReview = await getMyReviewByBookId(myBook.book.id, cookies); // 쿠키를 전달

    return {
      props: {
        id,
        myBook,
        miniReview,
      },
    };
  } catch (error) {
    console.error("Error fetching data:", error);
    return {
      notFound: true, // 데이터 가져오기 실패 시 404 페이지로 이동
    };
  }
}
```

이렇게하면 API 함수에도 파라미터로 쿠키를 넘겨야해서 수정해야 할 곳이 많아지고 번거로운 것 같다.
그리도 또 다른 문제로, 아래 에러가 발생했다.

```
⨯ ReferenceError: Worker is not defined
```

[Worker is not defined, in NEXT JS 9+ #10899](https://github.com/vercel/next.js/issues/10899)
이 이슈와 연관있어 보이고, 해결하려면 추가 설정이 필요하다. 패스...

### 헤더 인증 방식을 사용

SSR의 경우 서버 간 인증 시 Authorization 헤더에 토큰을 전달하여 별도의 인증 방식을 사용할 수도 있다. 하지만 이렇게 되면 인증 방식이 두 가지가 되어 관리포인트이고, 무언가 문제가 되면 토큰을 헤더에 전달했는지 쿠키로 전달했는지부터 체크해야하니 유지보수도 어려울 것 같다.
기존 쿠키 인증을 헤더 인증으로 바꾸고 인증 방식을 헤더 인증으로만 유지한다면 렌더링 방식(CSR, SSR)에 상관없이 동일한 인증 로직을 사용할 수 있다. 이는 인증 처리를 일관성 있게 유지할 수 있어 유지보수가 용이하다.

## 결론

처음 인증 방식을 설계할 때 SSR 환경을 고려하지 않은 점은 아쉽지만, 이 과정을 통해 쿠키와 SSR의 동작 방식을 다시 상기할 수 있어서 좋은 공부가 된 것 같다. 쿠키 기반 인증을 헤더 기반으로 고쳐야겠다.
