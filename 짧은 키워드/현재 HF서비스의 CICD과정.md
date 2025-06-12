# 현재 HF서비스의 CI/CD과정

## 1. Git에서 Commit / push하여 소스통합
	 local브랜치에서 작업 후 각(dev/stg/prod)프로젝트로 소스를 통합(merge)한다.

## 2. Jenkins에서 Git 소스 Build,Test&Publish 후 이미지화 (CI)
	jar, 메니페스트 file 등 소스,배포에 필요한 파일들 이미지 화

## 3. Docker에서 이미지 정보 받은 후 ArgoCD로 이미지 Pull
	 Jenkin에서 이미지화된 배포에 필요한 파일,소스들을 ArgoCD로 Pull한다.

## 4. ArgoCD로 이미지파일 K8S에 배포 (CD)
	 Jenkins에서 받은 이미지파일과 매니패스트파일을 기반으로 실제 서버에 배포한다.

## 5. K8S(쿠버네티스)에서 이미지파일 Docker를 통해 실행(서버실행)

### 각 과정에서 오류 및 예외 사항들을 찾고 대응 할 수 있도록
### 로그를 제공한다.

![[Pasted image 20240531162124.png]]
#### elastic 에서도 동작중인 서버의 모든 로그를 검색, 필터링 할 수 있다. 하나의 trace_id로 묶인 트렌젝션 단위를 기준으로 오류를 찾고 대응할 수 있다.

---

# 현재 HF서비스의 CICD과정

## 1. Git에서 Commit / push하여 소스통합
	 local브랜치에서 작업 후 각(dev/stg/prod)프로젝트로 소스를 통합(merge)한다.

## 2. Jenkins에서 Git 소스 Build,Test&Publish 후 이미지화 (CI)
	jar, 메니페스트 file 등 소스,배포에 필요한 파일들 이미지 화

## 3. Docker에서 이미지 정보 받은 후 ArgoCD로 이미지 Pull
	 Jenkin에서 이미지화된 배포에 필요한 파일,소스들을 ArgoCD로 Pull한다.

## 4. ArgoCD로 이미지파일 K8S에 배포 (CD)
	 Jenkins에서 받은 이미지파일과 매니패스트파일을 기반으로 실제 서버에 배포한다.

## 5. K8S(쿠버네티스)에서 이미지파일 Docker를 통해 실행(서버실행)

### 각 과정에서 오류 및 예외 사항들을 찾고 대응 할 수 있도록
### 로그를 제공한다.

![[Pasted image 20240531162124.png]]
#### elastic 에서도 동작중인 서버의 모든 로그를 검색, 필터링 할 수 있다. 하나의 trace_id로 묶인 트렌젝션 단위를 기준으로 오류를 찾고 대응할 수 있다.

---

# Hello CI-CD

![[Pasted image 20240722180706.png]]


---

```c
	            [ Client (Browser) ]'script 트리거 / REST 요청'
	                │       │
	         HTTP REST     WebSocket
	                ▼       ▼
┌─── '리버스 프록시' ─── [ NGINX ] '실질적 웹서버 / 인증 관리 등'
│		                │
│	      ┌─────────────┴─────┐
│	      ▼                   ▼
│	[ Frontend Server ]   [ Node.js WebSocket Server ]
│	 'html, js 정적리소스제공'  │    '리얼타임 경량 서버'
│	                          ▼
│	          ┌───────────────┴────────────────┐
│	          ▼                                ▼
└──▶[ Spring Boot API Server ]         [ Redis Server ] '캐시'
	          │        ▲                      ▲
	          │        └─── REST or RPC ──────┘
	          ▼
	     [ Database Server ]
```
