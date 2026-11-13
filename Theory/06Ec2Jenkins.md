# EC2 Jenkins

- [WSL과 Docker 설치](#wsl과-docker-설치)
- [서버 초기 설정](#서버-초기-설정)
- [JDK, Git, 빌드](#jdk-git-빌드)
- [MySQL 설치](#mysql-설치)
- [Jenkins 설치](#jenkins-설치)
- [Jenkins 연동과 자동 배포](#jenkins-연동과-자동-배포)

<br>

---

### WSL과 Docker 설치

> Windows에서 Docker Desktop은 WSL2 위에서 동작한다. 관리자 PowerShell에서 WSL을 설치하고 업데이트한 뒤 Hyper-V(가상화) 기능이 켜져 있는지 확인하고 Docker Desktop을 설치한다. 설치 후 앱을 열어 `docker images`로 동작을 확인한다.

<br>

```powershell
wsl --install
wsl --update
```

---

### 서버 초기 설정

> AWS 프리티어 EC2(리전은 서울)를 만들고 디스크는 프리티어 한도인 30GiB로 지정한다. 탄력적 IP를 연결하고 보안 그룹에 필요한 포트(SSH, 8080, 9090 등)를 연다. Windows에서는 PuTTY로 접속하며 발급받은 키 파일을 PuTTY 형식(`.ppk`)으로 바꿔 인증에 쓴다. 접속 계정은 `ec2-user`이고 `sudo su`로 관리자가 된다. 접속 후 시간대와 스왑 메모리를 먼저 설정한다. 프리티어는 메모리가 작아 빌드 중 메모리가 부족해지기 때문에 디스크 일부를 스왑으로 쓴다.

<br>

```bash
# 시간대를 서울로 변경
sudo rm /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime

# 2GB 스왑 파일 만들기
sudo dd if=/dev/zero of=/swapfile bs=128M count=16
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo swapon -s

# 재부팅 후에도 스왑이 유지되도록 /etc/fstab 마지막 줄에 추가
/swapfile swap swap defaults 0 0
free
```

<br>

| 명령 | 역할 |
| --- | --- |
| `dd if=/dev/zero of=/swapfile` | 빈 파일(스왑용) 생성 |
| `chmod 600` | 소유자만 읽고 쓰게 제한 |
| `mkswap`, `swapon` | 스왑 영역으로 만들고 즉시 활성화 |
| `free` | 메모리와 스왑 사용량 확인 |

---

### JDK, Git, 빌드

> 서버에 JDK 21과 Git을 설치하고 저장소를 내려받아 Gradle로 빌드한 뒤 `java -jar`로 직접 실행해 본다. 내려받은 `gradlew`에는 실행 권한이 없어 `chmod o+x gradlew`가 필요하다. 서버가 열리는지는 `http://서버IP:8080`으로 확인한다. Gradle이 JDK를 자동으로 찾도록 `settings.gradle`에 `foojay-resolver-convention` 플러그인을 추가한다.

<br>

```bash
yum install -y java-21 git
mkdir /app && cd /app
git clone https://github.com/YOUR_ACCOUNT/YOUR_REPO.git
cd YOUR_REPO
chmod o+x gradlew
./gradlew build
cd build/libs
java -jar demo-0.0.1-SNAPSHOT.jar
```

<br>

```groovy
plugins {
    id 'org.gradle.toolchains.foojay-resolver-convention' version '0.8.0'
}
rootProject.name = 'demo'
```

---

### MySQL 설치

> MySQL 공식 Yum 저장소 패키지(rpm)를 등록한 뒤 서버를 설치한다. 운영체제 버전과 저장소 경로가 맞지 않아 설치가 실패하면 저장소 파일의 버전 부분을 호환되는 값(9)으로 고정하고 캐시를 지운 뒤 다시 설치한다. 설치 후 `/var/log/mysqld.log`의 임시 비밀번호로 접속해 root 비밀번호를 바꾸고, 데이터베이스와 외부 접속용 사용자를 만든다. 외부(예: MySQL Workbench)에서 접속하려면 보안 그룹에서 3306 포트도 열어야 한다.

<br>

```bash
yum install https://dev.mysql.com/get/mysql84-community-release-el10-3.noarch.rpm
sudo sed -i 's|/el/\$releasever/|/el/9/|g' /etc/yum.repos.d/mysql-community*.repo
sudo yum clean all
sudo yum install mysql-community-server
systemctl restart mysqld
systemctl status mysqld
systemctl enable mysqld    # 부팅 시 자동 시작
```

<br>

```sql
alter user root@localhost identified by 'YOUR_DB_PASSWORD';
create database testdb;
create user dbuser@'%' identified by 'YOUR_DB_PASSWORD';
grant all privileges on testdb.* to dbuser;
```

---

### Jenkins 설치

> Jenkins는 Yum 저장소를 등록해 서버에 직접 설치하고 systemd 서비스로 실행한다. 기본 포트 8080은 Spring Boot와 겹치므로 서비스 파일의 포트 설정을 9090으로 바꾼다. 서비스를 다시 시작하고 `systemctl enable`로 자동 시작을 켠 뒤 보안 그룹에 9090 포트를 연다. 처음 접속하면 `/var/lib/jenkins/secrets/initialAdminPassword`의 초기 비밀번호를 입력하고 추천 플러그인을 설치한 뒤 관리자 계정을 만든다.

<br>

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/rpm-stable/jenkins.repo
sudo yum upgrade
sudo yum install fontconfig jenkins
sudo systemctl daemon-reload
sudo vim /usr/lib/systemd/system/jenkins.service   # JENKINS_PORT를 9090으로 변경
sudo systemctl restart jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```

<br>

> 빌드가 오프라인으로 표시되는 것은 임시 디렉터리 용량 부족이 흔한 원인이다. `/etc/fstab`에 `tmpfs /tmp tmpfs size=5g,mode=1777 0 0`을 추가하고 `mount -a`, 재부팅하면 `/tmp` 크기가 늘어난다.

---

### Jenkins 연동과 자동 배포

> 자동 배포 흐름은 GitHub에 푸시, 웹훅으로 Jenkins 호출, Gradle 빌드, 이전 프로세스 종료, 새 JAR 실행 순이다. GitHub에서 개인 액세스 토큰(classic)을 만들어 Jenkins의 Credentials에 등록하고, 시스템 설정의 GitHub 서버에 연결한다. JDK는 `alternatives --config java`로 나온 경로를, Gradle은 버전 이름을 Tools에 등록한다. GitHub Integration과 Post build task 플러그인을 설치하고 프리스타일 프로젝트를 만든다.

<br>

| 설정 위치 | 값 |
| --- | --- |
| 소스 코드 관리 | 저장소 주소, 브랜치 `*/main` |
| 빌드 유발 | GitHub hook trigger |
| Build Steps | Gradle 버전 지정, Tasks `clean build` |
| 빌드 후 조치 | Post build task, Log text `SUCCESS` 일 때 스크립트 실행 |

<br>

> 배포 스크립트는 실행 중인 이전 프로세스를 `pgrep -f`로 찾아 종료한 뒤 새 JAR를 `nohup`으로 백그라운드 실행한다. Jenkins가 빌드를 끝낼 때 자식 프로세스를 같이 죽이므로 `BUILD_ID=dontKillMe`를 지정해 살려 둔다.

<br>

```bash
JAR_PATH="/var/lib/jenkins/workspace/DEPLOY_SBONLY/build/libs/demo-0.0.1-SNAPSHOT.jar"
TARGET_PID=$(pgrep -f "java -jar $JAR_PATH")
if [ -n "$TARGET_PID" ]; then kill -9 "$TARGET_PID"; sleep 2; fi
BUILD_ID=dontKillMe nohup java -jar "$JAR_PATH" > /dev/null 2>&1 &
disown
```

<br>

- 토큰, 초기 비밀번호, DB 비밀번호, 서버 IP는 문서나 저장소에 적지 않는다. 토큰은 필요한 권한만 체크해 만들고 노출되면 바로 폐기한다.
- 예제 코드: [Spring Boot 단독 배포](../D06SpringBootOnly/build.gradle), [Jenkins Dockerfile](../D04ComposeDeploy/JENKINS/Dockerfile)
