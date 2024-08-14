---
title: 스프링부트 REST API 만들기 - 2. 다대다 관계 구현 (ManyToMany)
date: 2024-08-14 14:00:00 +/-TTTT
categories: [스프링부트]
tags: [스프링부트, springboot, restful api] # TAG names should always be lowercase
description: 스프링부트 RESTful API 실습하기 - @ManyToMany로 책과 태그 다대다 관계 구현
---

### tag

태그 정보를 저장하는 테이블. 이 프로젝트에서 태그 설정 기능 없이 테이블에 저장된 고정값만 사용하려고 한다.

| 컬럼 이름 | 데이터 타입  | 설명                    |
| --------- | ------------ | ----------------------- |
| id        | INT          | 고유 식별자 (자동 증가) |
| name      | VARCHAR(100) | 태그 이름               |

### book_tag

책과 태그 간의 다대다 관계를 표현하는 조인 테이블

| 컬럼 이름 | 데이터 타입 | 설명                     |
| --------- | ----------- | ------------------------ |
| book_id   | INT         | Books 테이블의 id를 참조 |
| tag_id    | INT         | Tags 테이블의 id를 참조  |

## Entity에 다대다 관계 설정

### Book

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

- `@ManyToMany`: 다대다 관계를 나타냅니다. `Book`과 `Tag` 사이의 다대다 관계를 정의합니다.
- `@JoinTable`: 조인 테이블을 정의합니다. 여기서는 `book_tag`라는 중간 테이블을 사용하여 `Book`과 `Tag` 간의 관계를 저장합니다.
- `@JoinColumn(name = "book_id")`: `book_tag` 테이블의 `book_id` 컬럼과 이 엔티티의 `id` 필드를 매핑합니다.
- `@JoinColumn(name = "tag_id")`: `book_tag` 테이블의 `tag_id` 컬럼과 `Tag` 엔티티의 `id` 필드를 매핑합니다.
- `private Set<Tag> tags = new HashSet<>()`: `Book`과 `Tag` 간의 관계를 표현하는 필드입니다. `Set`을 사용하여 중복된 태그를 방지합니다.

### Tag

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

- `@ManyToMany(mappedBy = "tags")`: `Book` 엔티티의 `tags` 필드에 의해 매핑된 다대다 관계입니다. 이는 `Tag` 엔티티에서 관계의 주인이 아님을 의미합니다.
- `private Set<Book> books = new HashSet<>()`: `Tag`와 `Book` 간의 관계를 나타냅니다. 각 `Tag`가 연결된 `Book` 객체를 저장합니다.

`BookTag` 엔티티를 별도로 생성할 필요가 있는지 여부는 애플리케이션의 요구 사항에 따라 다릅니다. 중간 테이블에 추가적인 정보를 저장하거나 복잡한 관계를 모델링할 필요가 없다면, `@ManyToMany` 어노테이션을 사용하여 중간 테이블을 자동으로 생성하고 관리하는 것이 좋습니다.

## Tag 리파지토리 추가

```java
package com.example.bookrating.repository;

import com.example.bookrating.entity.Tag;
import org.springframework.data.jpa.repository.JpaRepository;

public interface TagRepository extends JpaRepository<Tag, Integer> {

}

```

### DTO에 태그 추가

태그 속성 추가

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

Book 엔티티와 Tag 엔티티가 다음과 같이 양방향 관계를 가질 때, Book 객체를 JSON으로 직렬화할 때, Book의 tags 속성이 태그 객체들을 포함하게 되고, 각 Tag 객체는 다시 그와 연관된 Book 객체들을 포함하게 됩니다. 이렇게 서로를 참조하는 구조가 반복되면서 JSON 직렬화 과정에서 무한 재귀가 발생하고, 결국 StackOverflowError가 발생하거나 다른 형태의 오류가 나타날 수 있습니다.

API 응답으로 엔티티 대신 데이터 전송 객체(DTO)를 사용합니다. DTO는 데이터베이스 엔티티의 구조를 그대로 따르지 않아도 되므로, 서로간의 참조를 포함하지 않아도 됩니다. 각 엔티티를 DTO로 변환하는 과정에서 필요한 정보만 포함시킬 수 있습니다.

반환 타입을 List<Book>에서 List<BookDto>로 변경. 다른 API도 이후에 Dto가 반환되도록 변경할 예정이라 toDto 메서드 생성 후 해당 메서드를 사용하여 dto 리스트 반환

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

![get-api-test](/assets/img/posts/2024-08-14/get-api-test.png)

## 책 등록 시 태그도 저장되도록 개선

### Service 코드 수정. 반환값도 Dto로 변경

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
        BookDto book1 = new BookDto(nul

        l, "1234", "book", tagIds);
        BookDto saveBook = bookService.create(book1); // BookDto로 수정

        BookDto dto = new BookDto(null, null, "update", null);
        bookService.update(saveBook.getId(), dto);

        Book result = bookRepository.findById(saveBook.getId()).get();
        assertEquals(result.getTitle(), dto.getTitle());
    }
}
```

create 메서드를 사용하는 `책_제목_수정, 책_삭제` 테스트코드의 반환 타입도 Book에서 BookDto로 변경해야 한다.

컨트롤러 반환값을 Dto로 변경

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

서버 재시작 후 Talend에서 확인

```json
{
  "isbn": "10101011",
  "title": "book1",
  "tagIds": [3, 8]
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
  "title": "book1",
  "tagIds": [3, 4]
}
```

![patch-api-test](/assets/img/posts/2024-08-14/patch-api-test.png)

> 전체 코드는 [book-rating-backend 저장소의 chapter2-end 브랜치](https://github.com/suhyeoonn/book-rating-backend/tree/chapter2-end)에서 확인 가능합니다.
{: .prompt-info }