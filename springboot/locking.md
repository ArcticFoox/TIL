# 트랜잭션과 동시성 제어 패턴

# ACID와 트랜잭션 동시성 제어

모든 DB 작업은 트랜잭션 내에서 진행

- Auto-commit mode: Statement마다 암묵적으로 트랜잭션 시작/커밋
- 명시적 트랜잭션: BEGIN~COMMIT/ROLLBACK으로 여러 Statement를 하나의 작업 단위로 묶음

## ACID

### **Atomicity**

트랜잭션 내 모든 작업이 전부 성공하거나 전부 실패(rollback)

실패 시 중간 변경은 모두 이전 상태로 복원(undo log 등 활용)

### **Consistency**

트랜잭션 전후로 데이터가 항상 유효한 상태(모든 제약조건, 무결성 규칙 준수)

하나라도 위반되면 전체 트랜잭션 롤백

### **Isolation**

여러 트랜잭션이 동시에 실행되어도, 각각은 다른 트랜잭션의 중간 상태를 볼 수 없음

Serializable이 가장 높은 격리 수준(완전 직렬 실행과 동등)이지만 성능 저하 있음

### **Durabliity**

커밋된 트랜잭션의 결과는 서버 다운, 장애 후에도 항상 보존됨

Redo log, Write-Ahead Log(WAL), 트랜잭션 로그 등 사용

## DBMS별 ACID 구현

### **Atomicity와 Rollback**

모든 변경은 실제 데이터 구조(테이블, 인덱스 등)에 적용되지만, 커밋 전에는 언제든지 undo log 등으로 이전 상태로 롤백 가능해야 함

Oracle: undo tablespace에서 이전 데이터 버전을 관리

SQL Server: 트랜잭션 로그에 undo/redo 정보 모두 기록

PostgreSQL: 데이터 자체에 다중 버전(MVCC)으로 이전 상태를 보관, VACUUM으로 공간 회수

MySQL(InnoDB): undo 로그와 purge 프로세스로 이전 상태 관리

### **Durability**

커밋 시점에 모든 변경이 Redo Log, WAL 등에 안전하게 기록되어야 함

Oracle: Redo 로그 버퍼를 Log Writer가 디스크로 flush

SQL Server: 트랜잭션 로그를 commit마다 디스크로 flush (2014+ 버전은 configurable durability)

PostgreSQL: WAL을 commit마다 flush(비동기 가능)

MySQL: Redo 로그(innodb_flush_log_at_trx_commit=1이 권장)

### **Isolation**

Oracle: Read Committed, Serializable만 제공(Snapshot Isolation 기반)

SQL Server: 2PL 기본, Snapshot Isolation은 별도 설정 필요

PostgreSQL: 기본적으로 MVCC, Repeatable Read=Snapshot Isolation

MySQL: InnoDB는 기본 Repeatable Read(MVCC), Serializable은 락 기반

## 동시성 제어 방식

### **2PL(Two-Phase Locking)**

모든 트랜잭션이 확장 단계 (락 획득), 수축 단계 (락 해제) 구분

순수 직렬화 보장, 성능 저하, 데드락 발생 가능

### **MVCC(Multi-Version Concurrency Control)**

읽기/쓰기 간 락 최소화, 트랜잭션 별 snapshot/버전 활용

writer끼리만 충돌, reader-writer/reader-reader는 충돌 없음

## 동시성 현상(Phenomena)

Dirty Write: 커밋 전 데이터가 다른 트랜잭션에 의해 덮어써짐

Dirty Read: 커밋되지 않은 데이터를 읽음

Non-Repeatable Read: 같은 레코드 두 번 읽었을 때 값이 다름

Phantom Read: WHERE 조건에 부합하는 레코드의 수가 트랜잭션 중간에 달라짐

Read Skew / Write Skew / Lost Update: MVCC에서만 발생, 여러 행/테이블의 변경이 동기화되지 않음

# 비관적(Pessimistic) Locking

## Locking의 종류와 개념

### **비관적 락(Pessimistic Lock)**

물리적 락

실제 DB의 행, 페이지, 테이블 등 데이터에 대해 락을 걸어 동시성 충돌을 방지

트랜잭션 격리 수준에 따라 자동 발생

명시적으로 사용하는 방법: 쿼리에서 직접 FOR UPDATE, FOR SHARE 등으로 지정

