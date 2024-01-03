
[[중국 QA DB 증발]]
[[중국 DeliveryNew 실패]]


# 서버
---
* CharacterName does not exist in system TypeTree or TextDetect does not exist in system TypeTree
	→ 중국 서버를 실행하려고 했다면, ConfigTable\Develop.h의 define이 CHINA로 걸려 있는지 확인하세요

- Run_ChannelListIsNull
	오만가지 이유가 있을 수 있다. 일단 서버랑 미들웨어 버전이 잘 맞는지 부터 확인. 다 잘 맞는다면, Station 관련 파일 건드린 사람이 있나 도움을 요청한다

- CharacterSelectLoading_ResponseFail
	캐릭터 정보를 디비로 부터 로딩하는데 실패. 보통 DB 재설치 하면 해결된다.

- assert( Schema::EType::Text == this->schema->GetType( index ) ) 에서 중단점
	1. 동시 다발적으로 다양한 스키마에서 해당 중단점이 나타난다면, 높은 확률로 Develop.h에 국가 설정이 잘못 되었을 것. 간혹가다 제대로 되어 있는데도 그러는 경우가 있는데, 뭔가 국가 define이 꼬여 있는 것으로 추정. 게임 서버를 통째로 리빌드 하니 해결 되었다.
	2. 특정 스키마에서만 저런다면, MapleInfra가 최신인지 확인해 볼 것. 스키마 변경 사항이 제대로 반영이 안 되어 있을 수 있다.
	3. SourceTree 같은 걸로 MapleServer, MapleInfra, MapleResource의 브랜치가 통일되어 있는지, 최신화는 되어 있는지 천천히, 꼼꼼히 살펴볼 것

* SN_JoinField Deserialize Fail
	보통 서버와 클라 모두 최신화 하면 해결된다

- Init_WrongBranch, 0x100300B8
	말 그대로 클라 / 서버 브랜치가 안 맞다는 이야긴데, 다 정상이라면 게임 서버를 rebuild 하거나 로그인 서버를 rebuild 해 볼 것. 로그인 서버 rebuild 하니까 해결 되었음


# 기타
---
- EventLog 찾는 법
	해당 머신의 시간 / Date modified 은 무시하고, **현재 기준으로 내가 찾고 싶은 NTP 시간과 파일 명 뒤의 날짜가 일치하는 파일**이 내가 찾는 로그 파일이다.

- 투 클라 실행 시 lan.kor.tbl 불러올 수 없다며 추가 다운로드가 불가능한 오류
	→ MapleResource 버전이 같아야 함. 두 버전이 다를 시, 하나의 클라이언트가 실행 중일 때 다른 클라이언트를 실행하면 리소스 버전이 달라진 것을 인식하고 추다를 다시 하게 됨. 이 과정에서 이미 참조중인 lan.kor.tbl을 불러오려고 해서 오류가 뜸.
	→MapleResource는 매우 자주 갱신되는 편이므로, 동시에 pull을 받아서 하는 것을 추천
	→ 만약 리소스 폴더를 하나만 가지고 쓰는 경우
	→ 각 클라의 구동 환경 설정 → DevAppName 변경에서 아무거나 서로 다른 값을 준 후, 테이블 빌드 후 실행하면 된다.

- 기획자 로컬 서버 실행 후 Run.bat 파일이 사라지는 현상
	우선 게임 서버는 서버 실행 전, 10일 전 로그 파일을 모두 제거함. 파일을 지울 때, (로그파일 경로)\_._ 를 지우게 됨(Module::Toolkit::NLog::Output)
	기획자 로컬 서버를 실행하는 bat 파일을 열어보면 로그 파일 경로가 .\ → 현재 경로로 지정이 되어 있음. 때문에 10일 후에 실행하면 안에 있는 파일을 모조리 지워버리게 됨. 로그 파일 경로를 \LogFile\ 로 지정하도록 바꾸어 주어야 함
	이 때 뒤에도 \를 무조건 붙여야 함. 소스 코드 상에서 저 path 뒤에 *. *를 붙이는데, 따로 \를 붙여주지 않기 때문.
	10일이 한번이 아니라 매 번 사라지는 경우 → 아마도 로컬 시간 변경 치트 같은 걸 사용한 경우 시간 설정이 뭔가 꼬여서 그런 것으로 추정.

