## [미션]
# 1. 위의 링크, 그리고 워크북을 보며 API 응답 통일과 에러 핸들러 숙지하기
***
# 2. 반드시 본인 손으로 처음부터 끝까지 다 해보고 새 리포지토리 혹은 7주차 리포지토리에 새 브랜치에 push 후 해당 링크를 미션 기록지에 제출할 것
***
# 3. 미션 진행 시 반드시 중간 중간 **과정 인증샷**을 남길 것
![Image](https://github.com/user-attachments/assets/5e302dee-13ad-41b0-aaf6-07caff1e1475)
![Image](https://github.com/user-attachments/assets/1549b449-d90b-4792-8a05-5eb763bf5911)
***
# 4. ❗**필수**❗ ****RestControllerAdvice의 장점, 그리고 없을 경우 어떤 점이 불편한지도 조사하여 **미션 기록란**에 수록할 것
# RestControllerAdvice
`@ControllerAdvice`: 여러 컨트롤러에 대해 전역적으로 ExceptionHandler를 적용해 준다.

`@RestControllerAdvice`: @ControllerAdvice와 달리 **@ResponseBody가 붙어 있어 응답을 Json으로 내려준다.**

잘못된 URI를 호출하여 발생하는 NoHandlerFoundException 등 스프링 예외를 미리 처리해 둔 `ResponseEntityExceptionHandler`를 추상 클래스로 제공하고 있다. 따라서 ControllerAdvice 클래스가 이 추상 클래스를 상속받아 @Override 하면 된다.

- 하나의 클래스로 모든 컨트롤러에 대해 **전역적으로** 예외 처리가 가능하다.
- 직접 정의한 에러 응답을 **일관성 있게** 클라이언트에게 내려 줄 수 있다.
- 별도의 try-catch문이 없어 코드의 **가독성**이 높아진다.
***
# 5. ~~❗**필수**❗ **미션 목록 조회(진행중, 진행 완료) API 명세서** 작성하기 (이미 작성되어 있으면 상관 없음!)~~