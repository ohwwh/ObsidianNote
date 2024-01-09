#MapleAgent

## 개요
---
- python으로 구현된 MapleAgent/시간변경 봇/덤프 수집 봇 등 각종 봇 프로그램들을 통합하여 을 C#으로 포팅하는 프로젝트
- 슬랙 봇을 python 혹은 JS로만 구현 가능 하기 때문에, 현재 JS로 구현. 이 구현은 기능을 구현한 것이 아니라, C#으로 새로 만들 봇에 명령어를 토스해 주는 역할만 함. 
- ~~해당 C# 봇은 API 수준으로만 구현이 되어 있음~~
- 현재 MapleTools/Misc/WindowsServiceScheduler
- MapleTools/Jobs/MapleAgent 코드 한 번 까볼 것 → python으로 된 기존 MapleAgent


## MapleAgent란
---
- 슬랙 채팅창에 명령어를 입력하면 거기에 맞는 결과값을 보내 주는 슬랙 외부 봇
- 동접 현황에 라이브 이런 식으로 치면 결과 값 알려주는 역할


## 작동 방식
---
- 슬랙 app 관리에서 앱을 설치하면 앱 토큰이 발행된다
    - [Slack API: Applications | MapleM Slack]([https://api.slack.com/apps/A04LUMS3KRV/general?)](https://api.slack.com/apps/A04LUMS3KRV/general?))
- C# 코드 내에서 해당 토큰을 통해 슬랙 api를 호출하면, 해당 api가 슬랙의 메시지를 hooking 한다.
- 이 메시지가 어떤 채널에서 호출되었는지, 명령어가 무엇인지 여부에 따라 해당 기능을 실행한다.
- [[Slack] OAuth Token 생성하기 (tistory.com)](https://miaow-miaow.tistory.com/148)
- [.NET 7을 사용하여 사용자 지정 Slack 봇을 만드는 방법 – Daniel Donbavand](https://danieldonbavand.com/2023/05/03/how-to-create-a-custom-slack-bot-with-net-7/)

## TroubleShooting
---
- SlackNet.SlackException: Slack returned an error response: not_in_channel 남기고 죽는 이슈(2023-01-05)
	- ConcurrentUser()에서 오류 발생
	- Channel.json의 CURRENTUSER 그룹에 LogoutUserViewer 용으로 동접 하락 채널들을 넣어 놓았더니, ConcurrentUser 도 해당 채널로 메시지를 보냄
	- 해당 채널들에 MapleAgentTest 봇이 추가가 안 되어 있어서 exception 발생
	- 아예 ChannelGroup에서 LOGOUTUSER를 분리함
- ConcurrentUser() 돌 때 Channel이 null인 이슈
	- CURRENTUSER channelgroup에 location이 ALL 밖에 없음
	- 무슨 의도인지는 알겠는데 국가를 찾지를 못함
	- 아마 
- - ProcessAgent 에서 주기적으로 도는 코드들이 이따금 실행 안 되는 이슈
    - !\_agent.Ready() 여기서 걸림
    - \_awsManager가 실행이 안 된 상황
    - 아마도 data racing 문제. 
- 커밋 시 대량 충돌 이슈
	- 커밋을 한 동안 안 했더니 충돌이 너무 광범위하게 발생. 손 머지가 불가.....
	- 그냥 MapleToolsNew를 새로 클론 받아서 최신화 하고, 변경 사항 일일히 손으로 복사해 넣음
- 202 서비스 시작 안 되는 이슈
	- 테스트 용으로 MapleAgentTest 봇의 토큰을 넣었어야 했는데 실수로 오리지날 봇의 토큰을 넣음
	- 해당 봇이 이미 돌아가고 있었으므로 시작 중에서 멈춘 것으로 추정
- 202 서비스 실행 중 멈추는 이슈
	- System.IO.DirectoryNotFoundException: 'D:\Application\Dump' 경로의 일부를 찾을 수 없습니다
	- 일단 202에는 해당 기능 주석 처리 후 넣어 놓음
	- 로그 알아보기 힘들어 일단 로그 수정
	- 로그 확인 하니, AnalyzeDump가 끝난 후 위 경로를 못 찾아서 서비스가 사망하는 듯 함
	- 코드에  "down 받을 조건부로 받게끔...추가..작업 필요" 라고 주석 달려 있는 것(2024-01-09 기준)으로 보아 추가 작업 예정인 듯 함.
		- 형근님 피셜 아직 미완. 일단 주석 처리 해 놓으라고 하심

[[Window 서비스 등록]]
[[LogoutUserViewer]]
 