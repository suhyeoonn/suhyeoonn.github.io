---
title: 스프링부트 REST API 만들기 - 1. 프로젝트 생성
date: 2024-08-09 11:00:00 +/-TTTT
categories: [스프링부트]
tags: [스프링부트, springboot, restful api] # TAG names should always be lowercase
description: 스프링부트 RESTful API 실습하기 - 프로젝트 생성편
---

[이전](https://suhyeoonn.github.io/posts/springboot-todo1/)에는 MVC 패턴을 사용하여 메모리 기반의 투두리스트를 만들었다. 이번에는 책 리뷰 공유 웹사이트를 만들어 RESTful API를 실습하고 데이터베이스와 연동해보자. 이번 프로젝트에서는 프론트엔드 작업 없이 이미 완성된 프론트 결과물을 연동하여 백엔드와의 통합을 진행할 예정이다.

## 📋 프로젝트 소개

### 프로젝트명

Book Rating - 책 리뷰 공유 웹사이트

![결과-리스트화면](/assets/img/posts/2024-08-09/결과-리스트화면.png)
![결과-리뷰화면](/assets/img/posts/2024-08-09/결과-리뷰화면.png)

### 상세 기능

**책**

- 리스트에서 책 정보와 평균 별점을 확인할 수 있다.
- 책을 등록할 수 있다.
  - ISBN, 책 이름, 태그를 입력해야 한다.
  - 동일한 ISBN을 가지는 책은 중복 등록할 수 없다.
  - 태그는 여러 개 선택 가능하다.
- 책 이름과 태그를 수정할 수 있다.
- 책을 삭제할 수 있다.

> `ISBN`(International Standard Book Number)은 주민등록번호처럼 책을 고유하게 식별하기 위한 국제 표준 번호이다. 10자리 또는 13자리 숫자로 구성되며, 책의 제목, 저자, 출판사 등과 연결되어 책을 정확하게 찾고 관리하는 데 도움을 준다.
>
> - **ISBN-10**: 10자리로, 주로 2007년 이전에 사용됨.
> - **ISBN-13**: 13자리로, 현재 국제적으로 사용되는 형식이며, EAN-13 바코드와 호환됨.
>
> `978-4-87311-336-4` 이런 형태이고 보통 책의 바코드 부분을 보면 있다.
{: .prompt-info }

**리뷰**

- 리스트에서 책을 선택하면 책의 리뷰를 확인할 수 있다.
- 리뷰를 등록할 수 있다.
  - 별점은 1 ~ 5 중 하나를 선택한다.
  - 리뷰는 선택 사항이며, 값을 입력하지 않을 경우 화면에는 ‘도움 돼요’라는 기본 메시지가 표시된다.
- 리뷰를 수정할 수 있다.
- 리뷰를 삭제할 수 있다.

## 🌱 스프링부트 프로젝트 생성

[start.spring.io](https://start.spring.io/) 에 접속하여 프로젝트를 설정한다.

![프로젝트세팅](/assets/img/posts/2024-08-09/프로젝트세팅.png)

Project, Language, Spring Boot는 모두 기본값 그대로 두었다.

- Project: Gradle - Groovy
- Language: Java
- Spring Boot: 3.3.2
- Packaging: Jar
- Java: 17

**Metadata**

- Artifact에 bookrating 입력

**Dependencies**

디펜던시는 다음 4가지를 등록한다.

- Spring Web: 웹 애플리케이션 개발을 위한 기능을 제공
- H2 Database: 인메모리 또는 파일 기반의 경량 데이터베이스로, 개발 및 테스트에 유용
- Spring Data JPA: JPA를 이용해 데이터베이스와의 상호작용을 간소화하고, 객체-관계 매핑을 지원
- Lombok: 자바 코드에서 반복적인 작업을 줄여주는 어노테이션 기반의 라이브러리

프로젝트 실행 후 `localhost:8080`에 접속되는지 확인한다.

![localhost:8080 접속](/assets/img/posts/2024-08-09/루트접속확인.png)
_localhost:8080 접속_

## ⚙️ 애플리케이션 환경 설정

src/main/resources/application.properties에 데이터베이스 관련 설정 추가

```
spring.application.name=bookrating
# 데이터베이스 관련 설정
spring.h2.console.enabled=true
spring.jpa.defer-datasource-initialization=true
logging.level.org.hibernate.SQL=DEBUG
spring.jpa.properties.hibernate.format_sql=true
spring.datasource.url=jdbc:h2:mem:testdb
```

- spring.h2.console.enabled=true
  - 웹 브라우저에서 H2 데이터베이스 콘솔에 접근할 수 있도록 설정
- spring.jpa.defer-datasource-initialization=true
  - 데이터 소스가 완전히 초기화된 후에 더미 데이터(data.sql)가 생성되도록 설정
- logging.level.org.hibernate.SQL=DEBUG
  - Hibernate의 SQL 쿼리 로깅 수준을 DEBUG로 설정하여 SQL 쿼리를 로그에 상세히 기록
- spring.jpa.properties.hibernate.format_sql=true
  - 가독성을 위해 쿼리 줄바꿈
- spring.datasource.url=jdbc:h2:mem:testdb
  - 데이터베이스 URL 지정. 지정하지 않을 시 매번 바뀌어서 웹 콘솔로 접근 시 번거로움.

[localhost:8080/h2-console](http://localhost:8080/h2-console)에 접속

![h2-console접속](/assets/img/posts/2024-08-09/h2-console접속.png)

**JDBC URL**을 `jdbc:h2:mem:testdb`로 입력 후 Connect를 클릭하여 DB에 접속되는지 확인

![db접속성공화면](/assets/img/posts/2024-08-09/db접속성공화면.png)

## ✅ 테이블 생성 확인

> JPA에서 엔티티는 데이터베이스 테이블과 매핑되는 클래스이다. 이를 기반으로 테이블이 만들어진다.
{: .prompt-tip }

`id`, `isbn`, `title` 를 속성으로 가지는 Book 엔티티를 만들자.

### Book 엔티티

```java
package com.example.bookrating.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Entity
@AllArgsConstructor
@NoArgsConstructor
@Getter
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String isbn;
    private String title;
}

```

- JPA Entity에는 `@Enitity` 어노테이션이 필요하다.
- 생성자는 2개가 필요하다.
  - 기본 생성자: 매개변수가 없는 생성자로, JPA가 엔티티를 생성할 때 사용된다.
  - 모든 속성을 받는 기본 생성자: 데이터베이스에 저장될 인스턴스를 만들 때 사용한다. 이 생성자는 JPA에 필수는 아니고 사용자의 편의를 위해 추가하는 생성자이다.
- id 속성에는 `@Id` 어노테이션을 추가하여 JPA가 엔티티를 식별하도록 한다. 그리고 자동 증가되도록 `@GeneratedValue(strategy = GenerationType.IDENTITY)` 도 추가한다.
  - `GenerationType.IDENTITY` 은 기본키 생성을 데이터베이스에 위임하는 속성이다. 즉, 데이터베이스에서 자동으로 기본 키 값을 증가시키는 역할을 한다.
- 그 외 속성은 어노테이션 없이 그대로 둔다. 별도로 `@Column` 어노테이션을 사용하지 않으면, 엔티티의 필드명은 자동으로 데이터베이스 테이블의 컬럼명과 매핑된다.

### BOOK 테이블 생성 확인

서버 재시작 후 h2-console에 접속하여 BOOK 테이블이 추가되었는지 확인한다.

![book테이블생성확인](/assets/img/posts/2024-08-09/book테이블생성확인.png)

이렇게 기본적인 세팅이 완료되었다. 다음에는 Book API를 만들어보자.
