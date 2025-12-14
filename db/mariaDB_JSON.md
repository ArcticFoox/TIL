#MariaDB JSON
MariaDB 10.2 부터 지원
내부적으로 LONGTEXT로 저장되지만,
JSON_VALID() 같은 내장 함수를 통해 데이터 무결성 보장
  
MySQL의 네이티브 JSON 타입과 저장 방식(압축 여부)에서 차이가 있어,
MySQL과의 호환성에 주의 필요

## 예시
삽입
```sql
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    data JSON
);
INSERT INTO products (data) VALUES ('{"name": "Laptop", "price": 1200}');
```
추출
```sql
SELECT JSON_EXTRACT(data, '$.name') FROM products WHERE id = 1;
-- 결과: "Laptop"
```
인덱싱
```sql
ALTER TABLE products ADD COLUMN product_name VARCHAR(255) GENERATED ALWAYS AS (JSON_UNQUOTE(JSON_EXTRACT(data, '$.name'))) STORED;
CREATE INDEX idx_product_name ON products(product_name);
```

## 주요 함수
- JSON_VALID(json): JSON 형식 유효성 검사
- JSON_EXTRACT(json, path): JSON에서 특정 경로의 값 추출
- JSON_SET(json, path, value): JSON에서 특정 경로의 값 설정/수

그 밖은 [여기서](https://mariadb.com/docs/server/reference/sql-functions/special-functions/json-functions) 확인
