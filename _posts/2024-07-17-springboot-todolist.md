---
title: 스프링부트 TODO LIST 만들기
categories: [스프링부트]
tags: [스프링부트, 투두리스트, springboot] # TAG names should always be lowercase
description: 스프링부트, 타임리프, 부트스트랩을 사용하여 투두리스트 만들기
---

최근 자바 공부를 시작했는데 주변 추천을 받아 김영한님의 `스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술`을 완강했다. 생각보다 DI, 테스트 코드 작성 등 자세히 설명해 주셔서 왜 인기 많은지 이해가 갔다.

배운 걸 토대로 간단하게 연습해 보고자 투두리스트를 만들어 보려고 한다.

> 전체 코드는 [여기](https://github.com/suhyeoonn/springboot-todolist)에서 확인 가능합니다.
{: .prompt-tip }

---

## ✅ 기능 정의

![결과물](/assets/img/posts/2024-07-31/result.png)

- 할 일을 추가하고 삭제할 수 있다. (수정은 불가능👀)
- 완료된 할 일은 취소선으로 표시된다.
- 할 일을 추가할 때 카테고리를 선택할 수 있으며 리스트에 카테고리가 같이 표시된다.

## 🛠️ 사전 준비

[스프링 부트 스타터](https://start.spring.io/)로 프로젝트를 생성하고 환경 설정. 사용하는 라이브러리는 강의와 동일하게 `Spring Web`, `Thymeleaf`를 사용한다.

![프로젝트 설정](/assets/img/posts/2024-07-17/2024-07-17-springboot-todo1-image1.png)
_프로젝트 설정_

- 프로젝트 선택
  - Project: Gradle - Groovy Project
  - Spring Boot: 3.x.x
  - Language: Java
  - Packaging: Jar
  - Java: 17
- Project Metadata
  - group: com.todolist
  - artifactId: todolist
- Dependencies: Spring Web, Thymeleaf

빌드 후 실행해서 `http://localhost:8080` 에 접속이 잘 되는지 확인. 아직 컨트롤러 작업을 하지 않아 접속 시 에러가 나타난다.

## 📋 할일 리스트

### 템플릿 파일 추가

`src/main/resources/templates` 경로에 `todo` 폴더를 추가하고 list.html 파일을 만든다.
`li` 태그로 할 일을 임시로 추가하고, 할 일을 등록할 `input`, `button`도 추가해준다.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
  <head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>
  <body>
    <div>
      <form>
        <label for="content">할일 입력</label>
        <input type="text" id="content" />
        <button>추가</button>
      </form>
    </div>
    <ul>
      <li>
        <div>
          <label><input type="checkbox" />자바 강의 보기</label
          ><button>x</button>
        </div>
        <div>카테고리: 공부</div>
      </li>
      <li>
        <div>
          <label><input type="checkbox" />빨래 하기</label><button>x</button>
        </div>
        <div>카테고리: 집안일</div>
      </li>
    </ul>
  </body>
</html>
```

### 컨트롤러 추가

`src/main/java/com/todolist/controller` 경로에 `TodoController.java` 파일을 만들어 컨트롤러를 추가하고 `/` 에 접속 시 리스트화면이 출력되는지 확인한다.

```java
@Controller
public class TodoController {
    @GetMapping("/")
    public String list() {
        return "todo/list";
    }
}
```

추가한 리스트화면에 Todo 목록을 가져오도록 작업한다.

### Todo, Category 도메인 추가

**Todo 도메인 추가**

```java
package com.todolist.domain;

public class Todo {
    private Long id;
    private String content;
    private Category category;

    public Todo(String content, Category category) {
        this.content = content;
        this.category = category;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getContent() {
        return content;
    }

    public Category getCategory() {
        return category;
    }
}

```

**카테고리 도메인 추가**

```java
package com.todolist.domain;

public enum Category {
    STUDY, HOUSEWORK
}

```

### Repository 및 테스트 코드 추가

`TodoRepository` 클래스를 만들고, 강의처럼 `Map`에 할 일을 저장하고 모두 가져오는 메서드를 추가한다. `findById` 메서드는 테스트 용도로 추가했다.

```java
package com.todolist.repository;

import com.todolist.domain.Todo;
import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Repository
public class TodoRepository {
    private static Map<Long, Todo> store = new HashMap<>();
    private static long sequence = 0L;

    public Todo save(Todo todo) {
        todo.setId(++sequence);
        store.put(todo.getId(), todo);
        return todo;
    }

    public Todo findById(Long id) {
        return store.get(id);
    }

    public List<Todo> findAll() {
        return new ArrayList<>(store.values());
    }
}

```

리포지토리 코드가 잘 동작하는지 확인하기 위해 테스트 코드를 추가한다.

> `cmd + N` / `alt + Insert` > `테스트...` 에서 간편하게 테스트 메서드를 추가 가능
{: .prompt-tip }

```java
class TodoRepositoryTest {
    private final TodoRepository repository = new TodoRepository();

    @Test
    void save() {
        Todo todo = new Todo("todo", Category.STUDY);

        repository.save(todo);

        Todo result = repository.findById(todo.getId());
        assertThat(todo).isEqualTo(result);
    }

    @Test
    void findAll() {
        Todo todo1 = new Todo("todo1", Category.STUDY);
        repository.save(todo1);
        Todo todo2 = new Todo("todo2", Category.HOUSEWORK);
        repository.save(todo2);

        List<Todo> result = repository.findAll();

        assertThat(result.size()).isEqualTo(2);
    }
}
```

### Service 및 테스트 코드 추가

`TodoService` 클래스를 만들고 추가, 조회하는 메서드를 추가한다.

```java
package com.todolist.service;

import com.todolist.domain.Todo;
import com.todolist.repository.TodoRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class TodoService {
    private final TodoRepository repository = new TodoRepository();

    public Long createTodo(Todo todo) {
        repository.save(todo);
        return todo.getId();
    }

    public List<Todo> findTodos() {
        return repository.findAll();
    }
}

```

마찬가지로 테스트 코드를 작성한다.

```java
package com.todolist.service;

import com.todolist.domain.Category;
import com.todolist.domain.Todo;
import com.todolist.repository.TodoRepository;
import org.junit.jupiter.api.Test;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.*;
class TodoServiceTest {

    private final TodoService service = new TodoService();
    private final TodoRepository repository = new TodoRepository();

    @Test
    void createTodo() {
        Todo todo = new Todo("todo", Category.STUDY);

        Long id = service.createTodo(todo);

        assertThat(todo.getId()).isEqualTo(repository.findById(id).getId());
    }

    @Test
    void findTodos() {
        Todo todo1 = new Todo("todo1", Category.STUDY);
        service.createTodo(todo1);
        Todo todo2 = new Todo("todo2", Category.HOUSEWORK);
        service.createTodo(todo2);

        List<Todo> result = service.findTodos();
        assertThat(result.size()).isEqualTo(2);
    }
}
```

![테스트 코드 실행 결과](/assets/img/posts/2024-07-17/테스트코드실행결과.png)
_잘 된다👍_

### 템플릿 파일에 Todo 배열 전달

컨트롤러를 수정하여 Todo 리스트를 전달한다. 아직 등록 기능이 구현되지 않아 직접 2개 정도 추가하였다.

```java
@GetMapping("/")
public String list(Model model) {
    // 등록 기능 구현 후 제거 예정
    service.createTodo(new Todo("todo1", Category.STUDY));
    service.createTodo(new Todo("todo2", Category.HOUSEWORK));

    model.addAttribute("todos", service.findTodos());

    return "todo/list";
}
```

html 파일도 수정한다.

```html
<ul>
  <li th:each="todo : ${todos}">
    <div>
      <label><input type="checkbox" th:text="${todo.content}" /></label>
      <button>x</button>
    </div>
    <div th:text="${'카테고리: '+todo.category}">카테고리: 공부</div>
  </li>
</ul>
```

![리스트화면실행결과](/assets/img/posts/2024-07-17/리스트화면실행결과.png)
_리스트 화면 실행 결과 🎉_

---

이어서 할 일 등록 및 삭제 기능을 구현해보자.

## ➕ 할일 추가

### list.html 수정

카테고리를 선택할 수 있는 `select` 를 추가하고, `form` 태그에 `action`과 `method` 속성을 적용한다.

```html
<form action="/todos/new" method="post">
  <label for="category">카테고리</label>
  <select name="category" id="category">
    <option value="STUDY">스터디</option>
    <option value="HOUSEWORK">집안일</option>
  </select>
  <label for="content">할일 입력</label>
  <input type="text" id="content" name="content" />
  <button>추가</button>
</form>
```

### TodoCreateForm 추가

`form` 데이터를 받을 클래스를 추가한다.

```java
package com.todolist.controller;

public class TodoCreateForm {
    private String content;
    private String category;

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public String getCategory() {
        return category;
    }

    public void setCategory(String category) {
        this.category = category;
    }
}

```

### 컨트롤러 추가

`post` 컨트롤러를 추가하고, 이전에 테스트를 위해 list 메서드에 추가했던 `createTodo` 코드는 제거한다.

```java
public class TodoController {
    @GetMapping("/")
    public String list(Model model) {
        model.addAttribute("todos", service.findTodos());

        return "todo/list";
    }

    @PostMapping("/todos/new")
    public String createTodo(TodoCreateForm form) {
        Todo todo = new Todo(form.getContent(), Category.valueOf(form.getCategory()));
        service.createTodo(todo);
        return "redirect:/";
    }
}
```

실행해보면 할 일이 잘 추가된다.

### 할 일 빈 값 체크

그런데 지금은 할 일이 빈 값이라도 추가가 된다🤔 입력된 값이 없으면 경고창을 출력하도록 `list.html`에 자바스크립트를 추가하자.

```html
<body>
  ...

  <script>
    const $form = document.querySelector("form");
    $form.addEventListener("submit", (e) => {
      if (!$form.content.value) {
        alert("할 일을 입력하세요");
        e.preventDefault();
      }
    });
  </script>
</body>
```

이제 할 일 추가 시 입력된 값이 없으면 경고창이 출력된다.

![경고창출력](/assets/img/posts/2024-07-18/경고창출력.png)

## 🗑️ 할 일 삭제

Todo 객체는 각각 고유 `id`를 가진다. 삭제 버튼에 `data-id` 속성을 추가하고, 버튼 클릭 시 서버로 `id`를 전달하여 해당 `id`를 가진 객체가 삭제되도록 구현해 보자.

### list.html 수정

`th:attr`을 사용하여 `todo.id`를 연결시킨다.

```html
<li th:each="todo : ${todos}">
  <div>
    <label><input type="checkbox" th:text="${todo.content}" /></label>
    <button class="delete-btn" th:attr="data-id=${todo.id}">x</button>
  </div>
  <div th:text="${'카테고리: '+todo.category}">카테고리: 공부</div>
</li>
```

그리고 버튼 클릭 시 DELETE API를 호출하도록 자바스크립트 코드를 추가한다.

```jsx
const $deleteButtons = document.querySelectorAll(".delete-btn");
$deleteButtons.forEach((btn) => {
  btn.addEventListener("click", async ({ target }) => {
    await deleteTodo(target.dataset.id);
  });
});

async function deleteTodo(id) {
  try {
    await fetch(`/todos/${id}`, {
      method: "DELETE",
    });

    location.href = "/";
  } catch (e) {
    console.error(e);
  }
}
```

삭제가 정상적으로 완료되면 `/` 경로로 이동하여 리스트 데이터를 다시 불러온다.

### 리포지토리에 삭제 메서드 추가 및 테스트

`TodoRepository` 클래스에 아래 메서드를 추가한다.

```java
public class TodoRepository {
    // ...
    public void deleteById(Long id) {
        store.remove(id);
    }
}
```

그리고 잘 동작하는지 확인하기 위해 테스트 코드도 추가한다.

```java
class TodoRepositoryTest {
    // ...
    @Test
    void deleteById() {
        Todo todo1 = new Todo("todo1", Category.STUDY);
        repository.save(todo1);

        repository.deleteById(todo1.getId());

        List<Todo> result = repository.findAll();
        assertThat(result.size()).isEqualTo(0);
    }
}
```

그런데 `deleteById` 테스트 하나만 실행했을 땐 성공하나 리포지토리에 있는 테스트 코드 전체를 실행하면 `result.siz`e가 0이 아닌 3이 되어 실패한다. ☹️

`@AfterEach` 를 사용하여 각 테스트가 종료될 때마다 메모리 DB에 저장된 데이터를 삭제하기 위해 리포지토리에 `clearStore` 메서드를 추가한다.

```java
public class TodoRepository {
    // ...
    public void clearStore() {
        store.clear();
    }
}
```

그리고 테스트 파일 상단에 `AfterEach`를 추가한다.

```java
class TodoRepositoryTest {
    // ...
    @AfterEach
    public void afterEach() {
        repository.clearStore();
    }
}
```

다시 전체 테스트를 실행하니 모두 성공한다.👍

### 서비스에 삭제 메서드 추가 및 테스트

이어서 서비스를 수정한다.

```java
public class TodoService {
    // ...
    public void deleteTodo(Long id) {
        repository.deleteById(id);
    }
}
```

테스트 코드를 추가한다. 리포지토리와 마찬가지로 각 테스트가 종료될 때 메모리 데이터가 삭제되도록 `AfterEach`도 추가한다.

```java
class TodoServiceTest {
    @AfterEach
    public void afterEach() {
        repository.clearStore();
    }

    // 생략
    @Test
    void deleteTodo() {
        Todo todo1 = new Todo("todo1", Category.STUDY);
        service.createTodo(todo1);

        service.deleteTodo(todo1.getId());

        List<Todo> result = service.findTodos();
        assertThat(result.size()).isEqualTo(0);
    }
}
```

### DELETE API 추가

DELETE API를 추가한다. Todo 삭제 후 따로 반환할 값은 없기에 `ResponseEntity.noContent().build();` 로 204 상태 코드가 내려가도록 한다.

```java
public class TodoController {
	  @DeleteMapping("/todos/{id}")
    public ResponseEntity<Void> deleteTodo(@PathVariable("id") Long id) {
        service.deleteTodo(id);
        return ResponseEntity.noContent().build();
    }
}
```

잘 동작한다.

![결과](/assets/img/posts/2024-07-18/삭제기능완성.gif)

---

할 일 목록에서 각 항목의 완료 상태를 변경할 수 있도록 작업하자.

## ✍️ 할일 수정

### isDone 속성 추가

Todo 도메인에 `isDone` 속성을 추가한다. 완료된 할 일은 `isDone이` `true`이다.

```java
public class Todo {
    private Long id;
    private String content;
    private Category category;
    private boolean isDone; // 추가

    public Todo(String content, Category category) {
        this.content = content;
        this.category = category;
        this.isDone = false; // 기본값 false
    }

    // ...

    public boolean getIsDone() {
        return isDone;
    }

    public void setDone(boolean status) {
        isDone = status;
    }
}
```

### 리포지토리에 상태 변경 메서드 추가

`id`와 상태를 받아 Todo 객체의 `isDone`을 변경한다.

```java
public class TodoRepository {
  // ...
	public void updateStatus(Long id, Boolean status) {
	    Todo todo = store.get(id);
	    todo.setDone(status);
	}
}
```

그리고 테스트 코드도 추가하여 잘 동작하는지 확인한다.

```java
class TodoRepositoryTest {
    // ...
    @Test
    void updateStatus() {
        Todo todo = new Todo("todo1", Category.STUDY);
        repository.save(todo);

        repository.updateStatus(todo.getId(), true);
        assertThat(todo.getIsDone()).isEqualTo(true);

        repository.updateStatus(todo.getId(), false);
        assertThat(todo.getIsDone()).isEqualTo(false);
    }
}
```

### 서비스에 상태 변경 메서드 추가

조금 전 리포지토리에 추가한 메서드를 호출한다.

```java
public class TodoService {
    // ...
    public void updateStatus(Long id, boolean status) {
        repository.updateStatus(id, status);
    }
}
```

마찬가지로 테스트 코드를 작성하여 테스트한다.

```java
class TodoServiceTest {
    // ...
    @Test
    void updateStatus() {
        Todo todo = new Todo("todo", Category.STUDY);
        service.createTodo(todo);

        service.updateStatus(todo.getId(), true);

        Todo result = repository.findById(todo.getId());
        assertThat(result.getIsDone()).isEqualTo(true);

        service.updateStatus(todo.getId(), false);
        assertThat(result.getIsDone()).isEqualTo(false);
    }
}
```

### 상태 변경 API 추가

수정 작업이므로 HTTP 메서드는 `PATCH`를 사용하며, `body`로 `isDone` 값을 받는다. 만약 `body`에 `isDone`이 없다면 400 상태코드를 반환한다.

```java
public class TodoController {
    // ...
    @PatchMapping("/todos/{id}")
    public ResponseEntity<Void> updateTodo(@PathVariable("id") Long id, @RequestBody Map<String, Object> body) {
        if (!body.containsKey("isDone")) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).build();
        }
        service.updateStatus(id, (Boolean) body.get("isDone"));
        System.out.println(service.findTodos().get(0).getIsDone()); // test
        return ResponseEntity.noContent().build();
    }
}
```

#### 체크박스 클릭 이벤트 추가

체크박스 클릭 시 방금 추가한 API가 호출되도록 `list.html`을 수정한다.

#### 체크박스에 value 속성 추가

수정할 할 일의 `id`를 받아올 수 있도록 체크박스 `value`에 `id`를 할당한다.

```html
// ...
<ul>
  <li th:each="todo : ${todos}">
    <div>
      <label>
        <input
          type="checkbox"
          th:text="${todo.content}"
          th:attr="value=${todo.id}"
        />
      </label>
      <button class="delete-btn" th:attr="data-id=${todo.id}">x</button>
    </div>
    <div th:text="${'카테고리: '+todo.category}">카테고리: 공부</div>
  </li>
</ul>
```

#### 클릭 이벤트 코드 추가

체크박스 클릭 이벤트 코드를 추가한다.

```jsx
// 체크박스 클릭 이벤트
document.querySelectorAll("input[type=checkbox]").forEach((c) => {
  c.addEventListener("click", ({ target }) => {
    updateTodo(target.value, target.checked);
  });
});

async function updateTodo(id, isDone) {
  try {
    await fetch(`/todos/${id}`, {
      method: "PATCH",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ isDone }),
    });
    location.href = "/";
  } catch (e) {
    console.error(e);
  }
}
```

이제 실행해서 체크박스를 클릭하면 콘솔에 변경된 상태가 잘 찍힌다.

```
2024-07-26T13:39:47.176+09:00  INFO 56030 --- [todolist] [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 2 ms
true <-- 찍힌 상태값
```

### 완료된 할 일에 체크와 취소선 적용

이제 완료된 할 일은 체크박스가 선택되고 취소선이 표시되도록 수정하자.

#### 체크 표시

완료된 할 일은 `checked` 속성이 `true`로 적용되도록 `th:attr`에 `checked=${todo.isDone}`를 추가한다.

```html
<label>
  <input
    type="checkbox"
    th:text="${todo.content}"
    th:attr="value=${todo.id},checked=${todo.isDone}"
  />
</label>
```

#### 취소선 표시

`html` 상단에 `style` 태그를 추가하고 체크된 `li`에 취소선을 적용한다.

```html
<head>
  // ...
  <style>
    /* 자식 요소 중에 :checked 상태인 요소를 가지는 li */
    li:has(:checked) {
      text-decoration: line-through;
    }
  </style>
</head>
```

브라우저에서 잘 되는지 테스트 해보자.

![완성](/assets/img/posts/2024-07-26/완성.gif)

---

## 💄 부트스트랩 적용

이제 처음에 계획한 기능은 모두 구현하였다. 마지막으로 부트스트랩을 적용하여 투두리스트의 디자인을 수정하자.

[부트스트랩 > Introduction](https://getbootstrap.com/docs/5.3/getting-started/introduction/#quick-start)에 접속하여 `Quick start - 2번` 코드의 link 태그 복사

![bootstrap-link](/assets/img/posts/2024-07-31/bootstrap-link.png)

`llist.html` 파일의 `head` 태그 안에 복사한 link 태그 붙여넣기

```html
<head>
  <title>Hello</title>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  <link
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
  />
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

### html 코드 수정

html 코드에 부트스트랩 클래스 적용

```html
<body class="container mt-5 w-50 bg-body-secondary">
  <div class="bg-white p-5 rounded shadow">
    <form
      action="/todos/new"
      method="post"
      class="d-flex justify-content-center align-items-end mb-4"
    >
      <div class="me-2 flex-fill">
        <label for="category" class="form-label">카테고리</label>
        <select class="form-select" name="category" id="category">
          <option value="STUDY">스터디</option>
          <option value="HOUSEWORK">집안일</option>
        </select>
      </div>
      <div class="me-2 flex-fill">
        <label for="content" class="form-label">할일 입력</label>
        <input type="text" id="content" name="content" class="form-control" />
      </div>
      <button class="btn btn-primary">추가</button>
    </form>

    <ul class="list-group">
      <li
        th:each="todo : ${todos}"
        class="list-group-item border-0 rounded d-flex flex-column mb-3 shadow-sm"
        style="background-color: #f4f6f7;"
      >
        <div class="d-flex align-items-center justify-content-between">
          <label
            ><input
              type="checkbox"
              th:text="${todo.content}"
              class="form-check-input me-2"
              th:attr="value=${todo.id},checked=${todo.isDone}"
          /></label>
          <button
            class="delete-btn btn btn-danger btn-sm"
            th:attr="data-id=${todo.id}"
          >
            x
          </button>
        </div>
        <div
          th:text="${'카테고리: '+todo.category}"
          class="fs-6a"
          style="font-size: 13px"
        >
          카테고리: 공부
        </div>
      </li>
    </ul>
  </div>
</body>
```

---

완성!

![result](/assets/img/posts/2024-07-31/result.png)

이제 투두리스트 프로젝트를 모두 완료하였다. 매우 간단한 프로젝트였지만, 스프링부트에 대해 조금은 더 알게 된 것 같다. 다음에는 DB를 연동하는 새로운 프로젝트를 만들어봐야겠다.

> 전체 코드는 [여기](https://github.com/suhyeoonn/springboot-todolist)에서 확인 가능합니다.
{: .prompt-tip }
