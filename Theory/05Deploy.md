# Deploy

- [배포 구조](#배포-구조)
- [로컬용과 배포용 compose](#로컬용과-배포용-compose)
- [Nginx 프록시와 HTTPS](#nginx-프록시와-https)
- [Jenkins](#jenkins)
- [Spring Boot 단독 배포](#spring-boot-단독-배포)
- [비밀 정보 관리](#비밀-정보-관리)

<br>

---

### 배포 구조

> 서버(EC2 같은 클라우드 가상 머신)에 Docker와 Compose를 설치하고, 화면(Nginx), 서버(Spring Boot), DB(MySQL), 캐시(Redis)를 컨테이너로 실행한다. 외부에 열어 두는 것은 화면 컨테이너의 80, 443 포트뿐이고, 서버와 DB는 내부 네트워크에서만 접근하게 한다.

```
인터넷 -> Nginx(80, 443) -> /bn/ 요청은 Spring Boot(bn-container:8080) -> MySQL, Redis
```

---

### 로컬용과 배포용 compose

> `docker-compose-local.yml`은 개발 확인용으로 화면 3000번, 서버 8080번 포트를 호스트에 그대로 열어 둔다. 배포용 `docker-compose.yml`은 서버 포트를 열지 않고(주석 처리), 화면 컨테이너만 80과 443을 열며, 인증서 폴더를 읽기 전용으로 연결한다. 환경마다 달라지는 부분만 파일로 나눠 관리한다.

<br>

- 예제 코드: [배포용 compose](../D04ComposeDeploy/docker-compose.yml), [로컬용 compose](../D04ComposeDeploy/docker-compose-local.yml), [서버 Dockerfile](../D04ComposeDeploy/BN/Dockerfile), [DB Dockerfile](../D04ComposeDeploy/DB/Dockerfile), [Redis Dockerfile](../D04ComposeDeploy/REDIS/Dockerfile)

---

### Nginx 프록시와 HTTPS

> Nginx는 정적 파일(React 빌드 결과)을 제공하고, `/bn/`으로 시작하는 요청은 `proxy_pass`로 Spring Boot 컨테이너에 전달하는 역방향 프록시이다. 화면과 API가 같은 주소로 접근되어 CORS 문제가 줄어든다. `try_files $uri /index.html`은 React Router의 주소로 직접 접속해도 `index.html`을 돌려줘서 SPA 라우팅이 동작하게 한다. HTTPS는 Let's Encrypt 인증서(certbot으로 발급)를 컨테이너에 연결해 `443`에서 TLS로 서비스하고, `80`으로 온 요청은 `301`로 HTTPS로 보낸다. 인증서 갱신용 경로(`/.well-known/acme-challenge/`)는 리다이렉트에서 제외한다. 설정 파일의 도메인은 `www.example.com`으로 바꿔 두었으므로 본인 도메인으로 바꿔 쓴다.

<br>

- 예제 코드: [Nginx 설정](../D04ComposeDeploy/FN/nginx.conf), [화면 Dockerfile](../D04ComposeDeploy/FN/Dockerfile)

---

### Jenkins

> Jenkins는 코드가 바뀌면 자동으로 빌드하고 배포하는 CI/CD 도구이다. 이 저장소의 Jenkins 컨테이너 이미지는 공식 `jenkins/jenkins:lts`에 Docker CLI와 AWS CLI를 추가해, Jenkins 안에서 Docker 명령으로 이미지를 빌드하고 AWS를 다룰 수 있게 했다.

<br>

- 예제 코드: [Jenkins Dockerfile](../D04ComposeDeploy/JENKINS/Dockerfile)

---

### Spring Boot 단독 배포

> 화면 없이 Spring Boot만 배포하는 최소 프로젝트이다.

<br>

- 예제 코드: [단독 배포 안내](../D06SpringBootOnly/README.md), [build.gradle](../D06SpringBootOnly/build.gradle)

---

### 비밀 정보 관리

> DB 비밀번호, 서버 접속 정보, 토큰은 저장소에 올리지 않는다. 이 저장소의 compose와 Dockerfile 값은 `YOUR_DB_PASSWORD`로 바꿔 두었다. 실제 배포에서는 `.env` 파일(`.gitignore`에 등록)이나 서버 환경 변수, 비밀 관리 서비스를 사용하고, compose에서는 `${DB_PASSWORD}`처럼 참조한다.
