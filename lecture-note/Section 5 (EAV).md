# Section 5 (EAV)

<aside>
💡

EAV = Entity-Attribute-Value
얜 메인 테이블 + 속성을 저장하는 별도의 테이블을 구성함

</aside>

### 특징

속성을 컬럼이 아닌 행으로 저장

→ 속성이 자주 바뀌는 경우엔 좋음

Row Extense 문제로 Boolean만 처리하기에는 힘들어ㅜ

### Example

```sql
-- 메인 엔티티 테이블
CREATE TABLE restaurants (
    restaurant_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    address VARCHAR(200),
    phone VARCHAR(20),
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- EAV 테이블: 속성을 행으로 저장
CREATE TABLE restaurant_attributes (
    restaurant_id INT NOT NULL,
    attr_name VARCHAR(50) NOT NULL,
    attr_value VARCHAR(100),
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (restaurant_id, attr_name),
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
); # boolean으로 속성을 표현하지는 않음

# 1:N의 전략을 가짐 -> 값을 조인해서 들고오면 파싱은 어케함?
# 데이터 저장은 용이한데 가져오는 과정에서 병목이 생길수있음
# Json, Concat을 활용해서 쓰는게 좋음
# 그래도 추가적인 스키마 변경 없이 추가할 수 있음
```

### 속성 추가하는 방법

```sql
# 간단하게 Data Row만 추가하기
# 스키마 변경없이 추가할 수 있음

INSERT INTO restaurant_attributes (restaurant_id, attr_name, attr_value) VALUE (
    1, 'test', 'true');

INSERT INTO restaurant_attributes (restaurant_id, attr_name, attr_value) VALUE (
    1, 'has_parking', 'true');
    
# 속성 테이블도 가져올래!
SELECT * FROM restaurants AS r INNER  JOIN restaurant_attributes ra on r.restaurant_id = ra.restaurant_id;
```

### 요약

1. 대표적으로 속성을 행으로 저장함

→ ALT 테이블과 같이 스키마 변경하는 쿼리 없어도 새로운거 INSERT하기가 쉬움 → 유연성 굿

1. 그룹바이나 조인처럼 성능 저하를 유발하는 애들도 자주 쓰게 됨

두 개의 trade-off를 잘 고민하세요!