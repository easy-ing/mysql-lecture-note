# Section 4 (CASE-WHEN)

> SQL 안티패턴 알아보기
> 

### CASE-WHEN

이런 고급 패턴들은 어쩔수 없는 경우에만 쓰는게 좋음

왜? 

1. 중복된 로직이 많아짐 + 레거시 코드를 그대로 복사하는 경우가 많은데 만약 비효율적인 코드가 있다면 그게 계속 쌓이게 됨

같은 데이터에 대해서 서로 다른 결과를 반환 할 수 있음

2. 데이터의 신뢰성이 떨어짐 → 서로 다른 해석을 해서 오류가 생길 수 있음

### DDL

```sql
CREATE DATABASE IF NOT exists mysql_lecture;
USE mysql_lecture;

CREATE TABLE  IF NOT EXISTS status_dimension (
    status_code VARCHAR(10) PRIMARY KEY, # 1, 2, 3
    status_name VARCHAR(50) NOT NULL, # 재고 부족, 재고 충분, 품절
    status_description TEXT,
    is_active BOOLEAN DEFAULT FALSE
);

INSERT INTO status_dimension (status_code, status_name, status_description) VALUES
    ('1', '재고 없음', '현재 해당 상품의 재고가 없습니다'),
    ('2', '재고 부족', '재고가 부족한 상태입니다'),
    ('3', '재고 충분', '재고가 충분한 상태입니다');

CREATE TABLE  IF NOT EXISTS products (
    product_id BIGINT NOT NULL,
    product_name VARCHAR(100) NOT NULL,
    current_status VARCHAR(10) NOT NULL,
    stock_quantity INT DEFAULT 0,
    CONSTRAINT fk_status_code #제약 조건을 지키도록 해줌
        FOREIGN KEY (current_status)
        REFERENCES  status_dimension(status_code)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

INSERT INTO products (product_id, product_name, current_status, stock_quantity) VALUES
        (1, '노트북', '1', 0),
        (2, '마우스', '3', 100),
        (3, '키보드', '2', 5);
```

### VIEW

<aside>
💡

단순한 데이터 변환X / 비즈니스 규칙을 캡슐화함
다양한 애플리케이션에서 ‘일관된’ 데이터 접근을 허용함

뷰를 통해서 일관된 데이터를 불러올 수 있음!

</aside>

```sql
CREATE VIEW product_status_view AS
    SELECT
        p.product_id,
        p.product_name,
        p.current_status,
        sd.status_name,
        sd.status_description
    FROM products AS p
    LEFT JOIN status_dimension sd on p.current_status = sd.status_code
    WHERE sd.is_active = TRUE;

SELECT product_name FROM product_status_view WHERE current_status = '1';
```

Case-When 같은 고급함수 안쓰고 간단히 쓸 수 있음.

```sql
# CASE-WHEN을 사용하면 이렇게 길어지고
# 관리도 힘들어져요ㅠ

SELECT
    product_name,
    CASE
        WHEN current_status = '1' THEN '재고 없음'
        WHEN current_status = '2' THEN '재고 부족'
        WHEN current_status = '3' THEN '재고 충분'
        ELSE '알 수 없는 상태'
        END as status_name,
    CASE
        WHEN current_status = '1' THEN '현재 해당 상품의 재고가 없습니다'
        WHEN current_status = '2' THEN '재고가 부족한 상태입니다'
        WHEN current_status = '3' THEN '재고가 충분한 상태입니다'
        ELSE '상태를 확인할 수 없습니다'
        END as status_description
FROM products
WHERE current_status IN ('1', '2');
```

### 결론 - 차원테이블의 접근성

1. 데이터의 일관성 유지 - 상태코드 1개로 공통적으로 관리됨
2. 유지보수가 좋아짐 - 차원테이블만 수정하면 됨
3. 메모리에 캐싱되면서 빠르게 관리할 수 있음
4. 가독성도 좋아짐