# Section 3 (JOIN)

> GOAL : 다양한 다른 테이블을 참조하면서 무결성을 검증하고 데이터를 표현하는 방법을 배우기
> 

### INNER JOIN

> 두 테이블에서 조건이 일치하는 행만 반환하기
만약 매칭이 안된다면 해당 행은 결과에서 완전히 제외
> 

```sql
# 기본 문법
SELECT columns
FROM table1 INNER JOIN table2 on table1.column = table2.column;
```

그래서 이너 조인은 어떤 알고리즘 인가요?

매칭되는 행만 출력하고 끝!

매칭이 안된다면? 그건 바로 무시

그렇다면 핸들링하는 데이터가 적기 때문에 레프트 조인보단 최적화 하기 좋음

MYSQL OPTIMIZER 입장에서는 더 작은 테이블을 먼저 스캔하거나 인덱스를 활용한다

### LEFT JOIN = LEFT OUTER JOIN

> INNER JOIN의 확장 버전 = 왼쪽 테이블의 모든 행을 보존하면서 오른쪽에 매칭되는 행을 결합한 것
매칭되는 것이 없다면?! → 무시함

언제써용?
전체목록은 기본적으로 유지 + 관련된 정보 추가하기

주의할 점은 뭔가용?
LEFT JOIN 후 
where절에서 오른쪽 테이블의 컬럼을 필터링하면
INNER JOIN처럼 동작을 합니다
> 

```sql
# INNER JOIN과의 차이점

matched = true # 해당 플래그를 활용한다!

# matched가 뭔데!? -> 내부 루프를 완전히 순회한 후 평가한다
# 한번도 매칭이 안된다면 matched flag는 FALSE가 된다
```

### RIGHT JOIN = RIGHT OUTER JOIN

LEFT랑 반대임. 방향성만 달라짐

똑같아유

### FULL OUTER JOIN = LEFT JOIN + RIGHT JOIN

> 양쪽 테이블의 모든 행을 보존 해 버림 = 모든 행을 반환함
매칭되는 결과가 있으면 결합하고 없으면 각각 알아서 NULL취급!
FULL OUTER JOIN은 mysql에서는 지원 X

그럼 이거 왜씀?
데이터의 무결성을 검증하기 위해서!
> 

### CROSS JOIN

> 조건 없이 두 테이블에 모든 가능한 조합을 생성
Cartesian Product라고 부르기도 함!
두 테이블에 Cartesian 곱을 한 것임

쉽게 말해서 왼쪽 테이블의 각 행이
오른쪽 테이블의 모든 행과 결합되어 버림
즉, ON 조건이 따로 필요 없음!

언제 사용하는 건가요?
모든 조합을 생성할 경우
테스트 데이터 생성할때~
분석쿼리에서 기존 데이터셋 만들기 위해서

결과에 대한 행수가 많아질수록 조심해야함 🙂
>