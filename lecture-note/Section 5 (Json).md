# Section 5 (Modern Alternatives)

<aside>
💡

현대적인 모델링 = EAV 한계 극복 + mysql의 json 타입 활용 & 스키마 변경X & 유연한 속성 저장

</aside>

### MySQL - JSON(JavaScript Object Natation)

> 텍스트 형태로 저장되지만 JSON함수 써서 효율적으로
쿼리 사용할 수 있음

잘만쓰면 JSON경로의 인덱싱을 통해서 쿼리 성능 향상시킴
JSON 함수와 연산자를 통해서 씀
결론적으로는 대용량 데이터에서는 성능이 쬐끔 걱정됨
> 

```sql
-- MySQL JSON 테이블 생성
CREATE TABLE restaurants (
    restaurant_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    attributes JSON
);

-- JSON 데이터 삽입
INSERT INTO restaurants (restaurant_id, name, attributes) VALUES
    (1, 'Pizza Palace', '{"has_parking": true, "has_wifi": true}'),
    (2, 'Burger King', '{"has_parking": false, "has_wifi": true}');

CREATE INDEX idx_restaurants_parking ON restaurants
    ((CAST(attributes->>'$.has_parking' AS UNSIGNED)));

# JSON은 텍스트 형태로 저장을 해서 저장공간을 좀 많이 쓰긴함
# 함수를 쓸 수 있어서 유용하게 쓰기 괜찮음
```

### 인덱스 관점에선?

인덱스 효율은 그닥,, 성능 보장이 잘 안돼ㅜ

그리고 캐스팅 과정을 거쳐야함! (아래가 이유)

> 
> 
> 1. 인덱스 생성할때마다 인덱스를 정렬해야해
>     
>     그래서 파싱이 필요함
>     
> 2. 경로를 추출한 후 어떤 타입인지 파악을 해야함
> 3. 일반컬럼인덱스보다 구조가 복잡함
> 4. 경로랑 정보를 둘다 저장해야함
>     
>     JSON EXTRACT 이거 쓰면 파싱비용 많이 쓰임
>     

### 그럼 이걸 어떻게 쓸까?

JSON_EXTRACT, JSON_CONTAINS 활용해

```sql
SELECT name, JSON_EXTRACT(attributes, '$.has_parking') as has_parking FROM restaurants
WHERE JSON_EXTRACT(attributes, '$.has_parking') = true;
```

### JSON 방식은 하이브리드 방식

= 속성 컬럼 저장 + 확장 속성은 JSON 필드로 저장 + 테이블에 JSON 구조체 담기

```sql
CREATE TABLE restaurants (
    restaurant_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    has_parking BOOLEAN DEFAULT FALSE,
    has_wifi BOOLEAN DEFAULT FALSE,
    extended_attributes JSON
);
```

### 정리

<장>

정규화된 구조 활용

확장속성은 JSON필드 유연하게

서비스 성장하면서 요구사항이 다행해져서

필요한 상황에 쓰면 좋음!

<단>

일관성 유지가 힘들어

쿼리 작성 어려워

성능 최적화 어려워

유지보수 힘들어