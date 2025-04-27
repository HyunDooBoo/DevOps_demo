# 🛠️ **Kubernetes CI/CD 자동화 프로젝트**

이 프로젝트는 **Jenkins**, **ArgoCD**, **Gitea**를 이용하여 **Kubernetes** 클러스터에 자동으로 애플리케이션을 배포하는 **CI/CD 파이프라인**을 구축하는 시스템 데모이다.

코드가 **Gitea**에 푸시되면, **Jenkins**가 이를 감지하여 Docker 이미지를 빌드하고 레지스트리에 푸시한 후, **ArgoCD**가 Git 저장소를 감시하여 자동으로 **Kubernetes** 클러스터에 배포한다.

## 📋 **프로젝트 흐름**

1. **개발자가 Gitea에 코드 푸시** 💻  

2. **Jenkins가 Gitea의 Webhook을 통해 변경 사항을 감지** 📡  

3. **Jenkins가 코드 빌드, Docker 이미지 생성 후 Docker Hub에 푸시** 🚀  

4. **Kubernetes용 manifest (`deployment.yaml`)의 이미지 태그를 최신으로 업데이트** 🔧  

5. **ArgoCD가 Git 저장소에서 변경 사항을 감지** 🔍  

6. **ArgoCD가 Kubernetes에 자동 배포 (`kubectl apply`)** 🌐  

## 🏗️ **설치 과정**

### 1. **Gitea 설치** (Git 호스팅 서비스)

Gitea는 **경량화된 Git 호스팅 서비스**로, **코드 저장소**를 관리 해준다.

1. Gitea 패키지 다운로드

```bash
wget -O gitea https://dl.gitea.io/gitea/1.19.2/gitea-1.19.2-linux-amd64
```

2. Gitea 바이너리 실행 권한 부여

```bash
chmod +x gitea
```

3. Gitea 실행 디렉터리 생성
```bash
sudo mkdir -p /var/lib/gitea
sudo mkdir -p /etc/gitea
```

4. 서비스에 Gitea 등록 (자동 실행)
```bash
sudo nano /etc/systemd/system/gitea.service
```
실행 후 아래 내용 추가
```ini
[Unit]
Description=Gitea
After=syslog.target network.target

[Service]
User=gitea
Group=gitea
Type=simple
ExecStart=/path/to/gitea web
Restart=always
Environment=USER=gitea HOME=/home/gitea

[Install]
WantedBy=multi-user.target
```
자동으로 시작되게 설정
```
sudo systemctl daemon-reload
sudo systemctl start gitea
sudo systemctl enable gitea
```

