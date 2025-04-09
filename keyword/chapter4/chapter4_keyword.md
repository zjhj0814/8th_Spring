## [핵심 키워드]
# 프레임워크와 API의 차이
- `API(Application Programming Interface)`

  서로 다른 **소프트웨어** 컴포넌트 간의 **상호작용**을 가능하게 하는 인터페이스로, 내부 구현 세부사항을 추상화하여 사용자가 쉽게 사용할 수 있도록 한다.

- `라이브러리`

  **특정 작업을 수행**하기 위해 미리 작성된 코드의 집합으로, 개발자가 직접 라이브러를 호출하여 애플리케이션 흐름을 완전히 제어한다.

  ex. NumPy, Pandas…

- `프레임워크`

  **애플리케이션 개발을 위한 구조와 규칙을 제공**하는 재사용 가능한 컴포넌트 모음이다. 개발자가 특정 규칙에 따라 코드를 작성하도록 유도하며, 프레임워크는 기본적인 애플리케이션 흐름을 제어한다.

  ex. Angular, PHP, Tensorflow, Flutter…
***
# DI
`스프링 컨테이너`: 스프링 컨테이너에 객체를 스프링 빈으로 등록하면 스프링 컨테이너는 설정 정보를 참고해서 의존관계를 주입(DI)한다.

`스프링 빈` : 스프링 컨테이너에 등록된 객체.

[ 스프링 빈 저장소 예시 ]
| 빈 이름         | 빈 객체                     |
|----------------|----------------------------|
| memberService  | MemberServiceImpl@x01      |
| orderService   | OrderServiceImpl@x02       |

[ 스프링 컨테이너에서 스프링 빈을 찾는 코드 ]
```java
MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
```

[ 스프링 빈 등록 방법 ]
(1) `@Configuration`(설정 정보) + `@Bean`(의존관계 명시, 빈 등록)

스프링 컨테이너(BeanFactory 또는 ApplicationContext 인터페이스)는 @Configuration이 붙은 클래스를 설정 정보로 사용하여 @Bean이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다.  그 이후, 필요할 때마다 개발자는 스프링 컨테이너에서 스프링 빈을 찾아서 사용한다.
```java
@Configuration
public class AppConfig {
	@Bean
	public MemberService memberService(){
		return new MemberServiceImpl(memberRepository());
	}
	
	@Beab
	public MemberRepository memberRepository(){
		return new MemoryMemberRepository();
	}
}
```
(2) `@Component` (빈 등록) + `@Autowired` (의존관계 자동 주입)

설정 정보 없이 스프링 빈을 등록하는 컴포넌트 스캔 기능

`@Controller`, `@Service`, `@Repository`
```java
@Component
public class MemberServiceImpl implements MemberService {
	private final MemberRepository memberRepository;
	
	@Autowired
	public MemberServiceImpl(MemberRepository memberRepository){
		this.memberRepository = memberRepository;
	}
}
```

`DI(Dependency Injection)` : 하나의 객체에 다른 객체의 의존성을 제공하는 기술

[ 스프링 의존관계 자동 주입 방법 ]

1. 생성자 주입(⭕)
    - 생성자 호출 시점에 딱 1번만 호출되는 것이 보장된다.
    - **불변, 필수** 의존관계에 사용
    - 생성자가 한 개만 존재하면 @Autowired를 생략해도 자동 주입된다.

    ```java
    @Component
    public class MemberServiceImpl implements MemberService {
    	private final MemberRepository memberRepository;
    	
    	@Autowired
    	public MemberServiceImpl(MemberRepository memberRepository){
    		this.memberRepository = memberRepository;
    	}
    }
    ```

2. setter 주입
    - 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입한다.
    - **선택, 변경 가능성**이 있는 의존관계에 사용

    ```java
    @Component
    public class MemberServiceImpl implements MemberService {
    	private MemberRepository memberRepository;
    	
    	@Autowired
    	public void setMemberRepository(MemberRepository memberRepository){
    		this.memberRepository = memberRepository;
    	}
    }
    ```

