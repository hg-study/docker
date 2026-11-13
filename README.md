## Docker

### 목차

- [Docker](#docker)
- [Dockerfile](#dockerfile)
- [Compose](#compose)
- [Docker Hub](#docker-hub)
- [Deploy](#deploy)
- [EC2 Jenkins](#ec2-jenkins)

<br>

---

### Docker
- [Docker 이론](./Theory/01Docker.md)
    - [첫 Dockerfile](./D00Init/01/Dockerfile)

### Dockerfile
- [Dockerfile 이론](./Theory/02Dockerfile.md)
    - [Node Dockerfile](./D00Init/01/Dockerfile)
    - [서버 Dockerfile](./D01Images/BN/Dockerfile)
    - [화면 Dockerfile](./D01Images/FN/Dockerfile)
    - [서버 Dockerfile(컨테이너 단계)](./D02Container/BN/Dockerfile)
    - [DB Dockerfile](./D02Container/DB/Dockerfile)
    - [Redis Dockerfile](./D02Container/REDIS/Dockerfile)
    - [화면 Dockerfile(컨테이너 단계)](./D02Container/FN/Dockerfile)

### Compose
- [Compose 이론](./Theory/03Compose.md)
    - [compose 파일](./D03Compose/docker-compose.yml)
    - [서버 Dockerfile](./D03Compose/BN/Dockerfile)
    - [화면 Dockerfile](./D03Compose/FN/Dockerfile)

### Docker Hub
- [Docker Hub 이론](./Theory/04DockerHub.md)
    - [Docker Hub compose](./D05DockerHub/docker-compose.yml)

### Deploy
- [Deploy 이론](./Theory/05Deploy.md)
    - [배포용 compose](./D04ComposeDeploy/docker-compose.yml)
    - [로컬용 compose](./D04ComposeDeploy/docker-compose-local.yml)
    - [서버 Dockerfile](./D04ComposeDeploy/BN/Dockerfile)
    - [DB Dockerfile](./D04ComposeDeploy/DB/Dockerfile)
    - [Redis Dockerfile](./D04ComposeDeploy/REDIS/Dockerfile)
    - [Nginx 설정](./D04ComposeDeploy/FN/nginx.conf)
    - [화면 Dockerfile](./D04ComposeDeploy/FN/Dockerfile)
    - [Jenkins Dockerfile](./D04ComposeDeploy/JENKINS/Dockerfile)
    - [단독 배포 안내](./D06SpringBootOnly/README.md)
    - [build.gradle](./D06SpringBootOnly/build.gradle)

### EC2 Jenkins
- [EC2 Jenkins 이론](./Theory/06Ec2Jenkins.md)
    - [Spring Boot 단독 배포](./D06SpringBootOnly/build.gradle)
    - [Jenkins Dockerfile](./D04ComposeDeploy/JENKINS/Dockerfile)
