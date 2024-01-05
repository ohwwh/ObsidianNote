DB Sharding이란 같은 테이블 스키마를 가진 데이터를 다수의 데이터베이스에 분산하여 저장하는 방법을 의미한다.

용어가 굉장히 이질적이지만, 쉽게 말해 똑같은 구조의 데이터베이스를 여러 개 만들어 자료를 나누어 담는다고 보면 된다.

이러한 DB Sharding은 데이터베이스 차원에서 제공되지는 않는다. 즉, MySql에 DB를 분산하여 관리해 주는 기능이 따로 탑재된 것은 아니다. 나뉘어진 데이터들을 어떤 식으로 조회하고 수정하는지(= sharding을 어떻게 할 것인지)는 애플리케이션 레벨에서 정해야 한다. 즉, 프로그램을 코딩하면서 shardIndex별로 적절히 관리할 수 있도록 직접 구조를 짜야 한다는 뜻이다.

메이플M 계정 정보를 global_maple_account_1, global_maple_account_2에 나누어 담는 것이 예시이다(4까지 있음).

global_maple_account_1 DB의 account 테이블을 보면, user_no가 모두 4로 나누어 떨어지는( % 4 = 0인) 것을 볼 수 있다.

처음에 로그인을 하고 캐릭터를 생성할 때, Identity 서버에서 user_no를 발급한다. 이 때 user_no를 %4 한 값을 shard_index로 같이 저장한다. 나중에 account 관련 정보를 저장할 때, global_maple_account_1~4 중 어디에 저장할 지를 이 shard_index를 보고 판단하게 된다.

여기서 user_no를 **샤드 키**라고 하며, 해당 키를 이용해 도출된(여기서는 단순히 모듈러 연산을 이용하지만, 다양한 기법들이 있다. [https://nesoy.github.io/articles/2018-05/Database-Shard](https://nesoy.github.io/articles/2018-05/Database-Shard) 참조) 값을 **샤드 인덱스**라고 한다.