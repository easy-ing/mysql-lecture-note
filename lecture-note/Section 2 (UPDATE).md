# Section 2 (UPDATE)

> 어떤 update가 있고 
mysql에서 update는 어떤 방식으로 진행될까?!
> 

```sql
-- users 테이블
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    age INT,
    city VARCHAR(100),
    status VARCHAR(50),
    grade VARCHAR(50),
    priority VARCHAR(50),
    score INT DEFAULT 0,
    category VARCHAR(50),
    discount_rate DECIMAL(5,2) DEFAULT 0.00,
    total_purchase DECIMAL(12,2) DEFAULT 0.00,
    total_orders INT DEFAULT 0,
    phone VARCHAR(20),
    address VARCHAR(255),
    country VARCHAR(10),
    new_id VARCHAR(50),
    new_email VARCHAR(255),
    old_system TINYINT(1) DEFAULT 0,
    external_id INT,
    metadata JSON,
    last_login TIMESTAMP NULL,
    last_order_date DATE NULL,
    last_sync TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- user_preferences 테이블 (서브쿼리, JOIN 예제용)
CREATE TABLE user_preferences (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    preferred_city VARCHAR(100),
    city VARCHAR(100)
);

-- external_users 테이블 (데이터 동기화 예제용)
CREATE TABLE external_users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    status VARCHAR(50)
);

-- orders 테이블 (통계 업데이트 예제용)
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    order_date DATE NOT NULL
);

# 업데이트라는 쿼리는 어떤 쿼리일까?
'''
이미 존재하는 데이터를 수정하는 쿼리
INSERT를 통해 데이터 생성
우리는 추가한 데이터를 -변경- 하고싶을 수 있음

데이터의 생명주기 관점에서 중요함
데이터는 대부분 변동이 큼
비즈니스 로직, 사용자의 요구사항 등으로 변화함
생명주기가 굉장히 짧음

업데이트 잘못쓰면 데이터 일관성이 깨질 수 있음
잘못 수정한 데이터는 bin log에 기록이 되어서
신중하게 진행해야함!

단순히 값 변경이 아니라
db에 무결성을 유지하면서 진행해야함.

그럼 mysql은 update를 어케 이해함?
1. 쿼리 파싱
2. 권한 확인 (update 해도 됨?)
3. where 평가 : 맞는 레코드를 찾음
4. 잠금 : 외부에서 수정 못하게
5. 값 변경
6. 제약 조건 검사
7. 인덱스 재 정렬
8. 변경사항 최종으로 저장하는 커밋
'''

UPDATE database_name.table_name # 수정할 테이블 지정함 / 보통은 단일 테이블 지정 근데 join써서 여러개 업뎃할수도있음
SET colume1 = value1, colume2 = value2, .... # 새로운 값 설정 + 컬럼 매칭 
WHERE condition; # 옵션일수도 잇음 근데 거의 필수로 씀 (어떤 조건이냐면!!)

# set 값은 지정하는 방식이 다양함 
# 1) 직접 값을 지정 : SET age = 30 2)기존 값을 참조 : SET age = age + 1
# 3) 함수 사용 : SET name = UPPER(name) 4) 서브 쿼리 : SET city = ( SELECT city FROM table_name WHERE .. )
# 5) 조건부 : SET status = IF(age > 18, 'adult', 'minor')
# SET a = b, b = a -> 두 값 교환이 아니라 기존의 a,b값을 이용해서 씀

# where절을 어케 쓰는지에 따라 달라짐 / 내가 작성한 이 조건이 의도한 레코드만 선택했는지 꼭! 확인해야함
# 단순 비교 : WHERE id = 1
# 범위 : WHERE age BETWEEN 20 AND 30
# 패턴 : WHERE name LIKE '김%'
# NULL : WHERE email IS NOT NULL
# 복합 : WHERE age > 20 AND city = 'Seoul'
# IN 연산자 : WHERE city IN ('Seoul', 'Busan')
# 서브 쿼리 : WHERE id IN ( SELECT user_id FROM orders )

# ---- 업데이트 쿼리 패턴 ----

UPDATE users
    SET age = age  +1
WHERE city = 'Seoul'AND age < 30;
# 조건에 맞는 애들을 일괄적으로 처리 가능함
# 실행 순서 where 조건절을 실행해서 대상 되는 레코드를 찾을 예정임
# 그 다음 age 조건으로 필터링 하고 / 현재 age값을 읽고 다시 age값에 기록을 함
# 이 과정을 진행하면서 락을 걸어서 다른애들이 수정 못하게 막음 (원자적 = atomic)하게 처리함 + 동시성 문제 발생하지 않음

UPDATE users
SET city = 'Busan'
WHERE city IN ('Incheon', 'Gyeonggi');
# or로 안하는 이유? -> 간결함!
# in 연산자는 내부적으로 해시 테이블이나 정렬된 리스트를 사용해서 매칭 실행
# 옵티마이저는 in절에 있는 값의 개수나 테이블의 크기를 고려해서 실행계획을 세움
# 값이 많으면 임시테이블을 만들어서 해줌 (옵티마이저가 알아서 골라줌)
# 만약에 city columns에 인덱스가 있음 이걸 활용해서 훨신 더 빠르게 계산할 수 있음 = Index Range Scan / 전체 테이블 스캔보단 효과적임

UPDATE users
SET city = 'Sejong'
WHERE age BETWEEN 25 AND 35; 
# between은 and조건이랑 같지만 걔보다 효율적으로 수립함
# 시간 복잡도는 O(log n + m) n : 전체 레코드, m :범위 내 레코드

# ---- 수치 연산 ----

UPDATE users
SET age = age + 1;
# 동시성 환경에서 원자성 보장 안해주면 충돌 남 / 원자적인 레벨을 인정해줘야함

UPDATE users
SET score = score + 10
WHERE city = 'Seoul' AND age > 25;

# SELECT score FROM users WHERE id = 1; // 현재 점수: 100
# 애플리케이션에서 100 + 10 = 110 계산
# UPDATE users SET score = 110 WHERE id = 1;

# --> LB가 있다고 가정하고, 분산 락
# A : SELECT로 100을 읽음
# B : SELECT로 100을 읽음
# A : Update 110으로 저장
# B : UPdate 110으로 저장

# 덧셈 : price = price + 100
# 뺼셈 : stock = stock - 5
# 곱셈 : price = price * 1.1
# 나눗셈 : avg = total / count
# 나머지 : remain = value % 10
```

