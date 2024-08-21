---
title: JPA 엔티티 매니저
date: 2024-08-21 14:00:00 +/-TTTT
categories: [스프링부트]
tags: [스프링부트, TIL] # TAG names should always be lowercase
description: 엔티티 매니저에 대해 알아보고 JpaRepository와의 차이 파악하기
---

이때까지 아래처럼 JpaRepository를 사용해서 CRUD를 구현했었다.

```java
public interface BookRepository extends JpaRepository<Book, Integer> {
    Optional<Book> findByIsbn(String isbn);
}
```

그런데 `실전! 스프링 부트와 JPA 활용1` 강의에서 엔티티 매니저를 사용하길래 이게 무엇인지 알아보고자 한다.

---

엔티티 매니저(EntityManager)와 `JpaRepository`는 둘 다 JPA(Java Persistence API)를 사용하여 데이터베이스와 상호작용하는 도구이지만, 사용 방식과 목적이 다르다.

## 엔티티 매니저란?

엔티티 객체의 라이프사이클을 관리하고 데이터베이스 작업을 수행하는 데 사용된다.

🤔 TODO 라이프사이클 관리란?

엔티티 매니저는 보다 낮은 수준의 API로, 다음과 같은 작업을 할 수 있다.

- persist(): 엔티티를 데이터베이스에 저장
  - JpaRepository의 save인 것 같다. 아래에서 더 자세히 알아보자.
- find(): 데이터베이스에서 엔티티를 조회
- merge(): 엔티티 업데이트
- remove(): 엔티티 삭제
- JPQL: 엔티티 객체를 기반으로 한 쿼리 작성과 실행 지원

### JpaRepository와의 차이점

**추상화 수준**

`JpaRepository` 는 더 높은 수준의 추상화를 제공하여 개발자가 데이터베이스 작업을 쉽게 수행할 수 있게 한다. 반면, 엔티티 매니저는 개발자가 직접 데이터베이스 작업의 세부 사항을 관리할 수 있다.

**자동화**

`JpaRepository` 를 사용하면 Spring Data JPA가 자동으로 리포지토리 메서드를 구현해 주며, 개발자는 단지 인터페이스만 정의하면 된다. 엔티티 매니저를 사용하면 개발자가 모든 데이터베이스 연산을 직접 구현해야 한다.

결론적으로, 엔티티 매니저와 `JpaRepository` 는 둘 다 JPA를 사용하는 방법이지만 단순한 CRUD 작업을 주로 할 경우 `JpaRepository` 를 사용하는 것이 편리하고 생산성이 높으며, 복잡한 쿼리나 트랜잭션 관리가 필요한 경우 엔티티 매니저를 사용하는 것이 적합할 수 있다.

## 이미 엔티티 매니저를 사용하고 있었다!

`JpaRepository` 인터페이스의 구현체 `SimpleJpaRepository`를 보면 save 메서드가 있는데, 이 코드에서 이미 엔티티 매니저의 `persist`를 사용하고 있었다.

```java
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    private static final String ID_MUST_NOT_BE_NULL = "The given id must not be null";
    private final JpaEntityInformation<T, ?> entityInformation;
    private final EntityManager entityManager;
    // ...
		@Transactional
		public <S extends T> S save(S entity) {
		    Assert.notNull(entity, "Entity must not be null");
		    if (this.entityInformation.isNew(entity)) {
		        this.entityManager.persist(entity);
		        return entity;
		    } else {
		        return this.entityManager.merge(entity);
		    }
		}
}
```

이를 통해 `JpaRepository`는 `EntityManager` 기반으로 만들어진 걸 알 수 있다.

---

그런데 엔티티 매니저 방식을 실무에서 더 많이 사용하는건가? 아직 8강까지밖에 못 본 상태인데, 왜 강의에서 엔티티 매니저를 선택했는지 궁금하다.
