# Docker Hub

- [Docker Hub](#docker-hub)
- [이미지 올리기와 받기](#이미지-올리기와-받기)
- [이미지를 받아 쓰는 compose](#이미지를-받아-쓰는-compose)

<br>

---

### Docker Hub

> Docker Hub는 이미지를 저장하고 공유하는 레지스트리이다. 내 컴퓨터에서 만든 이미지를 올려 두면 다른 서버에서 소스나 빌드 도구 없이 이미지를 받아 바로 실행할 수 있다. 이미지 이름은 `계정/저장소:태그` 형식이다.

---

### 이미지 올리기와 받기

```bash
docker login                                         # 로그인
docker tag bn 계정/저장소:bn                           # 올릴 이름으로 태그 붙이기
docker push 계정/저장소:bn                             # 올리기
docker pull 계정/저장소:bn                             # 받기
```

<br>

<span style="color:brown">레지스트리에 올린 이미지에는 소스와 설정이 들어 있을 수 있으므로 비밀번호와 키는 이미지 안에 넣지 않고 실행할 때 환경 변수로 넣는다. 공개 저장소에는 올리지 않는다.</span>

---

### 이미지를 받아 쓰는 compose

> `build` 대신 `image: 계정/저장소:db`처럼 레지스트리의 이미지를 적으면 `docker compose up`이 이미지를 내려받아 실행한다. 서버에는 소스 없이 compose 파일만 두면 된다.

<br>

- 예제 코드: [Docker Hub compose](../D05DockerHub/docker-compose.yml)