### **낙관적 락(Optimistic Lock)**

논리적 락

실제 DB 락을 쓰지 않고, 버전(@Version 필드)이나 타임스탬프 등으로 충돌 감지

@Version 등 기본기능으로 제공

명시적으로 사용하는 방법: LockModeType.OPTIMISTIC_FORCE_INCREMENT 등 사용

## DBMS별 명시적 락 SQL

### **공유 락(Shared/Read)**

Oracle: FOR UPDATE(실제로는 배타적 락만 지원)

MySQL: LOCK IN SHARE MODE

SQL Server: WITH(HOLDLOCK, ROWLOCK)

PostgreSQL: FOR SHARE

DB2: FOR READ ONLY WITH RS

```sql
<-- Oracle -->
SELECT * FROM employee WHERE department_id = 1 FOR UPDATE;

<-- MySQL -->
SELECT * FROM employee WHERE department_id = 1 LOCK IN SHARE MODE;

<-- SQL Server -->
SELECT * FROM employee WITH (HOLDLOCK, ROWLOCK) WHERE department_id = 1;

<-- PostgreSQL -->
SELECT * FROM employee WHERE department_id = 1 FOR SHARE;

<-- DB2 -->
SELECT * FROM employee WHERE department_id = 1 FOR READ ONLY WITH RS;
```

### **배타적 락(Exclusive/Write)**

Oracle/MySQL/PostgreSQL/DB2: FOR UPDATE

SQL Server: WITH(UPDLOCK, HOLDLOCK, ROWLOCK)

```sql
<-- Oracle, MySQL, PostgreSQL -->
SELECT * FROM employee WHERE department_id = 1 FOR UPDATE;

<-- SQL Server -->
SELECT * FROM employee WITH (UPDLOCK, HOLDLOCK, ROWLOCK) WHERE department_id = 1;

<-- DB2 -->
SELECT * FROM employee WHERE department_id = 1 FOR UPDATE WITH RS;
```

### **Hibernate/JPA 락 모드**

NONE: 락 없음(default 값)

OPTIMISTIC/READ: 버전 체크, 낙관적

OPTIMISTIC_FORCE_INCREMENT/WRITE: 버전 증가 + 체크

PESSIMISTIC_READ: 공유 락(다른 트랜잭션의 배타전 락, 쓰기 막음)

PESSIMISTIC_WRITE: 배타적 락(다른 모든 읽기/쓰기 막음)

PESSIMISTIC_FORCE_INCREMENT: 배타적 락 + 버전 증가

```sql
// Hibernate 예시

// PESSIMISTIC_READ
Product product = session.get(Product.class, 1L);
session.buildLockRequest(new LockOptions(LockMode.PESSIMISTIC_READ)).lock(product);
// → SELECT ... FOR SHARE

// PESSIMISTIC_WRITE
Product product = session.get(Product.class, 1L);
session.buildLockRequest(new LockOptions(LockMode.PESSIMISTIC_WRITE)).lock(product);
// → SELECT ... FOR UPDATE
```

## Predicate Locking과 Table Locking

### **Predicate Locking**

쿼리 조건에 해당하는 모든 레코드(혹은 인덱스의 범위)에 락을 거는 방식

MySQL(REPEATABLE READ), SQL Server, PostgreSQL 등 일부 DB에서 지원

MySQL은 REPEATABLE READ에서 갭 락 (Gap Lock)까지 적용, INSERT 차단 가능

### **Table Locking**

일부 DB(Oracle 등)에서 predicate lock 미지원 시 전체 테이블 락

→ i.g. LOCK TABLE … IN SHARE MODE

## FOR UPDATE(NO WAIT/SKIP LOCKED)

### **NO WAIT**

락 이미 잡혀 있으면 즉시 예외 발생(대기하지 않음)

지원하는 DB: Oracle, PostgreSQL, SQL Server, MySQL(8.0+)

Hibernate/JPA: setLockTimeout(LockOptions.NO_WAIT)

### **SKIP LOCKED**

이미 락 잡힌 행은 건너뛰고 나머지 행만 즉시 반환 (큐 처리 등에서 유용)

**지원하는 DB:** Oracle, PostgreSQL, SQL Server, MySQL(8.0+)

