# Primary Key vs Unique Key

## Primary Key

테이블에서 각 행을 고유하게 식별하는 데 사용

중복, NULL 모두 허용되지 않음

테이블단 오직 하나만 존재

자동으로 클러스터형 인덱스(Clustered Index) 생성(InnoDB에서 실제 데이터 정렬 기준)

## Unique

해당 칼럼의 값이 중복되지 않도록 보장

NULL은 허용되며, 하나의 NULL만 저장 가능(MySQL 기준)

하나의 테이블에 여러 개 설정 가능

보조 인덱스(Secondary Index)로 동작

## 클러스터형 인덱스(Clustered Index)

InnoDB에서 데이블 자체가 Primary Key 기준으로 정렬되어 저장

즉, Primary Key 인덱스 자체가 데이터를 포함

PK 기준으로 검색 시 인덱스를 타고 바로 데이터 접근 가능

## 보조 인덱스(Secondary Index)

Unique 제약을 걸면 보조 인덱스가 생성

보조 인덱스는 실제 데이터를 저장하지 않고, Primary Key를 포인터처럼 저장

즉, Unique 인덱스로 검색한 후, 실제 데이터를 얻기위해 PK를 따라가야 함

## B+Tree 구조 비교

### PK

리프 노드(Leaf Node)에 전체 행 데이터 저장

즉, id가 PK라면, id → 전체 행 구조

테이블의 실제 데이터가 B+Tree에 들어가 있음

```java
[클러스터 인덱스 구조 (id 기준)]
         (내부 노드)
            [50]
           /    \
       [10][30] [60][80] ← 리프 노드 (id 기준 정렬)
         ↓        ↓
    Row(id=10, name=Alice)
    Row(id=30, name=Bob)
    Row(id=60, name=Tom)

```

### Unique

리프 노드에는 Unique Key 값과 해당 행의 PK 값만 저장

즉, email → id 형식

전체 데이터를 읽으려면, 보조 인덱스를 타로 클러스터 인덱스를 다시 조회

```java
[보조 인덱스 구조 (email 기준)]
         (내부 노드)
            [m@x.com]
           /        \
   [a@x.com][k@x.com][z@x.com] ← 리프 노드
         ↓         ↓
     id=10      id=30

[→ 다시 PRIMARY KEY 인덱스에서 id=10/30 찾음]

```

검색 시 : 보조 인덱스 탐색 → PK 탐색 → 데이터 읽기 (I/O 2번 발생)

UPSERT 작업 시

- `INSERT ... ON DUPLICATE KEY UPDATE` 시
    
    → `UNIQUE` 인덱스는 먼저 보조 인덱스 탐색
    
    → 해당 보조 인덱스가 가리키는 PK 인덱스를 다시 찾아서 기존 행을 업데이트
    

즉, 검색 → PK 재탐색 → 업데이트 or 무시 3단계

- 반면 PK 충돌이면 바로 리프 노드에서 해결 가능

```java
SQL 입력
   ↓
파싱 + 인덱스 로딩
   ↓
클러스터 인덱스 탐색 → 중복 키 검사
   ↓            ↓
[중복 없음]     [중복 있음]
   ↓               ↓
INSERT 수행      UPDATE 수행
   ↓               ↓
UNDO 로그         UNDO 로그
REDO 로그         REDO 로그
   ↓               ↓
  커밋 or 롤백 (단일 트랜잭션)
```

현재 Bookmark 구현에서 UPSERT 작업이 빈번하기 때문에,

보조 인덱스를 거치지 않고 PK 값으로 판단하는 것이 낫다고 봄