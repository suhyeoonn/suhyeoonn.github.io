---
title: 초보자를 위한 스프링부트 투두리스트 - 1. 홈 화면 추가하기
categories: [JAVA, 스프링 부트]
tags: [스프링부트, 스프링부트 실습, springboot] # TAG names should always be lowercase
description: 할일 관리 홈 화면 추가하기
---

최근 자바 공부를 시작했는데 주변 추천을 받아 김영한님의 `스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술`을 완강했습니다. 생각보다 DI, 테스트 코드 작성 등 자세히 설명해 주셔서 왜 인기 많은지 알겠더군요.

배운 걸 토대로 간단하게 연습하면서 스프링 부트와 친해지고, 저처럼 자바 개발에 도전하는 사람들에게도 도움이 되기를 바라며 `초보자를 위한 스프링부트 실습` 프로젝트를 시작해보려고 합니다.

아직 [스프링 입문]({{page.inflearn_link}}) 강의를 다 못 보셨다면 먼저 시청 후 이 글을 봐주시고, 포스트마다 미션이 주어지니 스스로 해보신 후 글을 봐주시면 좋을 것 같아요.😊

글에서 잘못된 부분이 있다면 알려주시면 감사하겠습니다.

---

## ✅ 할일 관리 프로젝트

- 회원 관리처럼 할일을 등록하고 조회만 할 수 있는 매우 간단한 프로젝트입니다.
- 이번 프로젝트는 DI를 사용하지 않습니다.👀

> DI(Dependency Injection)는 스프링 부트뿐만 아니라 프로그래밍 전반에서 널리 사용하는 패턴으로, 의존성을 줄이고 유연성을 높여줍니다. 하지만 처음 접하거나 익숙하지 않은 경우, DI의 필요성을 충분히 이해하기 어려울 수 있습니다. 이번 프로젝트에서는 DI 없이 작업해보면서 DI가 왜 중요한지 직접 체감해보려 합니다. 이를 통해 DI의 장점을 더 명확하게 이해하고, 실무에서의 사용 이유를 깊이 있게 알 수 있을 것입니다.
{: .prompt-info }

## 🛠️ 사전 준비

스프링 부트 스타터로 프로젝트를 생성하고 환경 설정을 해주세요.
사용하는 라이브러리는 강의와 동일하게 `Spring Web`, `Thymeleaf`를 사용합니다.

> 기억이 잘 안 나시나요? `스프링 입문: 세션1 - 프로젝트 환경설정` 강의를 참고해주세요.
{: .prompt-tip }

저는 프로젝트 설정을 다음과 같이 했으니 참고해 주세요!

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

빌드 후 실행해서 `http://localhost:8080` 에 접속이 잘 되는지 확인해 주세요.

## 🎯 미션! 홈 화면 추가하기

`할일 관리` 라는 h1 태그와 `할일 등록`, `할일 관리`라는 a 태그로 구성된 홈 화면을 추가해 주세요.
![홈 화면 예시](/assets/img/home.png)
_홈 화면_

> 기억이 잘 안 나시나요? `스프링 입문: 회원 웹 기능 - 홈 화면 추가` 강의를 참고해 주세요.
{: .prompt-tip }

## 🚀 구현하기
모두 완료하셨나요? 이제 결과를 확인해 봅시다!

### 컨트롤러

```java
package com.todolist.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {
    @GetMapping("/")
    public String home() {
        return "home";
    }
}

```

### 템플릿 파일

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<h1>할일 관리</h1>
<div>
    <a href="/todos/new">할일 등록</a>
    <a href="/todos">할일 목록</a>
</div>
</body>
</html>
```

전체 코드는 [missions/1](https://github.com/suhyeoonn/springboot-todolist/blob/missions/1/src/main/java/com/todolist/controller/HomeController.java) 브랜치에서 확인 가능합니다.

---

🎉 고생하셨습니다! 이제 다음 미션을 도전해 보세요.
