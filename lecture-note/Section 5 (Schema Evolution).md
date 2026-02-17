# Section 5 (Schema Evolution)

<aside>
💡

서비스의 지속가능성이나 추가적인 기능 요구사항을 고려할 때, 스키마의 변경과 진화에 대해 배워보자!

</aside>

스키마 진화시키는법

1. ALTER TABLE → 스키마 변경하기

### ALTER TABLE

스키마 변경 = high cost = down time + Disk

테이블 복사 방식으로 진행 ⇒ 온라인 DDL을 통해 어느정도 완화 가능

Ex) JSON, EAV 사용

```sql
# 기본 틀
ALTER TABLE <테이블 명> ADD COLUMN <컬러명> <옵션>;
# 전체 테이블을 재작성해야함 그렇다면 EXCLUSIVE LOCK 걸릴수밖에

# 다른 방식으로 쓸 수 있음 (MySQL 5.6이상의 버전, Online DDL)
# 다운타임 최소화 + 읽기랑 쓰기 하게 해줌
ALTER TABLE <테이블 명> ADD COLUMN <컬러명> <옵션>, ALGORITHM=INPLACE, LOCK=NONE;
```

### 실행 방식 제어하는 키워드

> 
> 
> 
> Algorithm 
> 
> Inplace : 테이블 복사 안하면서 인덱스 재구성
> 
> Copy : 기존 방식으로 테이블 복사
> 
> Inplace : 인덱스 재구성하는 방식 (Online DDL)
> 
> Default : mysql이 자동으로 고름
> 
> Lock 
> 
> None : lock 없음
> 
> Shared : 공유 lock
> 
> Exclusive : 베타락 씀
> 
> Default : mysql이 지가 고름
> 

### Online DDL

컬럼 추가 삭제 인덱스 추가 삭제

이런것들은 온라인 DDL 지원

근데 데이터 타입 변경 이런건 잘 지원 안해줌

```sql
-- Online DDL 미지원 작업 (COPY 방식으로 실행됨)
-- 1. 데이터 타입 변경
ALTER TABLE restaurants
    MODIFY COLUMN name VARCHAR(200),
    ALGORITHM=COPY, LOCK=EXCLUSIVE;

-- 2. 컬럼 이름 변경
ALTER TABLE restaurants
    CHANGE COLUMN name restaurant_name VARCHAR(100),
    ALGORITHM=COPY, LOCK=EXCLUSIVE;
```