3. 필드 주입 (❌)
    - 필드에 바로 주입하는 방법이다.
    - 외부에서 변경이 불가능해서 **테스트하기 힘들다.**

    ```java
    @Component
    public class MemberServiceImpl implements MemberService {
        @Autowired
    	private MemberRepository memberRepository;
    }
    ```


대부분의 의존관계는 애플리케이션 종료 전까지 변하면 안되고, 사용해선 안 되는 메서드를 public으로 열어두는 것은 위험하므로, 객체를 생성할 때 딱 1번 호출되는 **생성자 주입**을 사용하자!
***
# IoC
`IoC(Inversion of Control)`: 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하도록 하는 설계 원칙.

구현 객체는 자신의 로직을 실행하는 역할만 담당하고, 프로그램의 제어 흐름은 제어 객체가 담당한다.

💡[ IoC로 보는 프레임워크와 라이브러리 차이점 ]

- 프레임워크: 내가 작성한 코드를 제어하고, 대신 실행함
- 라이브러리: 내가 작성한 코드가 직접 제어 흐름을 조종함

`IoC 컨테이너` : 객체를 생성하고 관리하면서 의존관계를 연결해 주는 컨테이너로, 인스턴스 생성부터 소멸까지 생명주기를 컨테이너가 대신 한다
***
# 서블릿
`서블릿` : 클라이언트 요청을 처리하고 동적으로 응답을 생성하는 자바 프로그램.

[ 서블릿 특징 ]

- 클라이언트의 요청에 대해 동적으로 작동하는 웹 어플리케이션 컴포넌트.
- Java Thread를 이용하여 동작한다.
- MVC 패턴에서 Controller로 이용된다.

[ 서블릿 동작 방식 ]

- 사용자(클라이언트)가 URL을 입력하면 HTTP Request가 Servlet Container로 전송합니다.
- 요청을 전송받은 Servlet Container는 HttpServletRequest, HttpServletResponse 객체를 생성합니다.
- web.xml을 기반으로 사용자가 요청한 URL이 어느 서블릿에 대한 요청인지 찾습니다.
- 해당 서블릿에서 service메소드를 통해 클라이언트의 요청에 따라 doGet() 또는 doPost()를 호출합니다.
- doGet() or doPost() 메소드는 동적 페이지를 생성한 후 HttpServletResponse객체에 응답을 보냅니다.
- 응답이 끝나면 HttpServletRequest, HttpServletResponse 두 객체를 소멸시킵니다.

[ 서블릿 컨테이너 ]

서블릿을 관리하는 컨테이너.

1. 웹 서버와 통신 지원

   클라이언트의 요청을 받아주고 응답할 수 있게 웹서버와 통신한다.

2. 서블릿 생명주기 관리

   서블릿 클래스를 로딩하여 인스턴스화하고, 초기화 메소드를 호출하고, 요청이 들어오면 적절한 서블릿 메서드를 호출한다. 이때, 서블릿 객체를 종료하지 말고(서블릿 객체를 싱글톤으로 관리) 다른 요청에 대해서도 서블릿 객체를 재활용한 후, 서블릿 컨테이너가 종료될 때, 서블릿도 함께 종료시킨다.

3. 동시 요청을 위한 멀티 쓰레드 처리 지원
***
# AOP
`AOP(Aspect Oriented Programming)`: 공통 관심 사항과 핵심 관심 사항을 분리시키고 각각 모듈화하는 것을 의미한다. 이에 따라 가독성, 유지보수성을 높일 수 있다.

- Spring AOP 동작 원리

  Spring의 AOP는 기본적으로 프록시 방식으로 동작한다. ~~자세한 동작 원리는 더 공부해 봐야겠다.~~ 

- Spring AOP 동작 예시

  [ AOP 적용 전 스프링 컨테이너 ]

  memberController → memberService → memberRepository

  [ AOP 적용 후 스프링 컨테이너 ]

  프록시memberController → memberController
  → 프록시 memberService → memberService
  →  프록시 memberRepository → memberRepository
***
## [실습]
![Image](https://github.com/user-attachments/assets/9bb1bec2-8f3a-4c57-b055-d865f09a8e97)