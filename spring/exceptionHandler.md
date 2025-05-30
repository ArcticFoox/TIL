# @ExceptionHandler

Spring MVC에서 컨트롤러나 전역 예외 처리를 위한 @ControllerAdvice 클래스의 메서드에서 발생하는 예외를 처리하는데 사용

특정 예외를 처리하는 메서드를 지정하거나 메서드의 파라미터로 처리할 예외를 설정 가능

Spring MVC 애플리케이션에서 예외 발생 시,

DispatchterServlet이 적절한 HandlerExceptionResolver를 찾아 예외 처리

Spring에 기본적으로 등록된 HandlerExceptionResolver는 세 가지 존재

- ExceptionHandlerExceptionResolver : @ExceptionHandler 등록 예외 처리
- ResponseStatusExceptionResolver : 예외 발생 시 HTTP 상태 코드 설정
- DefaultHandlerExceptionResolver : Spring MVC가 제공하는 기본 예외 처리

<br>
위에서 부터 차례대로 우선순위

<br>
ExceptionHandlerExceptionResolver의 특징으로 예외가 WAS로 던져지지 않고 직접 처리

→ 사용자에게 친화적인 에러 메시지를 제공하거나 로깅 등의 추가 작업 수행 가능