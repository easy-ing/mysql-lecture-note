# Section 5 (Boolean Modeling)

<aside>
💡

설계가 왜 중요한지 → 안티패턴을 통해 배움
이제는 데이터 모델링에 집중해보자구!

</aside>

> 목표
: 대답이 Yes or No , 즉 Boolean 타입이 많은 엔티티 설계

논리적 모델링인 비즈니스의 불변의 개념 표현
물리적 모델은 설계 확장성을 위해 변해야함
> 

DDL

```sql
CREATE TABLE restaurants (
     restaurant_id INT PRIMARY KEY,
     name VARCHAR(100),
     address VARCHAR(200),
     #----boolean----
     has_parking BOOLEAN,
     has_wifi BOOLEAN,
     has_delivery BOOLEAN,
     is_vegetarian_friendly BOOLEAN,
     has_outdoor_seating BOOLEAN,
     has_live_music BOOLEAN
);

# 컬럼 값이 변할때 마다, alter table을 실행하고
# 그에 따라서 컬럼수가 증가하기 때문에 성능이 떨어짐
# 유지보수도 힘들겠죠
```

### Anchor

> 데이터 모델링에서 핵심적인 역할 
( 비지니스 도메인의 중심이 되는 엔티티 )

Ex) 레스토랑 설계
Anchor : 레스토랑 자체
> 

이 앵커를 변환하는것 보단

추가적인 정보를 넣고싶을때 어떤 방식을 쓸까?

1. one-table per 엔터티 : 앵커의 모든 속성을 하나의 테이블 컬럼으로 저장 → 컬럼이 많아질 수록 성능 + 유지보수 힘들어
2. Side table : 도메인 그대로 두고 필요한 기능을 따로 만들음 → 본 테이블에 대해서 Look up을 해줘야함 + 항상 조인과 관리를 해줘야함
3. Strategy 3 EAV : *EAV (Entity Attribute Value) : 1,2번은 결국 불리언으로 관리하는데 얘는 Attribute로 관리함 → 쿼리 복잡 + 기본적으로 가져오는 데이터가 많음*