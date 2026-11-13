# Compose

- [Compose란](#compose란)
- [서비스 정의](#서비스-정의)
- [네트워크](#네트워크)
- [환경 변수와 볼륨](#환경-변수와-볼륨)
- [healthcheck와 depends_on](#healthcheck와-depends_on)
- [실행 명령](#실행-명령)

<br>

---

### Compose란

> 여러 컨테이너(화면, 서버, DB, Redis)를 `docker run`으로 하나씩 띄우면 명령이 길고 순서를 지켜야 한다. Docker Compose는 `docker-compose.yml` 한 파일에 서비스들을 정의하고 한 번에 만들고, 시작하고, 지우게 한다.

---

### 서비스 정의

> `services` 아래에 서비스마다 `build`(Dockerfile 위치) 또는 `image`(받을 이미지), `container_name`, `ports`를 적는다. 서비스 이름(`db`, `redis`, `bn`, `fn`)은 같은 네트워크 안에서 호스트 이름처럼 쓸 수 있다.

<br>

- 예제 코드: [compose 파일](../D03Compose/docker-compose.yml), [서버 Dockerfile](../D03Compose/BN/Dockerfile), [화면 Dockerfile](../D03Compose/FN/Dockerfile)

---

### 네트워크

> 같은 Compose 파일의 서비스는 기본적으로 같은 네트워크에 들어가 서비스 이름이나 컨테이너 이름으로 서로를 찾는다. 이 예제는 `custom-network`를 만들어 대역(`172.30.0.0/24`)과 서비스별 고정 IP를 지정했다. 서버는 DB 주소를 `localhost`가 아니라 컨테이너 이름(`db-container:3306`)으로 쓴다. 컨테이너 안에서 `localhost`는 그 컨테이너 자신이기 때문이다.

---

### 환경 변수와 볼륨

> `environment`로 컨테이너에 환경 변수를 넘긴다. Spring Boot는 `SPRING_DATASOURCE_URL`처럼 대문자와 밑줄로 쓴 환경 변수로 `spring.datasource.url` 설정을 덮어쓴다. 코드나 이미지에 접속 정보를 박지 않고 실행 환경에서 주입하는 방식이다. `volumes`는 컨테이너가 지워져도 남길 데이터의 저장 위치이다. `db-data:/var/lib/mysql`처럼 이름 있는 볼륨을 DB 데이터 폴더에 연결하면 컨테이너를 다시 만들어도 데이터가 유지된다.

---

### healthcheck와 depends_on

> `healthcheck`는 컨테이너가 정상인지 주기적으로 확인하는 명령이다(MySQL은 `mysqladmin ping`, 서버는 `/actuator/health`). `depends_on`에 `condition: service_healthy`를 주면 의존하는 서비스가 정상일 때까지 기다렸다가 시작한다. DB가 준비되기 전에 서버가 접속을 시도해 실패하는 일을 막는다.

---

### 실행 명령

```bash
docker compose up -d --build     # 이미지를 빌드하고 모든 서비스를 백그라운드로 실행
docker compose ps                # 서비스 상태
docker compose logs -f bn        # 서비스 로그
docker compose down              # 컨테이너와 네트워크 삭제 (-v 를 붙이면 볼륨도 삭제)
```
