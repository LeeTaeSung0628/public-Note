# ⏰ Script 로딩 지연 이슈

#Script #로딩지연 #성능개선

---

# 원인
## - 특정 정적오브젝트의 로딩 지연
![[Pasted image 20241105110415.png|575]]
- mainLayout은 Contents loading 후 Script를 호출하기 때문에, 특정 정적 오브젝트의 로딩이 완전히 마무리 될때 까지 실행되지 않음.
**=> ex) 상품이 로딩되지 않은 상태로 최대 20초 가량을 대기 하게 됨**
![[Pasted image 20250227140437.png|600]]

---

# 로딩 지연 오브젝트

![[Pasted image 20250227140620.png]]
- icon(아이콘)
- font(폰트)

---

# 수정

#### document.addEventListener('DOMContentLoaded', function())
## DOMContentLoaded 리스너를 사용하게 된다면,
 현재 원인이 되는 외부자원(이미지/CSS)가 로딩되기 이전에 이벤트를 트리거 할 수 있다.
 이때, HTML DOM 트리가 준비 된 후(HTML 템플릿이 준비된 후)에 실행되기 때문에 `modelAndView.addObject()`로 전달한 데이터를 확정적으로 받아올 수 있다.

---
# Case

## APP **main** 진입시
### Local

- 기존 메인 진입 소요시간 case 1 (로컬)
![[Pasted image 20241105110637.png|500]]
**23초**

---

- 기존 메인 진입 소요시간 case 2 (로컬)
![[Pasted image 20241105110713.png|500]]
**22초**

---
### prod

- 기존 메인 진입 소요시간 case 1 (운영)
![[Pasted image 20241107115602.png|475]]
**21초**

---

- 기존 메인 진입 소요시간 case 2 (운영)
![[Pasted image 20241107115830.png|475]]
**21초**


---
# 변경 후 case

- 변경된 메인 진입 소요시간 case 1 (로컬)
![[Pasted image 20241107121044.png|500]]
**0.201초**

---

- 변경된 메인 진입 소요시간 case 2 (로컬)
![[Pasted image 20241107121202.png|500]]
**0.297초**

---

- 변경된 메인 진입 소요시간 case 3 (로컬)
![[Pasted image 20241107121939.png|500]]
**0.232초**

---
## APP **투자하기** 진입시
> 리로드 후 스크립트(상품 리스트)로드 시까지

- 기존 투자하기 소요시간 case 1
![[Pasted image 20250227114750.png]]
**22.4초**
![[Pasted image 20250227114826.png]]

---

- 기존 투자하기 소요시간 case 2
![[Pasted image 20250227115004.png]]
**22.6초**
![[Pasted image 20250227115015.png]]

---

- 기존 투자하기 소요시간 case 3
![[Pasted image 20250227115158.png]]
**22.5초**
![[Pasted image 20250227115148.png]]

---

# 변경 후

- 변경 투자하기 소요시간 case 1
![[Pasted image 20250227121009.png|500]]
**0.373초**

---

- 변경 투자하기 소요시간 case 2
![[Pasted image 20250227121046.png|500]]
**0.362초**

---

- 변경 투자하기 소요시간 case 3
![[Pasted image 20250227121942.png|475]]
**0.360초**


>[!info] #### main.html 템플릿이 완료됨과 동시에 출력이 가눙하다

---

# 스크립트 데이터 로드 실패 검증

## 👵기존 로직 로드
![[Pasted image 20250227122010.png]]
## 🍼변경 후 로직 로드
![[Pasted image 20250227122736.png]]

## ❌ **실패한 요청 목록**
1. **aceat.js?advid=1954816599**  - 에이스트레이더
	https://cdn.nhnace.com/libs/aceat.js?advid=195481
2. **synchronizer.js**  - 네이버 관련 서버
	https://ssl.pstatic.net/melona/libs/gfp-nac-module/synchronizer.js
3. **collect?en=page_view&dr=localhost&dl=http%...**  - 구글
	https://www.google.com/ccm/collect... - 구글
4. **869613409/?random=1740626824235&cv=11&f...**  
	https://www.google.com/pagead/1p...
5. **869613409/?random=1740626824235&cv=11&f...**  
6. **869613409/?random=1110219531&cv=11&fst=...**  

> 내부망 사용으로 인한 요청 실패, 중요 데이터 없음.

---

# 성능 요약 📊

![[output (14).png]]

## ✅ 성능 데이터 (단위: 초)

| 테스트 | Main (개선 전) | Main (개선 후) | ProductList (개선 전) | ProductList (개선 후) |
|--------|--------------|--------------|----------------------|----------------------|
| 첫 번째 | 23.0 | 0.201 | 22.4 | 0.373 |
| 두 번째 | 22.0 | 0.297 | 22.6 | 0.362 |
| 세 번째 | 21.0 | 0.232 | 22.5 | 0.360 |
**Main 페이지**: 기존 **21~23초 → 0.201~0.297초**로 약 **99% 속도 향상** 
**ProductList 페이지**: 기존 **22.4~22.6초 → 0.360~0.373초**로 약 **98% 속도 향상**

---

- *정상 투자 확인*
![[Pasted image 20250304121056.png|475]]
![[Pasted image 20250304121009.png|475]]
![[Pasted image 20250304121455.png|475]]
![[Pasted image 20250304121338.png|500]]

