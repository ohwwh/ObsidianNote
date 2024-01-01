## 개요
---
- [[2024-01-01]]
- 중국 긴급 점검 대응으로 Stage 빌드 후 DeliveryNew 실행
- Binary, Resource Reprocess 과정에서 Aborted 에러 띄우며 실패
## 원인
---
- DeliveryNew 잡 스크립트 내부의 ClientPipeline() 함수에, wait:false 파라미터가 들어감
- 해당 파라미터 설정한 경우, Reprocess 잡 실행 후 바로 다음 단계로 넘어감. 
- Reprocess의 ret이 null이라 실패하게 됨

## TroubleShooting
---

