# Section 4 (Index Columns)

<aside>
💡

인덱스 컬럼 & 함수 사용 쿼리 패턴에서 문제점 배우기

</aside>

### 미리 정리하자면!

> 
> 
> 
> 인덱스가 걸린 함수를 적용하는 패턴은
> 
> 데이터 베이스 성능에 영향을 많이 주는 안티 패턴이 될 수 있다!
> 

<aside>
💡

왜 ? 인덱스를 못탐

ㄴ DB의 인덱스는 B-tree로 이루어져 있음
쉽게 말해서 인덱스를 기준으로 정렬된 데이터가 따로 존재함

이 상황에서 인덱스에 함수를 걸어버리면 정렬된 구조를 쓸 수 가 없음 대신 모든 행을 순차적으로 스캔하고 함수 적용해야됨

</aside>

### 상황

```sql
CREATE TABLE  IF NOT EXISTS users (
    user_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL
)

# 이건 데이터 개수만큼 풀스캔을 때려야함!
# 너무 오래걸리고 메모리 낭비가 지려버림
SELECT
    user_id,
    name,
    email
FROM users
WHERE UPPER(name) = 'JOHN DOE'
   OR UPPER(email) = 'JOHN@EXAMPLE.COM';
```

### 해결법 - 함수 기반의 인덱스를 추가로 생성하기

```sql
-- 대소문자 구분 없는 검색을 위한 함수 기반 인덱스
CREATE INDEX idx_users_upper_name ON users (UPPER(name));
CREATE INDEX idx_users_upper_email ON users (UPPER(email));

-- 날짜 함수를 사용한 검색을 위한 인덱스
CREATE INDEX idx_orders_date ON orders (DATE(created_date));

-- 문자열 함수를 사용한 검색을 위한 인덱스
CREATE INDEX idx_products_category ON products (SUBSTRING(category_code, 1, 3));

# 기존 인덱스와 동작하는 방식은 같음
# 함수 기반으로 정리한 값을 또 정리해서 인덱스에 박아두는것임

'''
함수 기반의 인덱스의 내부 동작

DB엔진은 인덱스 생성 시 각 행에 대해
함수를 적용한 결과 값을 계산함

이 결과값을 기준으로 인덱스 구조를 구성

함수를 완전히 제거하고 인덱스를 직접 활용하는 방법임
'''
```