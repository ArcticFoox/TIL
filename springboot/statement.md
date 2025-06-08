# Statement

## 생애주기와 실행 계획

### Statement와 PreparedStatement의 실행 단계

Statement는 SQL을 문자열로 직접 실행하며 매번 파싱과 실행 계획 수립 필요

PreparedStatement는 아래 순서로 동작

- SQL을 미리 컴파일
- 바인드 변수로 파라미터화
- 여러 번 실행 시 실행 계획을 재사용하여 성능 향상
- 서버 측 Prepare/Execute로 동작

### 실행 단계

Parsing: SQL 문법/구문 오류 체크, 내부 파싱 트리 생성

Optimizer: SQL의 실행 계획(Execution Plan)을 생성하며 인덱스 선택, 조인 순서/알고리즘, 통계 기반 비용 산정

Executor: 실제 실행 계획에 따라 DB에서 데이터를 읽고 결과 생성

→ PreparedStatement는 실행 계획 캐싱 및 바인딩 값 적용

### 실행 계획 얻는 방법 DB별 예시

각 DB는 실행 계획을 시각적 또는 표 혹은 JSON 형태로 제공

실행 계획 해석 시 주요 정보

- 액세스 방식(인덱스 사용 여부, 풀 스캔, 조인 방식 등)
- 예상 비용 / 실행 시간 / 행 수
- 실제 행 수(ANALYZE 사용 시)

```sql
// Oracle
EXPLAIN PLAN FOR SELECT ...;
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY());

// SQL Server
SET SHOWPLAN_ALL ON;
SELECT ...;
SET SHOWPLAN_ALL OFF;

// SQL Server - 2
SET STATISTICS PROFILE ON;
SELECT ...;
SET STATISTICS PROFILE OFF;

// PostgreSQL
EXPLAIN ANALYZE SELECT ...;

// MySQL
EXPLAIN [EXTENDED|FORMAT=JSON] SELECT ...;
```

### Optimizer와 실행 계획의 역할

SQL은 ‘**무엇을**’ 할지 정의

‘**어떻게**’ 수행하지는 DBMS의 Optimizer가 판단 → 최적의 실행 계획

Optimizer는 파싱 트리를 기반으로 다양한 접근 경로와 조인 알고리즘을 평가

- 비용 기반 의사결정
- 테이블 / 인덱스 통계, 카디널리티, IO / CPU 비용 등

실행 계획은 DBMS가 정해진 시간/자원 예산 내에서 ‘충분히 좋은’ 플랜을 찾도록 설계

### 실행 계획 캐싱과 바인딩 변수

PreparedStatement와 실행 계획 캐싱 시

- 파싱 / 컴파일 / 실행 계획 생성 후 캐시에 저장되어 재사용됨
- 바인딩 변수 값에 따라 인덱스 선택이 달라질 수 있음 → 일부 DB는 바인딩 값별로 플랜을 재생성

**DDL 쿼리가 호출되면 캐시 된 실행 계획이 무효화될 수 있음**

### 실무 적용 팁

SQL 튜닝 / 최적화의 핵심은 실행 계획 분석

- 인덱스, 조인, 필터일, 정렬 등 실행 단계별 병목 파악
- 불필요한 풀스캔, 잘못된 조인 순서 / 알고리즘, 적합하지 않는 인덱스 사용 등 발견 가능

실행 계획은 DBMS마다 다르므로, DB별 도구와 명령어(EXPLAIN 등)를 반드시 숙지 필요

MySQL / PG / Oracle의 시각화 도구(pgAdmin, MySQL Workbench 등) 활용하는 것을 권장

→ 복잡한 쿼리의 실제 동작을 한눈에 파악 가능

## Statement 캐싱

### 필요성

SQL Statement의 파싱과 실행 계획 생성은 매우 자원 집약적인 작업

- Parser, Optimizer, Executor 단계 모두 CPU,메모리 소모

동일한 SQL이 반복 실행될 때 캐싱을 활용하면 파싱 / 컴파일 / 실행계획 생성 과정을 생략할 수 있고,

준비된 Statement의 재사용으로 인해 대량 처리 성능 극대화 가능

### 서버 측 Statement/Execution Plan 캐싱

Statement와 실행 계획을 DB 서버 내부에 캐싱

- SQL 구문, 실행 계획 등이 캐싱 대상
- 일부 DB의 경우 바인드 값별 커서도 대상

