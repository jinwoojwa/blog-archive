---
title: Docker 정리
---

## 1. Docker란?

- 도커(Docker)는 애플리케이션을 **컨테이너(Container)** 라는 격리된 환경에서 실행, 배포, 관리할 수 있게 해주는 플랫폼
- "내 컴퓨터에서는 잘 되는데, 다른 컴퓨터에서는 왜 안 되지?" 와 같은 환경 의존성 문제를 해결하고, 개발부터 배포까지 일관된 환경을 제공.

### 주요 특징

- **이식성 (Portability)**:
  - 도커 컨테이너는 어디서든 동일하게 동작
  - 로컬 PC, 테스트 서버, 클라우드(AWS, GCP, Azure) 등 환경에 구애받지 않고 애플리케이션을 실행할 수 있음.
- **격리성 (Isolation)**:
  - 각 컨테이너는 독립된 파일 시스템, 네트워크, 프로세스 공간을 가짐.
  - 이를 통해 프로그램 간 충돌을 방지하고 안정성을 높일 수 있음.
- **일관성 (Consistency)**:
  - 개발, 테스트, 운영 환경을 이미지(Image)를 통해 통일하여 환경 차이로 인한 문제를 원천적으로 차단.

## 2. 핵심 개념

### 이미지 (Image)

- 애플리케이션 실행에 필요한 **모든 것(코드, 런타임, 라이브러리, 환경 변수, 설정 파일 등)을 담고 있는 읽기 전용 템플릿**.
- '닌텐도 게임 칩'이나 'OS 설치 CD'와 같이, 이미지만 있으면 어디서든 동일한 환경의 컨테이너를 생성할 수 있음.

### 컨테이너 (Container)

- **이미지를 실행한 인스턴스**. 이미지가 클래스(Class)라면, 컨테이너는 그 클래스로부터 생성된 객체(Object)에 해당.
- 각 컨테이너는 격리된 공간에서 독립적으로 실행되며, 생성, 시작, 중지, 삭제될 수 있음.

## 3. 클라우드와 Docker

- 클라우드 환경(IaaS, PaaS)에서 애플리케이션을 배포할 때, 도커는 특정 클라우드 서비스에 대한 종속성을 줄여줌.
- 도커화된 애플리케이션은 가상 머신(IaaS) 위에서 독립적으로 동작하면서도, PaaS와 같은 효율적인 운영을 가능하게 함.
- 결론적으로, 도커를 사용하면 어떤 클라우드 환경에서든 **낮은 운영 비용과 높은 이식성**을 확보할 수 있음.

## 4. 주요 명령어

### 이미지 관련 명령어

#### 이미지 다운로드

```bash
# docker pull [이미지명]:[태그명]
$ docker pull nginx:latest
```

#### 이미지 목록 조회

```bash
# docker images
$ docker images
```

#### 이미지 삭제

```bash
# docker rmi [이미지 ID 또는 이미지명]
# docker image rm [이미지 ID 또는 이미지명]
# rmi = remove image
$ docker rmi nginx:latest
$ docker image rm nginx:latest
```

#### 모든 이미지 삭제

```bash
# docker rmi -f $(docker images -q)
$ docker rmi -f $(docker images -q)
```

### 컨테이너 생명주기 관련 명령어

#### 컨테이너 생성 및 실행

```bash
# docker run [옵션] [이미지명]:[태그명]
$ docker run -d -p 8080:80 --name my-web-server nginx
```

> **주요 옵션**
>
> - `-d`: 백그라운드(detached mode)에서 실행
> - `-p [호스트 포트]:[컨테이너 포트]`: 포트 포워딩
> - `--name [컨테이너명]`: 컨테이너에 이름 부여

#### 실행 중인 컨테이너 목록 조회

```bash
# docker ps
$ docker ps
```

#### 모든 컨테이너 목록 조회 (중지된 컨테이너 포함)

```bash
# docker ps -a
$ docker ps -a
```

#### 컨테이너 중지

```bash
# docker stop [컨테이너 ID 또는 이름]
$ docker stop my-web-server
```

