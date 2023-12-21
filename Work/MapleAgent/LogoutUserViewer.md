

- Secession 끄는 법
    - WindowsServiceScheduler의 Settings.settings를 들어가서, Enable_Secession을 false로 설정한다
- 서비스 이름 바꾸는 법
    - Settings.settings를 들어가서, ServiceName을 변경한다
- 서비스로 등록 안 한 이유
    - 서비스로 등록하면 SlackManager 외에도, 주기적으로 도는 기능(동접하락, 덤프 수집 등등)이 기존 MapleAgent와 겹치기 때문
- Logoutuser 구현 - 채널 827_동접하락, [https://maplem.slack.com/archives/G01DT948W3X/p1691634800101179](https://maplem.slack.com/archives/G01DT948W3X/p1691634800101179) 참조
- 기존 LogoutUserViewer 흐름
    1. [runserver.py](http://runserver.py), elif(serviceType==’Live’) 인 경우
        1. 리전(글로벌: asia1 asia2…) 별로 LRUDict(크기 제한 100000) 생성
        2. LocalDBInit 호출: 6시간 분의 log_connect 로그를 캐싱(왜??)
            - 로그아웃 한 유저의 connect 정보가 6시간 내로 남아 있다면 상관이 없는데, 6시간보다 전이라면??
            - 실제로 동접하락 채널을 보면…… 하나도 못 찾아 오는 것 같은데? 텅 비어있음
            - 그냥 logout_character에 ip/country/language/os 정보를 남기는 방향으로 가기로 함
        3. 스케쥴러에 LogoutuserViewer.Coruntine 등록
    2. [LogoutUserViewer.py](http://LogoutUserViewer.py)
        1. 일정 주기(REFRESHMINUTE = 1분)로 RefreshCache 실행하여 캐시 최신화
        2. GetCurrentUsersPair: log_current_user에서 현재 유저 목록 가져옴
        3. GetLogoutCharacter: log_character_logout에서 로그 아웃 캐릭터 가져옴
        4. MakeMessage
            1. 메시지의 대부분은 위 정보(log_current_user, log_character_logout)로 만들어짐
            2. GetUserSetInfo 1.
[https://confluence.nexon.com/display/MAPM/LogoutUserViewer](https://confluence.nexon.com/display/MAPM/LogoutUserViewer)

## 작업흐름
---
- log_character_logout 컬럼 추가
    - LogSchema.xml 수정
    - Protocol::NCache::Branch{Location}::Schema.h, Schema.cpp 수정
    - CharacterLogout.cpp 수정
- log_current_user SP 추가
    
- purchase SP 추가
    
- 메시지 작성 부 구현

## 쟁점
---
- 각각의 테이블을 어떻게 불러올 것인가?
    - 기존 MapleAgent의 경우 CommonDB에 띄우기 때문에 직접 DB에서 Select가 가능
    - MapleAgent(新)은 202에 떠 있기 때문에, 현재는 운영툴 API를 호출하여 로그를 불러오는 방식으로 작업되어 있음
    
    1. **log_current_user**
        - 무조건 SP 새로 만들어야 함
        - 2개 row(매 분)에 대한 user_count의 합만 필요한데, 기존 SP 사용 시 국내 라이브 기준 2만개의 row를 가져 와야 함
    2. **log_character_logout**
        - SP 없이 가도 될 것 같은데? group by char_no가 문제긴 한데, 1분 동안 같은 캐릭터가 수십 수백 번을 로그아웃 할 가능성이 적기 때문에 group by하지 않은 데이터와 크게 차이는 없을 것으로 보임(실제 국내 라이브 기준 7252 : 7220)
        - C#의 IEnumarable 클래스에서 자체적으로 GroupBy 메소드 제공
    3. **~~log_connect~~**
        - log_connect의 경우 운영툴에 적당한 API가 없음. 기존 LOG_CONNECT API의 경우 user_no를 인자로 받기 때문에 이 경우는 쓸 수 없음
        - 쿼리를 직접 실행하는 API도 없음. 사실 있으면 안 될 것 같기도 하고
        - MapleAgent 내부에서 쿼리를 직접 실행해 보려고 시도. 어차피 202에서 돌기 때문에 DB에 접근이 불가능 하다는 이슈가 있어 뻘짓으로 판명
        - 결국 기존 GOT_ReadLogConnect SP를 수정하거나(user_no가 안 들어오면 저절로 조건에서 배제시킨다던지), user_no를 안 받는 새로운 SP를 추가하거나……
        - 캐싱 이슈
            - 기존 MapleAgent의 경우 어떻게든 ip/country/language/os 정보를 얻기 위해 log_connect를 참고하고 있음
            - 하지만, 로그아웃 한 char_no이 언제 마지막으로 connect가 되었는지 알 수가 없기 때문에 그냥 6시간 분의 log_connect를 긁어와서 캐싱해 두고 refresh를 하는 방식으로 작동 중
            - 문제점
                - 위의 기존 흐름에서 보이듯 제대로 작동을 안 함: _마지막 connect가 6시간 전이라면 log_connect 캐싱 데이터에 없을 것_
                - 기존에는 CommonDB마다(global의 경우 region 마다) Agent를 띄우고 캐싱을 했지만, 이번에는 202에 뜬 하나의 Agent가 모든 정보를 캐싱해서 들고 있어야 함: _공간 복잡도 이슈_
            - 그냥 logout_character에 ip/country/language/os 정보를 남기는 방향으로 가는 것이 맞다
    4. **purchase**
        - SP 새로 필요할 것으로 보임. user_no에 대한 결제 건 수를 합친 값만 필요한데, 기존 SP 사용 시 모든 결제 건 만큼 row가 생기게 된다
- log_character_logout 테이블 컬럼 추가
    - ip/country/language/osType 정보를 해당 테이블에서 추가로 남기도록 작업    
    - 컬럼 추가
	    - LogScehma.xml 수정
	    - 데이터 insert
	        - Protocol::NProcess::NDispatch::NLog::CharacterLogout에 코드 추가
            ```cpp
            auto*	connectIp = aliver->GetIP();
            STRING	country = aliver->GetDeviceInfoDetail()->country;
            STRING	language = aliver->GetDeviceInfoDetail()->locale;
            INT		market = aliver->GetDeviceInfoDetail()->marketType;
            ```    
	    - 브랜치 별로 Schema.cpp, Schema.h 수정(QueryGenerator 돌리면 파일 자동 생성)
	    - 순서 이슈
		    - TLogSchema의 처리 때문에, LogSchema에 컬럼을 추가할 때는 무조건 마지막 부터 추가해야 함.
		    -  LogSchema 상의 순서(=실제 DB상의 컬럼 순서)와 서버 코드(Schema.h)의 순서가 다른 경우, typearray의 Index가 달라져 문제가 생김: 테스트 시 실제로 데이터가 이상하게 들어가는 것 확인![[Pasted image 20231215103616.png]]
		    - 서버 코드 수정, 점검 스크립트 메일 재 발송
- API를 호출할 때, 월드 별로 따로 호출하는 현재의 API를 쓸 것인가 vs 모든 월드 별로 쿼리를 실행하여 결과를 합쳐주는 새로운 API를 팔 것인가
	- 새로운 API를 파려면 log_current_user, log_character_logout 둘 다 새로 파야 함
	- 결정적으로, 서버 별로 돌면서 메시지를 생성해야 하는데 월드 별 쿼리 결과가 합쳐진 리스트 두 개를 순회하기가 굉장히 곤란해짐 
	- 따로 단점이 있는 것도 아니기 때문에
	- 월드 별로 따로 호출해야 할 듯
- makeMsg 함수를 따로 만들 것인가 vs 그냥 showFlag 설정하는 루프 돌면서 메시지도 함께 만들어 버릴 것인가
- commonConfig를 어떻게 없앨 것인가
	- 시차 정보는 어디에 저장해 놓고 불러올 것인가
	- 잡 인덱스 <-> 잡 이름 정보는 어디에 저장해 놓고 불러올 것인가
		- App_Data\\CodeData, Xml\\CodeValue 만지작 거려서 해결
		- GOT_ReadLogCharacterLogoutByTime을 다른 곳에서 쓸 때 문제가 안 될까??
		- log_character_logout 테이블의 job 컬럼 자체를 내가 최근에 추가한 거라, 다른데서 사용해도 job을 파싱해 오지 않을 것이라 괜찮을 듯
- 예외 처리를 어떻게 할 것인가?
	- try - catch는 지양. Trace.Assert or Trace.Write로 해결할 수 있다면 그걸로 처리
- 봇 분리(??)
	- 디버깅 용으로 Agent 띄워 놓으면, 기존 시간 변경 채널 명령어를 내 Agent가 후킹함
	- 문제는 뭔가 오류가 있어서 filter_key를 못찾는 에러 발생
	- 디버깅 용도로 쓸 때는 다른 봇을 쓰는 분리가 필요할 것 같은데.


## TroubleShooting
---
- 동접 유저 수가 한 자리 ~ 두 자릿수 밖에 안 된다고??
- Diff와 LogoutChar 가 다른 건 무슨 경우??
- Channel null인 이슈
    - App.json의 ChannelGroup 추가
- ExecuteLogConnect 실패 이슈
    - BufferSize 초과(32mb)
    - 근데 애초에 log_current_user가 아니라 log_connect를 불러와야 했던 문제라 의미 X
- log_connect 얻어오는 이슈
- GOT에서 GOT_ReadLogConnect 호출 시 빈 값 얻어오는 이슈
    - i_user_no가 없으니 validate가 실패함
    - validate에서 i_user_no는 무시하도록 validateIgnoreUserNo 함수 새로 만듦
- log_connect_prior에 접속 시도하는 문제
    - priror가 뭐지 대체
        - 테이블의 스키마 변경이 있을 시, 변경 전 테이블과 연관된 SP에 prior를 붙여서 관리
    - 이 케이스의 경우, 알파에서 테스트를 하여 prior 테이블은 없는데 실수인지 뭔지 prior가 붙은 SP는 지우지 않아서 발생
    - GOT_ReadLogConnectPrior 실행 시 log_connect_prior 테이블이 없어서 오류
- WebApi.json 못 찾는 문제
    - csproj 파일의 MapleInfra 위치가 잘못 됨
- Region.json Deserialize 시 Region 못 찾는 문제
    - Region이 아니라 Regions라고 되어 있었음
- KoreaAlpha 못 찾는 문제
- url 주소 오류
    - http:// 안 붙임
- _commands에서 _LOG_CONNECT 못 찾는 문제
    - WebApi.json의 command 에 log_connect 추가
- 대상 row가 계속 0개인 문제
    - 찾는 시간이 UTCNow가 아니라 UtcNow.AddHours(9) → UTC(+9)
- LOG_CONNECT result deserialize 실패 이슈
    - GOT query 실패 → 테이블 명 지정 안 함
    - 왜인지 ExecuteJObjectRaw 대신 ExecuteQueryJObjectRaw 돌리면 json에 result, msg, count가 안 뜸. CResponseLogConnect 구조체에서 해당 변수들 제거
    - CResponseLogConnectParam 구조체의 Language 변수 타입을 string이 아니라 int로 설정했었음
- ProcessAgent 이따금 실행 안 되는 이슈
    - !_agent.Ready() 여기서 걸림
    - _awsManager가 실행이 안 된 상황
- 로그 아웃 시 중단점 잡히는 이슈
    - Schema.h 말고 Schema.cpp도 수정해야 함(inputSchema.Append 호출)
- FTP.json 못 찾는 이슈
    - FTP.csproj의 FTP.json 경로 오류
- LOG_CHARACTER_LOGOUT API 실패 이슈
    - account_no 없어서 data.validate 실패
    - 애초에 char_no, account_no가 필요 없는 LOG_CHARACTER_LOGOUT_BY_TIME API를 이용하고 있었음
- LOG_CHARACTER_LOGOUT_ALL_WORLD API 실패 이슈
    - webApi.json에 API 이름 잘못 입력
    - API 없을 시 404 Not found가 뜨는데, 이 에러가 뜨면 그냥 return을 null로 받음.
    - Trace.Write로 로그만 남기도록 처리
- 중국 aws 인스턴스 못 찾는 이슈
    - Credential.json에 China EC2 무언가를 추가하니 해결
    - 하지만 AWS랑 런덱 필터랑 대체 무슨 상관이지??
        - 런덱 필터란, 다른게 아니라 어떤 장비에 설치된 런덱을 실행할 지 알려주는 것
        - 런덱은, 아마존 EC2 서버 장비에서 돌아간다. 따라서 해당 장비(인스턴스) 정보를 알아야 함
        - 최근 Credential.json 파일 구조 변경이 있었고, 이 때 China EC2 정보가 빠져 버린 것이 원인
        - [EC2와 S3의 차이?](https://www.notion.so/EC2-vs-S3-141dc92923dc426a980e5dcdba22b2c2?pvs=21)
- log_current_user 결과 값 갯수가 한 개라 outofrange exception 나는 이슈
	- 전체 월드에 대해 쿼리 실행 > 월드 별로 각각 실행으로 바꾼 후, GOT API 내부 리턴 양식 수정 안 함