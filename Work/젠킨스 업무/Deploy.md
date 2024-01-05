## 개요
---
QA, Stage 배포 잡
서버의 빌드할 항목을 체크하는 ServerCheck
클라이언트의 빌드할  항목을 체크하는 ClientCheck
빌드된 항목을 패치할 때 옵션을 체크하는 PatchCheck 
단계로 나뉜다.

## ServerCheck
---
파라미터로 Table, Game, Login, MiddleWare, DB, GOT를 선택할 수 있다.
각각의 파라미터 선택은, jenksin/workspace/Build 폴더의 개별 잡들(Build_* : ex. Build_Game)과 GOT 빌드 잡(ManageGOT > GOT > ExecuteALL)으로 연결 된다.

- **Build_Game, Build_Login, Build_MiddleWare, ExecuteALL:** 말 그대로 소스코드를 빌드하는 잡

- **Build_DB**: Modules/MetaData/Schema 내부의 스키마 파일들을 바탕으로 QueryGenerator를 실행하여 sql 파일들을 생성하고, 몹시 짜증나게도 **s3에 업로드 하는 것 까지 진행**(DB의 경우, 아래 Patch의 Upload 에서는 그냥 아무것도 안한다)

- **Build_Table**: 그냥 리소스 저장소를 최신으로 받아오는 일만 함


## ClientCheck
---
QA 빌드는 무조건 QA/Stage 마켓 버전을 바라보도록 클라에 하드코딩 됨.
Deploy시 Client 부분에 App 선택에 QA, IAP, Live를 선택할 수 있는데, 각각 클라 빌드를 따로 만들며 이 때 각 빌드에 어떤 마켓 버전을 바라볼지 하드코딩된다


## PatchCheck
---
각각의 파라미터 선택은, jenksin/workspacePatch 폴더의 개별 잡들과 연결 된다.

- **Validate**: GameServer - GameTable 간 검증. 간단히 말해 그냥 게임 서버를 특정 파라미터 값을 주고 실행한다. 이러면 서버가 끝까지 실행되지 않고, 게임 테이블 로딩이 잘 되는지 까지만 확인 후 종료된다.

- **Station**: Station 정보에 변화가 있을 때(ex. 미들웨어를 새로 추가), 터미널을 패치해준다.
	- 해당 옵션 체크 시, 관련 노드를 내리는 stop이 함께 실행된다.
	- Station만 단독으로 체크했을 경우는 이걸 다시 돌리는 Run도 실행된다.
	- Publish, Update와 함께 돌릴 경우 여기서 모든 서버를 죽였다가 다시 실행시키기 때문에 굳이 Run이 실행되지는 않는다.

- **DB:** s3에 업로드 된 DB 스키마를 적용한다. 따로 체크 항목이 없으며 Build_DB 실행 시 자동으로 패치까지 실행 됨

- **Upload**: s3에 빌드한 파일을 압축하여 업로드(Service\Storage에 업로드 됨. 예를 들어 중국 QA 빌드의 경우, \nxm-maplem-cn-deploy\Service\Storage\China\QA 로 업로드)

- **Publish**: 머신에 s3에 업로드 된 파일을 Contents 폴더에 다운로드(서버 파일 배포) - 이걸 **Patch**라고 하기도 함

- **Update**: 다운로드 된 파일의 압축을 해제하고 기존 파일과 교체 → 마켓버전 갱신, 배포된 바이너리 적용, 서버 재시작
	- Upload는, 어느 하나가 빌드에 실패하더라도 나머지가 영향 받지 않음 → 예를 들어 Game 빌드가 실패해도, Middleware나 Login등 다른 아이들은 Upload까지 잘 됨
	- Publish, Update는 영향을 받는다 → Game 하나만 실패해도, 나머지 모든 친구들의 Publish, Update가 이루어지지 않는다. → 싱크를 맞춰야 하기 때문.