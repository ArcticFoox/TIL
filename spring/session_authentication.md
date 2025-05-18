# Session Authentication

![image.png](/spring/img/image2.png)

세션 저장소는 일반적으로 외부 서버나 데이터베이스에 위치

## Java로 구현

### 로그인에 성공한 경우

```java
@DisplayName("로그인 성공 시 세션 아이디를 쿠키로 전달 받는다.")
@Test
void sessionLogin() {
    saveMemberAsAnna();

    String credentials = "anna@email.com" + ":" + "1234";
    String encoded = Base64.getEncoder().encodeToString(credentials.getBytes());
    String sessionId = RestAssured.given()
            .header("Authorization", "Basic " + encoded)
            .when().post("/login/session")
            .then().statusCode(200)
            .extract().cookie("sessionId");
    assertThat(sessionId).isNotNull();
}
```

서버는 이 인코딩 정보를 통해 세션 저장소에 정보를 저장하고 식별 값을 발급

```java
@PostMapping("/login/session")
public ResponseEntity<Void> loginSession(
        @RequestHeader("Authorization") String encodedCredentials,
        HttpServletRequest request,
        HttpServletResponse response
) {
    String encoded = encodedCredentials.split(" ")[1];
    String credentials = new String(Base64.getDecoder().decode(encoded));
    String[] emailAndPassword = credentials.split(":");

    Member member = authService.findByEmailAndPassword(new LoginRequest(
            emailAndPassword[0], emailAndPassword[1]
    ));

    HttpSession session = request.getSession(true);
    session.setAttribute("email", member.getEmail());
    session.setMaxInactiveInterval(3600);
    String sessionId = session.getId();
    sessionStorage.addSession(sessionId, session);

    response.addCookie(new Cookie("sessionId", sessionId));
    return ResponseEntity.ok().build();
}
```

세션 저장소의 경우 메모리에 저장하는 방식

```java
@Component
public class SessionStorage {
    private final Map<String, HttpSession> sessions = new HashMap<>();

    public void addSession(String sessionId, HttpSession session) {
        sessions.put(sessionId, session);
    }
}
```

### 세션 아이디를 통해 자원을 요청하는 경우

요청할 때 발급 받는 세션 아이디를 쿠키에 담아 보냄

```java
@DisplayName("세션 아이디를 쿠키에 담아 데이터를 요청하면 응답을 받는다.")
@Test
void getMemberBySessionId() {
    saveMemberAsAnna();

    String credentials = "anna@email.com" + ":" + "1234";
    String encoded = Base64.getEncoder().encodeToString(credentials.getBytes());
    String sessionId = RestAssured.given()
            .header("Authorization", "Basic " + encoded)
            .when().post("/login/session")
            .then().statusCode(200)
            .extract().cookie("sessionId");

    RestAssured.given()
            .cookie("sessionId", sessionId)
            .when().get("/login/member/session")
            .then().statusCode(200);
}
```

인터셉터에서 요청을 가로채 쿠키 값을 토대로 세션 정보가 있다면 검증에 성공하는 로직 생성

```java
@Component
public class LoginCheckInterceptor implements HandlerInterceptor {

    private final SessionStorage sessionStorage;
    private final MemberService memberService;

    public LoginCheckInterceptor(SessionStorage sessionStorage, MemberService memberService) {
        this.sessionStorage = sessionStorage;
        this.memberService = memberService;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        // 세션 아이디가 입력되지 않은 경우 예외 처리
        String sessionId = Arrays.stream(request.getCookies())
                .filter(cookie -> cookie.equals("sessionId"))
                .map(Cookie::getValue)
                .findAny()
                .orElseThrow(() -> new SecurityException("세션 정보를 입력해주세요."));
        return true;
    }
}
```

```java
@Component
public class SessionStorage {
    private final Map<String, HttpSession> sessions = new HashMap<>();

    public void addSession(String sessionId, HttpSession session) {
        sessions.put(sessionId, session);
    }

    public String getEmailBySessionId(String sessionId) {
        return (String) sessions.get(sessionId).getAttribute("email");
    }

    public boolean doesNotContains(String sessionId) {
        return sessions.containsKey(sessionId);
    }
}
```

### 잘못된 세션 아이디를 통해 자원을 요청하는 경우

```java
@DisplayName("잘못된 세션 아이디를 쿠키에 담아 요청 시 401 Unauthorized 응답을 받는다.")
@Test
void failGetMemberBySessionIdWhenIncorrectSessionId() {
    saveMemberAsAnna();

    RestAssured.given().log().all()
            .cookie("sessionId", "wrongSessionId")
            .when().log().all().get("/login/member/session")
            .then().statusCode(401);
}
```

임의로 세션 아이디를 지정해 요청을 보내는 경우 오류를 반환

인터셉터에서 세션 저장소로부터 세션 아이디를 조회 후 없는 아이디라면 예외를 반환

비즈니스 로직에 따라 세션 저장소를 활용해 다양한 예외 처리 가능

```java
@Component
public class LoginCheckInterceptor implements HandlerInterceptor {

    private final SessionStorage sessionStorage;
    private final MemberService memberService;

    public LoginCheckInterceptor(SessionStorage sessionStorage, MemberService memberService) {
        this.sessionStorage = sessionStorage;
        this.memberService = memberService;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        // 세션 아이디가 입력되지 않은 경우 예외 처리
        String sessionId = Arrays.stream(request.getCookies())
                .filter(cookie -> cookie.getName().equals("sessionId"))
                .map(Cookie::getValue)
                .findAny()
                .orElseThrow(() -> new SecurityException("세션 정보를 입력해주세요."));

        // 세션 아이디가 잘못된 경우 예외 처리
        if (sessionStorage.doesNotContains(sessionId)) {
            throw new SecurityException("존재하지 않은 세션 정보입니다.");
        }
        // 세션 아이디에 해당하는 이메일을 가진 회원이 없는 경우 예외 처리
        String email = sessionStorage.getEmailBySessionId(sessionId);
        memberService.findByEmail(email);

        return true;
    }
}
```

참조

[https://mingyum119.tistory.com/325](https://mingyum119.tistory.com/325)