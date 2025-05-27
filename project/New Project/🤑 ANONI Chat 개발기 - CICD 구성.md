# 🤑 ANONI Chat 개발기 - CICD 구성

#프로젝트 #개발 #개요 #구상 #인프라 #CIDE #Jenkins #Vultr

---

<br>

이전 시간에 Clude환경에 docker컨테이너를 구동시키는 것 까지 진행하였다.

▶ [[🤑 ANONI Chat 개발기 - infra setup]]

<br>

이번 시간에는 CICD환경을 구성하겠다.

---

<br>

# <font color="#76923c">Jenkins 연동하기</font>

<br>

## flow Graph

```c
graph TD
	A[GitHub Push]
	B[Jenkins Git Pull]
	C[Gradle Build Jar]
	D[Docker Build & Tag]
	E[DockerHub Push]
	F[서버 Pull + Deploy]
```
