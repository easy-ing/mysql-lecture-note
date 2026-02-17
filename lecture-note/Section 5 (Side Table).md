# Section 5 (Side Table)

<aside>
💡

분리 테이블 = Side Table

Boolean 속성을 별도 테이블로 분리해서
메인테이블의 복잡성을 줄임 

</aside>

### Side Table이란?

> 정의 : 메인 테이블에서 분리한 속성들을 별도의 테이블로 분리
→ 그 결과 메인 테이블 복잡성을 줄임 + 우리가 필요한 애들만 로드 가능
> 
> 
> 1:1 관계의 사이드 테이블은 메인 테이블과 분리될 때, 얻는 이점이 있음
> 
> 최대한 속성을 분리하고 사이드 테이블을 분리함
> 

### Example

```sql
-- 메인 테이블: 핵심 비즈니스 정보만 포함
CREATE TABLE restaurants (
    restaurant_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    address VARCHAR(200),
    phone VARCHAR(20),
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Side Table: Boolean 속성들을 별도 테이블로 분리
CREATE TABLE restaurant_flags (
    restaurant_id INT PRIMARY KEY,
    has_parking BOOLEAN DEFAULT FALSE,
    has_wifi BOOLEAN DEFAULT FALSE,
    has_delivery BOOLEAN DEFAULT FALSE,
    is_vegetarian_friendly BOOLEAN DEFAULT FALSE,
    has_outdoor_seating BOOLEAN DEFAULT FALSE,
    has_live_music BOOLEAN DEFAULT FALSE,
    accepts_credit_card BOOLEAN DEFAULT FALSE,
    is_24_hours BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
); # 기본적으로 1:1 관계를 가짐

SELECT restaurant_id, name, address, phone FROM restaurants WHERE name LIKE '%Pizza%';

SELECT * FROM restaurant_flags WHERE has_parking = TRUE;

SELECT * FROM restaurants INNER JOIN restaurant_flags rf on restaurants.restaurant_id = rf.restaurant_id;

```

### 특징

> 일단 얘도 컬럼 기반의 접근법임!
속성이 자주 추가 된다거나 인덱스의 관점에서는
근본적으로 성능적인 부분이 떨어짐
Exclusive Lock도 동일함 → 병목현상 유지
FK가 있어서 강제로 처리하기가 힘들어ㅜㅜ

그래도 장점은 명확함
테이블간의 관계가 명확해서 유지보수는 쉬움
> 

`@Transactional을 묶는 단점이 있다..!`

```sql
begin;

UPDATE  restaurants SET name = 'New Name' WHERE restaurant_id = 1;
update restaurant_flags set has_parking = true where restaurant_id = 1;

commit;
```