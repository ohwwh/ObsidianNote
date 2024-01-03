
서비스로 실행 시 로컬 시스템 계정이라 AWS 프로파일이 없음
→ mmjenkins 계정으로 실행하려고 하는데, 뭔가 구조가 이상함
→ 전체적으로 서비스 install / 실행 구조를 갈아 엎음
- Controller에서 install, uninstall, execute를 각각 호출하도록 구조 개선

## 쟁점
---
- Controller 클래스는 왜 만 들었나?
	- ManagedInstallerClass가 이미 만들어진 Installer 클래스를 이용하여 서비스를 설치한다는 설명이 있음
	- Installer 클래스로 서비스를 설치하는 "전통적인(without ManagedInstallerClass)" 방식에 대한 설명을 읽고, 바이너리 경로 지정, Install, Commit 등 필요한 요소가 없는 것을 보고 뭔가 미완성이라고 판단한 듯
	- 위 요소를 묶어서 처리하기 위해 Controller 클래스를 만든 듯 하다.
	- ManagedInstallerClass로 한번에 처리가 가능하기 때문에, 결과적으로는 불필요한 코드가 되어 삭제 필요
	
- System.Confugifuration.Install 모듈 내부에서 ManagedInstallerClass를 쓰는 것과 Installer 클래스를 쓰는 것의 차이?
	- Installer 클래스는 C#에서 서비스 등록 및 삭제 등을 위해 기본적으로 제공되는 클래스
	- 프로젝트 생성 시 프로젝트 형식: Windows Service를 선택하면 Program.cs와 함께 서비스 실행을 위한 디자이너 클래스가 생성된다
		- 기본 명은 Service1.cs, WindowsServiceScheduler 서비스에서는 Execute.cs라는 이름으로 존재한다
	- 이 디자이너 클래스를 더블 클릭해서 들어가면 빈 화면이 뜨는데, 여기서 우 클릭 -> 설치 관리자 추가를 통해 서비스 설치를 위한 디자이너 클래스가 자동으로 생성된다. 
		- 기본 명은 ProjectInstaller.cs, WindowsServiceScheduler 서비스에서는 Installer.cs
		- 위의 Installer 클래스를 상속받아 Install 추상 메소드를 구현한 채 생성 된다.
			- 기본적으로 처음 생성된 Install 메소드 내부에는 InitializeComponent()만 불리고 있음
	- 서비스 설치 과정
		1. 서비스를 설치할 바이너리 경로 지정
		2.  Install 메소드 호출
		3. Commit 메소드 호출
			- Commit이 정확히 뭔지 모르겠지만 Install 후 반드시 호출되어야 한다고 함([Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.configuration.install.installer.commit?view=netframework-4.8.1) 참조)
			- install transaction 이라는 걸 마무리한다는데... 서비스 설치/삭제 에 필요한 메타데이터를 저장한다는데 뭔지 잘 모르겠다
	- ManagedInstallerClass.Installhelper의 경우, 인자를 받아 조건에 따라 위의 경로 지정 + Install + Commit 혹은 Unintall을 수행한다.
		- 이 메소드의 경우 원래 InstallUtil.exe에서 쓰도록 만들어졌으며, 사용자 코드에서 직접 호출되도록 만들어 진 것은 아니다(출처: [Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.configuration.install.installer.commit?view=netframework-4.8.1))
		- 하지만 따로 금지된 것도 아니고, 설치 코드가 한 줄로 간단해 진다는 점에서 쓰는 것이 좋을 것 같다.

## TroubleShooting
---
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