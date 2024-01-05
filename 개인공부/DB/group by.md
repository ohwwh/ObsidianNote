
![[Untitled.png | 500]]

group by A 를 하면, 같은 값을 가진 행이 하나의 그룹으로 합쳐진다.
Distinct 처럼 중복된 행을 제거하는 것이 아니다.
만약 집계 함수와 함께 쓰이면, 그룹 별로 집계 함수를 적용한 결과값을 그룹 별 단일 행으로 반환한다.
```sql
select color, count(*) from table_example group by color
```
위와 같은 SQL의 경우, count(\*)에서 \*의 의미는 ‘그룹 전체’를 의미한다. 위의 예시에서 Red, Blue, NULL 3개의 그룹이 나왔으므로, count(\*) 연산을 3번 하는 꼴이 되며 3개의 행에 각각 Red = 2, Blue = 1, Null = 1 이런 식으로 출력된다.


**Q1. 처음에 select를 할 때, count(\*)를 뭘로 알아듣는거지? group by 시키기 전인데??**
[[SQL 형식의 실행 순서]] 참조
```SQL
select color, count(*) from table_example where price <= 3000 group by color having count(*) >= 0 order by count(*)
```
이 SQL을 예시로 들면 아래 같은 순서로 뽑힌다고 할 수 있다.
1. table_example 테이블에서 
2. price가 3000원 미만인 row를 일단 뽑는다
3. 이 가상의 테이블1에서, color가 같은 그룹 별로 묶는다.
4. 묶은 그룹 중, 그룹의 원소 개수(count(\*))가 0보다 큰 것들만 추린다
	1. Select \* from table_example where price <= 3000이라는 서브 쿼리를 뽑는다고 생각하자
5. 추린 가상의 테이블2에서, 그룹으로 묶인 color와 그룹 별 count(\*)를 각각 컬럼으로 만든 후 이어 붙여 테이블3를 만든다.
	- 테이블에서 Select 한다고 하기에는 count(\*)가 원래 테이블에 있는 컬럼은 아니니, 이런 식으로 설명하였다. 
	- ***group by 뒤에 있는 컬럼 외의 컬럼을 select나 order by 하면 안 된다.***
		- group화의 기준인 color를 제외한 나머지 컬럼의 경우, 수많은 레코드 중 랜덤한 값으로 합쳐지기 때문에 의미가 없는 값이 되어 버린다.
		- 예를 들어, color = red, price = 3000이라는 레코드와 color = red, price = 2000이라는 레코드가 group화 될 경우, price가 3000일지 2000일지 알 수가 없다는 것
6. 테이블3를 count(\*)에 따라 정렬한다


**Q2. group by 뒤에 속성이 2개 이상이라면 어떤 식으로 동작하나**
ex. group by color, price라고 하면, 가능한 모든 경우의 수로 그룹을 만들게 된다.
즉, (빨강, 3000원), (빨강, 1000원), (파랑, 2000원), (노랑, 1000원), (노랑, 500)원 같은 모든 경우에 수에 대해 그룹을 생성한다.

