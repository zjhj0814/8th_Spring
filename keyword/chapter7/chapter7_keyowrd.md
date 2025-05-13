## [키워드]
# RestControllerAdvice
`@ControllerAdvice`: 여러 컨트롤러에 대해 전역적으로 ExceptionHandler를 적용해 준다.

`@RestControllerAdvice`: @ControllerAdvice와 달리 **@ResponseBody가 붙어 있어 응답을 Json으로 내려준다.**

잘못된 URI를 호출하여 발생하는 NoHandlerFoundException 등 스프링 예외를 미리 처리해 둔 `ResponseEntityExceptionHandler`를 추상 클래스로 제공하고 있다. 따라서 ControllerAdvice 클래스가 이 추상 클래스를 상속받아 @Override 하면 된다.

- 하나의 클래스로 모든 컨트롤러에 대해 **전역적으로** 예외 처리가 가능하다.
- 직접 정의한 에러 응답을 **일관성 있게** 클라이언트에게 내려 줄 수 있다.
- 별도의 try-catch문이 없어 코드의 **가독성**이 높아진다.
***
# Lombok
| 어노테이션                  | 주요 기능                                                   | 비고                                                                 |
|----------------------------|------------------------------------------------------------|----------------------------------------------------------------------|
| @Getter / @Setter          | getter, setter 자동 생성                                   |                                                                      |
| @ToString                  | toString() 자동 생성                                       | exclude를 통해 특정 필드 제외 가능                                  |
| @NoArgsConstructor         | 기본 생성자 자동 생성                                     |                                                                      |
| @AllArgsConstructor        | 전체 필드 생성자 자동 생성                                |                                                                      |
| @RequiredArgsConstructor   | final/@NonNull 필드 생성자 자동 생성                      |                                                                      |
| @EqualsAndHashCode         | equals(), hashCode() 메서드 자동 생성                     |                                                                      |
| @Data                      | getter, setter, toString, equals, requiredArgsConstructor |                                                                      |
| @Builder                   | 빌더 패턴 적용                                            |                                                                      |
| @Value                     | 표현식 기반으로 값을 주입                                 |  |
| @Slf4j                     | 자동으로 Log 변수 선언 → 바로 로그 사용 가능             | slf4j: 로깅에 대한 추상 레이어를 제공하는 인터페이스 모음          |

***
# @JsonProperty
- 정의 및 용도

  @JsonProperty는 Jackson 라이브러리에서 제공하는 어노테이션으로, 자바 객체의 필드명과 JSON의 키가 다를 때 명시적으로 매핑을 지정할 수 있다.

  주로 JSON 데이터의 키와 자바 필드명이 다르거나, 네이밍 컨벤션이 다를 때 사용한다.

    ```java
    public class Student {
        @JsonProperty("my_name")
        private String myName;
    }
    ```

- 속성
    - JSON에서 사용할 속성명 지정
***
# @JsonPropertyOrder
- 정의 및 용도

  @JsonPropertyOrder는 JSON으로 직렬화할 때 필드의 순서를 지정할 수 있는 Jackson 어노테이션입니다.

  주로 JSON 응답의 필드 순서가 중요할 때 사용합니다.

    ```java
    @JsonPropertyOrder({"name", "age", "country"})
    public class Person {
        private String name;
        private int age;
        private String country;
    }
    ```

- 속성
    - 직렬화 시 필드의 순서를 배열로 지정
***
# @JsonInclude
- 정의 및 용도

  @JsonInclude는 객체를 JSON으로 직렬화할 때 특정 조건에 따라 필드를 포함하거나 제외할 수 있게 해주는 Jackson 어노테이션입니다.

  예를 들어, null 값이나 비어있는 값의 필드를 JSON에서 제외하고 싶을 때 사용합니다.

    ```java
    @JsonInclude(JsonInclude.Include.NON_NULL)
    public class TestClass {
        private String name;
        private String description;
    }
    ```

- 속성
    - JsonInclude.Include.ALWAYS: 항상 포함(기본값)
    - JsonInclude.Include.NON_NULL: null이 아닌 값만 포함
    - JsonInclude.Include.NON_EMPTY: null 또는 빈 값이 아닌 값만 포함
    - JsonInclude.Include.NON_DEFAULT: 기본값이 아닌 값만 포함