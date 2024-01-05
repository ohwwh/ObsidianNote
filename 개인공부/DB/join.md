## join
---
![[a8.27.21.png]]
select * from **topic** left join **author** on topic.author_id = [author.id](http://author.id/);

→ topic 테이블의 author_id과 autor 테이블의 id를 비교하여 둘이 같으면
1. 일치하는 각 행을 순서대로 붙여 새로운 행을 만든다(이 과정에서 중복되는 column은 삭제된다)
2. 만들어진 행을 하나씩 같다 붙여서 새로운 테이블을 만든다고
고 이해하면 된다.


## left join
---
left = left outer의 준말이다.
해당 키워드를 이해하기 위해, inner join, outer join의 개념을 이해해야 한다. 간단하게 말하면 교집합과 합집합 같은 느낌이다.
1. inner join
    
    조건을 비교해서, 일치하는 절만 출력한다. 무슨 말이냐면, 위의 예시에서 author_id 가 4인 행이 topic 테이블에 추가되고, id가 5인 행이 author 테이블에 추가된다고 하자. inner join을 하는 경우, 일치하지 않는 행은 양쪽 테이블에서 모두 버려지고 출력된다. 즉, 새로 추가된 두 행 모두 출력되지 않는다.
    
2. outer join
    
    조건을 비교해서, 일치하는 절만 출력하되 한 쪽 테이블은 보존한다. 무슨 말이냐면, 만약 left outer join을 하면 select 명령문의 왼쪽 - 위의 select문에서는 topic - 테이블의 행들은 모두 보존된다. 예를 들어 author_id = 4와 일치하는 id가 author쪽에 없다면, author쪽에 id = 5인 행은 날아가지만 author_id=4인 행의 오른쪽은 모두 null로 채워진다.
    


## 예시
---
직접 결과를 보는 것이 가장 이해가 빠르다:

다음과 같은 두 테이블에 대하여
![[a9.32.40.png | 500]]
- select * from topic inner join author on topic.author_id = [author.id](http://author.id); 명령어의 결과는
![[a9.35.06.png]]
- select * from topic left join author on topic.author_id = [author.id](http://author.id); 명령어의 결과는
![[a9.35.40.png]]
- select * from topic right join author on topic.author_id = [author.id](http://author.id); 명령어의 결과는
![[a9.36.01.png]]