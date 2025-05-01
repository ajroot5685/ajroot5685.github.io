---
title: 홈서버 구축 5 - Docker로 서비스 실행하기
date: 2025-04-11 16:22:00 +09:00
categories: [홈서버]
tags:
  [
    홈서버,
    미니pc,
    docker,
    ci/cd
  ]
---

## 들어가며

도커는 격리된 환경을 제공하여 서비스 간 간섭을 줄일 수 있고, 이식성도 매우 좋다.
<br>
어디서든 동일한 환경으로 쉽게 실행시킬 수 있으므로, 만약 홈서버가 다운되더라도 클라우드 서버에서 복구하는 시간이 적어진다.
<br>
이번 포스팅에서는 도커로 서비스를 실행하고, github actions의 셀프 호스팅으로 CI/CD 파이프라인을 구축하는 과정을 소개한다.

<br>

## 도커 설치

먼저 도커와 도커 컴포즈를 설치해야 한다.
<br>
꽤 많은 과정이 필요하기 때문에 한번에 해결 가능한 스크립트를 제공한다.
<br>
스크립트 내용을 복사하고 `./install_docker.sh`로 실행해서 도커와 도커 컴포즈를 설치하자

<https://github.com/ajroot5685/Script/blob/main/docker/install_docker.sh>

<br>

## 도커 컴포즈 설정

단일 도커 파일을 사용하면 이미지 빌드, 컨테이너 생성 및 실행, 볼륨/환경변수/네트워크 등을 모두 수동으로 설정해야 하므로 귀찮다.
<br>
도커 컴포즈를 이용해 위 작업들을 자동화하자

<br>

#### Dockerfile

도커 컨테이너를 실행하기 위해선 이미지가 필요하다.
<br>
`jar` 파일로 빌드된 애플리케이션 파일을 실행하는 도커 파일을 루트에 만들어준다

```dockerfile
FROM openjdk:17

ARG JAR_FILE=build/libs/*.jar

COPY ${JAR_FILE} app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

<br>

#### docker-compose.yml

```yaml
services:
  <앱 이름>:
    image: <도커 유저이름>/<이미지 이름>:latest
    container_name: app
    env_file:
      - .env
    ports:
      - "11000:8080"
```

`image` : 사용할 이미지를 지정한다.
<br>
`container_name` : 명시적으로 컨테이너 명을 지정한다.
<br>
`env_file` : 환경변수가 필요시 지정한다.
<br>
`ports` : 외부에서 요청을 전달하기 위해 호스트의 포트(`11000`)와 컨테이너의 포트(`8080`)를 연결한다.

> 사실 `build`로 도커파일을 지정해야 이미지 빌드까지 자동화되지만, 빌드 단계에서 오류가 나는지를 확인하기 위해 분리한다.
{: .prompt-info }

<br>

#### 테스트

CI/CD 환경에서 테스트하기 전에 로컬에서 파일들이 잘 작동되는지 먼저 검사하자
- 이미지 빌드 : `docker build -t <도커 유저이름>/<이미지 이름>:latest .`
- docker-compose 실행 : `docker-compose up -d`
- 이후 매핑한 호스트 포트(`11000`)로 요청이 잘 가는지 확인한다.

만약 도커 컴포즈 이름을 다르게 설정했다면 `-f` 옵션을 사용해 지정할 수 있다.
- `docker-compose -f docker-compose-app.yml up -d`

<br>

## Nginx 서버 블록 추가

[이전 포스팅](https://ajroot5685.github.io/posts/Home-Server-4/)을 참고해서 서버 블록을 추가하자.

<br>

## github actions 셀프 호스팅

깃허브 액션은 레포지토리를 public으로 설정하면 무료로 깃허브 서버로 파이프라인을 구축할 수 있다.
<br>
이 방식으로 홈서버에 배포하려면 깃허브 서버에서 구동되는 `runners`가 접근할 수 있도록 SSH 접속을 설정해야 한다.
<br>
하지만 나는 홈서버 SSH 보안을 위해 **SSH 접근을 최소화**하고 싶기 때문에 홈서버에서 파이프라인이 실행되는 **셀프 호스팅** 방식을 사용한다.

<br>

#### 셀프 호스팅 준비

[여기](https://danawalab.github.io/common/2022/08/24/Self-Hosted-Runner.html)에 잘 설명이 되어있지만, Linux 기준으로 다시 설명한다.

<img src="/assets/img/25/04/11/self_hosted.png" style="border-radius:5px" alt="self_hosted" width="800">

깃허브 액션을 사용하고자 하는 레포지토리에서 Settings - Actions - Runners로 이동한다.
<br>
`New self-hosted runner`를 누르고 Linux를 선택하면 설치 가이드가 있다.
<br>
원하는 위치에 가이드를 따라 설치하고 `./run.sh`로 실행이 잘되는지 확인한다.
<br>
백그라운드로 실행하기 위해서는 제공된 `./svc.sh`를 사용하면 된다.

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

가이드의 configure 단계를 따라 저장소에 잘 연결되었다면 깃허브의 runners에 보여질 것이다.

<img src="/assets/img/25/04/11/runners.png" style="border-radius:5px" alt="runners" width="800">

<br>

## 액션즈 환경변수 설정

레포지토리가 public이면 워크플로 파일에서 보안이 위험할 수 있다.
<br>
중요한 파일이나 변수들은 레포지토리의 비밀변수로 설정하여 노출되지 않도록 하자.

<img src="/assets/img/25/04/11/secrets.png" style="border-radius:5px" alt="secrets" width="800">

레포지토리의 Settings - Secrets and variables - Actions 에서 비밀 변수를 추가할 수 있다.
<br>
추가/수정/삭제만 되고 그 누구도 읽을 수 없으므로 노출될 염려가 없다.
<br>
이렇게 설정한 변수는 워크플로우 스크립트에서 <code class="language-plaintext highlighter-rouge">$&#123;&#123; secrets.DOCKER_IMAGE_NAME &#125;&#125;</code> 와 같이 설정 가능하다.
<br>
`yaml` 같이 띄어쓰기 형식이 중요할 때에는 base64로 인코딩하여 설정하고, 사용할 때 디코딩하여 사용해야 한다.

> base64 인코딩은 변환시켜주는 아무 웹사이트를 찾아서 사용하면 된다.

<br>

## 워크플로우 스크립트 설정

레포지토리에 `github/workflows/github-actions.yml` 파일을 만든다.
<br>
위에서 말했듯 이미지 빌드 오류가 나는지를 확인하기 위해 이미지 빌드 단계를 분리한다.
<br>
아래 파일을 추가하고 main 브랜치에 push했을 때 파이프라인이 정상 작동되는지 확인한다.

{% raw %}
```yaml
name: Manitto CI/CD 파이프라인

