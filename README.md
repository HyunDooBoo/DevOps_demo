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
기본적으로는 `http://localhost:8080`에서 접속 가능

#### Jenkins 초기 설정
1. **Jenkins UI**에 접속 후, 초기 관리자 비밀번호로 로그인한다.:
   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
2. 비밀번호를 입력하여 Jenkins의 초기 설정을 진행하고, 플러그인 설치 및 사용자 계정을 설정할 수 있다.

#### Jenkins의 주요 역할:
- **Gitea Webhook**을 통해 변경 사항을 감지
- **Docker 이미지** 빌드 및 **Docker Hub**로 푸시
- **Kubernetes manifest** 파일을 최신 이미지 태그로 업데이트

---

### 3. **ArgoCD 설치** (GitOps 기반 배포 도구)

**ArgoCD**는 **GitOps** 원칙을 따르며, Git 저장소에서 변경 사항을 감지하고 **Kubernetes 클러스터**에 자동으로 배포한다.

#### ArgoCD 설치:

Master Node에서 ArgoCD를 설치한다.
```bash
# ArgoCD 설치
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

#### ArgoCD UI 접근:
```bash
# 포트포워딩으로 UI에 접근
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**ArgoCD UI**에 접속하여, 리포지토리 설정 및 애플리케이션을 등록한 후, **자동 배포**가 이루어지도록 설정해준다. 🌟

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

---

### 5. **배포된 애플리케이션 확인**

배포가 완료된 후, **Kubernetes 클러스터**에서 애플리케이션이 정상적으로 실행되고 있는지 확인 🖥️

```bash
# 배포된 애플리케이션의 상태 확인
kubectl get pods
```

배포된 **Pod**를 확인하고, 애플리케이션이 정상적으로 실행 중인지 확인할 수 있다.

---

## 🔧 **기술 스택**
- **Gitea**: Git 호스팅 서비스
- **Jenkins**: CI 도구, 빌드 및 이미지 생성
- **Docker**: 컨테이너 이미지 빌드 및 레지스트리 푸시
- **ArgoCD**: GitOps 기반 자동 배포
- **Kubernetes**: 컨테이너 오케스트레이션
