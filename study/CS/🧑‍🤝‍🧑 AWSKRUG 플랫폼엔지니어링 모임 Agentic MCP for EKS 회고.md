# 🧑‍🤝‍🧑 AWSKRUG 플랫폼엔지니어링 모임 Agentic MCP for EKS 회고

#공부 #AWS #AWSKRUG #플랫폼 #모임 #회고 #EKS #MCP #AI 


---

<Br>

![[do-messenger_screenshot_2025-07-02_11_35_48.png|500]]

<br>
최근 나는 vultr플랫폼에서 ci/cd를 구현하여 운영중에 있다.

AWS가 아닌 vultr를 사용하게 된 계기는 역시 운영비 부분이 가장 컸다. 하지만, 이번 세미나를 통해서 다시 AWS로 운영플랫폼을 옮길까 고민을 하게 되었다. 그만큼 ai에이전트 기반 운영 자동화의 성능이 생각보다 더 뛰어났다는 소리다.

AWSKRUG 플랫폼엔지니어링 모임에 참여후 학습한 내용과 소감을 간단하게 정리하려고 한다.

<br>

# <font color="#8db3e2">개요</font> 

- AWSKRUG(AWS한국사용자모임)에서 주최한 Agentic/MCP for EKS 핸즈온랩 강의 이다.
- *Amazon Q Developer*와 연계된 AI에이전트가 어떻게 **k8s**기반 운영을 지원할 수 있는지.
- 코드 생성 + 실행 + 트러블슈팅이 하나의 흐름으로 연결되는 *Agentic Workflow*가 어떤 모습인지.
- 실제 Git 기반 설정 파일과 EKS 리소스를 자동으로 생성·관리하는 방식이 어떻게 되는지.

<br>

---

<br>

# <font color="#8db3e2">Agentic/MCP for EKS</font>

각각의 단어에 대해 알아보자.

<br>

## 1️⃣ Agentic (Agentic AI)

- `Agentic`는 자율성에 관한 유형 중 하나이다.
- `Agent`는 주로 **자율적으로 목표를 달성하고, 상황을 인식하며, 의사결정을 수행하는 독립 실행 단위**를 말한다.
- 예를 들어, AI Agentic 시스템은 단순히 명령만 수행하는 게 아니라, 스스로 다음 행동을 계획하고 실행 흐름을 제어한다.
- DevOps 맥락에서는 **운영 자동화 에이전트**, **자율 오케스트레이션**, **자기 복구(Self-Healing)** 시스템 같은 데에서 `Agentic` 개념이 쓰인다.

<br>

## 2️⃣ MCP (Model Context Protocol)

- `MCP`는 보통 **Model Context Protocol**의 줄임말로 사용된다.
- 2024년 11월 AI기업 Anthrophic이 오픈소스로 공개한 새로운 기술 표준이다.
- **AI가 여러 데이터와 도구를 통합해 활용할 수 있도록 하는 표준 프로토콜(규약), 연결 규칙**이다.
- ex) Google, MongoDB, Obsidian 등 MCP를 활용하면 CLI프롬프트 한번으로 여러작업을 모두 ai처리에 맡길 수 있음(읽기,쓰기 모두 가능)

<br>

## 3️⃣ EKS (Elastic Kubernetes Service)

- `EKS`는 **Elastic Kubernetes Service**의 약자.
- AWS가 제공하는 **Managed Kubernetes 클러스터 서비스**이다.
- 사용자는 직접 Kubernetes Control Plane을 설치하거나 유지보수하지 않고,
    - AWS가 API Server, etcd, Control Plane 확장 등을 자동으로 관리.
- 사용자는 워커 노드(Node Group)만 구성해서 Pod을 배포하면 된다.
- EKS는 AWS IAM, VPC, ELB, CloudWatch 같은 AWS 서비스와 통합된다.
- 온프레미스에서 직접 Kubernetes 운영하는 것보다 <u>운영 복잡성이 낮아진다</u>.


<br>

