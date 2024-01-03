
서비스로 실행 시 로컬 시스템 계정이라 AWS 프로파일이 없음
→ mmjenkins 계정으로 실행하려고 하는데, 뭔가 구조가 이상함
→ 전체적으로 서비스 install / 실행 구조를 갈아 엎음
- Controller에서 install, uninstall, execute를 각각 호출하도록 구조 개선

## TroubleShooting

- uninstall 이슈 - Error Occurerd: savedState 사전에 일관성 없는 데이터가 들어 있으며 손상되었을 수 있습니다.
    - Uninstall 메소드로 state 대신 null 보내서 해결
- 서비스로 실행 시 aws profile 못 찾는 이슈
    - mmjenkins로 서비스 install 시 mmjenkins 계정을 못 찾음
        - .\mmjenkins라고 해야 찾음
    - 로컬 시스템 계정으로 실행 시 aws profile 못 가져옴 - ProfileLocation이 Null. 파일에서 직접 가져오도록 뭔가 바꿔야 하는 듯
- currentuser 실행 시 채널이 null인 이슈
    - App.json에 ChannelGroup 설정이 안 되어 있음
    - App.json 최신화 및 LogCurrent / DumpCollect 등 주기적으로 도는 잡 비활성화
- 서비스가 계속 시작 중으로 뜨는 이슈 - Setting의 Debugging ⇒ False로 바꿈
    - 로컬에서 디버깅 하려면 True로 바꿔야 한다. False면 뭔가가 콘솔에 무한으로 출력되는 것을 볼 수 있음
    - 무조건 리빌드. 빌드만 하면 Setting 바뀐 것이 적용되지 않음
- actionthread들의 정체 - 비활성화 해도 딱히 문제는 안 생김
    - 스레드 풀인듯 함
    - 문제가 안 생길 것 같지는 않은게, 주기적으로 도는 잡들(주석 쳐 놓은)이 이 스레드풀을 이용함. 일단 주석 풀어 놓았음
- 서비스가 중지 되도 WindowServiceScheduler.exe가 계속 떠있는 이슈