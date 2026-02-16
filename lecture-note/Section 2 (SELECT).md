# Section 2 (SELECT)

> DB의 핵심 & 정수 & 꽃 = SELECT
인덱스 설계나 추가도 select로 부터 시작함!
저장된 데이터를 조회하고 분석하는 모든 작업 = 데이터 결과확인 = SELECT

SELECT를 실행하면 어떤일이 일어날까?
1. 쿼리파싱
2.권한확인&쿼리 최적화
3.인데스 선택, 데이터 읽기(옵티마이저가 함)
4.where절
5.정렬/그룹화
6.결과반환
> 

```sql
#select 기본 틀
SELECT columns FROM table_name WHERE condition GROUP BY columns HAVING condition ORDER BY columns LIMIT count;

#집계함수는 where에 쓸 수 없다

SELECT * FROM users WHERE city = 'Seoul';
SELECT * FROM users WHERE age > 30;
SELECT * FROM users WHERE email = 'userlk;@naver.com';

# 다양한 연산 조건이 있다
# =, != or <>, >, <, >=, <=, IS NULL, IS NOT NULL

SELECT * FROM users WHERE (city = 'Seoul' OR city = 'Busan') AND age >= 30;
SELECT * FROM users WHERE age BETWEEN  25 AND 35;
SELECT * FROM users WHERE  city IN ('Seoul', 'Busan');

-- 이름이 'User'로 시작하는 사용자들
SELECT * FROM users WHERE name LIKE 'User%';
# 정렬된 인덱스 범위 탐색이 가능함.

WHERE email LIKE '_est%';
# 비효율적이라서 인덱스를 완전히 사용하지는 않음

WHERE email LINK '%est%';
# 와일드카드가 붙어 있기때문에 정렬된 트리 사용이 불가능함 => 풀스캔 필수!

ALTER TABLE users
    ADD FULLTEXT INDEX ft_email_ngram (email)
        WITH PARSER ngram;
# 이 방식을 사용하면 훨씬 편해짐

WHERE email LIKE '%test%';
WHERE email LIKE '%example.com%';

ALTER TABLE users ADD email_rev VARCHAR(255)
    GENERATED ALWAYS AS (REVERSE(email)) STORED;
```

> select에 대해서 조금 더 딥하게 가보자구요
> 

```sql
# order by를 통해 보여주는 컬럼값을 수정할 수 있음.
SELECT * FROM users order by age;

# 정렬이 있을때, 어떤 방식으로 할까?
# 인덱스가 있는 컬럼이면 인덱스 기준으로 없다면? filesort - 모두 다 정렬

# Where 필터링할 조건이 있는지 확인하기
# 정렬하기 위한 버퍼 할당 = buffer_size 할당
# 정렬할 데이터 메모리로 로드
# 정렬 수행
# 정렬된 결과를 반환

SELECT * FROM users ORDER BY city, age DESC;
# 정렬값 2개 이상이면 그냥 , 기준으로 쓰면 된다

# LIMIT

SELECT * FROM users
ORDER BY age DESC
LIMIT 10;

# 100만개의 레코드에서 상위 10개를 찾는다고
# 일반정렬 : 100만 개를 모두 정렬 O(n Log n) -> 2천만번
# Top N : 10개짜리 힙을 유지하면서 순회 O(n log k) -> 2백만번

SELECT COUNT(*) FROM users;
SELECT COUNT(age) FROM users;
SELECT AVG(age) FROM users;
SELECT MAX(age), MIN(age) FROM users;
SELECT SUM(age) FROM users;

-- 도시별 사용자 수
SELECT city, COUNT(*) as user_count
FROM users
GROUP BY city;

-- 도시별 평균 나이
SELECT city, AVG(age) as avg_age
FROM users
GROUP BY city
ORDER BY avg_age DESC;

-- 도시별 최고/최저 나이
SELECT city, MAX(age) as max_age, MIN(age) as min_age
FROM users
GROUP BY city;

# -----------------------------------------

-- 사용자 수가 1000명 이상인 도시들
SELECT city, COUNT(*) as user_count
FROM users
GROUP BY city
HAVING user_count >= 1000;

-- 평균 나이가 30세 이상인 도시들
SELECT city, AVG(age) as avg_age
FROM users
GROUP BY city
HAVING avg_age >= 30;

# --------------------------------

SELECT city, AVG(age) as avg_age
FROM users
WHERE city = 'Seoul'  -- 레코드 필터링
GROUP BY city
HAVING avg_age >= 30;  -- 그룹 필터링

# --------------------------

-- 모든 도시 목록 (중복 제거)
SELECT DISTINCT city FROM users;

-- 여러 컬럼의 조합에서 중복 제거
SELECT DISTINCT city, age FROM users;

# 1. 정렬 기반 중복을 제거 : 결과를 정렬한 후, 인접한 중복값을 제거해버리는거요.
# 2. 해시 테이블 기반의 중복을 제거 : 각 고유한 값을 해시테이블로 저장하고, 이미 존재하는 값은 건너뛰어

# 10,000명의 사용자 -> 10개의 서로 다른 도시에 거주
# 10,000개의 레코드를 읽는다.
# 중복을 제거할꺼에요. 10개의 고유한 도시만을 반환
# 정렬 방식을 O(n log n) -> 13만 번의 비교
# 해시 방식을 O(n) => 1만번 계산

SELECT DISTINCT city, age FROM users;
# (Seoul, 25), (Seoul, 30), (Busan, 25)

-- 잘못된 이해: city만 중복 제거하고 싶지만
SELECT DISTINCT city, name FROM users;
-- 실제 동작: (city, name) 조합이 중복 제거됨

# DISTINCT : 단순히 진짜 오로지 단순히 중복을 제거한다. 의도가 명확
# GROUP BY : 그룹화 후 집계를 수행한다. 집계 함수 활용 ->
```