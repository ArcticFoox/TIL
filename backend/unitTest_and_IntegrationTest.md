# 단위 테스트 & 통합 테스트

# 단위 테스트

하나의 함수, 메서드, 클래스와 같은 가장 작은 단위의 코드가 기대한 대로 동작하는지 검증하는 테스트

## 목적

개별 로직 검증

리팩토링 시 안전망 제공

개발 초기에 빠른 피드백 제공

## 특징

| 항목 | 설명 |
| --- | --- |
| 테스트 범위 | 매우 작음 (하나의 클래스나 함수) |
| 외부 의존성 | 대부분 제거 (Mock 객체 사용) |
| 실행 속도 | 매우 빠름 |
| 실패 원인 | 대부분 로직 자체의 오류 |
| 도구 | JUnit, Mockito, AssertJ (Java 기준) |

```java
class Calculator {
    int add(int a, int b) {
        return a + b;
    }
}

@Test
void testAdd() {
    Calculator calculator = new Calculator();
    assertEquals(5, calculator.add(2, 3));
}

```

# 통합테스트

여러 컴포넌트(모듈, 클래스, 시스템)가 서로 연동되어 정상 작동하는지 검증하는 테스트

## 목적

실제 환경과 유사한 조건에서 동작 검증

컴포넌트 간 통신/호환성 확인

DB, 외부 API, 메시지 큐 등과의 연동 테스트

## 특징

| 항목 | 설명 |
| --- | --- |
| 테스트 범위 | 넓음 (여러 클래스/모듈) |
| 외부 의존성 | 존재 (실제 DB, 서버, 메시지 브로커 등 사용) |
| 실행 속도 | 느림 |
| 실패 원인 | 구성 요소 간 연동 문제, 설정 오류 등 |
| 도구 | SpringBootTest, TestContainers, Embedded DB 등 |

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void testGetUser() throws Exception {
        mockMvc.perform(get("/users/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("Alice"));
    }
}

```

# 비교

| 항목 | 단위 테스트 | 통합 테스트 |
| --- | --- | --- |
| 테스트 대상 | 메서드/클래스 하나 | 여러 컴포넌트 연동 |
| 외부 의존성 | 없음 (Mock 사용) | 있음 (DB, API 등) |
| 실행 속도 | 빠름 | 느림 |
| 테스트 환경 | 개발 환경과 유사 | 실제 환경과 유사 |
| 실패 원인 | 로직 오류 | 연동 오류, 환경 설정 문제 |