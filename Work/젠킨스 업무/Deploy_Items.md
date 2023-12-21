## Legacy, 현재 DeliveryNew로 대체되었습니다

[Deploy_items: 실행 과정 - MapleM - NK Confluence (nexon.com)](https://confluence.nexon.com/pages/viewpage.action?pageId=804735080)
해당 잡은 젠킨스 머신(42)에서 실행 됨

**`D:\\Jenkins\\workspace\\Build\\Deploy_Items`** 내부에서 모든 작업이 이루어진다(루트 디렉토리).

FTP에 올라가는 파일 경로와 동일하게 폴더를 만들고, 그걸 압축해서 업로드함

예를 들어 바이너리 파일의 경우, 루트 디렉토리 밑에 Application/Bin 폴더를 만들어 필요한 걸 집어넣고, 이걸 압축하여 전달함.

## Deploy_Items.jenkinsfile(스크립트 파일) 내부 플로우

1. ProcessingApplication
    1. res폴더 생성후, 아래 zip 파일 들을 bin → res로 옮겨준다.
        
        ```groovy
        MoveFile("""${BuildPath}\\\\Application\\\\Bin""", """${BuildPath}\\\\Application\\\\Res""", "Dynamic.zip")
        MoveFile("""${BuildPath}\\\\Application\\\\Bin""", """${BuildPath}\\\\Application\\\\Res""", "ExternalServer.zip")
        MoveFile("""${BuildPath}\\\\Application\\\\Bin""", """${BuildPath}\\\\Application\\\\Res""", "GameTable.zip")
        MoveFile("""${BuildPath}\\\\Application\\\\Bin""", """${BuildPath}\\\\Application\\\\Res""", "Language.zip")
        MoveFile("""${BuildPath}\\\\Application\\\\Bin""", """${BuildPath}\\\\Application\\\\Res""", "Physics.zip")
        MoveFile("""${BuildPath}\\\\Application\\\\Bin""", """${BuildPath}\\\\Application\\\\Res""", "Slope.zip")
        MoveFile("""${BuildPath}\\\\Application\\\\Bin""", """${BuildPath}\\\\Application\\\\Res""", "Schema.zip")
        MoveFile("""${BuildPath}\\\\Application\\\\Bin""", """${BuildPath}\\\\Application\\\\Res""", "Station.zip")
        ```
        
    2. ReprocessingApplicationBin, Res
        
        1. 필요한 파일들을 모두 다운받은 후, ExcludeFrom* 함수들을 호출→ ExcludeFromMarketVersionXml: Destination이 중국 PQA인 FakeAuth인 경우에만, 마켓버전 다운로드 후 PQA 버전 정보 제외하고 전부 삭제한 후, ChinaLiveMarketVersion.xml로 새로 저장한다. ExcludeFromCsv: 스테이션 파일들(Station\Node.csv, Channel.csv)에서 PQA 연결 정보 제외하고 전부 삭제 ExcludeFromXml: 스키마 파일들(Schema\*.xml)에서 PQA 정보 제외하고 전부 삭제 (필요한 파일들 압축 해제 후, 위 작업 수행 후 다시 압축한다)
2. Build_Parallel
    1. Bin(서버, 미들웨어 바이너리), Res(테이블, 스테이션, 스키마), DB, IDIP, Metadata, TLogSchema 폴더를 압축하고 md5 파일을 생성한다.
        - DB는 압축하기 전 MakeFullDB, MakeDiffDB를 실행한다.
            - MakeFullDB는 Build_DB 잡을 호출하여 스키마 파일을 새로 뽑는다.
            - MakeDiffDB는 Common/DBUtil/DBPatchScriptExtractor 잡을 호출하여 기존 DB와 패치된 DB의 스키마 파일의 diff 스크립트를 뽑는다.
        - IDIP는 압축하기 전 DB 패치, (ReprocessingIDIP)를 실행한다.
    2. Client도 ProcessClient가 실행되어 뭔가를 하지만, 정확히 뭘 하는지는 모르겠음
3. FTP_Upload
    1. FTP에 서버 파일들을 업로드 한다 - 위에서 만든 zip, md5 파일들
    2. FTP에 클라이언트 파일들을 업로드 한다 - TencetFTP_Upload 잡을 호출하여 실행
    3. 젠킨스 마스터 서버 DB에 git hash를 업데이트 한다.
        1. DB의 patchitme 테이블에서 Source에 해당하는 upload_hash, publish_hash, update_hash를 읽어와 Destination에 업데이트
        2. UploadPatchItems, PublishPatch, UpdatePatch에서 실행됨
4. ProcessGit FTP에 업로드한 내용을 MaplePatch\China\ 에도 푸쉬한다.

## TroubleShooting

- DB diff 뜨는데서 멈춘 것 같은데???
    - 멈춘게 아니라, 클라 바이너리 업로드 잡이 병렬로 돌고 있을 가능성이 높다. TencentFTP_upload 잡에 들어가서 돌아가고 있는지 확인할 것
- 만약 전달 과정에서 뭔가가 잘못되어 다시 전달해야 한다면
    - jenkinsDB의 patchitems 에서, source가 되는 location/region의 name=DB인 row의 upload_hash, publish_hash, update_hash 값을 이 전 값으로 롤백해 준다 (DB diff가 필요한 경우)
    - MaplePatch.git의 {location}\{region}\Compare 폴더의 내용을 롤백한다
        - 그냥 커밋 revert하면 된다. 날짜별 폴더에 올라가는 zip 파일은 그냥 백업용으로 롤백되어도 무방함