# main 브랜치에 push될 때 실행
on:
  push:
    branches:
      - main

# runners는 시스템 자원을 꽤나 많이 사용한다.
# 동시에 하나의 워크플로만 실행해야 시스템의 부담을 덜 수 있다.
concurrency:
  group: jg-runner-group # 같은 그룹의 워크플로는 동시에 실행되지 않음
  cancel-in-progress: false # 이미 실행중인 이전 워크플로를 취소하지 않고 기다림

jobs:
  build-and-deploy: # 작업 이름
    runs-on: self-hosted # runners의 태그, 여러 태그를 조합해 실행할 러너를 상세 지정 할 수 있다.
    steps:
      - name: Checkout Repository # 현재 레포지토리의 코드를 가져옴
        uses: actions/checkout@v3

      - name: Set up JDK 17 # 애플리케이션을 빌드하기 위해 JDK 17 설치
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Build Spring Boot Application # 테스트를 제외하고 빌드를 수행
        run: |
          chmod +x gradlew
          ./gradlew clean build -x test

      - name: Build Docker Image # 이미지 빌드
        run: |
          docker build --no-cache -t ${{ secrets.DOCKER_IMAGE_NAME }}:latest .

      - name: Add Private Files # 비밀 변수를 디코딩하여 파일에 저장
        run: |
          echo "${{ secrets.APPLICATION_ENV }}" | base64 --decode > .env
          echo "${{ secrets.DOCKER_COMPOSE }}" | base64 --decode > docker-compose.yml

      - name: Run New Container # 컨테이너 실행
        run: |
          docker-compose up -d --build

  clean:
    runs-on: self-hosted
    needs: build-and-deploy
    steps:
      - name: Cleanup Previous Workspace # 기록이 남지 않도록 전체 파일 삭제
        run: rm -rf ./*

      - name: Cleanup Docker and Cache # 도커의 찌꺼기 데이터들 제거
        run: |
          docker system prune -af --volumes
          rm -rf ~/.gradle/caches
```
{% endraw %}

<br>

## 주의 - ufw는 도커 방화벽을 제어할 수 없다

리눅스는 방화벽 시스템으로 `iptables`를 사용한다.
<br>
그리고 `ufw`는 iptables 설정의 복잡성을 간편하게 사용할 수 있게 해주는 도구이다.
<br><br>
이때 ufw는 도커 컨테이너의 방화벽을 제어할 수 없다.
<br>
그 이유는 도커는 iptables를 직접 조작하기 때문이다.
<br>
ufw도 iptables를 조작하지만 도커 방화벽 정책의 우선순위가 더 높다.
<br><br>
도커 컨테이너로 전달되는 요청은 내부 도커 네트워크를 통해 홈서버를 거쳐가는 것이기 때문에 `FORWARD` 체인을 사용한다.
<br>
여기서 설정되는 도커의 우선순위가 ufw 우선순위보다 높기 때문에 ufw로 도커 컨테이너 방화벽을 제어할 수 없는 것이다.

<img src="/assets/img/25/04/11/iptables.png" style="border-radius:5px" alt="iptables" width="600">

<br>

#### 그렇다면 어떻게 방화벽을 제어하는가?

도커 컨테이너는 기본적으로 컨테이너의 네트워크를 개방하기 때문에 보안상 안전하지 않은 상태가 된다.
<br>
이에 대한 2가지 대응 방법이 있다.

1. 컨테이너는 개방된 상태로 두고, 포트포워딩과 리버스 프록시를 이용해 외부 노출을 제한한다.
- 홈서버는 공유기를 통해 통신하므로 포트포워딩 설정이 없다면, 외부 접근은 불가능하다.
- 외부 요청은 Nginx와 같은 리버스 프록시를 통해 받고, Nginx가 받은 요청을 내부 컨테이너로 전달시킨다.
- 이때 Nginx가 사용하는 포트만 `ufw`로 관리하면, 컨테이너에 대한 외부 접근은 막으면서도 방화벽 제어가 가능하다.
2. iptables를 직접 조작한다.
- 이미지에 나와있듯이 커스텀 체인을 만들어서 컨테이너로 전달되기 전 트래픽을 제어할 수 있다.
- 꽤 난이도가 있는 작업이므로, 특수한 목적이 있을 때에만 권장한다.