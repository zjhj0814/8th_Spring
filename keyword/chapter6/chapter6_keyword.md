## 키워드

# 지연로딩과 즉시로딩의 차이

`즉시로딩`
: 객체를 조회할 때 연관된 객체들까지 모두 로딩하는 방법입니다.

- 필요없는 데이터까지 한 번에 로딩하여 성능이 저하될 수 있습니다.
- 예상치 못한 쿼리가 발생할 수 있습니다.
- 기본 엔티티 조회(ex. findAll()) 또는 JPQL 사용시, 기본 엔티티 조회 후 EAGER 감지로 인한 N쿼리가 추가로 발생할 수 있습니다.

> em.createQuery("select t from Team t",Team.class).getResultList();로 JPQL 호출
> ```
> SELECT * FROM TEAM;
> 
> SELECT * FROM MEMBER WHERE TEAM_ID = 1;
> 
> SELECT * FROM MEMBER WHERE TEAM_ID = 2;
> ```

`지연로딩`
: 연결 엔티티에 대해 프록시 상태로 두고, 연결 엔티티를 사용할 때 영속성 컨텍스트를 확인하고 없으면 쿼리를 통해 그때그때 프록시에 값을 채웁니다.

***
# Spring Boot에서 N+1 문제해결방법
# 1. 지연로딩 + 패치 조인
## Fetch Join
SQL 조인 종류가 아닌, JPQL에서 성능 최적화를 위해 제공하는 기능으로, 연관된 엔티티와 컬렉션을 한 번에 함께 조회합니다.
```mysql
[LEFT [OUTER]|INNTER] JOIN FETCH 조인 경로
ex. SELECT t FROM team t JOIN FETCH t.member
```
위 코드처럼 root entity(team)에 대해 조회할 때, lazy loading으로 설정되어 있는 연관관계를 join 쿼리를 통해 한 번에 조회할 수 있습니다.

### Fetch Join을 사용할 때 주의할 점
- 컬렉션을 패치 조인하고 Pagination까지 필요한 상황
    - **ToMany 관계(X)**

      many인 객체(ex. member)들이 one 객체(ex. team)에 fetch join 된다면 pagination 상황에서 개수를 판단하기 힘들기 때문에 fetch join을 사용할 경우 임의로 **인메모리에서** 이를 조정합니다.

      (일단 many 객체의 모든 값(member)을 조회해서 인메모리에 저장하고, application에서 필요한 페이지 만큼을 반환합니다.

      따라서 모든 many인 객체를 가져오기 때문에 페이징을 한 의미가 없고 `OutOfMemory`가 발생할 확률이 높습니다.)

      따라서 컬렉션 조회를 하는 경우, fetch join을 사용하지 않고 조회할 컬렉션 필드에 대해 `@BatchSize`를 걸어 해결합니다.(BatchSize 설명은 아래에 이어집니다.)

    - **ToOne 관계(O)**
- **둘 이상의 컬렉션 패치 조인(ToMany) (X)**
  `MultipleBagFetchException` 발생 ex. Team -> Members -> Orders
# 2. 지연로딩 + batch size 설정
객체를 조회할 때 그때그때 쿼리가 나가는 것이 아니라, 조회할 때 batch size 만큼 in 쿼리로 한 번에 객체를 가져와서 지연 로딩에 대해 미연에 방지합니다.
### hibernate.default_batch_fetch_size: 글로벌 설정
### @BatchSize: 개별 최적화
```java
@BatchSize(size = 100)
@OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
private Set<Member> members;
```

하지만 일반적인 케이스에서 최적화된 데이터 사이즈를 알기 힘들기 때문에 확실하게 알지 못한다면 좋지 않은 방법이 될 수 있습니다.
***
# @EntityGraph
- 지연로딩과 즉시로딩 전략은 정적으로 결정되고 이는 런타임에 변경하지 못하는 한계를 갖고 있었습니다.
  이때 엔티티를 로딩할 때 런타임 성능을 향상하기 위해 JPA Entity Graph가 도입되었습니다.
- 연관 엔티티를 단일 JOIN 쿼리로 한 번에 조회해서 N+1 쿼리 문제를 해결할 수 있습니다.
- 필요 필드만 선택적으로 로딩하여 메모리 사용량을 절감할 수 있습니다.
```java
@Override
@EntityGraph(attributePaths = {"member"})
List<Team> findAll();
```
***
# JPQL
: JPQL은 JPA에서 객체를 대상으로 데이터를 조회·검색하는 데 쓰는 쿼리 언어입니다. (JPA는 객체와 DB를 연결해주는 기술)

***
# Querydsl
: 타입 안전성을 보장하는 자바 기반의 쿼리 빌더 라이브러리입니다.

- 컴파일 시점에 쿼리의 오류를 잡을 수 있습니다.
- 쿼리를 자바 코드로 작성해서 메서드 체이닝을 통해 복잡한 쿼리를 작성하는 데 유리합니다.
- 동적 쿼리 작성이 편리합니다.
***
## 실습
![Image](https://github.com/user-attachments/assets/550f7046-2f79-4bcb-a198-30934ec400ea)