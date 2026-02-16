# Section 3 (Nested Loop join, Matched Function)

> GOAL : LEFT JOIN에 대해 딥하게 알아보기!
> 

오늘의 궁금증

1. Left Join을 실행할때, DB 엔진이 메모리에서 행을 어떻게 처리할까?
2. 집계 쿼리에서는 숫자가 틀릴 수 도 있다 → 왜?

### Nested Loop JOIN

> 실제 DB엔진이 메모리에서 행을 읽고 비교하는 구체적인 패턴
마치 이중 루프를 적용하는 것 처럼 DB에서도 반복문 실행함

3가지의 핵심 루프가 존재함
1. 외부루프 - 왼쪽 테이블 순차적으로 스캔 = driving row = 핸들링 하는 row값 = Join연산을 주도하는 행
2.내부루프 - driving row 1개당 오른쪽 테이블 전체를 스캔함
3.Matched Flag - Boolean 변수로 스택 메모리에 저장시킴 / 그리고 내부루프 다 순회하고 마지막에 평가로 되는것임
> 

```sql
FOR EACH row_left IN left_table: # 외부루프 
     matched = false

     FOR EACH row_right IN right_table: # 내부루프
         IF match_function(row_left, row_right) IS TRUE:
             OUTPUT (row_left + row_right)
             matched = true # matched flag  m  

     IF matched = false:
         OUTPUT (row_left + NULL)
```

### ON절

> Left Join에서는 이걸 Matched Function이라고 부름
단순한 필터조건은 아님. 함수에 가까움
Boolean 표현식을 평가하는 함수와 비슷함
> 

```sql
match(u,p) = (u.id = p.user_id)

#순서도
#u.id의 컬럼 값 추출
#CPU에서 p.user_id랑 단일 비교 연산을 할 것임 
#결과가 boolean값으로 나옴
#앞서 있던 matched를 해당 값을 기반으로 바꿈
#마지막에 출력되는 연결된 결과를 사용함
```

### matched flag

> 얘는 이진상태임. 
false → true로 한번만 전환되고
다시 false로 돌아가지 않음
왜? 최소 1개의 매칭이 있는지 없는지만 확인하는 용도임
>