동일한 SQL(특히 PreparedStatement) 실행 시 파싱, 컴파일, Optimizer, 실행 계획 준비 단계를 건너뛰고 곧바로 실행 가능

장점

- 파싱 / 컴파일 / 실행 플랜 생성 오버헤드를 제거하고 다양한 클라이언트, 세션에서 공유 가능
- 같은 데이터베이스 인스턴스 내 모든 클라이언트 / 세션에서 재사용 가능
- 단, SQL 텍스트 / 파라미터 / 스키마 등 완벽히 동일해야 함

단점

- 나쁜 실행 계획이 캐시에 남아 있으면 전체 성능 저하를 유발할 수 있음

<br>

**Oracle**

- **Soft Parse**: 이미 캐싱된 실행 계획이 있으면 재사용
    - SQL이 완전히 동일(대소문자, 공백까지)해야 함
- **Hard Parse**: 실행 계획이 없으면 새로 생성
- **Bind Peeking**: 첫 바인드 파라미터 값으로 인덱스 선택
- **Adaptive Cursor Sharing (11g+)**: 바인딩 값에 따라 여러 실행 계획을 상황별로 선택
- 캐시 미스 방지 위해 PreparedStatement 사용 권장

<br>

**SQL Server**

- **Execution Plan Cache**: 모든 Statement/PreparedStatement의 계획을 캐싱
- **Parameter Sniffing**: 첫 실행 파라미터로 계획 결정
- OPTION(RECOMPILE)로 계획 재생성 강제 가능

<br>

**PostgreSQL**

- **9.2 버전 이전:** Prepare 단계에서 실행 계획 고정
- **9.2+ 버전:** 바인드값에 따라 실행 시점에 계획 결정 (deferred optimization)
- prepareThreshold로 n회 이상 실행 시 서버 측 Prepare 전환

<br>

**MySQL**

- 기본적으로 Statement Plan Cache 없음
    - Connector/J 5.0.5부터 PreparedStatement를 모방
    - useServerPrepStmts, cachePrepStmts 옵션으로 서버 측 PreparedStatement 활성화 가능

### 클라이언트 측 Statement Caching

Statement/PreparedStatement 객체를 JDBC 드라이버, 커넥션 풀, 애플리케이션 레벨에서 캐싱

DB 서버에서의 실행 계획 캐싱과 별개로, 이미 생성 및 할당된 Statement 객체를 재사용

- 드라이버 / 커넥션 수준에서 Statement를 풀에 저장 및 재할당
- 메타데이터, 실행 상태, 데이터까지 일부 재사용 가능

장점

- 클라이언트단에서 Statement 객체 생명 주기 및 생성 비용을 감소
- DB 서버에서 불필요한 Prepare 요청 및 커서 할당 감소

단점

- 커넥션 단위로만 재사용
- 연결 해제 시 캐시 소멸
- 서버 측 캐싱만큼 폭넓은 재사용은 불가

<br>

**Oracle**

- **암묵적(implicit) 캐싱:** connection property로 전체 캐시 크기 지정
- **명시적(explicit) 캐싱:** Oracle 전용 API로 직접 Statement 키 관리

<br>

**SQL Server**

- JDBC 6.3+부터 클라이언트 캐싱 지원 (기본적으로 비활성화)

<br>

**PostgreSQL**

- 9.4-1202+부터 connection-bound client-side statement cache
    - preparedStatementCacheQueries, preparedStatementCacheSizeMiB로 조정

<br>

**MySQL**

- cachePrepStmts, prepStmtCacheSize, prepStmtCacheSqlLimit 등 연결 파라미터로 제어

### Statement Caching 성능

대부분의 DBMS에서 Statement Caching 활성화 시 20% ~ 55% 이상의 쿼리 처리량 향상(1분 동안 실행 가능한 퀄시 수 기준)

OLTP 시스템에서 캐싱 적용 시 전체 트랜잭션 처리량 / 응답시간에 큰 영향

### 실무 적용 팁

동적 SQL 및 Statement를 PreparedStatement로 변환하는 것을 권장

→ 파라미터 바인딩을 활용, 재사용성 극대화

데이터베이스별 캐시 옵션/최적화 전략 숙지 필요

- Oracle: Adaptive Cursor Sharing, 명시적/암묵적 캐싱
- SQL Server: 스키마까지 포함한 fully qualified name 사용
- PostgreSQL/MySQL: 드라이버 옵션, prepare threshold, cache 크기 조정

**Statement 캐싱을 반드시 커넥션 풀, 트랜잭션, 동시성 정책과 함께 고려 필요**