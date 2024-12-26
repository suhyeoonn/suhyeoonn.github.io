---
title: Next.js Router Cache로 인한 캐시 문제와 해결 방법
categories: [NextJS]
tags: [nextjs, book-rating] # TAG names should always be lowercase
description: Next.js에서 Router Cache로 인해 발생한 별점 업데이트 반영 문제를 분석하고, router.refresh()를 활용해 해결한 과정을 공유
---

책 별점 업데이트 기능 구현 중 캐시 문제를 겪었다.

## 문제 상황

리스트에서 항목 클릭 시 `router.push()`를 호출하여 상세 페이지로 이동한다. 그런데 별점을 변경 후 리스트 페이지에 갔다가 동일한 상세 페이지로 가면 별점을 업데이트 하기 전 상태로 표시되고 있었다.

![error](/assets/img/posts/2024-12/cache-error.png)
_별점을 2점에서 5점으로 수정하였지만 상세 페이지 재진입 시 여전히 2점으로 표시된다._

app/(main)/my-books/[id]/page.tsx

```tsx
import { MyBookDetailPage } from "@/pages/my-book-detail/ui";

export default function Page({ params: { id } }: { params: { id: string } }) {
  console.log("id", id);

  return <MyBookDetailPage id={+id} />;
}

export const MyBookDetailPage = async ({ id }: Props) => {
  const myBook = await getBook(id);
  const { review } = myBook;
  return (
    <div className="container md:w-3/4 lg:w-1/2 mx-auto py-8 px-4 md:px-6 flex flex-col  gap-6">
      <Breadcrumb links={[menus[1]]} pathName={myBook.book.title} />
      <BookInfo book={myBook} />
      <hr />
      <TiptapEditor id={id} review={review} />
    </div>
  );
};

//console
// id 14
```

동적 라우트 경로에 콘솔을 찍어보았다. 페이지에 진입할 때마다 id가 찍힐거라 예상했으나 실제로는 처음 한 번만 아이디가 찍히고 이후로는 찍히지 않았다.
구글링한 결과 라우트 캐시가 원인인 걸 알게되었다.

## Route cache

NextJS는 Router Cache를 통해 이전에 방문했던 경로의 데이터를 클라이언트 메모리에 저장하고, 동일 경로 재방문 시 캐시된 데이터를 반환한다.

> The React Server Component Payload is stored in the client-side [Router Cache](https://nextjs.org/docs/app/building-your-application/caching#client-side-router-cache) - a separate in-memory cache, split by individual route segment. This Router Cache is used to improve the navigation experience by storing previously visited routes and prefetching future routes.
>
> - [클라이언트에서의 Next.js 캐싱(라우터 캐시)](https://nextjs.org/docs/app/building-your-application/caching#4-nextjs-caching-on-the-client-router-cache)

### RSC Payload와 리액트 라우터

**서버 컴포넌트 렌더링 방식**

1. 리액트는 서버 컴포넌트를 React Server Component Payload(RSC Payload) 라는 특수한 데이터 형식으로 렌더링한다.
2. Next.js는 RSC payload와 클라이언트 컴포넌트를 서버에서 HTML을 렌더링한다.
   그런 다음 클라이언트에서
3. HTML로 화면을 미리 보여준다. 이는 최초 페이지 로드 시에만 사용된다.
4. RSC payload는 클라이언트와 서버 컴포넌트 트리를 조정하고 DOM을 업데이트하는데 사용된다.
5. 하이드레이션으로 애플리케이션을 상호작용되게 만든다.

> RSC Payload는 렌더링된 React Server Components 트리의 컴팩트한 바이너리 표현입니다. 클라이언트에서 React가 브라우저의 DOM을 업데이트하는 데 사용됩니다. RSC Payload에는 다음이 포함됩니다.
>
> - 서버 구성 요소의 렌더링된 결과

- 클라이언트 구성 요소를 렌더링해야 하는 위치와 해당 JavaScript 파일에 대한 참조
  > - 서버 구성 요소에서 클라이언트 구성 요소로 전달된 모든 props
  > - [React Server Component Payload(RSC)란 무엇인가요?](https://nextjs.org/docs/app/building-your-application/rendering/server-components#what-is-the-react-server-component-payload-rsc)

클라이언트 라우터는 RSC payload를 캐싱한다. 별점을 업데이트한 후 리스트로 돌아갔다가 동일한 상세 페이지를 방문하면, 캐시된 데이터를 반환해 변경된 별점이 반영되지 않는 것이다.

캐시를 무효화 하는 방법은 서버단에서 `revalidatePath`를 사용하거나 `router.refresh`를 호출하는 방법이 있다. router.refresh()는 현재 활성화된 경로에서 사용 중인 라우터 캐시를 무효화하고, 해당 페이지의 RSC Payload를 새로 요청한다.

별점이 변경되면 `router.refresh()`를 호출하도록 수정했다.

```ts
export const useUpdateRating = () => {
  const queryClient = useQueryClient();
  const router = useRouter();

  return useMutation({
    mutationFn: updateRating,

    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: bookQueries.list().queryKey });
      router.refresh();
    },
  });
};
```

이제 별점 변경 시 RSC Payload를 새로 가져오며, 상세 페이지 재진입시 마지막에 업데이트한 별점이 정상적으로 표시된다. 또한, 네트워크 탭에서 rating 업데이트 API 호출 후 rsc를 새로 가져오는 것을 확인할 수 있었다.
![rsc 재요청 network tab](/assets/img/posts/2024-12/rsc_refetching.png)
