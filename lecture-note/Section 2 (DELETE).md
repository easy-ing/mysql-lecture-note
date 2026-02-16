# Section 2 (DELETE)

> 어떤 삭제가 있는지, mysql에서는 어떤 방식으로 진행되는걸까?
Delete : DB에서 레코드를 물리적으로 완전히 제거하는 행위
함부로 지우면 위험해용 복구가 어렵거든요~

DELETE 전 확인할 것!!
1. where 조건
2.데이터 범위
3.백업
4.외래 키 관계
- 외래 키도 여러 옵션이 있음
- Restrict (기본값) / CASCADE / SET NULL / NO ACTION

실행하는 과정
1. 쿼리 파싱
2. 권한확인
3. where 조건 확인
4. 외래 키 검사, 잠금 획득 
5.레코드 삭제
6. undo log에 업뎃
7. Index 관련 업뎃
8. 다 완료? → 커밋
> 

```sql
# 기본 조건 where는 거의 필수임!
DELETE FROM table_name WHERE <조건>

# 조건은 다양하게 줄 수 있음
# -> 단순 비교, 범위, 패턴, NULL, 복합, IN, 서브쿼리

# DELETE은 단일 테이블 단위로 하세용
DELETE FROM users
WHERE age < 18;

DELETE FROM users
WHERE created_at < '2024-01-01';

DELETE FROM users
WHERE city = 'Seoul' AND age > 65;

DELETE FROM users
WHERE email IN ('user1@test.com', 'user2@test.com', 'user3@test.com');

DELETE FROM users
WHERE email LIKE '%@test.com';

-- 한 번에 1000개씩 삭제
DELETE FROM users
WHERE status = 'deleted'
LIMIT 1000;

-- 비효율적: 인덱스 사용 불가
DELETE FROM users
WHERE DATE(created_at) < '2024-01-01';

-- 효율적: 인덱스 사용 가능
DELETE FROM users
WHERE created_at < '2024-01-01';
```

> 조금 더 어렵고 고급화된 형태를 배워보아요
2가지 정도의 패턴 배워요!
> 

```sql
DELETE FROM users
WHERE id NOT IN (SELECT DISTINCT user_id FROM orders);
# 다른 테이블 참조하면서 데이터 지우기 가능
# NOT IN -> 목록에 없는거 지우기
# 서브 쿼리 실행 -> user_id 확인 -> 중복제거 -> 여러개 정보 중에서 In을 활용해서 묶고 NOT 확인하면서 없는거 지움
# 서브쿼리 -> 메모리 많이 씀

-- NOT EXISTS 사용 (위에 코드 보다 더 안전하고 효율적)
DELETE FROM users
WHERE NOT EXISTS (
    SELECT 1 FROM orders WHERE user_id = users.id
);

# 여러 조건 결합해서 복잡한 형태로 데이터 많이 관리할거야
# 여러 조건 결합 = 정확한 대상 고르기 가능
# where조건을 먼저 탈것임 -> 선택도가 높은거부터 고름
# where조건에는 레코드를 삭제하거나 필터링할수있는 애를 넣어주는게 좋음
DELETE FROM users
WHERE status = 'inactive'
  AND last_login < DATE_SUB(NOW(), INTERVAL 2 YEAR)
  AND NOT EXISTS (
    SELECT 1 FROM orders
    WHERE user_id = users.id
      AND order_date > DATE_SUB(NOW(), INTERVAL 1 YEAR)
);

# ---- 다른 테이블과 조인하여 삭제 ----
# 잘 안씀 왜? 무서웡 -> 대용량 데이터 막 지웠다가 롤백이 힘들어 

DELETE u FROM users u
      JOIN inactive_list il ON u.email = il.email
WHERE il.inactive_date < DATE_SUB(NOW(), INTERVAL 6 MONTH);

# Soft Delete ( 논리적 삭제 ) - 완전 삭제 대신 삭제 표시 쉽게 말해 데이터 복구가 가능해~
UPDATE users
SET deleted_at = NOW()
WHERE email = 'withdraw@example.com';
```