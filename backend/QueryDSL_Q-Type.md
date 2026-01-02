# Q-Type
엔티티를 컴파일 타임에 분석해서 만든 쿼리 전용 메타 클래스
```java
@Entity
public class Member {
    @Id Long id;
    String name;
    int age;
}
```
컴파일 타임에 Q-Type 생성
```java
public class QMember extends EntityPathBase<Member> {
    public static final QMember member = new QMember("member");

    public final NumberPath<Long> id = createNumber("id", Long.class);
    public final StringPath name = createString("name");
    public final NumberPath<Integer> age = createNumber("age", Integer.class);
}
```
- 엔티티 필드 1:1 대응
- 타입 정보 포함
- QueryDSL 내부 DSL이 메타 정보 저장

## Q-Type 필요성
**JPQL / Native Query 문제점**
```java
"where m.name = :name"
```
- 오타 -> 런타임 에러
- 리팩토링 -> 전수 테스트 필요
- IDE 지원 미약
```java
member.name.eq("kim")
```
- `name` 없으면 컴파일 에러
- 타입 불일치도 컴파일 에러
- IDE 자동완성 100%

## 내부 구조
```java
public class QMember extends EntityPathBase<Member> {
    public static final QMember member = new QMember("member");

    public final NumberPath<Long> id = createNumber("id", Long.class);
    public final StringPath name = createString("name");
    public final NumberPath<Integer> age = createNumber("age", Integer.class);
}
```
### Path
- 쿼리 AST(Abstract Syntax Tree) 노드
- SQL / JPQL 생성용 메타 데이터
ex)
```java
member.age.gt(20)
// 내부적으로
NumberPath(age) > Constant(20)
```
### Path 종류
| 엔티티 필드        | Q-Type             |
| ------------- | ------------------ |
| String        | StringPath         |
| int, Integer  | NumberPath         |
| boolean       | BooleanPath        |
| LocalDateTime | DateTimePath       |
| Enum          | EnumPath           |
| @ManyToOne    | Q엔티티               |
| @OneToMany    | ListPath / SetPath |

```java
@ManyToOne
Team team;
public final QTeam team = new QTeam("team");

// 사용 (조인 가능한 이유)
member.team.name.eq("dev")
```
## Q-Type 생성 과정
### 생성 주체
- Annotation Processing (APT)
- `querydsl-apt` 모듈이 처리
```scss
javac
 └─ annotation processor
     └─ @Entity 분석
         └─ Q 클래스 생성
```
### Gradle 기준 생성 위치
```bash
build/generated/querydsl
```
IDE에서 이 경로가 Source Root로 등록되어야 함 (안 되면 `QMember cannot be resolved` 발생)