#### 즉, `Agentic for EKS`는 Amazon Q Developer CLI와 `MCP`을 활용하여 Amazon `EKS` 환경에서 에이전트 기반 자동화와 운영 효율화를 실현하는 것이다.

---

<br>

# <font color="#8db3e2">Amazon Q Developer CLI Basics</font>

<br>

위 과정은 `Q CLI`를 셋팅하는 것 부터 시작한다.

```bash
curl --proto '=https' --tlsv1.2 -sSf \
  "https://desktop-release.q.us-east-1.amazonaws.com/latest/q-x86_64-linux.zip" \
  -o "q.zip"
  
✔ Do you want q to modify your shell config? → Yes
✔ Select login method → Use for Free with Builder ID

Confirm the following code in the browser:
Code: xxxx-xxxx  
Open this URL: https://view.awsapps.com/start/#/device?user_code=xxxx-xxxx

To learn more about MCP safety, see https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-mcp-security.html

Welcome to 

 █████╗ ███╗   ███╗ █████╗ ███████╗ ██████╗ ███╗   ██╗     ██████╗ 
██╔══██╗████╗ ████║██╔══██╗╚══███╔╝██╔═══██╗████╗  ██║    ██╔═══██╗
███████║██╔████╔██║███████║  ███╔╝ ██║   ██║██╔██╗ ██║    ██║   ██║
██╔══██║██║╚██╔╝██║██╔══██║ ███╔╝  ██║   ██║██║╚██╗██║    ██║▄▄ ██║
██║  ██║██║ ╚═╝ ██║██║  ██║███████╗╚██████╔╝██║ ╚████║    ╚██████╔╝
╚═╝  ╚═╝╚═╝     ╚═╝╚═╝  ╚═╝╚══════╝ ╚═════╝ ╚═╝  ╚═══╝     ╚══▀▀═╝ 
                                                        
```

- Q CLI는  터미널에서 실행되는 ai기반 도우미이다.
- AWS환경과 상호작용할 수 있는 CLI를 제공한다.

<br>

예를 들어 다음과 같은 질문도 가능하다.

```c
“EKS 클러스터를 보호하기 위한 몇 가지 모범 사례는 무엇인가요?”

---

EKS 클러스터를 보호하기 위한 모범 사례는 다음과 같습니다:

1. 네트워크 보안
   • 프라이빗 서브넷에 워커 노드 배치
   • 보안 그룹을 사용하여 트래픽 제한
   • AWS PrivateLink 또는 VPC 엔드포인트 사용

2. IAM 및 RBAC 구성
   • 최소 권한 원칙 적용
   • IAM 역할과 Kubernetes RBAC 통합
   • 서비스 계정에 IAM 역할 사용 (IRSA)

3. 암호화 및 보안 설정
   • EKS 클러스터 및 EBS 볼륨에 대한 암호화 활성화
   • Secrets 암호화 구현
   • AWS KMS 키 사용

4. 컨테이너 보안
   • 신뢰할 수 있는 이미지만 사용
   • 이미지 스캐닝 구현 (ECR 스캐닝 또는 서드파티 도구)
   • 컨테이너 실행 시 루트 권한 제한

... 생략

이러한 모범 사례를 구현하면 EKS 클러스터의 보안 태세를 크게 향상시킬 수 있습니다.
```

## 아니면, 다음과 같이 인프라 셋팅 파일 생성 또한 가능하다.

