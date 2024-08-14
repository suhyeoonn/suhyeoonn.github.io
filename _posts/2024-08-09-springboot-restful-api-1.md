---
title: 스프링부트 REST API 만들기 - 1. Book API CRUD
date: 2024-08-09 11:00:00 +/-TTTT
categories: [스프링부트]
tags: [스프링부트, springboot, restful api] # TAG names should always be lowercase
description: 스프링부트 RESTful API 실습하기 - 프로젝트 생성 및 Book API CRUD 구현
---

[이전](https://suhyeoonn.github.io/posts/springboot-todo1/)에는 MVC 패턴을 사용하여 메모리 기반의 투두리스트를 만들었다. 이번에는 책 리뷰 공유 웹사이트를 만들어 RESTful API를 실습하고 데이터베이스와 연동해보자. 이번 프로젝트에서는 프론트엔드 작업 없이 이미 완성된 프론트 결과물을 연동하여 백엔드와의 통합을 진행할 예정이다.

## 📋 프로젝트 개요

책 리뷰 공유 웹사이트 `Book Rating`은 사용자가 책을 등록하고 관리하며, 책에 대한 리뷰를 작성할 수 있다. 리뷰를 등록하면 별점이 책의 평균 점수에 반영되어 책의 평점을 업데이트한다.

![결과-리스트화면](/assets/img/posts/2024-08-09/결과-리스트화면.png)
![결과-리뷰화면](/assets/img/posts/2024-08-09/결과-리뷰화면.png)

## 🌱 스프링부트 프로젝트 생성

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

### 애플리케이션 환경 설정

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

## 요구사항 정리

책에 대한 요구사항은 다음과 같다.

- 리스트에서 책 정보와 평균 별점을 확인할 수 있다.
- 책을 등록할 수 있다.
    - ISBN, 책 이름, 태그를 입력해야 한다.
    - 동일한 ISBN을 가지는 책은 중복 등록할 수 없다.
    - 태그는 여러 개 선택 가능하다.
- 책 이름과 태그를 수정할 수 있다.
- 책을 삭제할 수 있다.
- `ISBN`(International Standard Book Number)은 주민등록번호처럼 책을 고유하게 식별하기 위한 국제 표준 번호이다. 10자리 또는 13자리 숫자로 구성되며, 책의 제목, 저자, 출판사 등과 연결되어 책을 정확하게 찾고 관리하는 데 도움을 준다.
    - **ISBN-10**: 10자리로, 주로 2007년 이전에 사용됨.
    - **ISBN-13**: 13자리로, 현재 국제적으로 사용되는 형식이며, EAN-13 바코드와 호환됨.
    
    `978-4-87311-336-4` 이런 형태이고 보통 책의 바코드 부분을 보면 있다.
    
    {:prompt_info}
    

이번 편에서는 태그와 평균 별점을 제외한 기본적인 CRUD 기능을 구현하자. 나머지는 다음 편에 이어서 작업할 예정이다.

## Book 엔티티 추가

책의 기본 정보를 저장하는 테이블은 다음과 같다.

| 컬럼 이름 | 데이터 타입 | 설명 |
| --- | --- | --- |
| id | INT | 고유 식별자 (자동 증가) |
| isbn | VARCHAR(13) | 국제 표준 도서 번호 (ISBN) |
| title | VARCHAR(255) | 책 제목 |

 Book 엔티티를 추가하고 테이블이 생성되는지 확인하자.

> JPA에서 엔티티는 데이터베이스 테이블과 매핑되는 클래스이다. 이를 기반으로 테이블이 만들어진다.
{: .prompt-info }
    

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
- `@Getter`는 모든 속성들의 게터를 등록해주는 어노테이션이다.
- id 속성에는 `@Id` 어노테이션을 추가하여 JPA가 엔티티를 식별하도록 한다. 그리고 자동 증가되도록 `@GeneratedValue(strategy = GenerationType.IDENTITY)` 도 추가한다.
    - `GenerationType.IDENTITY` 은 기본키 생성을 데이터베이스에 위임하는 속성이다. 즉, 데이터베이스에서 자동으로 기본 키 값을 증가시키는 역할을 한다.
- 그 외 속성은 어노테이션 없이 그대로 둔다. 별도로 `@Column` 어노테이션을 사용하지 않으면, 엔티티의 필드명은 자동으로 데이터베이스 테이블의 컬럼명과 매핑된다.

### BOOK 테이블 생성 확인

서버 재시작 후 h2-console에 접속하여 BOOK 테이블이 추가되었는지 확인한다.

![book테이블생성확인](/assets/img/posts/2024-08-09/book테이블생성확인.png)

## URL 설계

RESTful API는 url에는 명사로 자원을 표시하고, HTTP Method로 행동을 정의한다. CRUD API를 다음과 같이 정의하였다.

### 책 API

- 조회: `GET` /api/books
- 등록: `POST` /api/books
- 수정: `PATCH` /api/books/{id}
- 삭제: `DELETE` /api/books/{id}

> CRUD란?
> - C - create
> - R - read
> - U - update
> - D - delete
{: .prompt-tip }

## API 개발

### 조회 API

**repository**

```java
package com.example.bookrating.repository;

import com.example.bookrating.entity.Book;
import org.springframework.data.jpa.repository.JpaRepository;

public interface BookRepository extends JpaRepository<Book, Integer> {}

```

**service**

```java
package com.example.bookrating.service;

import com.example.bookrating.entity.Book;
import com.example.bookrating.repository.BookRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class BookService {
    @Autowired
    private BookRepository bookRepository;

    public List<Book> getBooks() {
        return bookRepository.findAll();
    }
}

```

- @Service : 해당 클래스를 서비스로 인식해 스프링 부트에 서비스 객체를 생성
- @Autowired: bookRepository 의존성 주입

> 원래라면 객체를 만들어야 한다.
>    
> ```java
> private BookRepository bookRepository = new BookRepository();
> ```
>    
> 그런데 `@Autowired` 어노테이션을 붙이면 객체를 만들 필요 없이 스프링 부트가 미리 생성해 놓은 객체를 가져와 주입해 준다. 이를 의존성 주입이라고 하고 영어로는 DI이다. (Dependency Injection)
{: .prompt-info }

**controller**

```java
package com.example.bookrating.controller;

import com.example.bookrating.entity.Book;
import com.example.bookrating.service.BookService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class BookController {
    @Autowired
    private BookService bookService;

    @GetMapping("/books")
    public List<Book> getBooks() {
        return bookService.getBooks();
    }
}

```

- `@RestController`: JSON이나 텍스트 같은 데이터를 반환하는 컨트롤러이다.
    - 반면, 투두리스트에서 사용했던 `@Controller` 어노테이션은 뷰 페이지를 반환한다.

[http://localhost:8080/books](http://localhost:8080/books) 에 접속하여 확인

![get_books_빈배열](/assets/img/posts/2024-08-09/get_books_빈배열.png)

아직 데이터가 없어서 빈 배열`[]`로 표시된다. 직접 데이터를 등록하기에는 번거로우니 data.sql에 INSERT문을 넣어서 더미 데이터를 추가해보자. src/main/resources 에 data.sql 파일을 추가 후 아래 내용을 넣는다. 데이터는 GPT한테 만들어달라고 요청했다.

```sql
INSERT INTO book (isbn, title) VALUES
('9780060935467', 'To Kill a Mockingbird'),
('9780451524935', '1984'),
('9780141439518', 'Pride and Prejudice'),
('9781594631931', 'The Kite Runner'),
('9780385490818', 'The Handmaid''s Tale'),
('9780345339683', 'The Hobbit');
```

추가한 데이터가 잘 나오는지 확인한다. 

![get_books_리스트](/assets/img/posts/2024-08-09/get_books_리스트.png)

> 크롬에 [JSON Formatter](https://chromewebstore.google.com/detail/json-formatter/bcjindcccaagfpapjjmafapmmgkkhgoa?hl=ko) 확장 프로그램을 설치하면 JSON 데이터를 보기 쉽게 바꿔준다.
{: .prompt-tip }
    

### 등록 API

ISBN이 동일한 책은 중복 등록이 불가능하다. 책을 저장하기 전 ISBN이 존재하는지 먼저 확인 후 없을때만 저장하려고 한다. `JpaRepository` 에서는 id에 대해서만 기본적으로 메서드를 지원하기에 ISBN으로 찾을 수 있도록 인터페이스에 `findByIsbn`을 추가한다.

**repository**

```java
package com.example.bookrating.repository;

import com.example.bookrating.entity.Book;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface BookRepository extends JpaRepository<Book, Integer> {
    Optional<Book> findByIsbn(String isbn);
}

```

**dto**

클라이언트의 요청 데이터를 객체로 받을 dto를 작성한다.

```java
package com.example.bookrating.dto;

import com.example.bookrating.entity.Book;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;

@AllArgsConstructor
@NoArgsConstructor
@Getter
public class BookDto {
    private Integer id;
    private String isbn;
    private String title;

    public Book toEntity() {
        return new Book(id, isbn, title);
    }
}

```

게터와 기본 생성자, 전체 생성자를 추가한다. `@NoArgsConstructor` 는 cannot deserialize from Object value (no delegate- or property-based Creator) 에러 발생해서 추가하였다.

**service**

만약 ISBN이 중복될 경우 에러를 반환하는 코드를 작성한다.

```java
@Service
public class BookService {
    @Autowired
    private BookRepository bookRepository;

    public List<Book> getBooks() {
        return bookRepository.findAll();
    }

    public Book create(BookDto book) {
        // isbn이 동일한 책은 중복 등록할 수 없음
        bookRepository.findByIsbn(book.getIsbn()).ifPresent(e -> {
            throw new IllegalStateException("이미 존재하는 책입니다");
        });;

        return bookRepository.save(book.toEntity());
    }
}

```

- `IllegalStateException`: Java의 표준 예외로, 프로그램의 상태가 메소드 호출에 적합하지 않을 때 발생시킬 수 있다.

### 테스트 코드 작성

`create`가 잘 동작하는지 확인하는 테스트 코드를 작성하자. 

BookService에서 커맨드 + N 을 눌러 테스트 메뉴를 클릭한다. create 메서드를 체크하고 확인을 누른다.

이어서 책이 등록되는지 확인하는 테스트 코드와 isbn이 중복될 경우 에러가 반환되는지 확인하는 테스트 코드를 작성한다.

![create_테스트코드](/assets/img/posts/2024-08-09/create_테스트코드.png)

```java
package com.example.bookrating.service;

import com.example.bookrating.entity.Book;
import com.example.bookrating.repository.BookRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
@Transactional
class BookServiceTest {
    @Autowired
    BookService bookService;
    @Autowired
    BookRepository bookRepository;

    @Test
    void 책_등록() {
        BookDto book = new BookDto(null, "1234", "book");

        bookService.create(book);

        Book result = bookRepository.findByIsbn(book.getIsbn()).get();

        assertEquals(result.getIsbn(), book.getIsbn());
    }

    @Test
    void isbn_중복_확인() {
        BookDto book1 = new BookDto(null, "1234", "book");
        bookService.create(book1);

        BookDto book2 = new BookDto(null, "1234", "book");
        IllegalStateException e = assertThrows(IllegalStateException.class, () -> bookService.create(book2));

        assertEquals(e.getMessage(), "이미 존재하는 책입니다");
    }
}
```

테스트가 모두 통과 되었다면 이어서 컨트롤러를 만든다. 

**controller**

```sql
@RestController
public class BookController {
    // ...

    @PostMapping("/books")
    public ResponseEntity<?> createBook(@RequestBody BookDto book) {
        try {
            Book savedBook = bookService.create(book);
            return ResponseEntity.status(HttpStatus.CREATED).body(savedBook);
        } catch (IllegalStateException e) {
            // 중복된 책일 경우 409 Conflict 상태 코드와 함께 오류 메시지 반환
            return ResponseEntity.status(HttpStatus.CONFLICT).body(Map.of("error", e.getMessage()));
        } catch (Exception e) {
            // 기타 예외 처리
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(Map.of("error", "An unexpected error occurred"));
        }
    }
}

```

### API 테스트

추가한 API는 POST 메서드이기 때문에 GET API처럼 브라우저에 접속하는 방식으로는 확인할 수 없다. API를 테스트하는 방식으로는 curl, HTTP Client, Postman 등 여러 방법이 있고 나는 HTTP client를 선호하지만 젯브레인 커뮤니티에서는 지원하지 않기에 [Talend API Tester](https://chromewebstore.google.com/detail/talend-api-tester-free-ed/aejoelaoggembcahagimdiliamlcdmfm) 크롬 확장 프로그램으로 테스트를 하려고 한다.

Talend 를 열고 `Requests` 탭에서 METHOD를 POST 선택하고 옆에 http://localhost:8080/books를 입력한다. Body에는 다음과 아래와 같이 입력 후 Send를 눌러 201과 책 정보가 반환되는지 확인한다.

```json
{
  "isbn":"1111",
  "title":"book1"
}
```

![post_api_test](/assets/img/posts/2024-08-09/post_api_test.png)

한번 더 Send를 누르면 409 상태코드와 함께 에러 메시지가 출력된다.

![post_api_test_409](/assets/img/posts/2024-08-09/post_api_test_409.png)

### 수정 API

book 엔티티에 patch 메서드를 추가하자. 이 메서드는 클라이언트로 전달 받은 타이틀이 있으면 해당 값을 저장한다.

```java
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String isbn;
    private String title;

    public void patch(Book book){
        if (book.title != null) {
            this.title = book.title;
        }
    }
}
```

이어서 서비스에 `update` 메서드를 추가한다. update를 하기 전에 전달받은 id가 존재하는지 확인한다. 존재하지 않을 경우 에러를 반환한다. 

```java
@Service
public class BookService {
	// ...
	public Book update(Integer id, BookDto dto) {
        Book target = bookRepository.findById(id).orElse(null);
        if (target == null) {
            throw new IllegalStateException("존재하지 않는 책입니다");
        }

        Book book = dto.toEntity();
        
        target.patch(book);

        return bookRepository.save(target);
    }
}
```

그리고 테스트 코드를 추가한다.

```java
@SpringBootTest
@Transactional
class BookServiceTest {    
		// ...
    @Test
    void 책_제목_수정() {
        BookDto book1 = new BookDto(null, "1234", "book");
        Book saveBook = bookService.create(book1);

        BookDto dto = new BookDto(null, null, "update");
        bookService.update(saveBook.getId(), dto);

        Book result = bookRepository.findById(saveBook.getId()).get();
        assertEquals(result.getTitle(), dto.getTitle());
    }
}
```

테스트에 통과하면 마지막으로 컨트롤러에도 코드를 추가하자.

```java
@RestController
public class BookController {
    // ...

    @PatchMapping("/books/{id}")
    public ResponseEntity<?> updateBook(@PathVariable Integer id,  @RequestBody BookDto book) {
        try {
            Book savedBook = bookService.update(id, book);
            return ResponseEntity.status(HttpStatus.OK).body(savedBook);
        } catch (IllegalStateException e) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(Map.of("error", e.getMessage()));
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(Map.of("error", "An unexpected error occurred"));
        }
    }

}
```

서버를 재시작 후 `Talend`에서 Method는 Patch, URL은 `http://localhost:8080/books/1`을 입력한다. 그리고 Body에 다음과 같이 입력하여 Send를 하면 수정된 객체가 반환되는지 확인하자.

```json
{ 
  "title":"book"
}
```

![patch_api_test](/assets/img/posts/2024-08-09/patch_api_test.png)

### 삭제 API

서비스에 delete 메서드를 추가한다. delete도 마찬가지로 id가 있는지 확인 후 있을 경우에만 삭제를 한다.

```java
@Service
public class BookService {
    // ...

    public void delete(Integer id) {
        Book target = bookRepository.findById(id).orElse(null);
        if (target == null) {
            throw new IllegalStateException("존재하지 않는 책입니다");
        }

        bookRepository.deleteById(id);
    }
}
```

그런데 책이 없으면 에러를 반환하는 코드가 `update`에도 있어서 중복된다. `findBookOrThrow` 메서드를 추가해서 코드 중복을 제거한다.

```java
@Service
public class BookService {
    // ...

    public Book findBookOrThrow(Integer id) {
        Book target = bookRepository.findById(id).orElse(null);
        if (target == null) {
            throw new IllegalStateException("존재하지 않는 책입니다");
        }
        return target;
    }
    
    public Book update(Integer id, BookDto dto) {
        Book target = findBookOrThrow(id);

        Book book = dto.toEntity();
        target.patch(book);

        return bookRepository.save(target);
    }

    public void delete(Integer id) {
        findBookOrThrow(id);

        bookRepository.deleteById(id);
    }
}
```

책이 삭제되는지와 존재하지 않는 책일 경우 에러를 반환하는지 확인하는 테스트 코드를 작성한다.

```java
@SpringBootTest
@Transactional
class BookServiceTest {    
    @Test
    void 책_삭제() {
        BookDto book1 = new BookDto(null, "1234", "book");
        Book saveBook = bookService.create(book1);

        bookService.delete(saveBook.getId());

        Book result = bookRepository.findById(saveBook.getId()).orElse(null);
        assertEquals(null, result);
    }

    @Test
    void 존재하지_않는_책_확인() {
        long count = bookRepository.count();

        int id = (int) (count + 1);
        IllegalStateException e = assertThrows(IllegalStateException.class, () -> bookService.validate(id));

        assertEquals(e.getMessage(), "존재하지 않는 책입니다");
    }
}
```

테스트를 통과했다면 컨트롤러 코드도 추가하자.

```java
@RestController
public class BookController {
    // ...
    @DeleteMapping("/books/{id}")
    public ResponseEntity<?> deleteBook(@PathVariable("id") Integer id) {
        try {
            bookService.delete(id);
            return ResponseEntity.status(HttpStatus.OK).build();
        } catch (IllegalStateException e) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(Map.of("error", e.getMessage()));
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(Map.of("error", "An unexpected error occurred"));
        }
    }
}
```

정상적으로 삭제되었을 땐 반환할 데이터가 없어 build()로 빈 데이터를 전달한다.

서버를 재시작 후 `Talend`에서 Method는 DELETE, URL은 `http://localhost:8080/books/1`을 입력한다. Send를 눌러 정상적으로 삭제하는지 확인하자. 

![delete_api_test_200](/assets/img/posts/2024-08-09/delete_api_test_200.png)

이어서 한번 더 Send를 클릭하여 400 에러가 발생하는지 확인하자.

![delete_api_test_400](/assets/img/posts/2024-08-09/delete_api_test_400.png)

이렇게 기본적인 CRUD를 모두 구현하였다. 다음 편에서는 책 등록 시 태그도 같이 넘기고 태그 수정이 가능하도록 개선하자.

> 전체 코드는 [book-rating-backend 저장소의 chapter1-end 브랜치](https://github.com/suhyeoonn/book-rating-backend/tree/chapter1-end)에서 확인 가능합니다.
{: .prompt-info }