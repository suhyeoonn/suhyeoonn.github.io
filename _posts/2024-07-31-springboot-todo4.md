---
title: 스프링부트 TODO LIST - 4. 부트스트랩 적용
date: 2024-07-31 11:00:00 +/-TTTT
categories: [스프링부트]
tags: [스프링부트, 투두리스트, springboot, bootstrap] # TAG names should always be lowercase
description: 스프링부트 투두리스트를 Bootstrap 사용하여 디자인 변경
---

부트스트랩을 적용하여 투두리스트의 디자인을 수정하자

## 🎯 부트스트랩 CDN 추가

[부트스트랩 > Introduction](https://getbootstrap.com/docs/5.3/getting-started/introduction/#quick-start)에 접속하여 `Quick start - 2번` 코드의 link 태그 복사

![bootstrap-link](/assets/img/posts/2024-07-31/bootstrap-link.png)

`llist.html` 파일의 `head` 태그 안에 복사한 link 태그 붙여넣기

```html
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">
    // ...
</head>
```

다시 부트스트랩 코드에서 `script` 태그 복사 후 `list.html`의 `body` 닫는 태그 바로 위에 붙여넣기

![bootstrap-script](/assets/img/posts/2024-07-31/bootstrap-script.png)

```html
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

## 🎯 html 코드 수정

html 코드에 부트스트랩 클래스 적용

```html
<body class="container mt-5 w-50 bg-body-secondary">
<div class="bg-white p-5 rounded">
    <form action="/todos/new" method="post" class="d-flex justify-content-center align-items-end mb-4">
        <div class="me-2 flex-fill">
            <label for="category" class="form-label">카테고리</label>
            <select class="form-select" name="category" id="category">
                <option value="STUDY">스터디</option>
                <option value="HOUSEWORK">집안일</option>
            </select>
        </div>
        <div class="me-2 flex-fill">
            <label for="content" class="form-label">할일 입력</label>
            <input type="text" id="content" name="content" class="form-control">
        </div>
        <button class="btn btn-primary">추가</button>
    </form>

    <ul class="list-group">
        <li th:each="todo : ${todos}" class="list-group-item border-0 rounded d-flex flex-column mb-2"
            style="background-color: #f4f6f7;">
            <div class="d-flex align-items-center justify-content-between">
                <label><input type="checkbox" th:text="${todo.content}" class="form-check-input me-2"
                              th:attr="value=${todo.id},checked=${todo.isDone}"></label>
                <button class="delete-btn btn btn-danger btn-sm" th:attr="data-id=${todo.id}">x</button>
            </div>
            <div th:text="${'카테고리: '+todo.category}" class="fs-6a" style="font-size: 13px">카테고리: 공부</div>
        </li>
    </ul>
</div>
```

---

완성!

![result](/assets/img/posts/2024-07-31/result.png)

이제 투두리스트 프로젝트를 모두 완료하였다. 매우 간단한 프로젝트였지만, 스프링부트에 대해 조금은 더 알게 된 것 같다. 다음에는 DB를 연동하는 새로운 프로젝트를 만들어봐야겠다.

>전체 코드는 [여기](https://github.com/suhyeoonn/springboot-todolist)에서 확인 가능합니다.
{: .prompt-info }