실행 사진
![image](https://github.com/user-attachments/assets/c539cf2e-5bff-4079-a639-739d77f2e7c8)


---

### 2. **Jenkins 로컬 설치** (CI 도구)

**Jenkins**는 코드 빌드, 테스트, Docker 이미지 생성을 담당한다. 아래는 로컬 환경에 Jenkins를 설치하는 과정이다.

#### Jenkins 설치

```bash
# Jenkins 설치에 필요한 패키지 설치
sudo apt update
sudo apt install openjdk-11-jdk

# Jenkins 리포지토리 추가
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee /etc/apt/trusted.gpg.d/jenkins.asc
sudo sh -c 'echo deb http://pkg.jenkins.io/debian/ / > /etc/apt/sources.list.d/jenkins.list'

# Jenkins 설치
sudo apt update
sudo apt install jenkins

# Jenkins 시작
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

설치가 완료되면, **Jenkins UI**에 접속 가능
![image](https://github.com/user-attachments/assets/8a88c9d1-9c6e-4c9c-8761-005b29d50315)


#### Jenkins 초기 설정
1. **Jenkins UI**에 접속 후, 초기 관리자 비밀번호로 로그인한다.:
   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
   ![image](https://github.com/user-attachments/assets/ec3e9822-0ec0-4c2c-b6e2-31157a3facbb)

2. 비밀번호를 입력하여 Jenkins의 초기 설정을 진행하고, 플러그인 설치 및 사용자 계정을 설정할 수 있다.

#### Jenkins의 주요 역할:
- **Gitea Webhook**을 통해 변경 사항을 감지
- **Docker 이미지** 빌드 및 **Docker Hub**로 푸시
- **Kubernetes manifest** 파일을 최신 이미지 태그로 업데이트

---

### 3. **ArgoCD 설치** (GitOps 기반 배포 도구)

**ArgoCD**는 **GitOps** 원칙을 따르며, Git 저장소에서 변경 사항을 감지하고 **Kubernetes 클러스터**에 자동으로 배포한다.

#### ArgoCD 설치:
1. Helm 리포 등록 및 업데이트
```
helm repo add argo <https://argoproj.github.io/argo-helm>
helm repo update
```

2. Master Node에서 ArgoCD를 설치
```bash
# ArgoCD 설치
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

3. ArgoCD 설치 확인
```bash
kubectl get all -n argocd
```


#### ArgoCD UI 접근:
젠킨스가 8080 포트에서 열려있으므로 8082 포트로 접근 가능하게 해준다.
```bash
# 포트포워딩으로 UI에 접근
kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8082:443
```
![image](https://github.com/user-attachments/assets/03324d9e-f94c-48e9-8eae-c253edfecb1b)


**ArgoCD UI**에 접속하여, 리포지토리 설정 및 애플리케이션을 등록한 후, **자동 배포**가 이루어지도록 설정해준다. 🌟
![image](https://github.com/user-attachments/assets/44fc875b-54ac-46a4-866c-dc176c09d7f1)
![image](https://github.com/user-attachments/assets/a3d72733-f755-4232-a40b-343385a9ad28)
![image](https://github.com/user-attachments/assets/6d65f983-055d-4ba5-9268-051a86fafea4)
![image](https://github.com/user-attachments/assets/7f8e6e19-61d6-4fdc-8423-a266907cf0cc)
![image](https://github.com/user-attachments/assets/91afef73-75aa-4f24-b3b9-7247da36e518)
![image](https://github.com/user-attachments/assets/e7f2578d-1048-4646-8abc-61de60d176d5)
![image](https://github.com/user-attachments/assets/cdb7bd3f-86ef-42f1-836b-2c1b97f9b89c)

---

### 4. **배포 파이프라인**

#### 1. Gitea에 코드 푸시
개발자가 **Gitea**에 코드를 푸시하면, Webhook을 통해 **Jenkins**가 변경 사항을 실시간으로 감지

#### 2. Jenkins 빌드 및 Docker 이미지 푸시
**Jenkins**는 **Gitea**에서 변경된 코드를 **빌드**하고, **Docker 이미지**를 생성하여 **Docker Hub**에 푸시

#### 3. Kubernetes manifest 파일 업데이트
**Jenkins**는 **Kubernetes**의 `deployment.yaml` 파일을 최신 이미지 태그로 자동으로 업데이트

#### 4. ArgoCD 자동 배포
**ArgoCD**는 **Git** 저장소의 변경 사항을 감지하고, 변경된 **manifest 파일**을 **Kubernetes 클러스터**에 자동으로 배포

Jenkinsfile
```bash
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "hyundooboo/my-app"
        HELM_REPO_URL = 'http://10.0.2.104:3000/hyundoo/my-app-helm.git'
        SPRING_DATASOURCE_URL = 'jdbc:mysql://10.0.2.104:3306/test?useSSL=false&allowPublicKeyRetrieval=true'
        SPRING_DATASOURCE_USERNAME = 'doo'
        SPRING_DATASOURCE_PASSWORD = 'vmffjtm1064!'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'http://10.0.2.104:3000/hyundoo/my-app.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    echo "🔨 Building Docker image: ${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                    docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        docker.withRegistry('', 'dockerhub') {
                            echo "📤 Pushing image to Docker Hub..."
                            docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").push()
                        }
                    }
                }
            }
        }
        stage('Update Helm values.yaml') {
            steps {
                script {
                    echo "🛠️ Cloning Helm chart repo..."
                    sh 'rm -rf my-app-helm'
                    sh "git clone ${HELM_REPO_URL} my-app-helm"

                    echo "🔧 Updating image tag to ${env.BUILD_NUMBER}..."
                    sh """
                      sed -i 's/^  tag: .*/  tag: "${env.BUILD_NUMBER}"/' my-app-helm/values.yaml
                    """

                    dir('my-app-helm') {
                        sh 'git config user.name "jenkins"'
                        sh 'git config user.email "jenkins@example.com"'
                        withCredentials([usernamePassword(credentialsId: 'gitea-token', usernameVariable: 'GITEA_USERNAME', passwordVariable: 'GITEA_PASSWORD')]) {
                            sh """
                            git remote set-url origin http://${GITEA_USERNAME}:${GITEA_PASSWORD}@10.0.2.104:3000/hyundoo/my-app-helm.git
                            git commit -am 'Update image tag to ${env.BUILD_NUMBER}'
                            git push origin HEAD:main
                            """
                        }
                    }
                }
            }
        }
    }
}
```

Dockerfile
```bash
# 1단계: 빌드
FROM eclipse-temurin:17-jdk AS build
WORKDIR /app
COPY . .
RUN chmod +x gradlew
RUN ./gradlew clean build -x test

# 2단계: 실행
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=build /app/build/libs/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

### 5. **배포된 애플리케이션 확인**

배포가 완료된 후, **Kubernetes 클러스터**에서 애플리케이션이 정상적으로 실행되고 있는지 확인 🖥️

```bash
# 배포된 애플리케이션의 상태 확인
kubectl get pods -n spring
```
![image](https://github.com/user-attachments/assets/5b423b96-c588-44f0-8dac-29a539f7da1b)


배포된 **Pod**를 확인하고, 애플리케이션이 정상적으로 실행 중인지 확인할 수 있다.

---

## 🔧 **기술 스택**
- **Gitea**: Git 호스팅 서비스
- **Jenkins**: CI 도구, 빌드 및 이미지 생성
- **Docker**: 컨테이너 이미지 빌드 및 레지스트리 푸시
- **ArgoCD**: GitOps 기반 자동 배포
- **Kubernetes**: 컨테이너 오케스트레이션
