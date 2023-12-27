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



[[LogoutUserViewer]]
 