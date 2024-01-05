## 정의
---

데이터베이스에 대한 일련의 작업을 정리한 절차를 관계형 데이터베이스 관리 시스템에 저장한(지속성) 것
= **SQL 묶음**

## 사용법
---

```sql
DELIMITER $$
CREATE PROCEDURE 프로시저_이름 (IN 또는 OUT parameter)
BEGIN
//내용
END$$
DELIMITER ;
```

**Q1. Delimiter는 뭐죠?**

구분자를 ;에서 일시적으로 \$$로 바꿔주는 것. 이걸 안 쓰면, SP내부의 쿼리의 끝에 ;가 나왔을 때 이게 개별 SQL의 끝인지 전체 프로시저의 끝인지 모호해진다.