> 이제 고급으로 가보자구요. case문 sub query를 활용하자요
> 

```sql
# 조금 더 어려운 업데이트 문
# case문을 활용한 조건부 업데이트
# 복잡한 비즈니스 요구사항과 로직을 처리할때 자주 씀

UPDATE users SET grade = CASE
    WHEN age < 20 THEN 'Junior'
WHEN age BETWEEN 20 AND 30 THEN 'Senior'
WHEN age > 30 THEN 'Expert'
ELSE 'Unkonwn'
END;

# CASE문은 위에서 아래로 순차적으로 돌리면서 제일 먼저 참인걸로 골라줌
# 제일 매칭이 잘 되는값을 위에 쓰면 성능 향상에 좋음
# 구체적인 조건을 앞단에 써주는게 좋음

CASE age
    WHEN 20 THEN '120'
    WHEN 30 THEN '1201'
    ELSE '40'
END

#검색 케이스가 유연하게 작동하기 좋음
CASE
    WHEN age < 20 THEN 'Junior'
    WHEN age BETWEEN 20 AND 30 THEN 'Senior'
    WHEN age > 30 THEN 'Exprt'
ELSE 'Unkonwn';

#---- 서브쿼리써서 업뎃하기 ----
# 다른 테이블에 데이터를 참조하면서 복잡한 업데이트 로직 구현

UPDATE users AS u SET city = (
    SELECT city FROM user_preferences AS up WHERE up.user_id = u.id
)
WHERE EXISTS(
    SELECT 1 FROM user_preferences AS up WHERE up.user_id = u.id
)

# 위의 코드 순서도
# u 테이블의 각 레코드를 순회
# 각 레코드에 대해서 EXIST
# up에 해당 user_id
# 존재하면다면 -> SET -> city
# O(n * log m)
# n : users 레코드 수, m : user_preferences 레코드 수
# Exists랑 In을 비교하면 Exist가 조금 더 안정적임 -> 일치하는 레코드 찾으면 더 진행 안함

# -------------------------------------------

UPDATE users AS u # 10,000개 있다고 가정하면 1만번의 서브쿼리를 진행해야함. 결론적으로 말하자면 조인 쓰는게 좋음
    JOIN user_preferences AS up on u.id = up.user_id
SET u.city = up.preferred_city
    WHERE up.preferred_city IS NOT NULL;
    
# 다른테이블과의 관계를 동기화 하면서 업뎃 가능
# join을 활용해서 안정적으로 동기화 가능함
# 쿼리 분석하면서 조인 순서 결정함
# 조인들 간의 알고리즘들 중 괜찮은 걸 옵티마이저가 골라줌
# 잠금을 어케 진행할건지도 고민함
# 조인된 테이블에 레코드만 읽는 경우가 많음 -> 읽는 요청에 대해서는 shared lock(읽기전용락을 진행함)

# --------------------------------------------
# Limit을 활용해서 배치단위로 데이터 처리하기 

UPDATE users AS u SET status = 'inactive'
WHERE last_login < DATE_SUB(NOW(), INTERVAL 1 YEAR)
LIMIT 1000;

# limit을 1000개만 하는거니까 데이터 핸들링하는거 자체는 양이 좀 적어짐
# 메모리 사용양을 줄이면서 안전하게 처리하게됨
# 그래서 limit을 왜 걸까? 
# -> 기본적인 메모리 사용량 + 한계가 없으면 수십만개를 한번에 업뎃함 -> 업뎃하는 도중에는 lock이 걸려서 undo log값도 커짐(롤백위해서 변경전 데이터 저장하는거)

UPDATE users AS u
    SET city = IF(age > 30, 'Seoul', 'Busan'),
        status = IF(city = 'Seoul', 'pr', 'st')

# IF : 참/거짓, 중첩도 가능 (가독성 저하)
# CASE : 조건이 많아도 깔끔, 순차적인 평가(shrot-circuit), 가독성 좋음
```