# Section 6 (Event)

<aside>
💡

이벤트에 대한 정의 + 물리적으로 저정하는 방법 & 개념
그에 따른 DDL 작성

</aside>

### 용어를 정리해보아용

> Anchor : DB에서 중심이 되는 역할 + 카운트가 가능한 실제 엔티티 (USER, Day Event) → 비즈니스 도메인에 있는 실제!
> 

Anchor 설정이 참 중요해용

```sql
# 이것이 유저를 표현하는 Anchor
CREATE table  users (
    user_id VARCHAR(50) PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(100),
    timezone VARCHAR(50) NOT NULL DEFAULT  'UTC',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

# user_id를 사용자가 고유하게 식별할 수 있는 기본키로 씀
```

Day Event 작성하기

DayEvent ( 하루 단위로 시작/끝이 있는 이벤트 )
begin_date == end_date

```sql
CREATE TABLE  day_events (
    event_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    begin_date DATE NOT NULL,
    end_Date DATE NOT NULL,
    user_id VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);
```

---

유저와 데이 이벤트 관계는 1:N의 형태를 가짐

이걸 어떻게 Link로 관리해서 연결해줄까?
-> User creates several DayEvents,
-> DayEvent is created by only one User

이렇게 쓰는 이유가 뭐야? 처음부터 한개에 때려넣지

관계의 의미를 명확하기 하기 위함!

관계방향을 명확하게 해야지 로직을 잘 맞추어서 할 수 있음!

```sql
-- 시간대 테이블 (Timezone Anchor)

CREATE TABLE timezones (
    timezone_id VARCHAR(50) PRIMARY KEY,
    timezone_name VARCHAR(100) NOT NULL, # Asia/Seoul, America/New_York
    utc_offset VARCHAR(10) NOT NULL,
    is_dst BOOLEAN DEFAULT FALSE
);

-- 요일 테이블 (DayOfTheWeek micro-anchor)

CREATE TABLE days_of_week ( # 반복이벤트에서 특정 요일만 반복하는것
    day_id VARCHAR(10) PRIMARY KEY,
    day_name VARCHAR(20) NOT NULL, # Monday, Tuesday
    day_order TINYINT NOT NULL # 요일의 순서
);

-- 시간 이벤트 테이블 (TimeEvent Anchor)

CREATE TABLE time_events ( # 특정시간에 시작하고 끝나는 이벤트
    event_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    start_time DATETIME NOT NULL,
    end_time DATETIME NOT NULL,
    location VARCHAR(255),
    user_id VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- 시간 이벤트와 요일 관계 테이블 (M:N Link)

CREATE TABLE time_event_weekdays (
    event_id VARCHAR(50) NOT NULL,
    day_id VARCHAR(10) NOT NULL,
    PRIMARY KEY (event_id, day_id),
    FOREIGN KEY (event_id) REFERENCES time_events(event_id) ON DELETE CASCADE,
    FOREIGN KEY (day_id) REFERENCES days_of_week(day_id) ON DELETE CASCADE
);

# 얘로 다대다 관계를 정의함.
# 중복도 확인해서 데이터 무결성을 지켜줄것임
```

---

> Attribute : 독립적으로 존재하지 않고 Anchor에 대해 붙어서 쓰임. 특정 Anchor에 대한 특징이 됨.
> 

대부분 논리타입으로 많이 쓰여요 string, data, integer