# Jenkins 파이프라인 설정

#프로젝트 #개발 #인프라 #CIDE #Jenkins #도구 

---

<br>

## Credential 설정

<br>


- credential은 도커 허브, 깃 허브 등 젠킨스가 빌드하고 배포하는 과정에서 **로그인에 필요한 정보들**이라고 생각하면 된다.

#### 설정은 Jenkins 관리 > Security > Credential로 진입

![[do-messenger_screenshot_2025-06-04_10_50_01.png]]

#### 이후 system > global credential > 우측 상단에 *add credential*

![[do-messenger_screenshot_2025-06-04_10_49_09.png]]
여기서 도커 허브와 git hub 등 인증 정보를 기입하면 된다.

>[!tip] 참고
> 처음엔 Docker Hub를 통해 이미지를 관리하였지만, 이후 팀원과 공유하기 위하여
> Git Packages로 이미지 저장소를 이전하였다.
> >
> [[🔑 Docker Image를 Git repo로 관리하기]]
<br>

---

<br>

## pipeline 구성

<br>

파이프라인을 구성할 아이템을 선택한다.

![[do-messenger_screenshot_2025-06-04_10_56_17.png]]
자동 빌드보다는 수동빌드가 더 안전하다고 생각해서 폴링이나 웹훅 방식을 사용하지 않는다.

![[Pasted image 20250529235718.png|725]]
- 그래서 트리거는 아무것도 설정하지 않았다.

(직접 젠킨스 어드민 페이지 에서 빌드를 눌러야 빌드가 시작됨)

<br>

#### 스크립트 작성

```c
pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'ghcr.io/anonichat/app/anonichat' // docker image 레포지토리 주소
        DOCKER_IMAGE_TAG = '${BUILD_NUMBER}' // 이미지태그 자동 넘버링
        DOCKER_LATEST_TAG = 'latest' // 가장 최근 넘버를 할당
        CONTAINER_NAME = 'anonichat' // 컨테이너 명
        CONTAINER_PORT = '8000:8080' // 호스트포트 : 컨테이너 포트
    }

    stages {
        stage('1. Git Clone') {
            steps {
                echo '📂 GitHub에서 최신 코드 가져오기...'
                checkout([$class: 'GitSCM', // Jenkins git 플러그인
                    branches: [[name: '*/prod']], // 브렌치 명
                    userRemoteConfigs: [[
                        url: 'https://github.com/AnoniChat/App.git', // git repo url
                        credentialsId: 'githubRepository' // git 접속 credentials ID
                    ]]
                ])
                sh 'git log --oneline -5' // 최근 커밋기록 5개 로그 출력
            }
        }

        stage('2. Build') {
		    steps {
		        echo '🔨 anonichat 애플리케이션 빌드 중...'
		        script {
		            if (fileExists('gradlew')) {
		                sh 'chmod +x ./gradlew' // gradle 실행권한 부여
		                sh './gradlew clean build -x test' // 테스트 제외
		            } else {
		                error('gradlew 파일이 없습니다. Gradle 프로젝트인지 확인하세요!')
					}
				}
			}
		}

        stage('3. Test') {
		    steps {
		        echo '🧪 단위 테스트 실행 중...'
		        sh './gradlew test' // gradle 테스트 실행
		    }
		    post {
		        always {
		            junit 'build/test-results/test/*.xml' // 테스트 결과를 Junit 플러그인으로 리포트
		        }
		    }
		}

        stage('4. Docker Image Build') {
		    steps {
		        echo '🐳 Docker 이미지 빌드 중...'
		        script {
		            // Gradle 빌드 결과 JAR 파일 찾기
		            def jarFile = sh(
		                script: 'find build/libs -name "*.jar" | grep -v plain', // jar파일 탐색
		                returnStdout: true
		            ).trim()
		            
		            if (jarFile) {
		                echo "JAR 파일 발견: ${jarFile}"
		                sh """
		                docker build -t ${DOCKER_HUB_REPO}:${DOCKER_IMAGE_TAG} . // docker 빌드
		                docker tag ${DOCKER_HUB_REPO}:${DOCKER_IMAGE_TAG} ${DOCKER_HUB_REPO}:${DOCKER_LATEST_TAG} // 이미지 태깅 (항상 최근 버전에 `latest` 태그 부여)
		                """
		            } else {
		                error('JAR 파일을 찾을 수 없습니다!')
		            }
		        }
		    }
		}

        stage('5. Docker Image Push') {
            steps {
                echo '📤 GHCR에 이미지 푸시 중...'
                script { // githubPackage에 최신버전 이미지 push
                    withCredentials([usernamePassword(credentialsId: "githubPackage", 
                                                    passwordVariable: 'DOCKER_PASSWORD', 
                                                    usernameVariable: 'DOCKER_USERNAME')]) {
                        sh """
                        echo ${DOCKER_PASSWORD} | docker login ghcr.io -u ${DOCKER_USERNAME} --password-stdin
                        docker push ${DOCKER_HUB_REPO}:${DOCKER_IMAGE_TAG}
                        docker push ${DOCKER_HUB_REPO}:${DOCKER_LATEST_TAG}
                        docker logout
                        """
                    }
                }
                echo "✅ 이미지 푸시 완료!"
            }
        }

        stage('6. Deploy') {
            steps {
                echo '🚀 최신 이미지로 배포 중...'
                script {
                    try { //기존 컨테이너 삭제 후, 최신 이미지로 컨테이너 build/run
                        sh "docker stop ${CONTAINER_NAME} || true"
                        sh "docker rm ${CONTAINER_NAME} || true"
                        sh "docker rmi ${DOCKER_HUB_REPO}:${DOCKER_LATEST_TAG} || true"
                        sh "docker pull ${DOCKER_HUB_REPO}:${DOCKER_LATEST_TAG}"
                        sh """
                        docker run -d \
                          --name ${CONTAINER_NAME} \
                          --restart always \
                          -p ${CONTAINER_PORT} \
                          ${DOCKER_HUB_REPO}:${DOCKER_LATEST_TAG}
                        """
                        sh "sleep 10"
                        sh "docker ps | grep ${CONTAINER_NAME}"
                    } catch (Exception e) {
                        echo "❌ 배포 실패: ${e.getMessage()}"
                        throw e
                    }
                }
            }
        }
    }

    post {
	    always {
	        script {
	            node {  // node context 추가
	                echo '🧹 빌드 후 정리 작업...'
	                sh 'docker image prune -f || true'
	            }
	        }
	    }
	    success {
	        echo '🎉 CI/CD 파이프라인이 성공적으로 완료되었습니다!'
	    }
	    failure {
	        echo '❌ CI/CD 파이프라인이 실패했습니다.'
	    }
	}
}
```

다음 일련의 과정을 거쳐, 배포가 가능하다.

`Git repo(prod branch) push 후 → Jenkins 수동 빌드` 시

![[do-messenger_screenshot_2025-06-04_11_29_26.png]]

![[do-messenger_screenshot_2025-06-04_11_29_14.png]]

#### 적용 완료.