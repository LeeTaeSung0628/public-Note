# 🔒 투자하기 서비스 락(Lock)기법 개선안

#공부 #Lock #DB #SPRING #동시성

---

# hello 투자 flow

1. *investView.html* / 상품 선택 후 투자하기버튼 클릭
2. *custom.invest.js* / kyc체크, 개인투자자 체크 -> go-invest modal(investView.html)
3. *go-invest modal* / 투자위험 동의체크, 유효성 체크 후 투자 실행 -> InvestController
4. *InvestController* / MP_INVEST investAction -> investService.investAction
5. *investService.investAction* / 투자신청기록, api상태체크 -> SERVICE 프로젝트
6. *SERVICE 프로젝트* investmentsService.p2pCenterInvestmentRegister(투자신청기록)
7. *p2p(금결원) 투자신청기록* / 투자 유효성검사(시간 내 재투자, 금액체크) -> p2p센터 투자잔액 조회(api)
8. 잔액조회 후 / 수익률,기간 업데이트 등등 .. 로직 처리 후 **P2PCenter 투자신청 등록**
9. 여기까지 정상 처리되면, async(비동기)로 기표처리