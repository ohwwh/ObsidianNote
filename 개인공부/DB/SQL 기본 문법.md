- group_concat 문법
- case, sum 문법
	- sum 안에 case를 쓰는 경우
- in 문법
```SQL
SELECT * FROM table_example WHERE c1 IN (p1, p2, p3)
SELECT * FROM table_example WHERE (c1, c2) IN ((p11, p12), (p21, p22), (p31, p32))
SELECT * FROM table_example WHERE c1 IN t1 #t1은 서브쿼리
```
1. c1 in (p1, p2, p3)는, (c1 = p1) or (c1 = p2) or (c1 = p3)과 같은 표현이다.
2. (c1, c2)과 같이 복수의 컬럼을 넣어서 비교할 수도 있다.  단, IN 뒤의 괄호 속 원소들 역시 예시 처럼 괄호로 묶인 복수의 원소를 가져야 함 => 형식이 맞아야 함
3. in 뒤에 subquery를 넣을 수 도 있다. 이 subquery는, Select c1 혹은 Select c1, c2 같은 식으로 앞에서 비교하는 컬럼의 갯수와 형식이 맞아야 함