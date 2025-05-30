<br>
<br>


Docker 이미지를 Docker Hub가 아닌 GitHub 팀 저장소에서 관리하려면, GitHub Container Registry(GHCR)를 활용하여 이미지를 저장하고 배포할 수 있습니다. 아래에 그 과정을 단계별로 자세히 설명하겠습니다.

---

## 1️⃣ GitHub Container Registry(GHCR) 이해하기

GHCR은 GitHub에서 제공하는 컨테이너 이미지 저장소로, 개인 계정이나 조직 계정에 이미지를 저장하고 관리할 수 있습니다. 이미지를 특정 저장소와 연결하거나 독립적으로 관리할 수 있으며, 퍼블릭 또는 프라이빗으로 설정할 수 있습니다.

---

## 2️⃣ Personal Access Token(PAT) 생성 및 설정

GHCR에 이미지를 푸시하려면 인증이 필요합니다. 이를 위해 Personal Access Token(PAT)을 생성하고 설정해야 합니다.[GitHub Docs+6CTO.ai+6Shipyard+6](https://cto.ai/blog/build-and-deploy-a-docker-image-on-ghcr/?utm_source=chatgpt.com)

1. GitHub 계정의 **Settings > Developer settings > Personal access tokens**로 이동합니다.[CTO.ai+1Medium+1](https://cto.ai/blog/build-and-deploy-a-docker-image-on-ghcr/?utm_source=chatgpt.com)
    
2. **Generate new token**을 클릭하고, 다음 권한을 선택합니다:[CTO.ai+1DEV Community+1](https://cto.ai/blog/build-and-deploy-a-docker-image-on-ghcr/?utm_source=chatgpt.com)
    
    - `write:packages`[CTO.ai+3Shipyard+3DEV Community+3](https://shipyard.build/blog/gha-recipes-build-and-push-container-registry/?utm_source=chatgpt.com)
        
    - `read:packages`[Stack Overflow+1Medium+1](https://stackoverflow.com/questions/74326389/unable-to-push-docker-image-to-github-package-using-github-actions?utm_source=chatgpt.com)
        
    - `delete:packages` (선택 사항)
        
3. 토큰을 생성하고 안전한 곳에 저장합니다.
    
4. 로컬 환경에서 다음 명령어로 Docker에 로그인합니다:
    
    bash
    
    복사편집
    
    `echo $CR_PAT | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin`
    

여기서 `CR_PAT`는 생성한 토큰을 환경 변수로 설정한 것입니다.

---

## 3️⃣ Docker 이미지 빌드 및 태깅

Dockerfile이 있는 디렉토리에서 다음 명령어로 이미지를 빌드하고 태깅합니다:

bash

복사편집

`docker build -t ghcr.io/OWNER/REPOSITORY/IMAGE_NAME:TAG .`

- `OWNER`는 GitHub 사용자명 또는 조직명입니다.
    
- `REPOSITORY`는 이미지와 연결할 GitHub 저장소명입니다.
    
- `IMAGE_NAME`은 이미지의 이름입니다.
    
- `TAG`는 이미지의 태그(예: `latest`, `v1.0.0`)입니다.[GitHub Docs](https://docs.github.com/packages/working-with-a-github-packages-registry/working-with-the-container-registry?utm_source=chatgpt.com)
    

예시:

bash

복사편집

`docker build -t ghcr.io/my-org/my-repo/my-app:latest .`

---

## 4️⃣ 이미지 푸시

빌드한 이미지를 GHCR에 푸시합니다:[GitHub+5DEV Community+5CTO.ai+5](https://dev.to/willvelida/pushing-container-images-to-github-container-registry-with-github-actions-1m6b?utm_source=chatgpt.com)

bash

복사편집

`docker push ghcr.io/OWNER/REPOSITORY/IMAGE_NAME:TAG`

예시:

bash

복사편집

`docker push ghcr.io/my-org/my-repo/my-app:latest`

푸시가 완료되면, 해당 이미지는 GitHub의 **Packages** 섹션에서 확인할 수 있습니다.

---

## 5️⃣ GitHub Actions를 통한 자동화 (선택 사항)

CI/CD 파이프라인의 일환으로, GitHub Actions를 사용하여 코드 변경 시 자동으로 이미지를 빌드하고 GHCR에 푸시할 수 있습니다.

1. 저장소의 `.github/workflows` 디렉토리에 YAML 파일을 생성합니다.
    
2. 다음과 같은 워크플로우를 정의합니다:
    
    yaml
    
    복사편집
    
    `name: Build and Push Docker Image  on:   push:     branches: [ main ]  jobs:   build:     runs-on: ubuntu-latest      steps:       - name: Checkout code         uses: actions/checkout@v3        - name: Log in to GHCR         uses: docker/login-action@v2         with:           registry: ghcr.io           username: ${{ github.actor }}           password: ${{ secrets.GITHUB_TOKEN }}        - name: Build and push Docker image         run: |           docker build -t ghcr.io/${{ github.repository_owner }}/${{ github.repository }}:latest .           docker push ghcr.io/${{ github.repository_owner }}/${{ github.repository }}:latest`
    

이 워크플로우는 `main` 브랜치에 푸시될 때마다 트리거되어 이미지를 빌드하고 GHCR에 푸시합니다.

---

## 6️⃣ 이미지 접근 및 사용

GHCR에 푸시된 이미지는 다음 명령어로 사용할 수 있습니다:

bash

복사편집

`docker pull ghcr.io/OWNER/REPOSITORY/IMAGE_NAME:TAG`

또한, Kubernetes 등의 오케스트레이션 도구에서 해당 이미지를 참조하여 배포할 수 있습니다.

---

이러한 과정을 통해 Docker 이미지를 GitHub 팀 저장소에서 효과적으로 관리하고 배포할 수 있습니다. 추가적인 자동화나 보안 설정이 필요하다면, GitHub Actions의 다양한 기능과 GHCR의 접근 제어 설정을 활용할 수 있습니다.
