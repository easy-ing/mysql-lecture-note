# Section 4 ( View - SELECT ALL)

<aside>
💡

View에서 SELECT * 을 사용하는 경우의 문제점

DB 스키마가 변경되면 쉽게 깨지는 안티패턴임!
예측가능성을 해칠 수 있음
단순하게 성능저하 X , DB 스키마 변경에 취약함

</aside>

### DDL

```sql
CREATE TABLE IF NOT EXISTS users (
     user_id BIGINT PRIMARY KEY AUTO_INCREMENT,
     name VARCHAR(100) NOT NULL,
     email VARCHAR(255) NOT NULL UNIQUE,
     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
     updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE  IF NOT EXISTS orders (
   user_id BIGINT,
   product_id BIGINT
);

CREATE TABLE  IF NOT EXISTS products (
   product_id BIGINT
);

```

### Problem View

```sql
CREATE VIEW user_summary AS
SELECT *
FROM users u
     LEFT JOIN orders o ON u.user_id = o.user_id
     LEFT JOIN products p ON o.product_id = p.product_id;
     
# DB 스키마에 변화가 생기면 문제가 생겨유
# MySQL 엔진에서 다룰 row가 너무 많아져요 ㅜ
```

 

### Solution View

```sql
# 필요한 것만 데려오셈
# 데이터 타입이 정확히 나눠져있어서 보기도 좋음

CREATE VIEW user_summary AS
SELECT
    u.user_id,
    u.name,
    u.email,
    u.created_date,
    o.order_id,
    o.order_date,
    o.total_amount,
    p.product_name,
    p.category
FROM users u
         LEFT JOIN orders o ON u.user_id = o.user_id
         LEFT JOIN products p ON o.product_id = p.product_id;
```

### 정리

> SELECT *은 
DB스키마 변경에 매우 취약함

새로운 컬럼이 추가되면 뷰는 자동으로 이 컬럼을
포함하게 됨 → View 입장에선 컬럼이 하나 추가되어버림

의존성 관리도 어려움 → 기본 테이블의 모든 컬럼에 의존해버림
>