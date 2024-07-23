---
title: 초보자를 위한 스프링부트 투두리스트 - 2. 할일 등록 리포지토리 구현 및 테스트하기
date: 2024-07-19 11:00:00 +/-TTTT
categories: [JAVA, 스프링 부트]
tags: [스프링부트, 스프링부트 실습, springboot] # TAG names should always be lowercase
description: 할일 등록 리포지토리 구현 및 테스트하기
---

지난 포스트에서는 홈 화면을 추가하였습니다. 이번에는 할일 등록과 관련된 리포지토리와 테스트 코드를 작성해 봅시다.

## 🎯 미션! 1. 투두 도메인과 리포지토리 만들기
`Todo` 도메인을 생성하고 메모리에 저장되도록 리포지토리를 만들어 주세요.
- Todo는 `id`와 `content` 속성을 가집니다.
- Map을 사용해서 메모리에 저장합니다.
- 리포지토리에서 `sequence`를 관리하며, 객체가 추가될 때마다 1씩 증가합니다.
- `id` 값을 추가할 객체에 저장하고 `Map`에 객체를 저장하는 `save` 메서드를 구현합니다.
- 테스트코드에서 사용하기 위해 id를 전달하면 그에 해당하는 겍체를 반환하는 `findOne` 메서드도 추가합니다.

> 기억이 잘 안 나시나요? `스프링 입문: 세션3. 회원 관리 예제 - 백엔드 개발 > 회원 도메인과 리포지토리 만들기` 강의를 참고해주세요.
{: .prompt-tip }

### 도메인

```java
public class Todo {
    private Long id;
    private String content;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}

```
> 게터, 세터는 단축키 `cmd + N` > `getter 및 setter` 로 간편하게 추가 가능합니다. (윈도우: `alt + Insert`)
{: .prompt-tip }

> `id` 데이터 타입을 `Long` 대신 `Int`를 사용하면 안 될까요? 궁금하시다면 [여기](https://suhyeoonn.github.io/posts/long-vs-int/)를 참고해 주세요.
{: .prompt-info }

### 리포지토리
```java
public class TodoRepository {
    private static long sequence = 0L;
    private static Map<Long, Todo> store = new HashMap<>();

    public void save(Todo todo) {
        todo.setId(++sequence);
        store.put(todo.getId(), todo);
    }

    public Todo findOne(Long id) {
        return store.get(id);
    }
}
```
> `++sequence` 와 `sequence++`의 차이점을 아시나요? 모르신다면 [이 글](https://suhyeoonn.github.io/posts/%EC%A6%9D%EA%B0%80%EC%97%B0%EC%82%B0%EC%9E%90/)을 참고해 주세요.
{: .prompt-warning }

## 🎯 미션! 2. 리포지토리 테스트코드 작성하기
방금 추가한 메서드가 잘 동작하는지 확인하기 위해 테스트코드를 작성해 봅시다.
- `todo` 객체를 `save` 후 `findOne` 메소드로 추가한 객체를 가져와서 두 객체가 동일한지 확인하는 테스트 코드를 작성하세요.

> `cmd + N` / `alt + Insert` > `테스트...` 에서 간편하게 테스트 메서드를 추가할 수 있습니다.
{: .prompt-tip }

### 테스트
```java
package com.todolist.repository;

import com.todolist.domain.Todo;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.*;

class TodoRepositoryTest {
    private TodoRepository repository = new TodoRepository();

    @Test
    void save() {
        Todo todo = new Todo();
        todo.setContent("todo1");

        repository.save(todo);

        Todo result = repository.findOne(todo.getId());
        assertThat(result).isEqualTo(todo);
    }
}
```

---

🎉 고생하셨습니다! 이제 다음 미션을 도전해 보세요.

전체 코드는 [missions/2](https://github.com/suhyeoonn/springboot-todolist/blob/missions/2/src/main/java/com/todolist/domain/Todo.java) 브랜치에서 확인 가능합니다.
