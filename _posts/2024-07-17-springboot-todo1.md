---
title: 스프링부트 TODO LIST - 1. 리스트화면 구현
categories: [스프링부트]
tags: [스프링부트, 투두리스트, springboot] # TAG names should always be lowercase
description: 스프링부트 투두리스트 할일 관리 리스트화면 구현하기
---

최근 자바 공부를 시작했는데 주변 추천을 받아 김영한님의 `스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술`을 완강했다. 생각보다 DI, 테스트 코드 작성 등 자세히 설명해 주셔서 왜 인기 많은지 이해가 갔다.

배운 걸 토대로 간단하게 연습해 보고자 투두리스트를 만들어 보려고 한다.

---

## ✅ 할일 관리 프로젝트

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

## 🎯 리스트 화면 추가

### list.html

`li` 태그로 할 일을 임시로 추가하고, 할 일을 등록할 `input`, `button`도 추가해준다.

경로: src/main/resources/templates/todo/list.html

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

### 컨트롤러

컨트롤러를 추가하여 `/` 에 접속 시 리스트화면이 출력되는지 확인한다.

경로: src/main/java/com/todolist/controller/TodoController.java

```java
@Controller
public class TodoController {
    @GetMapping("/")
    public String list() {
        return "todo/list";
    }
}
```

## 🎯 Todo 리스트 가져오기

추가한 리스트화면에 Todo 목록을 가져오도록 작업한다.

### Todo, Category 도메인 추가

Todo 도메인 추가

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

카테고리 도메인 추가

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

다음에는 할 일 등록 및 삭제 기능을 구현해보자.

> 전체 코드는 [여기](https://github.com/suhyeoonn/springboot-todolist)에서 확인 가능합니다.
{: .prompt-info }
