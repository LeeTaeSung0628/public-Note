# 🚢Kubernetes(k8s)란

#Tools #Kubernetes #k8s

---
# <font color="#9bbb59">등장 배경과 필요성</font>

### ✅ 기존 인프라의 한계
- 과거에는 서버 1대에 하나의 앱을 설치하는 방식 → 리소스 낭비
- 가상화(VM)는 개선책이었지만 느리고 무겁다

### ✅ 컨테이너의 등장
- **Docker**를 사용한 컨테이너(꼭 Docker가 아니더라도..) 기술은 경량화된 격리 실행환경 제공
- 문제: 수백~수천 개의 컨테이너를 **관리, 스케줄링, 모니터링**를 사람이 직접 한다는것은 많이 어렵다.

### ✅ Google 내부의 **Borg 시스템**에서 착안
- 구글이 자사 서비스에서 10년 넘게 써온 내부 컨테이너 오케스트레이터 “Borg”의 아이디어를 오픈소스로 풀어낸 것이 **Kubernetes**(줄여서 *k8s*)이다.


## 그래서 k8s가 정확히 무엇인가?
- 위에서 설명했듯 <u>여러개의 컨테이너를 자동으로 배포, 스케일링, 복구, 관리</u>하는 컨테이너 **오케스트레이션 플랫폼**이다.

---

#### k8s의 핵심 개념

|개념|설명|
|---|---|
|**Pod**|컨테이너의 가장 작은 배포 단위. 하나 이상의 컨테이너 묶음 (보통 1개)|
|**Node**|물리적/가상 머신으로 컨테이너가 실행되는 워커 노드|
|**Cluster**|여러 노드의 집합|
|**Deployment**|선언적 방식으로 Pod 배포 및 버전 관리|
|**Service**|Pod에 고정된 접근 경로를 제공 (Load Balancer, ClusterIP 등)|
|**Namespace**|논리적 그룹화 단위. 여러 팀이 공유 클러스터를 사용할 때 격리 목적|
|**ConfigMap / Secret**|설정 및 민감 정보를 외부에서 주입 가능하게 함|
|**Volume / PV / PVC**|스토리지 추상화 및 연결|
#### k8s의 구성
``` c

                 +-------------------+
                 |     Master Node   |
                 |-------------------|
                 | - API Server      |
                 | - Scheduler       |
                 | - Controller Mgr  |
                 | - etcd (DB)       |
                 +---------+---------+
	                       |
              ┌────────────┴────────────┐
              ↓                         ↓
      +-------------+          +---------------+
      |  Worker Node|          |  Worker Node  |
      |-------------|          |---------------|
      | kubelet     |          | kubelet       |
      | kube-proxy  |          | kube-proxy    |
      | containerd  |          | containerd    |
      +-------------+          +---------------+
             ↓                         ↓
           [Pod: 컨테이너 + 네트워크 + 볼륨]
```

| 구성 요소                  | 설명                                         |
| ---------------------- | ------------------------------------------ |
| **API Server**         | 모든 명령의 입구 (RESTful API)                    |
| **etcd**               | 분산 키-값 저장소, 클러스터 상태 저장                     |
| **Scheduler**          | 새로 생성된 Pod을 어떤 Node에 배치할지 결정               |
| **Controller Manager** | 상태를 모니터링하고 필요한 조치를 수행 (복제, 자동 복구 등)        |
| **kubelet**            | Node 내 컨테이너 상태 보고 및 유지                     |
| **kube-proxy**         | 네트워크 트래픽을 올바른 Pod으로 라우팅                    |
| **Container Runtime**  | ex) containerd, cri-o (Docker는 더 이상 기본 아님) |

---

# <font color="#9bbb59">k8s와 ArgoCD</font>
<br/>

k8s가 컨테이너 *오케스트레이션 플랫폼* (어디에, 어떠게 배포할지 결정 )이라면,
argoCD는 [[GitOps]]기반의 _Continuous Delivery 도구_ (무엇을, 언제, 어떤 방식으로 배포할지 관리) 이다.

## k8s만으로는 충분하지 않은가?

#### SSOT(Single Source of Truth) - 단일 진실 공급원
- k8s의 모든 배포는 yaml파일을 통해 이뤄진다. 
→이때, 배포를 수행하는 개발자마다 각자의 PC에서 yaml파일을 만들어 관리한다면, 많은 노력이 들어가게 된다.
따라서, [[GitOps]]방법론에 따라, 배포와 관련된 모든 코드를 Git을 통해 관리하도록 하는것이다.


즉, <u>ArgoCD는 Kubernetes를 위한 GitOps 자동 배포 계층이다</u>.

Git Repository (Manifest 저장) ← (*감시*) ← Argo CD → (*반영*) → Kubernetes Cluster

---
## Argo CD가 Kubernetes 배포 전략에 주는 변화

| 항목                        | 기존 방식 (`kubectl`, Jenkins 등) | Argo CD 방식                  |
| ------------------------- | ---------------------------- | --------------------------- |
| 배포 주체                     | 운영자 또는 자동 스크립트               | **Git 상태가 배포 기준**           |
| 이력 관리                     | CI/CD 도구의 로그 또는 수동 기록        | Git commit log              |
| 롤백                        | 수동 실행                        | Git commit revert + sync    |
| 다중 환경 관리 (dev/stage/prod) | 스크립트 복잡도 증가                  | Git repo 분리 or overlay로 단순화 |
| 다중 클러스터 배포                | CI 스크립트 복잡함                  | Argo CD의 multi-cluster 지원   |


# <font color="#9bbb59">요약</font>
1. Argo CD는 Kubernetes의 배포 자동화 담당
2. Kubernetes는 애플리케이션을 실행하고, Argo CD는 그것을 배포/관리함
3. Git을 소스 오브 트루스로 삼고, Argo CD는 상태를 감시하고 Kubernetes와 싱크 맞춤
4. 실시간 배포 이력 추적, 자동 복구, 롤백, 멀티클러스터 지원 등 GitOps 핵심 도구
5. Kubernetes를 쓰는 조직이라면 Argo CD는 필수에 가까움 (Jenkins는 CI로만 남게 됨)
