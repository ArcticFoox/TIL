# Custom Exception

## Always Provide a Benefit

JDK가 제공하고 있는 예외들과 비교했을 때,

커스텀 예외가 더 나은 점이 없다면 표준 예외 중 하나를 사용하는 것이 좋다.

## Follow the Naming Convention

JDK가 제공하는 예외 클래스들 모두 Exception으로 끝남

만들고자 하는 커스텀 예외 클래스도 이 네이밍 규칙을 따르는 것이 좋다.

## Provide JavaDoc Comment for Your Exception Class

많은 커스텀 예외들이 JavaDoc 코멘트 조차 없이 만들어 진다.

기본적으로 API의 모든 클래스, 멤버 변수, 생성자들에 대해서 문서화하는 것이 좋다.

클라이언드와 직접 관련된 메서드 중 하나가 예외를 던지면, 그 예외는 API 중 하나가 된다.

JavaDoc에는 예외가 발생할 수 있는 상황과 예외의 전반적인 의미를 기술

다른 개발자들이 API를 이해하고 일반적인 에러 상황을 피하도록 도와야 함

## Provide a Constructor That Sets the Cause

커스텀 예외를 사용할 때 자주 사용하는 형식은 표준 예외를 캐치해서 커스텀 예외로 전환해서 던지는 것

```java
public class ExceptionEx {
		public static void main(String[] args){
				try {
						//do Something
				} catch(RuntimeException e){
						throw new MyCustomException("발생", e);
				}
		}
}

class MyCustomException extends RuntimeException{
		public MyCustomException(){
		}
		
		public MyCustomException(String message, Throwable cause){
				super(message, cause);
		}
}
```

캐치된 예외에는 발생한 오류를 분석하는데 필요한 정보가 포함되어 있음

```java
public void wrapException(String input) throws MyBusinessException {
    try {
        // do something
    } catch (NumberFormatException e) {
        throw new MyBusinessException("A message that describes the error.",
         e, ErrorCode.INVALID_PORT_CONFIGURATION);
    }
}
```

NumberFormatException은 에러에 대한 상세 정보 제공

MyBusinessException을 생성할 때 cause 정보를 설정하지 않으면 이전에 발생시켰던 에러 정보 손실

Exception과 RuntimeException은 예외의 원인을 기술하고 있는,

Throwable을 받을 수 있는 생성자 메서드 제공

커스텀 예외 또한 생성자에서 예외의 원인을 인자로 받는 것이 좋음

발생한 Throwable을 파라미터를 통해 가져올 수 있는 생성자를 최소한 하나 구현하고,

super 생성자에 Throwable를 전달해줘야 함

```java
public class MyBusinessException extends Exception {
    public MyBusinessException(String message, Throwable cause, ErrorCode code) {
            super(message, cause);
            this.code = code;
        }
        ...
}
```

출처

[https://parkadd.tistory.com/69](https://parkadd.tistory.com/69)