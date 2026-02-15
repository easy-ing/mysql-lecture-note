# Section 2 (INSERT)

기본적인 개념이 되는 쿼리 정리

### INSERT

> 가장 기본적인 처리, DB에 저장되는 행위
> 

```sql
#INSERT

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    age INT,
    city VARCHAR(100),
    metadata JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

INSERT INTO TABLE_NAME (col1, col2 ,,, ) VALUES (value1, value2, ,,, )
#컬럼에 대해 어떤 값이 들어가는지 확인 할 수 있음 + 특정 컬럼이 추가된다고 해도 상관없음
#not null 조건은 꼭 값을 넣어주고, default는 생략시킬수있다

INSERT INTO users VALUES (NULL, 'a', 'a@naver.com', 30, 'Busan')
# users에 있는 값을 순서대로 넣은 것 -> 테이블 구조 변경하면 쿼리도 바꾸어줘야함

# 단일 ROW 삽입패턴
INSERT INTO users (name, email, age, city) VALUE ('철수','kim@naver.com',13,'Seoul')

# 다중 ROW 삽입패턴

INSERT INTO users (name, email, age, city) VALUE
    ('김철수', 'kim@naver.com', 13, 'Seoul'),
    ('김철수', 'kim@naver.com', 13, 'Seoul'),
    ('김철수', 'kim@naver.com', 13, 'Seoul'),
    ('김철수', 'kim@naver.com', 13, 'Seoul');
    # 단일로 4번쓰는것보단 네트워크에 따른 mysql을 왔다갔다하는 비용을 낮춤
    # 그리고 mysql은 쿼리가 들어오면 파싱해서 어떤식으로 활용할지 sequence를 짜는데 이걸 한번만 하면됨
   
   
# Bulk Write, STMT
# 다중 행을 입력했을때 실패가 생기면 누구때문인지 알기 어려움 -> 한개씩 진행시킬 수 있음

# 테이블간의 데이터를 복사
# 언제씀? 다른 테이블 데이터를 조회하면서 특정 테이블에 추가하는 Data Migration

# --> 다른 테이블을 조회하여 복사
INSERT INTO users (name, email, age, city)
SELECT name, email, age, city
FROM old_users
WHERE status = 'active';

'''
old uers에서 active 상태 인 애들을 밸류값으로
가지고 users에 값을 집어 넣은 상태임  
'''

# --> 조건부 데이터 복사
INSERT INTO users (name, email, age, city)
SELECT
    CONCAT('New_', name) as name,
    CONCAT('new_', email) as email,
    age + 1 as age,
    'Seoul' as city
FROM old_users
WHERE age > 25;

'''
데이터 변환과 삽입을 한번에 처리할수있음
mysql엔진에서 하나의 쿼리로 실행하는게
mysql입장에선 좋음 근데 이렇게까지 진행하는 경우는
거의 없음

그리고 뭐가 틀린지 확인하기 어려움
검증과정을 넣어야함
'''

# * ON DUPLICATED KEY UPDATE
# 중복된 키가 발생했을때, 업데이트를 수행하게 하는 방법임
# -> MongoDB Upsert과 비슷함

INSERT INTO users (id, name, email, age, city)
    value (100,'김철수', 'kim@naver.com', 13, 'Seoul')
ON DUPLICATE KEY UPDATE
    name = VALUES(name),
    age = VALUES(age),
    city = VALUES(city),
    updated_at = NOW();

# 기능이 유연함. 우리가 insert 진행하는 과정에서
# 유일성이 무너지면 아예 실행이 안됨.
# 근데 on duplicate key update를 쓰면 동적으로 진행하면서 (없으면 넣어주고 잇으면 업뎃해줘)

INSERT IGNORE INTO users (id, name, email, age, city)
    value
    (100,'김철수', 'kim@naver.com', 13, 'Seoul'),
    (100,'김철수', 'kim@naver.com', 13, 'Seoul');
    
# 중복키가 있다면 그건 에러로 생각하지 않음
# 일종의 컨티뉴와 같음 -> 중복키가 있으면 건너뛰기임
# 데이터가 미리 존재하는지 확인할 필요가 없음

# 그럼 이 2개의 쿼리 차이는 뭐임?
# 둘다 key값을 기준으로 select를 진행함
# 이후에 나머지 작업을 진행함. 

# REPLACE INTO
REPLACE INTO users (id, name, email, age,city)
VALUE (1, 'a', 'b', 3, 's')

# 이름과 같이 교체함
# id값이 있는지 확인하고 -> 있다면 이전것을 지우고 새로 삽입
# 외래키 제약조건이 있으면 문제가 생김
# 참조 무결성 문제도 생길수있음
# 그래서 이거보다는 위에 2개를 활용하는게 더 좋음 (삭제를 안하니까)

```

### INSERT(고급)

> 다양하고 최적화 하는것에 집중해서 보자구용
> 

```sql
# 대량의 데이터를 삽입할때 최적화하는 기법 중 한개
-- 트랜잭션으로 묶어서 성능 향상
START TRANSACTION;
INSERT INTO users (name, email, age, city) VALUES
    ('User1', 'user1@test.com', 20, 'Seoul'),
    ('User2', 'user2@test.com', 21, 'Busan'),
    -- ... 수천 개의 레코드
    ('User1000', 'user1000@test.com', 30, 'Daegu');
COMMIT; # 커밋하면서 트랜잭션 종료

# 원자성 보존에 좋음 - 중간에 문제가 발생해도 일관성 보존
# 커밋을 한번만 해서 IO 관점에서 효율적임
# 단위가 적으면 굳이 트랜잭션을 쓸 필요가 없음
# 데이터가 1만개 이상이면 쓰는게 좋음

INSERT INTO users (name, email, age, city) SELECT '새 사용자', 'test메일', 25, 'seoul'
    WHERE NOT EXISTS(
        SELECT 1 FROM users WHERE email = 'test메일' # 필요한 데이터가 있는지 확인하기 이후에 insert 작동
    );
    
    
# 존재와 삽입을 하나의 쿼리로 하면 성능상의 이점이 있음

INSERT INTO users (name, email, age, city)
    VALUE ('User1', 'user1@test.com', 20, 'Seoul')
ON DUPLICATE KEY UPDATE
    age = VALUES(age) + 1, -- 기존 나이에 1을 추가
    name = CONCAT('Updated_', VALUES(name)); -- 복잡한 로직을 하나의 쿼리로 쓸 수 있음 / 원래 있던 값에 추가를 해주는 것임
    
# 쿼리 상에 보이는거에 insert시도한 값에 +1을 업뎃 해줌
# values는 참조 할 수있게 해줌

# -------- 제약 조건 조심하기 ---------

-- 유니크 값 위반 안하게 조심하기
-- Not null 제약 조건 위반 안하게 조심하기 (유효한 값 제공해야함)
-- auto increment 값은 따로 공간을 잡아먹으면서 관리함 / on duplicated key update에는 ulid uuid를 주는게 좋음

# ------- 성능 최적화 --------

# insert에 대해서도 인덱스 관리해야함 -> insert가 많으면 힘들어
# 왜? 인덱스가 많으면 각 인덱스마다 업뎃해야하는 비용이 필요함

# ------ 파일에서 대량의 데이터 가져오기 --------
LOAD DATA INFILE '/path/to/users.csv'
    INTO TABLE users
    FIELDS TERMINATED BY ','
    ENCLOSED BY '"'
    LINES TERMINATED BY '\n'
    (name, email, age, city);
    
# 성능이 제일 빠르고 효율적인 방식임
```