---
title: 스프링부트 REST API 만들기 - 2. 다대다 관계 구현 (ManyToMany)
date: 2024-08-14 14:00:00 +/-TTTT
categories: [스프링부트]
tags: [스프링부트, springboot, restful api] # TAG names should always be lowercase
description: 스프링부트 RESTful API 실습하기 - @ManyToMany로 책과 태그 다대다 관계 구현
---

[이전 편](https://suhyeoonn.github.io/posts/springboot-restful-api-1/)에서는 태그 기능을 제외한 Book CRUD API를 작업했다. 이제 `tag` 엔티티를 추가하고 `book`과 `tag`에 다대다 관계를 설정하여 책 등록 및 수정 시 태그 정보를 받도록 하자.

### 요구사항 정리

책에 대한 요구사항은 다음과 같다.

- 리스트에서 책 정보와 평균 별점을 확인할 수 있다.
- 책을 등록할 수 있다.
    - ISBN, 책 이름, 태그를 입력해야 한다.
    - 동일한 ISBN을 가지는 책은 중복 등록할 수 없다.
    - 태그는 여러 개 선택 가능하다.
- 책 이름과 태그를 수정할 수 있다.
- 책을 삭제할 수 있다.

### 추가할 테이블 정보

### tag

태그 정보를 저장하는 테이블. 이 프로젝트에서 태그 설정 기능 없이 테이블에 저장된 고정값만 사용하려고 한다.

| 컬럼 이름 | 데이터 타입 | 설명 |
| --- | --- | --- |
| id | INT | 고유 식별자 (자동 증가) |
| name | VARCHAR(100) | 태그 이름 |

### book_tag

책과 태그 간의 다대다 관계를 표현하는 조인 테이블

| 컬럼 이름 | 데이터 타입 | 설명 |
| --- | --- | --- |
| book_id | INT | Books 테이블의 id를 참조 |
| tag_id | INT | Tags 테이블의 id를 참조 |

## Entity에 다대다 관계 설정

### Book Entity

Book 엔티티에 ManyToMany로 다대다 관계를 설정한다.

```java
package com.example.bookrating.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.util.HashSet;
import java.util.Set;

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

    @ManyToMany(cascade = CascadeType.ALL)
    @JoinTable(
            name = "book_tag",
            joinColumns = @JoinColumn(name = "book_id"),
            inverseJoinColumns = @JoinColumn(name = "tag_id")
    )
    private Set<Tag> tags = new HashSet<>();

    public void patch(Book book){
        if (book.title != null) {
            this.title = book.title;
        }
    }
}

```

- `@ManyToMany`:  `Book`과 `Tag` 사이의 다대다 관계를 정의.
- `@JoinTable`: 조인 테이블을 정의. 여기서는 `book_tag`라는 중간 테이블을 사용하여 `Book`과 `Tag` 간의 관계를 저장.
- `@JoinColumn(name = "book_id")`: `book_tag` 테이블의 `book_id` 컬럼과 이 엔티티의 `id` 필드를 매핑.
- `@JoinColumn(name = "tag_id")`: `book_tag` 테이블의 `tag_id` 컬럼과 `Tag` 엔티티의 `id` 필드를 매핑.
- `private Set<Tag> tags = new HashSet<>()`: `Book`과 `Tag` 간의 관계를 표현하는 필드이며, `Set`을 사용하여 중복된 태그를 방지합니다.

### Tag Entity

```java
package com.example.bookrating.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.util.HashSet;
import java.util.Set;

@Entity
@AllArgsConstructor
@NoArgsConstructor
@Getter
public class Tag {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;

    @ManyToMany(mappedBy = "tags")
    private Set<Book> books = new HashSet<>();

    public Tag(Integer id, String name) {
        this.id = id;
        this.name = name;
    }
}

```

- `@ManyToMany(mappedBy = "tags")`: `Book` 엔티티의 `tags` 필드에 의해 매핑된 다대다 관계. 이는 `Tag` 엔티티에서 관계의 주인이 아님을 의미함.
- `private Set<Book> books = new HashSet<>()`: `Tag`와 `Book` 간의 관계를 나타냄. 각 `Tag`가 연결된 `Book` 객체를 저장합니다.

### BookTag 엔티티도 추가해야 할까?

`BookTag` 엔티티를 별도로 생성할 필요가 있는지 여부는 애플리케이션의 요구 사항에 따라 다르다. 중간 테이블에 추가적인 정보를 저장하거나 복잡한 관계를 모델링할 필요가 없다면, `@ManyToMany` 어노테이션을 사용하여 중간 테이블을 자동으로 생성하고 관리해도 좋다.

지금은 간단하게 구현하는 것이 목표이므로, `BookTag` 엔티티를 추가하지 않고 `ManyToMany` 어노테이션만을 사용하려고 한다.

## Tag 리파지토리 추가

```java
package com.example.bookrating.repository;

import com.example.bookrating.entity.Tag;
import org.springframework.data.jpa.repository.JpaRepository;

public interface TagRepository extends JpaRepository<Tag, Integer> {

}

```

### DTO에 태그 추가

태그 ID 리스트를 관리할 tagIds 속성을 추가한다. 그리고 `toEntity` 에서 tags를 전달받도록 매개변수를 추가하자.

```java
package com.example.bookrating.dto;

import com.example.bookrating.entity.Book;
import com.example.bookrating.entity.Tag;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.util.Set;

@AllArgsConstructor
@NoArgsConstructor
@Getter
public class BookDto {
    private Integer id;
    private String isbn;
    private String title;
    private int[] tagIds;

    public Book toEntity(Set<Tag> tags) {
        return new Book(id, isbn, title, tags);
    }
}

```

## Tag 더미 데이터 추가

이 프로젝트는 태그 정보를 관리하는 기능을 제공하지 않는다. 고정된 태그 정보를 받아올 수 있도록 data.sql에 더미 데이터를 추가하자.

```sql
INSERT INTO book (isbn, title) VALUES
// ...

INSERT INTO tag (name) VALUES
('Classic'),
('Dystopian'),
('Romance'),
('Historical'),
('Fantasy'),
('Literary Fiction'),
('American'),
('British'),
('Young Adult'),
('Drama');

INSERT INTO book_tag (book_id, tag_id) VALUES
(1, 1),  -- 'To Kill a Mockingbird'에 'Classic' 태그를 연결
(1, 7),  -- 'To Kill a Mockingbird'에 'American' 태그를 연결
(2, 2),  -- '1984'에 'Dystopian' 태그를 연결
(2, 7),  -- '1984'에 'American' 태그를 연결
(2, 8),  -- '1984'에 'British' 태그를 연결
(3, 3),  -- 'Pride and Prejudice'에 'Romance' 태그를 연결
(3, 8),  -- 'Pride and Prejudice'에 'British' 태그를 연결
(4, 4),  -- 'The Kite Runner'에 'Historical' 태그를 연결
(5, 5),  -- 'The Handmaid''s Tale'에 'Fantasy' 태그를 연결
(6, 5),  -- 'The Hobbit'에 'Fantasy' 태그를 연결
(6, 9);  -- 'The Hobbit'에 'Young Adult' 태그를 연결
```

## GET API 리스폰스 스키마 변경

서버를 재시작한 후, `/books` API를 호출해보자. 이전과는 다르게 엄청나게 많은 데이터가 보인다.

![books_api_재귀호출](/assets/img/posts/2024-08-14/books_api_재귀호출.png)

Book 엔티티와 Tag 엔티티가 양방향 관계를 가질 때, Book 객체를 JSON으로 직렬화하면 Book의 tags 속성이 태그 객체들을 포함하게 된다. 그런데 각 Tag 객체도 다시 연관된 Book 객체를 포함하게 되면서, 서로를 계속 참조하는 구조가 반복되어 JSON 직렬화 과정에서 무한 재귀가 발생할 수 있다. 이로 인해 StackOverflowError가 발생하거나, 다른 형태의 오류가 나타날 수 있다.

이를 방지하기 위해 API 응답에서는 엔티티 대신 데이터 전송 객체(DTO)를 사용하는 것이 좋다. DTO는 데이터베이스 엔티티의 구조를 그대로 따르지 않고, 서로 간의 참조를 포함하지 않으므로 필요한 정보만을 포함할 수 있다.

이에 따라 반환 타입을 `List<Book>`에서 `List<BookDto>`로 변경하자. 다른 API도 이후에 DTO를 반환하도록 변경할 예정이므로, `toDto` 메서드를 생성하고 해당 메서드를 사용하여 DTO 리스트를 반환한다.

> **StackOverflowError**는 Java 프로그램 실행 중에 발생할 수 있는 에러로, 주로 **스택 메모리**가 가득 차서 더 이상 사용할 수 없을 때 발생한다.
> 

{ :.prompt_info }

```java
@Service
public class BookService {
    @Autowired
    private BookRepository bookRepository;
    @Autowired
    private TagRepository tagRepository;

    public List<BookDto> getBooks() {
        List<Book> books = bookRepository.findAll();
        return books.stream()
                .map(this::toDto).collect(Collectors.toList());
    }

    private BookDto toDto(Book book) {
        int[] tagIds = book.getTags().stream()
                .mapToInt(Tag::getId)
                .toArray();
        return new BookDto(book.getId(), book.getIsbn(), book.getTitle(), tagIds);
    }
}
```

컨트롤러도 반환 타입 수정

```java
@RestController
public class BookController {
    @Autowired
    private BookService bookService;

    @GetMapping("/books")
    public List<BookDto> getBooks() {
        return bookService.getBooks();
    }
}
```

다시 서버를 재시작한 후, API를 호출하면 이제 정상적으로 데이터를 가져오는 것을 확인할 수 있다.

![get-api-test](/assets/img/posts/2024-08-14/get-api-test.png)

## 책 등록 시 태그도 저장되도록 개선

### Service 코드 수정

`BookService`에 `findTagsByIds` 메서드를 추가한다. 이 메서드는 전달된 `tagIds` 배열을 기반으로 데이터베이스에서 해당하는 태그들을 검색하여 Set<Tag>로 반환한다. 데이터베이스에 등록되지 않은 태그는 별도의 에러를 반환하지 않고 Set에서 제외할 예정이다. `findTagsByIds` 에서 반환된 `tags`는 `toEntity` 메서드로 전달된다.

```java
@Service
public class BookService {
    // ...

    public BookDto create(BookDto dto) {
        // isbn이 동일한 책은 중복 등록할 수 없음
        bookRepository.findByIsbn(dto.getIsbn()).ifPresent(e -> {
            throw new IllegalStateException("이미 존재하는 책입니다");
        });

        Set<Tag> tags = findTagsByIds(dto.getTagIds());

        Book saved = bookRepository.save(dto.toEntity(tags));

        return toDto(saved);
    }
    
    private Set<Tag> findTagsByIds(int[] tagIds) {
        Set<Tag> tags = new HashSet<>();
        for (int tagId : tagIds) {
            Optional<Tag> tag = tagRepository.findById(tagId);
            if (tag.isPresent()) {
                tags.add(tag.get());
            }
        }
        return tags;
    }
}

```

### 테스트 코드 수정

```java
@SpringBootTest
@Transactional
class BookServiceTest {
    @Autowired
    BookService bookService;
    @Autowired
    BookRepository bookRepository;
    @Autowired
    TagRepository tagRepository;

    int[] tagIds = {1,3};

    @Test
    void 책_등록() {
        BookDto book = new BookDto(null, "1234", "book", tagIds);

        bookService.create(book);

        Book result = bookRepository.findByIsbn(book.getIsbn()).get();

        assertEquals(result.getIsbn(), book.getIsbn());

        assertEquals(tagIds.length, result.getTags().size());
    }
    
    // ...
    
    @Test
    void 책_제목_수정() {
        BookDto book1 = new BookDto(null, "1234", "book", tagIds);
        BookDto saveBook = bookService.create(book1); // BookDto로 수정

        BookDto dto = new BookDto(null, null, "update", null);
        bookService.update(saveBook.getId(), dto);

        Book result = bookRepository.findById(saveBook.getId()).get();
        assertEquals(result.getTitle(), dto.getTitle());
    }
}
```

create 메서드를 사용하는 `책_제목_수정, 책_삭제` 테스트코드의 반환 타입을 `Book`에서 `BookDto`로 변경한다. 

그리고 이어서, 컨트롤러에서 `bookService.create(dto)`의 반환 타입도 BookDto로 변경한다.

```java
@RestController
public class BookController {
    // ...

    @PostMapping("/books")
    public ResponseEntity<?> createBook(@RequestBody BookDto dto) {
        try {
            BookDto savedBook = bookService.create(dto); // BookDto로 변경
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

서버 재시작 후 Talend에서 아래 값을 Body로 전달하여 수정이 정상적으로 되는지 확인한다.

```json
{
  "isbn":"10101011",
  "title":"book1",
  "tagIds":[3,8]
}
```

![post-api-test](/assets/img/posts/2024-08-14/post-api-test.png)

## PATCH API 수정

entity patch 메서드에 tag 값이 있을 경우 해당 값으로 교체하는 코드 추가

```java
@Entity
@AllArgsConstructor
@NoArgsConstructor
@Getter
public class Book {
    // ...
    public void patch(Book book){
        if (book.title != null) {
            this.title = book.title;
        } if (!book.tags.isEmpty()) {
            this.tags = book.tags;
        }
    }
}
```

태그가 변경되는지 확인하는 테스트 코드 추가

```java
@SpringBootTest
@Transactional
class BookServiceTest {
    // ...
    @Test
    void 책_제목_및_태그_수정() {
        BookDto book1 = new BookDto(null, "1234", "book", new int[]{1});
        BookDto saveBook = bookService.create(book1);

        BookDto dto = new BookDto(null, null, "update", new int[]{2,3});
        bookService.update(saveBook.getId(), dto);

        Book result = bookRepository.findById(saveBook.getId()).get();
        assertEquals("update", result.getTitle());

        assertEquals(2, result.getTags().size());

        Set<Integer> tagIds = result.getTags().stream().map(Tag::getId).collect(Collectors.toSet());
        assertTrue(tagIds.containsAll(Arrays.asList(2,3)));
    }
}
```

Dto가 반환되도록 서비스 수정

```java
@Service
public class BookService {
		// ...
		public BookDto update(Integer id, BookDto dto) {
        Book target = findBookOrThrow(id);

        Set<Tag> tags = Optional.ofNullable(dto.getTagIds())
                .map(this::findTagsByIds)
                .orElse(Collections.emptySet());

        Book book = dto.toEntity(tags);

        target.patch(book);

        return toDto(bookRepository.save(target)); // toDto 사용하여 Dto로 변환
    }
}
```

컨트롤러도 수정

```java
@RestController
public class BookController {
		@PatchMapping("/books/{id}")
    public ResponseEntity<?> updateBook(@PathVariable("id") Integer id,  @RequestBody BookDto dto) {
        try {
            BookDto savedBook = bookService.update(id, dto); // BookDto로 변경
            return ResponseEntity.status(HttpStatus.OK).body(savedBook);
        } catch (IllegalStateException e) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(Map.of("error", e.getMessage()));
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(Map.of("error", "An unexpected error occurred"));
        }
    }
}
```

```json
{ 
  "title":"book1",
  "tagIds":[3,4]
}
```

![patch-api-test](/assets/img/posts/2024-08-14/patch-api-test.png)

## GET Tag API 추가

프론트에서 책 추가 시 tags API를 호출하여 태그 정보를 받으려고 한다. 태그 정보를 조회하니 HTTP 메서드는 GET, url은 `/tags`로 지정하자.

**Tag DTO 추가**

```java
package com.example.bookrating.dto;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;

