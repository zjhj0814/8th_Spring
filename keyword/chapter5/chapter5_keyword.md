## 키워드

# Domain
우리가 해결하고자 하는 특정 비즈니스 영역이나 문제 영역을 의미합니다. SW 개발에서 도메인은 개발 대상을 가리킵니다.

도메인 모델: 소프트웨어 도메인의 핵심 개념을 객체, 관계, 속성 등으로 나타낸 모델입니다. -> DDD
***
# 단방향 매핑
`단방향 매핑`: 한 엔티티만 다른 엔티티를 참조하는 방식입니다.(연관관계 주인만 참조 대상을 아는 것 A→B)
> `연관관계 주인`: 외래키를 관리하는 엔티티

단방향 매핑은 구조가 단순하고, 설계와 관리가 쉽습니다. 

반면 한 방향만 탐색이 가능하여, 참조 대상이 연관관계 주인에 대한 정보가 필요할 때(B→A) 불편할 수 있습니다.

# 양방향 매핑
`양방향 매핑`: 양쪽 엔티티가 서로를 참조하는 방식입니다. (A↔B)

양방향 매핑은 객체 그래프 탐색이 자유롭지만, 설계가 복잡해지고 상태 관리가 어려워질 수 있습니다.
이때 참조 대상은 연관관계 주인을 탐색할 수 있지만, 참조 대상 객체에서 연관관계를 변경(등록, 수정, 삭제)해도 데이터베이스에 반영되지 않습니다.

[ 양방향 매핑에서 고려해야 하는 사항 ]
1. 순환 참조

순환 참조 문제는 양방향 매핑에서는 두 엔티티가 서로를 참조함으로써 발생하는 문제입니다.

한 객체를 직렬화(json 변환)하거나 toString, equals 등을 호출할 때 양쪽 모두 서로를 무한히 참조하면서 반복적으로 접근하게 됩니다.
```text
member.getTeam()-> team.getMembers() -> member.getTeam() -> ...
```

이러한 문제는 @JsonIgnore 같은 Jackson 어노테이션으로 직렬화 대상을 제한하거나 DTO를 사용해 필요한 데이터만 추려서 반환함으로써 방지할 수 있습니다.


2. 두 엔티티의 상태 동기화 
3. 연관관계 주인만 연관관계 설정하는 점

2번 + 3번 => `연관관계 메서드`를 사용하자!

연관관계 메서드란? 연관관계 주인 설정 + 참조 대상 설정 코드가 하나인 것처럼 사용하는 메서드입니다.
```java
member.setTeam(team);
team.getMembers().add(member);
```

☞ 기본적으로 단방향으로 시작하고 필요할 때, 양방향 매핑을 사용할 것.
***
# N+1
`N+1`: 조회 시 1개의 쿼리를 예상했으나 예상치 못한 조회 쿼리가 N개 더 발생하는 문제

DBMS에서 직접 쿼리를 날릴 때가 아니라,
mybatis, JPA를 사용하며 자동화된 쿼리문이 날리면서 발생하는 문제입니다.

ex. 팀:팀원 = 1:N 일 때, 팀 객체 조회(1번의 쿼리) 후, 팀 객체에서 팀원 조회(N번의 쿼리) 시 N+1 문제 발생

***
# Spring Boot에서 N+1 문제해결방법
> ### N+1 문제를 해결하기 위해 즉시로딩을 사용하면 안되는 이유
> - 필요 없는 데이터까지 한 번에 로딩하여 성능이 저하될 수 있습니다.
> - 예상치 못한 쿼리가 발생할 수 있습니다.
> - JPQL에서 N+1 문제가 발생합니다.
>
## 1. 지연로딩 + 패치 조인
- `지연 로딩(lazy loading)`: 연결 엔티티에 대해 프록시 상태로 두고, 연결 엔티티를 사용할 때 영속성 컨텍스트를 확인하고 없으면 쿼리를 통해 프록시에 값을 채웁니다.
- `패치 조인(fetch join)`: SQL 조인 종류가 아닌, JPQL에서 성능 최적화를 위해 제공하는 기능으로, 연관된 엔티티와 컬렉션을 한 번에 함께 조회합니다.

  [ LEFT [OUTER] | INNTER ] JOIN FETCH 조인 경로

```jpaql
SELECT t FROM team t JOIN FETCH t.member
```
👉 root entity(team)에 대해 조회할 때, lazy loading으로 설정되어 있는 연관관계를 join 쿼리를 발생시켜 한 번에 조회할 수 있습니다.

### ❗ fetch join을 사용할 때 주의할 점
- 컬렉션을 패치 조인하고 Pagination까지 필요한 상황
    - **ToMany 관계(X)**

      many인 객체(ex. member)들이 one 객체(ex. team)에 fetch join 된다면 pagination에서 개수를 판단하기 힘들기 때문에 fetch join을 사용할 경우 임의로 **인메모리에서** 이를 조정합니다.

      (일단 many 객체의 모든 값(member)을 조회해서 인메모리에 저장하고, application에서 필요한 페이지 만큼을 반환합니다.

      따라서 페이징을 한 이유가 없고 out of memory가 발생할 확률이 높습니다.)

      따라서 컬렉션 조회를 하는 경우, fetch join을 사용하지 않고 조회할 컬렉션 필드에 대해 @BatchSize를 걸어 해결합니다.(BatchSize 설명은 아래에 이어집니다.)

    - **ToOne 관계(O)**
- **둘 이상의 컬렉션 패치 조인(ToMany) (X)**
  `MultipleBagFetchException` 발생 ex. Team -> Members -> Orders

## 2. 지연로딩 + batch size 설정
객체를 조회할 때 그때그때 쿼리가 나가는 것이 아니라, 조회할 때 batch size 만큼 in 쿼리로 한 번에 객체를 가져와서 지연 로딩에 대해 미연에 방지합니다.
### hibernate.default_batch_fetch_size: 글로벌 설정
### @BatchSize: 개별 최적화
```java
@BatchSize(size = 100)
@OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
private Set<Member> members;
```

하지만 일반적인 케이스에서 최적화된 데이터 사이즈를 알기 힘들기 때문에 확실하게 알지 못한다면 좋지 않은 방법이 될 수 있습니다.