**사용 예시:** 여러 워커가 동시에 작업 큐에서 할당받을 때 프로세스/쓰레드 충돌 없이 동시에 안전하게 작업을 분배할 수 있음

- 현재 트랜잭션에서 아직 락이 걸리지 않은 (=다른 워커가 처리 중이 아닌) 행만 읽어서 락을 획득
- 이미 다른 워커가 락을 잡은 행은 "건너뛰고" 다음 행을 선택

```java
// Job Queue 에시
List<Post> posts = entityManager.createQuery(
    "select p from Post p where p.status = :status order by p.id", Post.class)
    .setParameter("status", status)
    .setMaxResults(postCount)
    .setLockMode(LockModeType.PESSIMISTIC_WRITE)
    .setHint("javax.persistence.lock.timeout", LockOptions.SKIP_LOCKED)
    .getResultList();
```

### **PostgreSQL Advisory Lock**

세션/트랜잭션 단위의 임의 key 기반 소프트 락

- 실제 행/테이블과 무관, 애플리케이션 레벨의 동시성 제어
- i.g. 특정 리소스별로 임계구역 동기화, 분산 환경에서 다중 노드 동기화 가능

```sql
SELECT pg_advisory_lock(12345);
-- 락 해제
SELECT pg_advisory_unlock(12345);
```

## 낙관적(Optimistic) Locking

## 낙관적 Locking 원리

실제 데이터베이스 락을 잡지 않고, 엔티티에 버전 혹은 타임스탬프 컬럼을 사용하여 동시성 충돌을 감지하는 방식

@Version 필드를 활용하면, 엔티티를 읽을 때 버전값을 함께 읽고, UPDATE/DELETE 시점에 WHERE 절에 버전값까지 명시

→ “id=… AND version= …”

누군가 이미 버전을 변경했다면, 해당 쿼리는 업데이트되지 않고 애플리케이션에서는 OptimisticLockException 예외가 발생

```java
@Entity
public class Post {
  
    @Id 
    private Long id;
  
    @Version 
    private int version;
  
    private String title;
}

// Update 시
// before: version=0
UPDATE post SET title='...', version=1 WHERE id=1 AND version=0
// 다른 트랜잭션이 먼저 커밋했다면 updateCount=0, OptimisticLockException 발생

// DELETE 시
DELETE FROM post WHERE id=1 AND version=1
```

## @Version vs Timestamp

### 타임스탬프 기반

DB/시스템 시간 동기화 문제(동기화, NTP, time zone, leap second 등)로 인해 실무에서 추천하지 않음

각 DBMS별로 정밀도/동작 차이 있음

### int/short 기반 버전 필드

대부분의 엔터프라이즈 환경에서 권장하는 방식

@Version 필드에 int 대신 short도 사용 가능, 대용량 테이블에서 스토리지 절약 효과

트랜잭션이 짧다면 short도 충분 → 하나의 레코드가 64,000번 이상 연속 변경되는 경우는 드묾

## Bulk Update시 유의사항

대량 UPDATE/DELETE 쿼리는 WHERE 절에 반드시 버전 조건을 포함해야 함

→ 동시성 충돌(lost update) 방지

```sql
UPDATE post SET status=?, version=version+1 WHERE status=? and version=?
```

## Versionless Optimistic Locking

Hibernate는 @Version 필드 없이도 @DynamicUpdate와 @OptimisticLocking(type=DIRTY)로 변경된 컬럼만 WHERE 조건에 포함하여 동시성 제어 가능

→ 단, 일반적인 상황에서는 버전 필드를 쓰는 것이 더 명확하고 안전

## 실무 팁

낙관적 Locking은 단일 트랜잭션 내 Lost Update 방지에는 유용한 방식

논리적 트랜잭션(여러 HTTP 요청/폼 제출 등)에도 버전 필드만 잘 사용하면 안정하게 동시성 충돌 감지 가능

대량 업데이트/삭제, 복잡한 집계(aggregate) 변경, 버전 컬럼 없는 엔티티 등은 별도 설계 및 조치 필요

- Aggregate 내부의 모든 변경을 부모 버전에 반영하려면 Hibernate 커스텀 이벤트 리스너, @OptimisticLocking(type = ALL/Dirty), Force Increment 등 고급 기능을 활용해야 함

출처

[https://jaimemin.tistory.com/2723](https://jaimemin.tistory.com/2723)