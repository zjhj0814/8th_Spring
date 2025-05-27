## 키워드
### 페이징 요청 정보

Sort와 / Pageable 객체는 리포지토리 메서드의 파라미터로 넘겨주어
데이터베이스 쿼리의 정렬과 / 정렬과 페이징 정보를 전달합니다.

### Sort

Sort는 **데이터 정렬 정보**를 담는 인터페이스입니다.

특정 컬럼을 기준으로 오름차순/내림차순 정렬을 지정할 수 있고 여러 필드에 대해 복합 정렬도 가능합니다.

```java
List<Book> books = bookRepository.findAll(Sort.by("author").ascending().and(Sort.by("title").descending()));
```

### Pageable (Sort 내부적으로 포함)

Pageable은 사용자가 **요청한 페이지 번호(zero-based index), 한 페이지에 보여줄 데이터 개수, 정렬 방식 등의 페이징 요청 정보**를 담는 인터페이스입니다.

**PageRequest**는 Pageable 인터페이스의 대표적인 **구현체**로 페이징 처리에 필요한 정보를 담는 **불변 객체**입니다.

```java
//PageRequest.of(int page, int size, Sort sort)

Pageable pageableWithoutSort = PageRequest.of(0,10);
Pageable pageable = PageRequest.of(0,10,Sort.by("createdAt").descending());
Page<Book> bookPage = bookRepository.findAll(pageable);
```

### 페이징 결과

### Slice

Slice는 페이징된 데이터의 “한 조각”을 표현하는 인터페이스입니다.

Slice는 현재 페이지 데이터 목록, 현재 Slice의 번호, 크기, 실제 데이터 수, 정렬 정보, 현재 Slice가 첫 번째/마지막인지, 다음/이전 Slice가 존재하는지 여부를 제공합니다.

```java
public interface Slice<T> extends Streamable<T> {
	int getNumber(); //현재 페이지
	int getSize(); //페이지 크기 
	int getNumberOfElements(); //현재 페이지에 나올 데이터 수 
	**List<T> getContent()**; //조회된 데이터
	boolean hasContent(); //조회된 데이터 존재 여부 
	Sort getSort(); //정렬 정보
	boolean isFirst(); //현재 페이지가 첫 페이지 인지 여부 
	boolean isLast(); //현재 페이지가 마지막 페이지 인지 여부 
	**boolean hasNext();** //다음 페이지 여부
	**boolean hasPrevious();** //이전 페이지 여부
	Pageable getPageable(); //페이지 요청 정보 
	Pageable nextPageable(); //다음 페이지 객체 
	Pageable previousPageable();//이전 페이지 객체
	<U> Slice<U> map(Function<? super T, ? extends U> converter); //변환기
}
```

> 💡 Slice는 전체 데이터 개수나 전체 페이지 수를 제공하지 않아, 전체 개수를 세는 쿼리를 실행하지 않기 때문에 성능상 이점이 있습니다.

주로 **무한 스크롤**이나 더보기 버튼 등 전체 개수 정보가 필요 없는 UI에 적합합니다.

```java
Slice<Member> slice = memberRepository.findSliceBy(pageable);
List<Member> members = slice.getContent();
boolean hasNext = slice.hasNext();
```

### Page (Slice 상속)
Page는 Slice를 상속하는 하위 인터페이스로, Slice가 가진 모든 기능을 그대로 포함합니다. Page는 Slice의 기능에 더해, **전체 데이터 개수와 전체 페이지 수**를 추가로 제공합니다.

Page는 내부적으로 전체 데이터 개수를 구하기 위한 추가 쿼리(count 쿼리)를 실행합니다. 이 때문에 전체 페이지 내비게이션이 필요한 UI(예: 페이지 번호를 보여주는 페이지네이션)에 적합합니다

전체 페이지 수, 전체 데이터 개수가 꼭 필요한 일반적인 페이징 UI(페이징 번호 내비게이션 등)에 적합합니다.

```java
public interface Page<T> extends Slice<T> {
	**int getTotalPages();** //전체 페이지 수 
	**long getTotalElements();** //전체 데이터 수
	<U> Page<U> map(Function<? super T, ? extends U> converter); //변환기
}
```

```java
Page<Member> page = memberRepository.findAll(pageable);
List<Member> members = page.getContent();
long totalCount = page.getTotalElements();
int totalPages = page.getTotalPages();
```

> 💡 [ 카운트 쿼리 분리하기 ]
> 
> 카운트 쿼리는 DB의 모든 데이터를 카운트해야 하기 때문에 무거운 기능입니다.
> 
> 따라서 필요한 경우에만 카운트 쿼리를 분리해야 합니다.
> 
> 예를 들어, left outer join 시 countQuery에서도 left outer join을 할 필요는 없으니 다음 코드와 같이 카운트 쿼리를 분리해 줍니다.
> 
```java
@Query(value = "select m from Member m left join m.team t", countQuery = "select count(m.username) from Member m")
```
***
### 객체 그래프 탐색
**객체 그래프 탐색**은 JPA에서 엔티티(객체) 사이의 연관관계를 따라가며 연관된 객체들을 자유롭게 조회하는 것을 의미합니다. 

즉, 한 객체에서 다른 연관 객체로 점(.)을 찍어 접근하는 방식으로, 실제 비즈니스 로직에서 필요한 연관 데이터를 자연스럽게 탐색할 수 있도록 해줍니다.

```java
member.getTeam();
member.getOrder().getDelivery();
```

JPA는 이러한 객체 그래프 탐색을 지원하기 위해, 연관된 객체를 프록시 객체로 채우고 실제로 사용할 때 데이터베이스에서 조회하는 **지연 로딩(Lazy Loading)** 기능을 제공합니다. 즉, 연관 객체를 처음 사용할 때 SELECT 쿼리가 실행되어 필요한 데이터를 가져오게 됩니다.

반대로, 연관 객체를 미리 함께 조회하는 방식도 있는데, 이를 **즉시 로딩**이라고 합니다.

객체 그래프를 무한정 탐색하는 것은 가능하지만, 실제로 모든 연관 객체를 한 번에 메모리에 올리는 것은 비효율적입니다. 
따라서 필요한 범위만 탐색하고, 필요에 따라 **fetch join**이나 **엔티티 그래프(Entity Graph)** 같은 최적화 기법을 활용해야 합니다.
