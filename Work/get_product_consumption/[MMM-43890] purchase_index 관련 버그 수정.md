---
Bug:
---
## 개요
---
[MMM-43890](https://jiradev.nexon.com/browse/MMM-43890)

purchase_index가 같은 상품을 2회 이상 구매한 경우, order_id로 소비 상태 조회 시 다른 order_id의 소비 상태도 함께 조회 됨

## 원인
---
1. log_mail 조회 시, 성장 패키지 류 상품의 경우 메일 발송 시 서버에서 order_id를 가지고 있지 않아 receipt_key 컬럼이 비어 있음
2. 주어진 order_id에 대해 메일을 수령했는지 여부를 알 수 없음
3.  따라서, order_id 대신 purchase_index를 대신 남기도록 처리 -> 성장 패키지류의 경우 상품 하나를 계정 당 1회만 구매가 가능하기 때문에 order_id <=> purchase_index가 1대1 매칭이 가능

다만 성장 패키지류가 아닌, 다 회 구매가 가능한 상품의 경우 다른 order_id이지만 같은 purchase_index를 가질 수 있음(같은 상품을 여러번 구매)

메일을 읽어오는 GOT_ReadLogMailIdAll SP의 조건이
((receipt_key = i_order_id) OR (reserve_int_01 = i_item_idx)) 이기 때문에 다른 order_id, 같은 purchase_index를 가진 메일 목록까지 불러오게 됨
(reserve_int_01, i_item_idx = purchase_index)
## 작업 내용
---
1. API 코드 상에서, receipt_key가 null인 경우에만 purchase_index가 일치하는 메일 로그를 읽어오도록 수정(임시 조치, 아래 SP 업데이트 이후 원복 예정)
2. GOT_ReadLogMailIdAll SP 수정
	- (receipt_key = i_order_id) OR (reserve_int_01 = i_item_idx) (before)
	- (receipt_key = i_order_id) OR (receipt_key =null and reserve_int_01 = i_item_idx) (after)

## TroubleShooting
---
- 수정 후 receipt_flag = 1 -> 0이 되는 이슈
	- ReceiptKey가 비어 있다면, ""로 파싱됨
	- == null이 아니라 IsNullOrWhiteSpace() == true로 비교해야 함