- 서버 시간 바꾸기
	Assets\Resources\BuildIn\Define.xml을 바꿔 주어야 제대로 시간이 바뀜(기간 한정 이벤트 참여 가능)
	[https://confluence.nexon.com/pages/viewpage.action?pageId=69868879](https://confluence.nexon.com/pages/viewpage.action?pageId=69868879) 참조

- Login 서버 실행 실패
	DB 재설치 후 다시 하니까 잘 되네???

- Monitor가 안되요
	최신화를 해 보자. 우클릭 후SVN update하면 됨.

- mRemoteNG가 안되요
	최신화를 해 보자. [](https://confluence.nexon.com/display/MAPM/3.5.10.+Profile+%28for+mRemoteNG%29+Generator)[https://confluence.nexon.com/display/MAPM/3.5.10.+Profile+(for+mRemoteNG)+Generator](https://confluence.nexon.com/display/MAPM/3.5.10.+Profile+(for+mRemoteNG)+Generator) 여기서 ConnectionGen.zip을 다시 다운받은 후, 연결 파일 정보를 새로 생성하여 적용한다.

- 무슨 브랜치에 커밋을 해야 되나요
	브랜치 분리에 대해서는 [젠킨스 업무](https://www.notion.so/e3aa1cbc44d74a52b74e4ed5e9382fc7?pvs=21)의 브랜치 분리 참조
	만약 현재 1.87 브랜치가 분리된 상태에서
	1. 전 국가 공통으로 바뀌어야 하는 뭔가를 개발 했는데 이걸 1.87 업데이트에 꼭 넣어야 한다 → 마스터 + 전 국가 1.87 브랜치에 모두 커밋한다.
	2. 공통이지만 꼭 1.87에 안 들어가도 된다 or 들어가면 안된다(ex. 다음 버전부터 적용될 전 국가 밸런스 패치) → 마스터에만 커밋한다(1.88 브랜치 분리 시 자동으로 포함 됨).
	3. 특정 국가 한정 컨텐츠이다 → 마스터와 해당 국가 브랜치에 모두 커밋한다.

- 젠킨스 - `MSBUILD : error MSB4166: 자식 노드 "9"이(가) 중간에 종료되었습니다. 종료하는 중입니다`
	그냥 실패한 친구만 재빌드 돌려라

- 젠킨스 - `curl: (28) Failed to connect to 10.10.56.202 port 3000 after 21031 ms: Timed out`
	모든 패치가 끝난 후 버전 정보를 버전 대시보드(10,10,56,202:3000)에 쏴주는 curl이 머신의 문제나 다른 이유로 인해 실패한 것. 202머신이나 네트워크 문제가 있는지 확인 후 해당 curl을 다시 쏴 줄 것.

- DB 패치 - `ERROR 7 (HY000) at line 15984: Error on rename of '.\\scania_maple_log\\#sql-11bc_54.frm' to '.\\scania_maple_log\\log_customize.frm' (Errcode: 13 - Permission denied)`
	프로세스가 해당 파일을 잡고 있는 것으로 추정. 적당히 시간 차를 두고 재 실행 해 볼것



- 컴파일 - error C2065 corecrt_wstdio.h
	높은 확률로 당신이 \<iostream\>같은 표준 헤더를 추가 했을 것. 왜 인지 이런 표준 헤더는 못 쓴다고 한다.

- 캐릭터 외형 정보를 찾을 수 없습니다.
	서버, 리소스, 클라를 최신화 후, DB 패치를 해준다. 특히 DB 패치가 원인일 가능성이 매우 높음

- 클라이언트 - 실행 버튼 눌렀는데 넥슨 로고가 안 보이고 그냥 멈춰 있는 현상(첫 화면이 회색이 아니고 하늘색 그라데이션)
	MapleProto를 다시 열어주면 해결

- 터미널 로그인 실패
	```xml
	fail command : TerminalLogin
	2023-05-17 18:09:32	retry(4) command : TerminalLogin
	2023-05-17 18:09:32	Channel.TryConnect(18.180.48.107:7302) (18.180.48.107) / InterNetwork
	2023-05-17 18:09:32	Callback.OnConnect() 18.180.48.107:7302 System.Net.Internals.SocketExceptionFactory+ExtendedSocketException (10060): 연결된 구성원으로부터 응답이 없어 연결하지 못했거나, 호스트로부터 응답이 없어 연결이 끊어졌습니다. 18.180.48.107:7302
	```
	- 해당 장비 터미널의 IP가 모종의 사유로 변경되어 연결에 실패
	- 일단 준원님한테 터미널 IP가 바뀐게 맞는지 확인
	- IP 정보는 패치 콘솔이 service.csv에서 읽어 온다.
	- 해당 파일의 경로는MapleTools\Patch\Client\PatchConsole\TableData\China\Table\Service
	- nxm-maplem-cn-deploy/App/2.0.3/Config/China/Table/Service(s3 경로)
	- IP 주소가 진짜로 바뀐건지 실수로 바뀐건지 알 수 없으므로, 일단 s3에 올라간 service.csv의 ip주소를 직접 수정 후 publish/update 진행
	- 실제로 바뀐게 맞다고 확인 되면, MapleTools에 있는 Service.csv를 수정한 후, [Manager [PatchTool] [Jenkins]](http://10.10.56.42:8080/view/Common/job/PatchTool/job/Manager/) 잡을 실행하여 실제로 패치콘솔에 적용 후 해당 빌드 머신에 배포해 준다

- Daemon 사망
	```jsx
	System.ComponentModel.Win32Exception (0x80004005): ReadProcessMemory 또는 WriteProcessMemory의 일부 작업만을 마쳤습니다
	```
	뭔가 프로세스 권한 관련 문제로 추정. 머신 재시작 하면 해결 된다.

- TStage World0Set 미들웨어들 전부 사망
	- Terminal에 Station 패치가 안 됨
	- Terminal이 대체 뭐하는 놈이고, Station 패치라는게 뭔지도 모르겠음
	    - Copy잡이 원래 Infra가 바뀌었을 때 터미널에 Station 패치까지 해주는 옵션을 줘야 하는데, 없음. 따라서 Station 패치가 되지 않았다.
	- 일단 MapleInfra의 China/release/0.895 브랜치에서 최신 Station 파일들을 터미널 장비의 D:\Application\Table\Station으로 교체
	- 원래는 TStage말고 Stage나 QA에서 이런 일이 발생하면, Deploy 잡에서 맨 밑의 PatchCheck에 Station을 체크하여 Station 패치를 걸어준다. 하지만 모종의 이유로 TStage, TQA는 선택지에서 빼 버렸기 때문에, 위와 같이 수동으로 해야 함
	- 수동으로 바꿔줬는데, 여전히 서버는 죽어 있고 state가 unbind라는 애들도 뜬다??
	- Station 파일의 내용이 변경 되었으니, 이제 바뀐 내용 대로 각각의 파일들을 찾겠지? 근데 없다. 왜냐하면, patch를 돌릴 때
	- TStage 장비들 비밀번호 → [3.7.2. 텐센트 클라우드 서버 연결 - MapleM - NK Confluence (nexon.com)](https://confluence.nexon.com/pages/viewpage.action?pageId=413509297)

- 클라이언트 시간을 변경하고 싶습니다
	Game ResetFakeLocalTime 2020 04 23 13 00 00
	치트 사용 전, MapleClient/Asset/Resources/BuildIn/Define.xml에 다음을 추가(혹은 변경)해 준다.
	```json
	<?xml version="1.0" encoding="utf-8"?>
	<Root>
		<Code value="_DEVELOP_FAKE_SERVER_TIME" />
		<!--Code value="_EXTERN_TABLE_2017_06"/-->
	</Root>
	```

- 빌드 했는데, 파일 버전이 이상해요
	```sql
	D:\\Jenkins\\workspace\\Build\\Build_Game>D:\\Jenkins\\WORKSPACE\\_FileVersion\\FileVersion.exe /i D:\\LinkJapanStage\\Binary\\x64\\MapleGameRelease\\Game.exe 1.920.2.3867 
	14:17:38 ================================================================================
	14:17:38 Command-line arguments:
	14:17:38 argv[ 0 ] - D:\\Jenkins\\WORKSPACE\\_FileVersion\\FileVersion.exe 
	14:17:38 argv[ 1 ] - /i 
	14:17:38 argv[ 2 ] - D:\\LinkJapanStage\\Binary\\x64\\MapleGameRelease\\Game.exe 
	14:17:38 argv[ 3 ] - 1.920.2.3867 
	14:17:38 Start..... Updating Version[1.920.2.3867]
	14:17:38 EndUpdateResource_Fail
	14:17:38  GetLastError : 110
	```
	위와 같이, 파일의 버전을 입력하는 fileversion.exe가 제대로 안 돌아서 실패했을 수 있다. 보통 Build_에서 실행되므로 이곳의 console output을 잘 뒤져 볼 것

- CommonDB 장비 Dummy DB 로그인이 안 되는 문제
	ProgramData\MySQL\MySQL Server 5.7\Data_ 내부의 테이블 데이터를 새로 교체하면 해결 됨
	→ DB 설치를 다시 하라는 뜻

- git cli 로그인이 안되요
	인증이 풀려서 로그인을 다시 하라고 했을 때,
	
	```powershell
	remote: HTTP Basic: Access denied
	fatal: Authentication failed for '<https://gitlab.nexon.com/maplem/JenkinsScripts.git/>'
	```
	이렇게 로그인이 안 될 때가 있다.
	- ID는 넥슨 id. **단, @nexon.co.kr은 떼야 한다.**
	- 비밀번호는 넥슨 pwd. **단, 로그인 시 token으로 들어가야 한다**.
	 대체 왜 이따위인지는 모르겠다.

- DBManager 실행 시, 아래 에러 발생:
	```jsx
	Exception.ToString()이 실패하여 예외 문자열을 인쇄할 수 없습니다.
	```
	에러를 stdout에서 Log 내부의 로그파일로 redirect 하기 때문에 print 하지 않음. 저 에러는 무슨 이야긴지는 모르겠다. 어쨌든 저런 게 뜨면, Binary/Log 로 들어가 로그 파일을 보면 무슨 에러로 종료되었는지 볼 수 있다.

- 클라이언트 실행 시, 컴파일 에러 난다고 safe 모드로 들어가라고 하는 경우
	Asset, Library 백날 날려봐야 똑같이 뜸. Unity Hub에서 안드로이드로 Swtich Platform 안 해서 뜨는 에러일 가능성 99%

- 아웃룩(Outlook), 팀즈(Teams) 연결 안 됨 이슈
	컴퓨터 재부팅 후 해결

- 위챗 로그인이 안 되요
	- 휴대폰 분실 후 새로운 디바이스에서 로그인 하려면 이전 디바이스에서 로그아웃을 해야 한다고 함.
	- 왜인지 하루 지나니까 해결 되었음

- QQ 아이디가 뭐더라
	[위챗/QQ 테스트 계정 - MapleM - NK Confluence (nexon.com)](https://confluence.nexon.com/pages/viewpage.action?pageId=473096901)에서 아무 ID나 골라서 내가 사용한다고 적고 쓰면 된다.

- AWS VPN 접근 - SAML을 인증에 필요합니다

- 중국 QA 빌드 접근 시 C001 연결 에러
	사내 wifi에 연결해 보자

- 유저 정보를 찾을 수 없습니다
	내 경우는 게임 서버만 브랜치가 달랐었음. 확인 잘 할 것

- mmjenkins 계정 로그인 불가 이슈
	- `1327: 계정 제한으로 인해 이 사용자가 로그인할 수 없습니다. 빈 암호 사용, 로그온 시간 제한 또는 정책 제한으로 인해 이러한 문제가 발생할 수 있습니다.`
	- 계정 설정 들어가서 암호 사용 기간 제한 없음 체크 시 해결

- Workbench DB 접속 불가 오류
	- SSL connection error: error: 1425F102:SSL routines:ssl_choose_client_version:unsupported protocol
	- Edit Connection > SSL > Use SSL 을 No로 설정 시 해결

- Visual Studio: 코드 업데이트 확인 중
	- 한 줄 씩 디버깅 시 F10 누를 때 마다 "코드 업데이트 확인 중" 이라고 뜨며 응답 없음 상태에 빠짐
	- Visual Studio 재 시작 시 해결되었음

- Incredibuild가 Stand Alone 으로 돌아가는 경우
	- Message 창을 보면 initiator 등록이 안 되었다는 경고 메시지가 있음. 
	- 승욱 님에게 문의하여 initiator로 등록해주자