```c
~/environment/myeks 디렉토리에 nginx-deployment.yaml이라는 이름으로 기본 Kubernetes nginx 배포를 위한 YAML 파일을 생성해주세요.

---

Kubernetes nginx 배포를 위한 기본 YAML 파일을 생성하겠습니다.


🛠️  Using tool: fs_write
 ⋮ 
 ● Path: /home/ec2-user/nginx-deployment.yaml

+     1: apiVersion: apps/v1
+     2: kind: Deployment
+     3: metadata:
+     4:   name: nginx-deployment
+     5:   labels:
+     6:     app: nginx
+     7: spec:
+     8:   replicas: 3
+     9:   selector:
+    10:     matchLabels:
+    11:       app: nginx

... 중략
+    35:   selector:
+    36:     app: nginx
+    37:   ports:
+    38:   - port: 80
+    39:     targetPort: 80
+    40:   type: ClusterIP


Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:

> t
Creating: /home/ec2-user/nginx-deployment.yaml

 ⋮ 
 ● Completed in 0.0s


nginx-deployment.yaml 파일을 성공적으로 생성했습니다. 이 파일에는 다음 내용이 포함되어 있습니다:

1. Deployment 리소스:
   • 3개의 복제본(replicas)
   • nginx:1.14.2 이미지 사용
   • 포트 80 노출
   • 리소스 제한 설정:
     • CPU 제한: 0.5 코어, 요청: 0.2 코어
     • 메모리 제한: 512Mi, 요청: 256Mi

2. Service 리소스:
   • ClusterIP 타입 서비스
   • 포트 80을 통해 Deployment에 연결

```

 1. docker image 다운로드
 2. 셋팅 yml파일 작성
 3. 권한부여
 4. 리소스 차트 작성
 5. 컨테이너 실행
 6. 테스트

<br>

#### 전 과정을 자연어 스크립트 기반으로 처리할 수 있다.
또한, 위 과정은 아주 단편적인 기능일 뿐 활용도는 무궁무진 하며 캐시데이터로 학습이 지속적으로 이루어졌을 때는 성능이 더욱 높아질 여지가 충분하다.



---

<br>

# <font color="#8db3e2">Q CLI / MCP Architecture 의 구조와 동작방식</font>

<br>

```c
┌────────────┐       ┌────────────┐       ┌────────────┐
│  터미널 입력│ ─▶    │ Q CLI      │─▶     │ MCP Client │
└────────────┘       └────────────┘       └────────────┘
		                                          │
		                           JSON-RPC over stdio/stdout
		                                          │
		                                  ┌────────────┐
		                                  │ MCP Server │
		                                  └────────────┘
		                                          │
		                                  ┌────────────┐
		                                  │ 로컬 Tools │
		                                  └────────────┘

```

#### 실행 프로세스

1. 사용자 입력
2. Q CLI 입력 처리
3. 컨텍스트 수집 시작
4. 컨텍스트 수집 완료
5. MCP Client 초기화
6. 도구 발견
7. LLM 페이로드 구성
8. LLM 분석 요청
9. LLM 분석 완료
10. 도구 실행 시작
11. kubectl 명령 실행
12. 결과 분석 및 응답 생성
13. 사용자에게 응답 전달

<br>

## 왜 AWS Q CLI 에서 동작시킬까?

1. **보안**
    - 민감한 AWS 자격증명을 로컬에서만 처리 → <u>민감정보가 로컬을 벗어날 때에는 자체 마스킹 처리</u>
    - Kubeconfig 같은 클러스터 접근정보 보호
2. **성능**
    - 로컬 파일 시스템 접근 빠름
    - <u>네트워크 호출 최소화</u>
3. **일관성**
    - 사용자 환경 맞춤 <u>컨텍스트 자동 수집</u> → 위에서 기술한 대로, 학습하여 성능이 더욱 개선(최적화) 될 수 있다.
4. **실제 실무 파일**
    - AWS 프로파일, 자격증명, K8s 설정, 현재 디렉터리 사용

<br>

#### 하지만 Q CLI도 만능은 아니다.

질문자의 역량에 따라서, 불필요한 반복 역질문과 같이 토큰을 소모하거나, 보안/운영상의 큰 위험요소를 남길 수 있다.

이를 미연에 방지하고, 최적화 하는 방법이 바로 **프로파일과 컨텍스트**이다.

<br>

---

<br>

# <font color="#8db3e2">프로파일과 컨텍스트</font>

<br>

컨텍스트를 활용하여 Amazon Q가 다양한 작업을 더 효과적으로 도와줄 수 있다.

## 컨텍스트(Context): 
- *개발 규칙, 프로젝트 세부사항, 코딩 표준* 등을 포함한 정보
- Amazon Q의 응답을 더 정확하고 맞춤화되게 만든다.

