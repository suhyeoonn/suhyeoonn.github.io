---
title: 스프링부트 TODO LIST - 3. 할 일 상태 변경 기능 구현
date: 2024-07-26 11:00:00 +/-TTTT
categories: [스프링부트]
tags: [스프링부트, 투두리스트, springboot] # TAG names should always be lowercase
description: 스프링부트 투두리스트 할 일 상태 변경하기
---

할 일 목록에서 각 항목의 완료 상태를 변경할 수 있도록 작업하자.

## 🎯 상태 변경 API 추가

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

### 컨트롤러에 메서드 추가

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

## 🎯 체크박스 클릭 이벤트 추가

체크박스 클릭 시 방금 추가한 API가 호출되도록 `list.html`을 수정한다.

### 체크박스에 value 속성 추가

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

### 클릭 이벤트 코드 추가

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

## 🎯 완료된 할 일에 체크와 취소선 적용

이제 완료된 할 일은 체크박스가 선택되고 취소선이 표시되도록 수정하자.

### 체크 표시

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

### 취소선 표시

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

---

브라우저에서 잘 되는지 테스트 해보자.

![완성](/assets/img/posts/2024-07-26/완성.gif)

이제 처음에 계획한 기능은 모두 구현하였다. 다음에는 디자인을 수정해서 지금보다는 조금 더 이쁘게 바꿔보자.

>전체 코드는 [여기](https://github.com/suhyeoonn/springboot-todolist)에서 확인 가능합니다.
{: .prompt-info }