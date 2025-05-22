## 키워드
## 예외(Exception)란?

- 사용자의 잘못된 조작이나 개발자의 코딩 실수로 인해 발생하는 프로그램 오류.
- 예외가 발생되면 프로그램은 곧바로 종료된다는 점에서 에러와 동일하나, 예외는 예외 처리를 통해 프로그램을 종료하지 않고 정상 실행 상태가 유지되도록 할 수 있다.

## 1. Checked Exception

: 명시적으로 처리해야 하는 예외로 쉽게 생각해보자면 코딩을 하다가 빨간 줄이 뜨면서 예외 처리를 하지 않으면 실행을 못하는 경우에 해당한다. 주로 외부 환경(파일, 네트워크, 데이터베이스 등)과의 입출력 작업에서 발생한다.

- IOException: 파일이 존재하지 않거나 접근할 수 없을 때, 또는 네트워크 등 입출력 작업 중 오류가 발생할 때 발생
- ClassNotFoundException: 클래스 로더가 지정한 클래스 파일을 찾지 못할 때 발생
- SQLException: 데이터베이스 연결에 실패하거나 잘못된 SQL 쿼리 실행 등 데이터베이스 작업 중 오류가 발생할 때 발생
- FileNotFoundException: 파일을 읽거나 쓸 때 지정한 파일이 존재하지 않거나 경로가 잘못되었을 때 발생

## 2. Unchecked Exception

: RuntimeException 상속하는 예외로, 명시적인 예외 처리를 강제하지 않는다. 쉽게 생각해보자면 실행 도중 발생하는 경우가 해당한다. 발생 위험이 있는 경우에 지나치고 넘어갈 수 있어 주의해야 한다.

- NullPointerException: 객체가 null인 상태에서 객체를 사용하려 할 때 발생
- ArrayIndexOutOfBoundsException: 배열에서 인덱스 범위를 초과하여 사용할 때 발생
- NumberFormatException: 숫자로 변환할 수 없는 문자열을 숫자로 변환할 때 발생
- ClassCastException: 억지로 타입 변환을 시도할 경우 발생
- IndexOutOfBoundException
- ArithmeticException

## Error란?

: 시스템에 비정상적인 상황이 발생한 경우로 앞선 checked/unchecked와는 다른 특수한 형태의 에러인데, 애플리케이션 단에서는 복구할 수 없는 상황을 주로 일컫는다.

이와 같은 오류는 하드웨어 오류 또는 운영 체제와 같은 문제로 인해 발생하며 프로그램의 제어 범위를 벗어납니다. 따라서 예외처럼 처리하거나 포착할 수 없으며 신중한 설계나 테스트를 통해 방지해야 합니다.

- StackOverflowError: 호출 스택의 메모리 한도를 초과할 때 발생
- OutOfMemoryError: JVM이 객체를 위한 메모리를 할당할 수 없을 때, 즉 힙(Heap)이나 기타 메모리 공간이 부족할 때 발생
***
## @Valid

: 객체의 필드 값이 정의된 제약 조건(Validation Constraints)을 만족하는지 자동으로 검사하는 어노테이션

- **용도**

  메서드의 파라미터(특히, 컨트롤러에서 @RequestBody, @ModelAttribute 등으로 전달받는 객체)에 붙여 사용하며, 해당 객체의 필드에 선언된 검증 어노테이션(@NotNull, @Size 등)에 따라 값의 유효성을 자동으로 검사한다.

- **동작 원리**

  @Valid가 붙은 객체의 각 필드에 선언된 제약 조건을 Bean Validator가 검사한다. 유효성 검증에 실패하면, Spring MVC에서는 **`MethodArgumentNotValidException`**이 발생하며, BindingResult를 함께 사용하면 예외를 직접 처리할 수 있다.

- **검증 대상**
    - @RequestBody, @ModelAttribute, @PathVariable, @RequestParam 등 다양한 방식으로 전달받은 객체에 적용 가능
    - 객체의 필드에 선언된 다양한 검증 어노테이션(@NotNull, @Size, @Pattern 등)과 함께 사용
- **대표적인 검증 어노테이션 예시**:
    - **`@NotNull`**: null 불가
    - **`@NotEmpty` :** null, “”, 빈 컬렉션 불가
    - **`@NotBlank`**: null, “”, “ “
    - **`@Size(min=, max=)`**: 문자열/배열의 길이 제한
    - **`@Pattern(regex=)`**: 정규식 패턴 만족 여부
    - **`@Min`**, **`@Max`**: 숫자 범위 제한
    - **`@Email`**: 이메일 형식 검사
    - **`@Past`**, **`@Future`**: 과거/미래 날짜 검사