@AllArgsConstructor
@NoArgsConstructor
@Getter
public class TagDto {
    private Integer id;
    private String name;
}

```

**Tag Service 추가**

```java
package com.example.bookrating.service;

import com.example.bookrating.dto.TagDto;
import com.example.bookrating.repository.TagRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class TagService {
    @Autowired
    private TagRepository tagRepository;

    public List<TagDto> getTags() {
        return tagRepository.findAll().stream()
                .map(tag -> new TagDto(tag.getId(), tag.getName()))
                .collect(Collectors.toList());
    }
}

```

**Tag Controller**

```java
package com.example.bookrating.controller;

import com.example.bookrating.dto.TagDto;
import com.example.bookrating.service.TagService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class TagController {
    @Autowired
    private TagService tagService;

    @GetMapping("/tags")
    public List<TagDto> getTags() {
        return tagService.getTags();
    }

}

```

서버 재시작 후 브라우저나 Talenad에서 http://localhost/tags 를 호출하여 데이터가 출력되는지 확인하자.

![get_tags_api](/assets/img/posts/2024-08-14/get_tags_api.png)

---

이렇게 태그 테이블을 추가하여 다대다 구현을 완료하였다. 다음 편에서는 책에 대한 리뷰를 CRUD 하는 API를 작업하자.

> 전체 코드는 [여기에서](https://github.com/suhyeoonn/book-rating-backend/tree/chapter2-end) 확인 가능합니다. (chapter2-end 브랜치)
{: .prompt-info }