# 👩‍👧‍👦 Analytics 공통모듈 ADMIN용 데이터 추출

#프로젝트 #개발 #SPRING #AOP

---

# **작업 개요 먼저 보기**
# ->[[👩‍👧‍👦 marketing Analytics 공통모듈 제작기]]

---

# DB에서 관리? -> ADMIN페이지로 생성?

*hf_marketing_code*

| KEY `hitCode` | `codeName` |
| ------------- | ---------- |
| kakao1st      | 카카오-비즈보드   |
| naver2nd      | 네이버광고      |
| google1st     | 구글광고       |

*hf_marketing_target*

| KEY `idx` | `targetClass`          | `targetMethod`              | `methodParameters`             | `pageName` |
| --------- | ---------------------- | --------------------------- | ------------------------------ | ---------- |
| 3         | marketingHitController | gateLoanView                | "String","HttpServletResponse" | 대출하기 페이지   |
| 2         | LoanController         | noticeView                  | ""                             | 한도조회 버튼    |
| 1         | LoanController         | identityVerificationRequest | "G5Member"                     | 한도조회 확인버튼  |

*hf_marketing_hit_log*

| KEY `idx` | `hitCode` | `hit_uid`                            | `targetMethodKey` | `insert_date`       |
| --------- | --------- | ------------------------------------ | ----------------- | ------------------- |
| 3         | kakao1st  | 7fd530dd-672d-422b-aea6-e48062e41fd0 | 3                 | 2024-12-31 13:07:19 |
| 2         | naver2nd  | bd4a96fb-63f8-4a85-b859-fa30faf15f24 | 2                 | 2024-12-31 13:07:19 |
| 1         | google1st | bd4a96fb-63f8-4a85-b859-fa30faf15f24 | 1                 | 2024-12-31 13:07:19 |


---
# 1. enum객체의 상수값들을 빌드시에 DB에서 셋팅할 수 있는지?
# 2. pageURL/pageType 과 targetClass/targetMethod/methodParameters중 하나로 특정
# 3. DB 최적화를 진행하자 -> 중복데이터 제거

---

# 포인트 컷 생성시 DB 호출하여 타겟 메서드 세팅
![[Pasted image 20250102181842.png]]

---

# ADMIN에서 어떻게 받을것인가?
| 날짜    | 광고 유형    | 유입(인원) | 히트  | 페이지      | 페이지별 히트 |
| ----- | -------- | ------ | --- | -------- | ------- |
| 12/02 | 카카오-비즈보드 | 4      | 19  | 대출하기 페이지 | 10      |
|       |          |        |     | 한도조회 버튼  | 5       |
|       |          |        |     | 이용안내 페이지 | 4       |
| 12/02 | 구글광고     | 3      | 10  | 대출하기 페이지 | 7       |
|       |          |        |     | 한도조회 버튼  | 3       |

=> 해당형태로 컬럼을 고정시키고, 동적으로 유입 수 를 출력한다.
#### =>  기존 EXCEL로직을 사용하기 위해 service로직에서 row를 병합하여 view에 건내주도록 개발하였다.

---

# repositoty에서 세부데이터를 List형태로 받을 수 있도록 쿼리 작성
-> 그 후 페이지에 뿌려주기

```
Date: 01-02
Ad Type: 구글광고
Total Inflow: 4
Total Hit: 27
Detail Information:
  Page Name: 대출하기 페이지
  Hit: 9
  Page Name: 한도조회 버튼
  Hit: 12
  Page Name: 이용안내 페이지
  Hit: 6
```
