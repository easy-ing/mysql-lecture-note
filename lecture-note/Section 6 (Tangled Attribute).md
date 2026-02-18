# Section 6 (Tangled Attribute)

<aside>
💡

반복되는 이벤트 처리를 위한 이벤트 모델링하기!
Ex) 반복주기, 주기단위 등 체계적으로 구조화하기

</aside>

### 다뤄야 하는 것

1. 반복 주기 → 문자열로 처리해서 유연성 확보 (다양한 패턴 지원)
2. 주기 단위 → 정수로 처리해서 정확하게 처리하기

### Tangled Attribute

→ 의미가 있는 속성 = 특정한 조건이 있는 경우에만 작동되는것!

이거 왜써? 데이터 일관성은 유지하지만 유연하게 처리 가능하니까!

어떤식으로 만들어? 일단 개념은 ‘조건부 속성’을 통해 처리하기!

2가지 방식이 있어

1. 별도 테이블로 구성
2. 조건부 컬럼을 만들기

```sql
# for repeated events only

CREATE TABLE recurring_rules (
    rule_id VARCHAR(50) PRIMARY KEY,
    event_id VARCHAR(50) NOT NULL,
    recurrence_type VARCHAR(20) NOT NULL, # 반복 주기
    recurrence_interval INT NOT NULL DEFAULT 1, # 반복 간격
    end_type VARCHAR(20) NOT NULL, # 반복이 종료가 되는 조건 "forever" ,"until_date", "N_repetitions == count"
    end_date DATE,
    end_count INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
); 

# 하나의 테이블에서 3가지 종료 조건을 다룰수있어서 좋음
```