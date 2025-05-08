# SQL 인젝션

## 공격 방법

### 클래식 SQL 인젝션

사용자 입력값을 그대로 SQL 쿼리에 삽입하여 악의적인 SQL 코드를 실행하는 기본적인 공격 방법

```sql
SELECT * FROM users WHERE username = '[입력값]' AND password = '[입력값]';

// admin'; -- 입력시
SELECT * FROM users WHERE username = 'admin'; --' AND password = '[입력값]';
```

password 조건이 주석처리되어 비밀번호 검증 없이 액세스 가능

### 블라인드 SQL 인젝션

공격자가 데이터베이스의 데이터를 직접 조회하지 않고, 참/거짓의 결과를 통해 정보를 추출하는 방법

```sql
admin' and (실행할 쿼리문) and 1 = 1 (참)
```

### UNION 기반 SQL 인젝션

UNION 연산자를 이용해 원래 쿼리 결과에 추가적인 선택 결과를 결합하는 방식

```sql
(실행 쿼리) UNION SELECT username, password FROM users
```

### 에러 기반 SQL 인젝션

데이터베이스가 에러 메시지를 반환할 때, 그 메시지를 통해 데이터베이스의 정보를 추출하는 방법

CAST 함수와 같은 함수를 사용하여 강제적으로 에러 발생, 해당 에러 메시지에서 데이터베이스 정보 추출

### Out-of-band SQL 인젝션

데이터베이스 서버가 직접 외부와 통실할 수 있을 때 사용되는 방법

데이터베이스 함수를 이용하여 공격자의 서버로 특정 데이터를 전송

### 스택쿼리 SQL 인젝션

하나의 SQL 문장 뒤에 추가적인 SQL 문장을 넣어서 실행하는 방식

세미콜론(;)을 사용하여 기존 쿼리 뒤에 DROP TABLE users; 와 같은 악의적인 쿼리 추가

## 예방법

### input 값을 받을 때, 특수문자 여부 검사

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class InputValidation {

    private static final String EMAIL_PATTERN = 
        "^[A-Za-z0-9+_.-]+@([A-Za-z0-9-]+\\.)+[A-Za-z]{2,6}$";

    private Pattern pattern;
    private Matcher matcher;

    public InputValidation() {
        pattern = Pattern.compile(EMAIL_PATTERN);
    }

    public boolean validateEmail(String email) {
        matcher = pattern.matcher(email);
        return matcher.matches();
    }

    public static void main(String[] args) {
        InputValidation validator = new InputValidation();

        String email = "user@example.com";
        boolean isValid = validator.validateEmail(email);

        if (isValid) {
            System.out.println("Email is valid.");
        } else {
            System.out.println("Email is invalid.");
        }
    }
}
```

로그인 전, 검증 로직을 통해 미리 차단

### SQL 서버 오류 발생 시, 해당 에러 메시지 숨김

```sql
-- 원본 테이블 예시: Users
CREATE TABLE Users (
    UserID INT PRIMARY KEY,
    UserName VARCHAR(100),
    UserEmail VARCHAR(100)
);

-- 뷰 생성
CREATE VIEW PublicUsers AS
SELECT UserName, UserEmail
FROM Users;

-- 사용자는 이 뷰를 통해서만 데이터를 조회
SELECT * FROM PublicUsers;
```

view를 활용해 테이블 접근 권한을 높임

### preparedStatement 사용

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class DatabaseExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/yourdatabase";
        String user = "username";
        String password = "password";

        try (Connection con = DriverManager.getConnection(url, user, password)) {
            String query = "INSERT INTO Users (UserName, UserEmail) VALUES (?, ?)";

            try (PreparedStatement pst = con.prepareStatement(query)) {
                pst.setString(1, "John Doe");
                pst.setString(2, "john@example.com");

                pst.executeUpdate();
            }
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
    }
}
```

특수문자를 자동으로 escaping 해줌

### JPA 사용

Hibernate는 항상 PreparedStatement 사용

[흥미있는 실험 내용](https://velog.io/@baekgwa/JPA%EB%A5%BC-%EC%93%B0%EB%A9%B4-SQL-%EC%9D%B8%EC%A0%9D%EC%85%98-%EA%B3%B5%EA%B2%A9%EC%9D%84-%EB%B0%A9%EC%96%B4%ED%95%A0%EA%B9%8C)

원글

[https://velog.io/@k4minseung/DB-SQL-Injection-공격과-방어-방법](https://velog.io/@k4minseung/DB-SQL-Injection-%EA%B3%B5%EA%B2%A9%EA%B3%BC-%EB%B0%A9%EC%96%B4-%EB%B0%A9%EB%B2%95)