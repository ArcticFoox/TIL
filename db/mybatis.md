# mybatis

MyBatis는 **Java에서 SQL 기반의 데이터베이스와 쉽게 연동할 수 있게 도와주는 퍼시스턴스 프레임워크** 

JPA(Hibernate)와 같은 ORM(Object Relational Mapping) 프레임워크와는 다르게, 

SQL을 직접 작성할 수 있도록 하여 **개발자에게 SQL 제어권을 부여하는 특징을 가짐**

## MyBatis의 핵심 개념

### **SQL Mapper 프레임워크**

MyBatis는 XML 또는 애너테이션을 사용해 SQL을 명시적으로 작성

SQL과 Java 객체를 연결(Mapping)하는 데에 집중

```xml
<!-- XML 기반 SQL 매핑 예시 -->
<select id="selectUser" parameterType="int" resultType="User">
  SELECT * FROM users WHERE id = #{id}
</select>
```

### **객체-관계 매핑(O/R Mapping)**

DB에서 조회된 결과를 Java 객체로 변환 (역직렬화)

insert/update 시 객체를 파라미터로 넘기면 SQL에 맞게 매핑

```java
User user = userMapper.selectUser(1);
System.out.println(user.getName());
```

## MyBatis 주요 구성 요소

| 구성 요소 | 설명 |
| --- | --- |
| `SqlSessionFactory` | MyBatis 설정 및 DB 커넥션 관리를 담당하는 팩토리 객체 |
| `SqlSession` | 실제 쿼리 실행의 중심 객체 (`selectOne`, `insert`, `update`, `delete`) |
| `Mapper` | 인터페이스와 XML 매핑 파일로 구성된 DAO 인터페이스 |
| `Mapper XML` | SQL을 포함하는 XML 매퍼 파일 (예: `UserMapper.xml`) |
| `Configuration` | mybatis-config.xml을 통해 전반적인 설정 관리 |

## 사용 흐름

**설정 파일 작성**

```xml
<!-- mybatis-config.xml -->
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/test"/>
        <property name="username" value="root"/>
        <property name="password" value="1234"/>
      </dataSource>
    </environment>
  </environments>

  <mappers>
    <mapper resource="mapper/UserMapper.xml"/>
  </mappers>
</configuration>
```

**매퍼 인터페이스**

```java
public interface UserMapper {
    User selectUser(int id);
    void insertUser(User user);
}
```

**XML Mapper**

```xml
<mapper namespace="UserMapper">
  <select id="selectUser" parameterType="int" resultType="User">
    SELECT * FROM users WHERE id = #{id}
  </select>

  <insert id="insertUser" parameterType="User">
    INSERT INTO users(name, email) VALUES(#{name}, #{email})
  </insert>
</mapper>
```

**사용 예시**

```java
SqlSession session = sqlSessionFactory.openSession();
UserMapper mapper = session.getMapper(UserMapper.class);
User user = mapper.selectUser(1);
```