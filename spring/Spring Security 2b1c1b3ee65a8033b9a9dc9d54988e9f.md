# Spring Security

## 전체과정

![image.png](/spring/img/secuimage.png)

![image.png](/spring/img/secuimage1.png)

### Request

사용자가 로그인 정보를 요청

AuthenticationFilter 는 사용자의 세션 ID(JSESSIONID)가 Security Context에 있는지 확인

없으면 다음 로직 수행

### 토큰 생성

사용자가 보낸 아이디와 비밀번호를 AuthenticationFilter가 받아서 UsernamePasswordAuthenticationToken 을 생성

생성된 UsernamePasswordAuthenticationToken 는 AuthenticationManager 의 인증 메서드를 호출하는 데 사용

AuthenticationManager 는 단순한 인터페이스이며 실제 구현은 ProviderManager 가 수행

ProviderManager 에는 사용자 요청 인증에 필요한 AuthenticationProvider 목록이 존재

ProviderManager 는 제공된 각 AuthenticationProvider를 살펴보고 전달된 UsernamePasswordAuthenticationToken 를 기반으로 사용자 인증 시도

→ 토큰을 처리할 수 있는 AuthenticationProvider 선택

![image.png](/spring/img/secuimage2.png)

AuthenticationProvider 는 사용자의 이름(username)을 기반으로 사용자의 세부정보를 검색하기 위해 UserDetailsService 를 사용할 수 있음

AuthenticationProvider 인터페이스에는 authenticate() 메서드를 오버라이딩해 인증용 객체를 파라미터로 받아 로그인 페이지에서 입력한 사용자 정보를 들고 올 수 있음

UserDetailsService 는 DB에 저장된 회원의 비밀번호와 입력한 비밀번호가 일치하면 UserDetails 인터페이스를 구현한 객체를 반환

AuthenticationProvider 인터페이스에 의해 사용자가 성공적으로 인증되면, 완전히 채워진 인증개체가 반환 실패 시 AuthenticationException 발생

인증 메커니즘 지원하는 AuthenticationEntryPoint에 의해 처리

AuthenticationManager는 획득한 완전히 채워진 인증개체를 관련 인증 필터(AuthenticationFilter)로 다시 반환