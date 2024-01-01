## 개요
---
- [[2023-12-22]]
- 오전 QA 빌드 이후, DB 빌드 실패 & QA 빌드 증발
- 런덱 잡이 실패
	- [MapleM China - DB_Update](http://54.168.224.205:4440/project/MapleStoryM-China/job/show/723175b9-ab68-4693-9108-ab2edea0947a)
- DBManager가 뽑은 스크립트(C:\SE_DIR\Patch\Application\Sql\China\QA)들을 확인해 보니, 모든 데이터베이스를 Drop하는 SQL을 만들어 놓음

## 원인
---
- 중국 QA DB에서, 대소문자는 무시하도록 설정 변경
- QA Dummy DB는 해당 작업이 누락됨
	- QueryGenerator에서 쿼리를 뽑을 때 모든 테이블 이름은 대문자로 바뀌어서 나온다.
	- 하지만 대소문자 무시가 되어 있다면(lower_case_table_names_ = 1), 쿼리를 실행해도 테이블 이름이 자동으로 소문자로 바뀐다
	- 대소문자 무시를 하지 않으면(lower_case_table_names_ = 0), 대문자로 등록됨
- DBManager가 Dummy의 대문자 테이블과 원본의 소문자 테이블을 비교하여 Diff가 제대로 안 나오게 됨
	- 이게 원인인듯 한데??(추정)
	- 하지만 그렇다면 원본의 소문자 테이블을 모두 지우고 대문자 테이블을 만들어야 할 텐데, 왜 drop만??
	- 그리고 런덱 로그 상으로 이 drop sql의 실행은 실패한 것으로 보임 => 이미 QA DB는 사라져 있었음
	- 전혀 모르겠다


## TroubleShooting
---
- 테스트 용으로 내 로컬 MySQL 인스턴스의 lower_case_table_names를 바꾸고자 함
	- set lower_case_table_names = 0 으로 바뀌지 않음
		- my.ini를 수정하고 인스턴스 재시작해야 함
	- my.ini 수정 후 재시작 시 중단됨
		- lower_case_table_names = 1인 경우에만 정상 실행되고, 0이면 중단됨......
		- [MySQL :: MySQL 8.0 Reference Manual :: 5.1.8 Server System Variables](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_lower_case_table_names)
			- You should _not_ set [`lower_case_table_names`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_lower_case_table_names) to 0 if you are running MySQL on a system where the data directory resides on a case-insensitive file system (such as on Windows or macOS). It is an unsupported combination that could result in a hang condition when running an ``INSERT INTO ... SELECT ... FROM _`tbl_name`_`` operation with the wrong _`tbl_name`_ lettercase. With `MyISAM`, accessing table names using different lettercases could cause index corruption.
			- **윈도우에서는 저걸 0으로 설정할 수 없음**
