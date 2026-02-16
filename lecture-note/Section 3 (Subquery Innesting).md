# Section 3 (Subquery Innesting)

> 다중 집계 쿼리 설계패턴 공부하기!
여러가지 통계를 동시에 조회함
> 

오늘의 궁금증

1. 다중 Left Join이 메모리에서 어떻게 행을 생성하는지 파악하기

다중집계쿼리를 작성하는 방법

1. 직접 조인하기

```sql
# 직접 JOIN
SELECT
    u.id,
    u.name,
    COUNT(p.id) AS post_count,
    COUNT(c.id) AS comment_count
FROM users u
         LEFT JOIN posts p ON u.id = p.user_id
         LEFT JOIN comments c ON u.id = c.user_id
GROUP BY u.id, u.name;
# 두 개 이상의 1:N 관계를 동시에 물리적으로 가져온다.
# 1:N & 1:M으로 행이 N*M으로 기하급수적으로 늘어남
```

1. CTE - 각 집계를 독립적인 CTE로 분리하고 최종적으로 1:1 관계만들어서 조인함 (근데 안정성이 1번 보다 좋음)

```sql
WITH user_base AS (
    SELECT id, name FROM users
),
     post_stats AS (
         SELECT user_id, COUNT(*) AS post_count
         FROM posts
         GROUP BY user_id
     ),
     comment_stats AS (
         SELECT user_id, COUNT(*) AS comment_count
         FROM comments
         GROUP BY user_id
     )
SELECT
    u.id,
    u.name,
    COALESCE(p.post_count, 0) AS post_count, # 2
    COALESCE(c.comment_count, 0) AS comment_count # 3
FROM user_base u
         LEFT JOIN post_stats p ON u.id = p.user_id
         LEFT JOIN comment_stats c ON u.id = c.user_id;
# Dimension Reduction ( 차원 축소 )
# 나눠서 CTE 진행하니까 최대 N을 가진 유니크 키로 줄여짐
# 줄여진 값을 가지고 조인하니까 1:1 관계가 된다
```

1. Correlated Subquery - Select절에서 서브쿼리를 써서 각 집계를 독립적으로 사용하기(가독성 베리굿)

```sql
SELECT
    u.id,
    u.name,
    (SELECT COUNT(*) FROM posts p WHERE p.user_id = u.id) AS post_count,
    (SELECT COUNT(*) FROM comments c WHERE c.user_id = u.id) AS comment_count
FROM users u;

# 개념적으로 외부쿼리의 각 행마다 서브쿼리를 실행함 (느림ㅜ)
# 잘 안씀 CTE 를 쓰세용 이건 비추비추
# 조건 잘 맞추면 CTE 처럼 쓸 수 있긴해 => AST 맞추기
```