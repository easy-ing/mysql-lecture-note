# Section 4 (DISTINCT)

<aside>
💡

DISTINCT 남용으로 인한 중복 문제 해결하기

</aside>

> DISTINCT는 언제쓰나요?
→ 잘못된 조인으로 중복된 결과가 많이 생길때!
→ 임시로 중복된 결과 지우면서 무결성 챙기기

데이터 정합성 해칠수있음!

HOW? 
데이터의 관계가 1 : N 일때 
DISTINCT가 다른건데 같은걸로 이해함
그래서 없애버림..!
> 

### 문제의 상황

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

SELECT DISTINCT
    u.user_id,
    u.name,
    u.email,
    o.order_id,
    o.order_date,
    o.total_amount
FROM users u
         LEFT JOIN orders o ON u.user_id = o.user_id;
         
# 사용자와 주문정보를 조인하고 조회할것
# LEFT JOIN 써서 주문있는 사용자, 주문없는 사용자 모두 합쳐짐 -> 즉 중복된 결과가 나옴
# DISTINCT를 사용하면 중복 제거 하지만 다른 주문이지만 같이 사라질수도있음
```

### HOW to Solve?

<aside>
💡

조인로직 보강 or WHERE 조건 강화

</aside>

```sql
SELECT
    u.user_id,
    u.name,
    u.email,
    o.order_id,
    o.order_date,
    o.total_amount
FROM users u
         LEFT JOIN orders o ON u.user_id = o.user_id
WHERE o.order_id IS NOT NULL OR u.user_id NOT IN (
    SELECT DISTINCT user_id FROM orders
);

# 주문있고 없는걸 명확하게 구분함
```

### 그래서 왜 DISTINCT가 안티패턴을 초래할 수 있는가?

```sql
# 데이터 무결성 문제를 해침
# 중복 제거한다고 쿼리 시간 시간이 더 늘어남

SELECT DISTINCT
    p.product_id,
    p.product_name,
    p.price,
    c.category_name
FROM products p
         LEFT JOIN categories c ON p.category_id = c.category_id
         LEFT JOIN product_tags pt ON p.product_id = pt.product_id
         LEFT JOIN tags t ON pt.tag_id = t.tag_id;

SELECT DISTINCT
    o.order_id,
    o.order_date,
    o.total_amount,
    u.name,
    p.product_name
FROM orders o
         JOIN users u ON o.user_id = u.user_id
         JOIN order_items oi ON o.order_id = oi.order_id
         JOIN products p ON oi.product_id = p.product_id;

SELECT
    u.user_id,
    u.name,
    COUNT(DISTINCT o.order_id) as order_count,
    SUM(DISTINCT o.total_amount) as total_spent
FROM users u
         LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id, u.name;
```

### DISTINCT 안쓰고 중복해결하기

```sql
SELECT
    u.user_id,
    u.name,
    u.email,
    o.order_id,
    o.order_date,
    o.total_amount
FROM users u
         JOIN orders o ON u.user_id = o.user_id
WHERE o.order_date >= '2024-01-01'
ORDER BY u.user_id, o.order_date;

SELECT
    u.user_id,
    u.name,
    u.email,
    p.phone_number,
    p.address
FROM users u JOIN user_profiles p ON u.user_id = p.user_id;

SELECT
    u.user_id,
    u.name,
    COUNT(o.order_id) as order_count,
    SUM(o.total_amount) as total_spent,
    MAX(o.order_date) as last_order_date
FROM users u LEFT JOIN orders o ON u.user_id = o.user_id
    GROUP BY u.user_id, u.name;
```

### 정리

<aside>
💡

설계가 불분명하거나 무너지면

DISTINCT 쓸수도 있음

JOIN로직 강화하고 데이터간의 관계를 깔끔하게 해야함

</aside>