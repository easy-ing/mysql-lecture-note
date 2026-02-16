# Section 3 (LEFT JOIN의 WHERE vs ON)

오늘의 목표

> 실무에서 가장 많이 실수하는게 
On절과 Where 조건절의 차이 이해랍니다.
이해합시다 우리

뭐가 달라용?
실행시점과 NULL처리 방식이 완전하게 다름

그래서 뭐가 중요해요?
Null Padding이 중요해요
ON은 Null Padding 전에 평가하지만 
Where은 Null Padding 이후에 합니다
> 

```sql
# FOR EACH row_left IN users:
#     matched = false
#     FOR EACH row_right IN posts:
#         IF match_function(row_left, row_right):  ← ON 절이 여기서 평가 / 내부루프 안에서 각 행 쌍마다 평가를 함
#             OUTPUT (row_left + row_right)
#             matched = true
#     IF matched = false:
#         OUTPUT (row_left + NULL)  ← NULL 패딩은 ON 절 평가 후, 매칭 규칙임

# ---- 이렇게 하는게 정배임 
SELECT u.id, u.name, p.id AS post_id, p.title, p.status
FROM users u
         LEFT JOIN (
    SELECT * FROM posts WHERE status = 'published' # draft
) p ON u.id = p.user_id; # 여기가 핵심임!!
```