#### 컨테이너 시작

```bash
# docker start [컨테이너 ID 또는 이름]
$ docker start my-web-server
```

#### 컨테이너 삭제

```bash
# docker rm [컨테이너 ID 또는 이름]
$ docker rm my-web-server
```

#### 모든 컨테이너 삭제

```bash
# docker rm -f $(docker ps -qa)
$ docker rm -f $(docker ps -qa)
```

### 컨테이너 모니터링 및 관리 명령어

#### 컨테이너 로그 조회

```bash
# docker logs [옵션] [컨테이너 ID 또는 이름]
$ docker logs -f --tail 10 my-web-server
```

> **주요 옵션**
>
> - `-f`: 실시간 로그 출력 (follow)
> - `--tail [줄 수]`: 마지막 N줄만 출력

#### 컨테이너 내부 접속 (명령어 실행)

```bash
# docker exec -it [컨테이너 ID 또는 이름] [명령어]
$ docker exec -it my-web-server bash
```

> `-it` 옵션: 상호작용 가능한 터미널(tty)로 접속
>
> - `-i (interactive):` 표준 입력을 열어두고 키보드 입력을 계속 받을 수 있게 함
> - `-t (tty):` 터미널 환경을 붙여서 bash처럼 프롬프트가 보이도록 함

## 5. 데이터 영속성: 볼륨 (Volume)

- 컨테이너는 삭제되면 내부의 데이터도 함께 사라짐.
- 데이터베이스 데이터나 중요한 로그처럼 영구적으로 보존해야 할 데이터는 **볼륨(Volume)** 을 사용해 컨테이너 외부(호스트)에 저장해야 함.
- 볼륨은 호스트의 특정 디렉터리를 컨테이너의 디렉터리와 연결(마운트)하여 데이터를 공유하는 기능

### 사용법

`-v` 또는 `--volume` 옵션을 사용함.

```bash
# docker run -v [호스트 디렉터리]:[컨테이너 디렉터리] [이미지명]
$ docker run -d -p 3306:3306 -v /my/data:/var/lib/mysql --name my-db mysql:8.0
```

- 위 예시에서 컨테이너의 `/var/lib/mysql` 디렉터리에 저장되는 데이터는 실제로는 호스트의 `/my/data` 디렉터리에 저장됨.
- 따라서 `my-db` 컨테이너를 삭제하고 새로 생성해도 데이터는 그대로 유지됨.

## 6. 이미지 생성: Dockerfile

- `Dockerfile`은 나만의 도커 이미지를 만들기 위한 **설계도(레시피)** 파일
- 이 파일에 명시된 명령어들을 통해 베이스 이미지에 원하는 애플리케이션과 설정을 추가할 수 있음.

### 주요 Dockerfile 명령어

- **`FROM`**: 베이스가 될 이미지를 지정 (예: `FROM openjdk:17-jdk-slim`)
- **`COPY`**: 호스트의 파일이나 디렉터리를 이미지 내부로 복사 (예: `COPY ./build/libs/app.jar /app.jar`)
- **`RUN`**: 이미지 빌드 과정에서 실행할 셸 명령어를 정의 (예: `RUN apt-get update && apt-get install -y vim`)
- **`CMD`**: 컨테이너가 시작될 때 실행할 기본 명령어를 설정 (예: `CMD ["java", "-jar", "/app.jar"]`)
- **`WORKDIR`**: 명령어들이 실행될 기본 디렉터리를 설정
- **`EXPOSE`**: 컨테이너가 외부에 노출할 포트를 지정

### 이미지 빌드

`Dockerfile`이 있는 경로에서 아래 명령어를 실행하여 이미지를 빌드함.

```bash
# docker build -t [이미지명]:[태그명] [Dockerfile 경로]
$ docker build -t my-app:1.0 .
```

> **`.dockerignore`**
>
> `.gitignore`와 유사하게, 이미지 빌드 시 포함하지 않을 파일이나 디렉터리를 지정하여 불필요한 파일이 이미지에 포함되는 것을 막고 빌드 속도를 향상시킬 수 있다.
