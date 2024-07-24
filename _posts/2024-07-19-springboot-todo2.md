---
title: 스프링부트 TODO LIST - 2. 할일 추가, 삭제 기능 구현
date: 2024-07-19 11:00:00 +/-TTTT
categories: [스프링부트]
tags: [스프링부트, 투두리스트, springboot] # TAG names should always be lowercase
description: 스프링부트 투두리스트 할 일 추가, 삭제 기능 구현하기
---

## 🎯 할일 추가

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
    <input type="text" id="content" name="content">
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
    const $form = document.querySelector('form')
    $form.addEventListener('submit', (e) => {
        if (!$form.content.value) {
            alert('할 일을 입력하세요');
            e.preventDefault();
        }
    })
</script>
</body>
```

이제 할 일 추가 시 입력된 값이 없으면 경고창이 출력된다.

![경고창출력](/assets/img/posts/2024-07-18/경고창출력.png)

## 🎯 할 일 삭제

Todo 객체는 각각 고유 `id`를 가진다. 삭제 버튼에 `data-id` 속성을 추가하고, 버튼 클릭 시 서버로 `id`를 전달하여 해당 `id`를 가진 객체가 삭제되도록 구현해 보자.

### list.html 수정

`th:attr`을 사용하여 `todo.id`를 연결시킨다.

```html
<li th:each="todo : ${todos}">
    <div>
        <label><input type="checkbox" th:text="${todo.content}"></label>
        <button class="delete-btn" th:attr="data-id=${todo.id}">x</button>
    </div>
    <div th:text="${'카테고리: '+todo.category}">카테고리: 공부</div>
</li>
```

그리고 버튼 클릭 시 DELETE API를 호출하도록 자바스크립트 코드를 추가한다.

```jsx
const $deleteButtons = document.querySelectorAll('.delete-btn')
$deleteButtons.forEach(btn => {
    btn.addEventListener('click', async ({target}) => {
        await deleteTodo(target.dataset.id)
    })
})

async function deleteTodo(id) {
    try {
        await fetch(`/todos/${id}`, {
            method: 'DELETE',
        })

        location.href = '/'
    } catch(e) {
        console.error(e)
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

### 컨트롤러 수정

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

---

## 🎉 완성

잘 동작한다.

![결과](/assets/img/posts/2024-07-18/삭제기능완성.gif)

다음에는 할 일의 상태를 변경해보자.

>전체 코드는 [여기](https://github.com/suhyeoonn/springboot-todolist)에서 확인 가능합니다.
{: .prompt-info }
