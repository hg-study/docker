# Docker

- [Docker란](#docker란)
- [이미지와 컨테이너](#이미지와-컨테이너)
- [기본 명령](#기본-명령)
- [Docker 사용 이유](#docker-사용-이유)
- [컨테이너 네트워크](#컨테이너-네트워크)

<br>

---

### Docker란

> Docker는 애플리케이션과 실행에 필요한 환경(런타임, 라이브러리, 설정)을 하나로 묶어 어디서나 똑같이 실행하게 해 주는 컨테이너 도구이다. "내 컴퓨터에서는 되는데 서버에서는 안 된다"는 환경 차이 문제를 줄이고, 개발, 테스트, 배포를 같은 환경으로 맞춘다. 가상 머신이 운영체제 전체를 흉내 내는 것과 달리 컨테이너는 호스트 운영체제의 커널을 공유해서 가볍고 빠르게 시작된다.

---

### 이미지와 컨테이너

| 구분 | 설명 |
| --- | --- |
| 이미지(Image) | 실행에 필요한 파일과 설정을 담은 읽기 전용 템플릿. 클래스에 해당 |
| 컨테이너(Container) | 이미지를 실행한 인스턴스. 객체에 해당하며 하나의 이미지로 여러 개를 띄울 수 있음 |
| 레지스트리(Registry) | 이미지를 저장하고 공유하는 저장소. Docker Hub가 대표적 |
| Dockerfile | 이미지를 만드는 순서를 적은 파일 |

---

### 기본 명령

```bash
docker build -t myapp .              # Dockerfile 로 이미지 만들기 (-t 는 이름:태그)
docker images                        # 이미지 목록
docker run -d -p 8080:8080 --name app myapp   # 컨테이너 실행 (-d 백그라운드, -p 호스트포트:컨테이너포트)
docker ps -a                         # 컨테이너 목록 (-a 는 중지된 것 포함)
docker logs -f app                   # 로그 보기
docker exec -it app sh               # 실행 중인 컨테이너에 접속
docker stop app && docker rm app     # 중지하고 삭제
docker rmi myapp                     # 이미지 삭제
```

<br>

> 포트 매핑 `-p 3000:80`은 내 컴퓨터의 3000번 포트로 들어온 요청을 컨테이너의 80번 포트로 전달한다. 컨테이너를 지우면 안에 쓴 데이터도 사라지므로 보존할 데이터는 볼륨(volume)에 저장한다.

---

### Docker 사용 이유

> 팀원 모두가 같은 환경으로 실행하고, 서버 설치 과정을 Dockerfile 코드로 남겨 재현할 수 있으며, 여러 서비스(DB, Redis, 서버, 화면)를 한 번에 띄우고 지울 수 있다. 이 저장소는 React 화면, Spring Boot 서버, MySQL, Redis를 컨테이너로 묶어 실행하고 서버에 배포하는 과정을 단계별로 담았다.

<br>

- 예제 코드: [첫 Dockerfile](../D00Init/01/Dockerfile)

---

### 컨테이너 네트워크

> 컨테이너끼리 이름으로 통신하려면 같은 사용자 정의 네트워크에 넣는다. 네트워크를 만들고 `--network`로 컨테이너를 연결하면 컨테이너 이름이 주소 역할을 해서 `ping db-container`처럼 부를 수 있다. 컨테이너 안에서 `localhost`는 자기 자신이라 다른 컨테이너에 접근할 때는 반드시 이름을 쓴다. 포트 연결 `-p 호스트:컨테이너`는 호스트에서 접속하기 위한 것이고, 컨테이너 간 통신에는 컨테이너 포트를 그대로 쓴다.

<br>

```bash
docker network create --driver bridge my-network
docker network ls
docker run -d --name db-container --network my-network -p 3330:3306 mysql:1.1
docker run -d --name redis-container --network my-network -p 6479:6379 redis:1.1
docker exec -it db-container bash
docker run -d --name bn-container --network my-network -p 8080:8080 bn:1.1
docker run -d --name fn-container --network my-network -p 3000:80 fn:1.1
docker start bn-container fn-container
docker network rm 네트워크ID
```

<br>

- 같은 설정을 파일 하나로 선언하는 방법이 Compose이다. [Compose 이론](03Compose.md)에서 다룬다.

