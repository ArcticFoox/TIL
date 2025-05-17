# HTTP Authentication & HTTP Authorization

HTTP는 기본적으로 Basic Authentication 인증 방식을 제공

HTTP 헤더를 통해 인증 방식에 대한 정보를 클라이언트에게 제공하고,

클라이언트는 인증 정보를 HTTP 헤더에 담아 서버에 전송

![image.png](/spring/img/image.png)

## Java 코드

### 인증 정보가 없는 경우

```java
@DisplayName("인증 정보 없이 요청하면 서버에서 401 unauthorized를 반환한다.")
@Test
void requestWithoutAuthorization() {
    RestAssured.given()
            .when().get("/login/member")
            .then()
            .statusCode(401)
            .header("WWW-Authenticate", "Basic realm=\"dev server\"");
}
```

자원 요청을 보냈을 때 인증 정보가 없으면 401 응답

인터셉트를 사용해 등록

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    private final LoginCheckInterceptor loginCheckInterceptor;

    public WebConfig(
            LoginCheckInterceptor loginCheckInterceptor,
    ) {
        this.loginCheckInterceptor = loginCheckInterceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginCheckInterceptor)
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/", "/signup", "/login", "/logout", "/css/**", "/*.ico", "/error", "/js/**",
                        "/docs/**");
    }
}
```

인터셉터에서는 헤더에 인증 정보가 없다면 진입을 막음

Authorization 헤더에 Basic으로 시작하는 문자열이 있다면 통과

### 올바른 인증 정보를 보낸 경우

클라이언트에서 올바른 아이디와 비밀번호를 인코딩한 정보를 헤더에 담아 자원 요청 전송

```java
@RestController
public class BasicAuthController {
private final AuthService authService;

		public BasicAuthController(AuthService authService) {
		    this.authService = authService;
		}
		
		@PostMapping("/login/basic")
		public ResponseEntity<Void> loginBasic(@RequestHeader("Authorization") String encodedCredentials) {
		    String encoded = encodedCredentials.split(" ")[1];
		    String credentials = new String(Base64.getDecoder().decode(encoded));
		    String[] emailAndPassword = credentials.split(":");
		
		    authService.findByEmailAndPassword(new LoginRequest(
		            emailAndPassword[0], emailAndPassword[1]
		    ));
		    return ResponseEntity.ok().build();
		}
}
```

자원 요청을 받은 서버는 문자열을 디코딩하여 아이디와 비밀번호를 추출하고 데이터베이스에서 회원을 조회

올바른 아이디와 비밀번호인 경우 성공 응답 전송

### 올바르지 않은 아이디와 비밀번호인 경우

```java
@DisplayName("올바르지 않은 아이디와 비밀번호를 인코딩하여 전송하면 로그인에 실패한다.")
@Test
void loginFailure() {
    String credentials = "anna@email.com" + ":" + "12345";
    String encoded = Base64.getEncoder().encodeToString(credentials.getBytes());
    RestAssured.given()
            .header("Authorization", "Basic " + encoded)
            .when().post("/login/basic")
            .then().statusCode(401);
}
```

올바르지 않은 비밀번호 입력 시 401 응답

```java
public class AuthService {
    private final MemberRepository memberRepository; 
    
    public AuthService(MemberRepository memberRepository) {
    	this.memberRepository = memberRepository;
    }
	
    public Member findByEmailAndPassword(LoginRequest loginRequest) {
        return memberRepository.findFirstByEmailAndPassword(loginRequest.email(), loginRequest.password())
                .orElseThrow(() -> new SecurityException(HttpStatusCode.valueOf(401), "일치하지 않는 이메일 또는 비밀번호입니다."));
    }   
}

```

요청한 아이디와 비밀번호를 데이터베이스에서 조회하여 일치하지 않으면 예외 발생

예외 클래스를 핸들링하여 오류를 포함한 응답을 클라이언트에게 전달

```java
@ExceptionHandler(SecurityException.class)
    public ResponseEntity<SecurityErrorResponse> paymentExHandler(SecurityException exception) {
        logger.error(exception.getMessage(), exception);
        return ResponseEntity.status(exception.getStatusCode())
                .body(new SecurityErrorResponse(exception.getMessage()));
    }
```

참고

[https://mingyum119.tistory.com/324](https://mingyum119.tistory.com/324)