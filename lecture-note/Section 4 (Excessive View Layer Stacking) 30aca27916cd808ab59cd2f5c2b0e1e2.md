# Section 4 (Excessive View Layer Stacking)

<aside>
💡

뷰 중첩에 대한 안티패턴 다루기!

</aside>

### 뷰(View)

장점 > 비즈니스 로직을 간편화 + DB엔진 부하 방지

단점 > 뷰가 많아지면 종속성 체인의 복잡성이 생김

DDL

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    created_date DATE
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2)
);

```

### 기본 뷰 (테이블 참조 → 깊이 1)

```sql
-- 기본 뷰 --

CREATE VIEW user_basic_info AS
SELECT user_id, name, email, created_date
FROM users;

CREATE VIEW order_basic_info AS
SELECT order_id, user_id, order_date, total_amount
FROM orders;

```

### 중간 뷰 (테이블 & 기본뷰 → 깊이 2)

```sql
-- 중간 뷰 --

CREATE VIEW user_order_summary AS
SELECT
    u.user_id,
    u.name,
    u.email,
    COUNT(o.order_id) as order_count,
    SUM(o.total_amount) as total_spent
FROM user_basic_info u
         LEFT JOIN order_basic_info o ON u.user_id = o.user_id
GROUP BY u.user_id, u.name, u.email;
```

### 상위 뷰 (중간 뷰 보다 1단계 더 깊음 → 깊이 3)

```sql
-- 상위 뷰 --

CREATE VIEW user_analytics AS
SELECT
    user_id,
    name,
    email,
    order_count,
    total_spent,
    CASE
        WHEN total_spent > 1000 THEN 'VIP'
        WHEN total_spent > 500 THEN 'Gold'
        ELSE 'Silver'
        END as customer_tier
FROM user_order_summary;
```

### 최상위뷰 (상위를 참조함 → 깊이 4)

```sql
-- 최상위 뷰 --

CREATE VIEW customer_segmentation AS
SELECT
    customer_tier,
    COUNT(*) as customer_count,
    AVG(total_spent) as avg_spent
FROM user_analytics
GROUP BY customer_tier;

CREATE VIEW complex_analytics AS
SELECT
    customer_tier,
    COUNT(*) as customer_count,
    AVG(total_spent) as avg_spent,
    MAX(total_spent) as max_spent,
    MIN(total_spent) as min_spent
FROM user_analytics
WHERE customer_tier IN ('VIP', 'Gold')
GROUP BY customer_tier;
```

최상위 뷰를 실행하기 위해 이전에 모든 뷰를 실행하게 됨 ㅜ

각 뷰마다 추가적인 메모리할당과 처리실행이 필요함 → 처리시간 급증

> 
> 
> 
> 뷰 중첩에는 성능적인 부분을 생각해야함!
> 
> 그리고 디버깅 과정에서의 문제가 생김
> 
> 내부적으로 고급패턴을 사용하면 어떤 뷰를 생성하는데 문제가 생겼는지 알 수 없음
> 
> 새로운 컬럼 추가하려면 여러 뷰를 추가해야하고
> 
> 많은 뷰를 수정해야함 ㅜ
> 

### How to solve?

1. 뷰의 평탄화 과정 : 뷰를 평탄화하는게 좋음
복잡한 과정을 단순화하기 / 굳이 중간뷰 만들지 않기
→ 메타데이터 조회 굿, 디버깅도 굿
2. materialization : 복잡한 연산이 있는 경우 뷰를 많이 쓰는데
    
    multi-realization → 결과를 물리적으로 저장
    이 테이블에 대해 인덱스를 등록하거나 활용하기에 좋음
    집계합수를 굳이 사용할 필요가 없음