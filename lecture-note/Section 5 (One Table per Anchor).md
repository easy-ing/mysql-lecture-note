# Section 5 (One Table per Anchor)

<aside>
💡

One Table Per Anchor
장점 : 초기 프로토는 활용과 적용이 쉬움(Join X, Table X)
단점 : 사이즈가 커지면 성능 + 유지보수가 힘듬

</aside>

DDL

```sql
CREATE TABLE restaurants (
    restaurant_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    address VARCHAR(200),
    phone VARCHAR(20),
    has_parking BOOLEAN DEFAULT FALSE,
    has_wifi BOOLEAN DEFAULT FALSE,
    has_delivery BOOLEAN DEFAULT FALSE,
    is_vegetarian_friendly BOOLEAN DEFAULT FALSE,
    has_outdoor_seating BOOLEAN DEFAULT FALSE,
    has_live_music BOOLEAN DEFAULT FALSE,
    accepts_credit_card BOOLEAN DEFAULT FALSE,
    is_24_hours BOOLEAN DEFAULT FALSE
#     has_rooftop BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_restaurants_parking ON restaurants(has_parking);
CREATE INDEX idx_restaurants_wifi ON restaurants(has_wifi);
CREATE INDEX idx_restaurants_vegetarian ON restaurants(is_vegetarian_friendly);
CREATE INDEX idx_restaurants_delivery ON restaurants(has_delivery);
```

알 수 있는 점

1. 컬럼수가 증가하면, 내부 자원을 정말 많이 씀
Ex) PostgreSQL에서는 8KB 페이지 크기에서
컬럼수가 증가하면 페이지당 레코드 수가 적어짐
하나의 페이지당 레코드가 적으면 → 페이지 많이 땡겨와야함 → 메모리 부족해짐
2. 직관적이고 조인도 필요 없어 → 개이득 직관적이넹

컬럼을 추가하고 싶어!

```sql
ALTER TABLE restaurants ADD COLUMN  has_rooftop BOOLEAN DEFAULT FALSE;

# 엄연히 테이블 구조 자체를 변경하는 과정
# Mysql은 내부적으로 Exclusive Lock을 실행하면서
# 다른 모든 작업 차단 + 기존 데이터 백업 + 새 스키로 테이블 재생성
# 그 다음 Lock 해제 -> 들어오는 모든 작업을 병목시킴!!
```