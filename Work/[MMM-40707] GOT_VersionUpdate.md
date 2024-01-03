## 개요
---
[[MMM-40707] GOT_VersionUpdate](https://jiradev.nexon.com/browse/MMM-40707)
1. [[공통] 패치이력 - MapleM - NK Confluence (nexon.com)](https://confluence.nexon.com/pages/viewpage.action?pageId=181312261) 페이지
2. Jenkins Global DB
에 GOT binary, table 패치 이력(버전, hash)를 기록하는 잡 개발


## 작업 내용
---
- ManageGOT > GOT > [Deploy_GOT_VersionUpdate](http://10.10.56.42:8080/view/ManageGOT/job/GOT/job/Deploy_GOT_VersionUpdate/) 에 생성

1. {GOT주소}/VERSIONDETAIL (GOTGate.sln 내부의 `public string GetVersionDetail()`에서 GOT 버전 정보를 얻어온 후
    
2. [[공통] 패치이력 - MapleM - NK Confluence (nexon.com)](https://confluence.nexon.com/pages/viewpage.action?pageId=181312261)
    
    위 문서의 curl 커맨드로 지라에 패치 이력(버전, commit hash) 전송(issuetype: Epic)
    
3. Jenkins Global DB에 패치 이력 저장
    

## TroubleShooting
---
진짜 결과물 대비 삽질 제일 심한 듯

- FreeStype Project에서 환경변수를 가져오질 못함
    
- 파이프라인 프로젝트로 만들어도 못가져옴
    
    - 그냥 위에서 inject 하지 않고 pipeline 스크립트 위로 가져와 버림
- 프로토콜이 없다고 Http request를 못함
    
- 스트림이 끊겼다고 출력을 못함
    
    - getInputStream().getText()를 여러 번 호출 할 수 없음
- \\n을 인식을 못함
    
- Common Library 로딩을 못함(갑자기 그럼)
    
- Cannot find current thread 이슈
    
    - **`Cannot contact buildserver_175: java.io.IOException: cannot find current thread`**
    - 모든 빌드에서 저걸 띄우면서 실패, 근데 지라에 올라는 감
    - 파이프라인 스크립트에 try, catch 구문 추가하니 sucess로 뜸. 근데 왜 sucess 뜨는 건지를 모르겠음. 에러 메시지는 똑같이 뜸……
- java.io.NotSerializableException 이슈
    
    - **`Caused: java.io.NotSerializableException: groovy.json.internal.LazyMap`**
    - pipeline.getdatabaseconnection 함수 불러올 때, LazyMap 객체를 serlalize 할 수 없다며 오류 띄움
- [**java.io](http://java.io) 관련 Exception 전부 아래 방법으로 해결됨**
    
    - 기존에 URL 객체 생성 후 openConnection, get 함수로 response 값 얻어오던 방식에서
        
    - 아래 같은 방식으로 변경
        
        ```cpp
        def response = httpRequest(
            url: "<http://$>{GOTIP}/VERSIONDETAIL",
            httpMode: 'GET'
        )
        def responseBody = response.getContent()
        def responseStatus = response.status
        ```
        
        openconnection() 함수의 return이 non-serializable한 객체였던 것으로 추정.
        
- 중국 stage만 timeout 뜨는 이슈
    
    - **Treating class org.apache.http.conn.HttpHostConnectException(Connect to 35.76.95.61:8080 [/35.76.95.61] failed: Connection timed out: connect) as 408 Request Timeout**
    - 이렇게 뜨면서 VERSIONDETAIL에 접근을 못함
    - 브라우저나 postman으로는 잘 되는데, VersionDashBoard나 젠킨스 상으로는 접근을 못함
- HTTP 500 에러 리턴하면서 GOT가 동작을 안 함
    
    - 에러 로그 보면 보통 이런 식으로 찍혀있음
        
        ```cpp
        Internal Server error Could not find a part of the path 'C:\\Got\\GotGateStage\\Xml\\MetaData'.
        ```
        
    - Build / Deploy MetaData 해주면 원상복구(Deploy만 할 경우 안 생김)
        
    - 근데 종종 이걸 해 줘도 다시 그럴 때가 종종 있는데, 원인을 모르겠음. MetaData가 언제 사라지는거지??
        
- **분명히 versionupdate까지 갔는데 지라에 안 올라 감**
    
    - 특히 ExecuteAll로 Table, Binary 동시에 Deploy 하는 경우 보통 Table이 지라에 업데이트가 안 됨
    - 이것도 원인을 모르겠음……
    - 뭔가 동시에 curl을 쏴서 그런가? 싶기도 해서 일단 잡 옵션에서 concurrent 빌드 막아 놓음
    - 9/14 아침 빌드 돌 때 업데이트 되는지 확인
- [[2024-01-02]]
	- 스크립트 수정
		- 형근님 요청
		- 젠킨스 DB의 patch 테이블에 GOT 버전이 들어가지 않으면, FTP 업로드 시 폴더 이름이 이상하게 들어가는 문제를 수정할 수 있음
		- patch.Update()대신 patch.DBUpdatePatchItems()만 실행하면 patch 테이블에는 기록되지 않고, patchitem 테이블에만 기록됨
	- 종종 응답 못 받아서 실패하는 이슈
	- BuildType 존재 의미??
		- Binary와 Table 둘 중 하나만 Deploy하는 경우, 버전 지라에 구분하여 이력을 남기기 위해 사용한 것으로 추정
		- DB 업데이트 칠 때는 어차피 한꺼번에 하고 있었으므로 의미 없음
		- 현재는 Deploy_Binary와 Deploy_MetaData 잡이 Update_GOT로 합쳐져서 역시 의미가 없어짐
		- 없애버려도 될 듯?