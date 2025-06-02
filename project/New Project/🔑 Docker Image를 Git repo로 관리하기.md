# 🔑 Docker Image를 Git repo로 관리하기

#프로젝트 #개발 #인프라 #docker #git

---

<br>
<br>

#### private한 Docker이미지를 팀 레포에서 관리하기 위해 DockerHub → Git repo 로 옮기는 작업이다.

Docker 이미지를 Docker Hub가 아닌 GitHub 팀 저장소에서 관리하려면,

**GitHub Container Registry(GHCR)** 를 활용하여 이미지를 저장하고 배포할 수 있다. 

---

<br>

# <font color="#76923c">GHCR의 주요 특징</font>

1. **GitHub와의 통합**
	- GHCR는 GitHub와 통합되어 있어, 저장소와 연동하여 컨테이너 이미지를 관리할 수 있다.  
2. **퍼블릭 및 프라이빗 이미지 지원**
	- 이미지를 퍼블릭 또는 프라이빗으로 설정할 수 있어, 접근 제어가 가능하다.
3.  **세분화된 권한 관리**
	- Personal Access Token(PAT)을 사용하여 세분화된 권한 설정이 가능하다.
4. **GitHub Actions와의 연동**
	- GitHub Actions를 통해 CI/CD 파이프라인을 구축하고, 자동으로 이미지를 빌드하고 GHCR에 푸시할 수 있다.


---

# <font color="#76923c">Personal Access Token(PAT) 생성 및 설정</font>

<br>


GHCR에 이미지를 푸시하려면 인증이 필요하다. 이를 위해 Personal Access Token(PAT)을 생성하고 설정해야 한다.

1. GitHub 계정의 **Settings > Developer settings > Personal access tokens**로 이동한다.

2. **Generate new token**을 클릭하고, 권한을 선택
    
    - `write:packages`
        
    - `read:packages`
        
    - `delete:packages` (선택 사항)
        ![[Pasted image 20250530173935.png]]
3. 토큰을 생성하고 안전한 곳에 저장 후
    
4. 로컬 환경에서 다음 명령어로 Docker에 로그인한다.

```bash
$  docker login ghcr.io -u "gitHub아이디" 
Password: #<Pesonal Access Token> 입력
```   


---

<br>


# <font color="#76923c">Docker 이미지 빌드 및 태깅(선택)</font>

<br>


Dockerfile이 있는 디렉토리에서 다음 명령어로 이미지를 빌드하고 태깅한다.

```bash
docker tag "이미지ID" ghcr.io/"gitHub아이디"/"repo이름"/"이미지:태그"
```

---

<br>

# <font color="#76923c">이미지 푸시</font>

<br>

위에서 빌드(태깅)한 이미지를 **GHCR**에 푸시한다.

```bash
docker push ghcr.io/"gitHub아이디"/"repo이름"/"이미지:태그"
```


푸시가 완료되면, 해당 이미지는 GitHub의 **Packages** 섹션에서 확인할 수 있다.

![[Pasted image 20250602104005.png|900]]
![[Pasted image 20250602104018.png|925]]

---

<br>

# <font color="#76923c">이미지 접근 및 사용</font>


<br>


**GHCR**에 푸시된 이미지는 다음 명령어로 사용할 수 있다.

```bash
docker pull ghcr.io/OWNER/REPOSITORY/IMAGE_NAME:TAG
```

![[Pasted image 20250602112229.png|725]]
또한, Kubernetes 등의 오케스트레이션 도구에서 해당 이미지를 참조하여 배포할 수 있다.

![[Pasted image 20250602113419.png|775]]

---

<br>

이러한 과정을 통해 **Docker 이미지를 GitHub 팀 저장소에서 효과적으로 관리하고 배포할 수 있다**.

또한, <u>추가적인 자동화나 보안 설정이 필요하다면</u>, GitHub Actions의 다양한 기능과 GHCR의 접근 제어 설정을 활용할 수 있다.