<br>

## 프로파일(Profile):
- Amazon Q가 사용자와 시스템에 맞춰 반응하도록 설정된 *컨텍스트 모음*입니다.

<br>

## 글로벌 컨텍스트(Global context):
- 모든 프로파일에 공통적으로 적용되는 규칙

<br>

## 워크스페이스 컨텍스트(Workspace context):
- 특정 프로파일에만 적용되는 규칙

<br>
컨텍스트 파일은 일반적으로 Markdown 형식이며, 다음과 같은 내용을 포함할 수 있다.

- 프로젝트 요구사항
- 개발 가이드라인
- 보안 정책
- 배포 표준 등

<br>

ex)
```c
# shell 에서 실행합니다.
cat > ~/environment/myeks/kubernetes_standards.md << 'EOF'
# Kubernetes 배포 표준

## 일반 가이드라인
- 모든 Kubernetes 리소스는 다음과 같은 라벨을 반드시 포함해야 합니다:
  - app: {애플리케이션 이름}
  - environment: {dev|staging|prod}
  - team: platform-engineering
  - cost-center: cc-12345

## 배포 구성
- 리소스 요청(requests) 및 제한(limits)을 반드시 명시해야 합니다
- CPU 요청: 최소 100m, 최대 1000m
- 메모리 요청: 최소 128Mi, 최대 1Gi
- 배포 전략은 롤링 업데이트를 사용하며, maxSurge는 25%, maxUnavailable도 25%로 설정
- 모든 컨테이너에 readiness 및 liveness probe를 설정해야 합니다
- 이미지 풀 정책은 "IfNotPresent"로 설정할 것

## 컨테이너 보안
- 컨테이너는 root가 아닌 사용자로 실행해야 함 (runAsNonRoot: true)
- 가능하면 읽기 전용 루트 파일 시스템을 사용할 것 (readOnlyRootFilesystem: true)
- 권한 상승 허용은 false로 설정 (allowPrivilegeEscalation: false)
- 모든 capabilities는 제거하고 필요한 항목만 명시적으로 추가

## 네트워킹
- 서비스 포트는 프로토콜에 따라 명확히 이름 지정 (예: http, https, grpc)
- Ingress 리소스는 TLS를 사용하고, HTTP 요청은 HTTPS로 리디렉션해야 함
- 네임스페이스 간 트래픽 제한을 위해 NetworkPolicy 사용

## 예시: 컨테이너 포트 설정
yaml
ports:
• name: http
  containerPort: 8080
  protocol: TCP
EOF
```

<br>

일련의 과정을 통해서 각 서비스 마다의 *프로파일*을 최적으로 구성한다면,

업무 효율은 어마어마 할것 같다.

---

## <font color="#8db3e2">후기</font>

<br>

이전, *claude* 를 사용해 <u>fileSystem, git, obsidian</u> MCP를 연결하여 포스팅 자동화 시스템을 구축한 경험이 있다.

이때에는 단순히 *작업 플로우를 줄이는데*에 목적이 있다고 생각했다. 하지만, 이번 세미나를 통해서 배운 지식은 그것을 훨씬 뛰어넘었다.


보통의 경우엔 컨트롤하지 못하는 영역까지 ai의 범위가 넓어졌고, 양질의 정보만을 선택적으로 학습한 전문적인 ai솔루션이 나옴에 따라

사용 경험이 이전과는 확연하게 달라짐을 느꼈다. 앞으로는 AI로 인하여, 업무 학습방법이 이전과는 크게 달라질 것으로 생각된다.

<u>코딩(행위)이 아닌 개념과 아키텍처로?</u>

이는 비단, AWS의 Q CLI뿐 아니라 모든 AI 에이전트에 해당되는 사항일 것이다.
<br>

그러나 AI는 여전히 생산성을 높여주는 보조도구일 뿐, 전체적인 시스템과 세부 내용들을 이해하지 못하고 사용한다면 위험한 무기가 될 수 도있다고 생각한다. *때문에 다방면으로 넓은 지식을 쌓을 필요가 생겼다.*