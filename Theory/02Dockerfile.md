# Dockerfile

- [Dockerfile 지시어](#dockerfile-지시어)
- [Node 앱](#node-앱)
- [Spring Boot 앱 다단계 빌드](#spring-boot-앱-다단계-빌드)
- [React 앱 다단계 빌드](#react-앱-다단계-빌드)
- [DB와 Redis](#db와-redis)
- [레이어 캐시](#레이어-캐시)

<br>

---

### Dockerfile 지시어

| 지시어 | 설명 |
| --- | --- |
| `FROM` | 기반이 될 이미지 |
| `WORKDIR` | 이후 명령을 실행할 작업 디렉터리 |
| `COPY` | 호스트의 파일을 이미지로 복사 |
| `RUN` | 이미지를 만드는 동안 실행할 명령 (설치, 빌드) |
| `ENV` | 환경 변수 |
| `EXPOSE` | 컨테이너가 사용하는 포트 표시(실제 공개는 `-p`) |
| `CMD` | 컨테이너가 시작될 때 실행할 기본 명령 |

---

### Node 앱

> `package*.json`을 먼저 복사해 `npm install`을 하고 그 다음 소스 전체를 복사한다. 소스가 바뀌어도 의존성 설치 단계는 캐시를 재사용하므로 빌드가 빠르다.

<br>

- 예제 코드: [Node Dockerfile](../D00Init/01/Dockerfile)

---

### Spring Boot 앱 다단계 빌드

> 다단계 빌드(multi-stage)는 빌드용 이미지와 실행용 이미지를 나눈다. 1단계에서 JDK와 Gradle로 `jar`를 만들고, 2단계에서 가벼운 JDK 이미지에 완성된 `jar`만 복사한다. 최종 이미지에 빌드 도구와 소스가 들어가지 않아 작고 안전하다. `COPY --from=build`로 앞 단계의 결과물을 가져온다.

<br>

```dockerfile
FROM gradle:8.14.1-jdk21 AS build
WORKDIR /app
COPY . .
RUN gradle build --no-daemon -x test

FROM eclipse-temurin:21-jdk-alpine
WORKDIR /app
COPY --from=build /app/build/libs/*.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

<br>

- 예제 코드: [서버 Dockerfile](../D01Images/BN/Dockerfile)

---

### React 앱 다단계 빌드

> 1단계에서 Node로 `npm run build`를 해 정적 파일(`build`)을 만들고, 2단계에서 Nginx 이미지의 웹 루트(`/usr/share/nginx/html`)에 복사한다. 실행 이미지에는 Node가 필요 없고 Nginx만 있으면 된다. 기본 Nginx 설정을 바꾸려면 `nginx.conf`를 복사해 덮어쓴다.

<br>

- 예제 코드: [화면 Dockerfile](../D01Images/FN/Dockerfile)

---

### DB와 Redis

> MySQL과 Redis는 공식 이미지(`mysql:8.0`, `redis`)를 기반으로 환경 변수나 설정만 덧붙인다. MySQL은 `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD` 환경 변수로 초기 계정을 만든다. 비밀번호를 Dockerfile이나 저장소에 적으면 노출되므로 이 저장소에는 `YOUR_DB_PASSWORD` 자리 표시자만 두었고, 실제 값은 실행할 때 환경 변수나 별도 파일로 넣는다.

<br>

- 예제 코드: [서버 Dockerfile(컨테이너 단계)](../D02Container/BN/Dockerfile), [DB Dockerfile](../D02Container/DB/Dockerfile), [Redis Dockerfile](../D02Container/REDIS/Dockerfile), [화면 Dockerfile(컨테이너 단계)](../D02Container/FN/Dockerfile)

---

### 레이어 캐시

> Dockerfile의 각 줄은 이미지의 층(layer)이 되고, 앞의 층이 바뀌지 않으면 캐시를 재사용한다. 자주 바뀌는 소스 복사는 아래에, 잘 바뀌지 않는 의존성 설치는 위에 두면 빌드가 빨라진다. `.dockerignore`로 `node_modules`, `build` 같은 불필요한 파일은 이미지에 넣지 않는다.
