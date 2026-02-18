# Section 6 (SLOT)

<aside>
💡

실행 모델에 대해서 설계하기
반복되는 이벤트에 실제 인스터를 표한하고
렌더링하는 SLOT이라는 구조를 도입하기

</aside>

### SLOT?

필요에 따라 속성이나 렌더링과의 기록을 분리하는 패턴

복잡한 반복 이벤트를 효율적으로 설계하고 처리함\

### DDL

```sql
-- 하루 종일 Slot 테이블
CREATE TABLE day_slots (
    slot_id VARCHAR(50) PRIMARY KEY,
    event_id VARCHAR(50) NOT NULL,
    slot_date DATE NOT NULL, # "2024-01-01"
    is_skipped BOOLEAN DEFAULT FALSE, # 매주 월요일 회의 1월 15일만 취소하고싶다,
    is_rescheduled BOOLEAN DEFAULT FALSE, # 1월8일 월요일 회의가 오후 2시 -> 오후 3시
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (event_id) REFERENCES day_events(event_id) ON DELETE CASCADE
);

-- 시간 Slot 테이블
CREATE TABLE time_slots (
    slot_id VARCHAR(50) PRIMARY KEY,
    event_id VARCHAR(50) NOT NULL,
    slot_datetime DATETIME NOT NULL, # "2024-01-01 14:00:00"
    is_skipped BOOLEAN DEFAULT FALSE,
    is_rescheduled BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (event_id) REFERENCES time_events(event_id) ON DELETE CASCADE
);
```

반복 이벤트는 한개의 정의지만

실제는 여러개의 인스턴스가 생성되기 때문에

체계적으로 관리하는 구조가 필요함

특히 반복규칙이 커질수록 시간도 오래걸림

특정 DB레벨에 대해 오버헤드 없이 값을 뱉어냄

### How to run?

캐시, 뷰, 테이블 생각하기

반복 이벤트 정의를 기반으로 실제 인스턴스를 미리 생성해서 저장

필요할때 빠르게 조회!

### How to make?

Pre-generation : 반복 이벤트가 생성될 때, 미리 앞으로 발생할 모든 인스턴스를 계산하기

미리 계산한걸 SELECT로 가져오니까 시간을 줄일 수 있다!

하지만 저장공간을 많이 씀 ㅜ

이건 DDL 설게보단 애플리케이션에서 어떻게 설계한 것인지에 대한 관점

On-demand : 사용자가 캘린더를 조회할때 마다, 그 시점에 필요한 인스턴스만 동적으로 계산

저장공간 아낄수있지만 매번 계산하니까 CPU 사용이 많고

로딩 시간이 좀 더 걸림

우린 이 2개를 하이브리도 쓰는거지요!

### 랜더링과 기록의 분리

랜더링이란? 사용자가 캘린더 화면을 보면 이벤트들을 화면에 표시하는 과정 (UI)

랜더링된 데이터는 빠르게 보여질 수 있도록 최적화된 데이터임.

기록욕 데이터 : 데이터의 정확성과 무결성이 중요

삭제하거나 복구할때 얘를 기준으로 함