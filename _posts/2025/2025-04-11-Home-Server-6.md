---
title: 홈서버 구축 6 - blue&green 무중단 배포 설정하기
date: 2025-04-11 18:22:00 +09:00
categories: [홈서버]
tags:
  [
    홈서버,
    미니pc,
    nginx,
    무중단 배포
  ]
---

## 들어가며

배포를 진행하면 잠깐의 다운타임이 발생한다.
<br>
사용자는 잠깐의 다운타임에도 불쾌하고 불편함을 느끼기에 무중단으로 배포되도록 설계가 필요하다.
<br>
이번 포스팅에서는 nginx, docker를 사용하는 환경에서 무중단 배포를 구성하는 방법을 소개하겠다.

<br>

## blue&green 무중단 배포

무중단 배포에는 여러 방식이 있지만 가장 간단하고 조금의 다운타임도 없는 방식은 blue&green이다.
<br>
기존 컨테이너를 유지한 채로 새로운 컨테이너를 띄우고 정상 연결이 확인되면 트래픽을 새로운 컨테이너로 연결한 뒤 기존 컨테이너를 종료한다.
<br>
일시적으로 컨테이너가 2배가 되기 때문에 리소스 분배에 유의해야 한다.

<br>

## 흐름

간단히 흐름을 설명하면 다음과 같다.
<br><br>
예시 : green 컨테이너가 띄워져 있는 상황

<img src="/assets/img/25/04/11/blue_green1.png" style="border-radius:5px" alt="blue_green1" width="300">

1. 새로운 파이프라인이 실행된다.
2. 다른 포트로 다른 버전의 컨테이너가 실행된다

    <img src="/assets/img/25/04/11/blue_green2.png" style="border-radius:5px" alt="blue_green2" width="300">
3. nginx는 새로운 컨테이너에 연결한다.

    <img src="/assets/img/25/04/11/blue_green3.png" style="border-radius:5px" alt="blue_green3" width="300">
4. 기존 컨테이너는 제거된다.

    <img src="/assets/img/25/04/11/blue_green4.png" style="border-radius:5px" alt="blue_green4" width="300">

<br>

## deploy.sh

간단한 편에 속하지만, 배포 자체의 복잡성은 올라갈 수 밖에 없다.
<br>
스크립트로 배포 로직을 묶어서 간편하게 관리하자.

<https://github.com/ajroot5685/Script/blob/main/docker/deploy_with_blue_green_strategy.sh>

위 링크에서 예시 스크립트를 사용할 수 있다.
<br>
`APP_NAME`을 원하는 컨테이너명으로 바꿔서 사용하기 바란다.
<br><br>
컨테이너가 실행된다고 바로 요청을 처리할 수 있는 상태는 아니다.
<br>
따라서 애플리케이션에 헬스 체크하는 API를 추가하여 준비된 상태인지를 체크하는 스크립트가 포함되었다.
- `curl -s "http://localhost:$PORT/health"`가 그것으로, `/health` 엔드포인트에 단순히 OK 문자열을 반환하도록 추가하면 되겠다.

주석으로 필요한 사전 작업을 요구하고 있는데, 하나씩 설명하겠다

<br>

#### deploy.sh 위치

runners가 `deploy.sh`를 제대로 인식하고 실행할 수 있도록 올바른 위치에 넣어주어야 한다.
<br>
runners가 설치된 디렉토리의 `_work` 폴더의 워크스페이스 폴더 안에 넣자

```
/home/actions-runner/_work/
└── my-repo/            # 1차: 워크스페이스 폴더 (레포 단위)
    ├── deploy.sh       # 배포 스크립트
    └── my-repo/        # 2차: 실제 체크아웃된 Git 레포 (실제 작업 디렉토리)
        └── ...         # .git, 소스 코드 등
```

<br>

## docker-compose.yml 정의

**관련 스크립트 코드**

```shell
SERVICE_NAME="${APP_NAME}-${COLOR}"
OLD_SERVICE_NAME="${APP_NAME}-${OLD_COLOR}"

# 주의 : docker-compose.yml 파일과 호환되어야함
docker-compose up -d --build "$SERVICE_NAME"
```

도커 컴포즈 파일 또한 blue&green에 맞게 설정해주어야 한다.
<br>
컨테이너명과 포트를 제외한 구성이 똑같은 앱을 정의한다.

```yaml
services:
  app-blue:
    image: ajroot/app:latest
    container_name: app-blue
    env_file:
      - .env
    ports:
      - "11000:8080"
  app-green:
    image: ajroot/app:latest
    container_name: app-green
    env_file:
      - .env
    ports:
      - "11001:8080"
```

<br>

## nginx.conf에 들어갈 블록 정의

[홈서버 구축 4 - Nginx로 트래픽 분배하기](https://ajroot5685.github.io/posts/Home-Server-4/)

위 포스팅을 보고 해당 서비스의 server블록을 정의하길 바란다.
<br>
server 블록 명은 `APP_NAME`으로 통일해야 한다.

<br>

## 블록의 템플릿 정의

**관련 스크립트 코드**

```shell
export PORT
envsubst '$PORT' < /etc/nginx/template/${APP_NAME} > /etc/nginx/sites-available/${APP_NAME}
```

마찬가지로 nginx에 들어갈 블록도 정의해주어야 한다.
<br>
`/etc/nginx/template`이라는 새 디렉토리를 만들고 앱의 템플릿 블록을 만들자
<br>
스크립트에는 `APP_NAME`으로 템플릿 파일을 조회하므로 `APP_NAME`과 동일한 파일로 생성하자
<br>

```conf
# 가독성을 위해 ssl 설정은 생략했다.
server {
    listen 10000;
    server_name domain.mooo.com;

    root /var/www/app;
    index index.html;

    location / {
        proxy_pass http://localhost:$PORT; # $PORT가 스크립트에 정의된 변수로 대치된다.
    }

    include /etc/nginx/global_security.conf;
}
```

<br>

## 변경될 nginx 파일 경로의 소유자 변경

스크립트에서는 `sites-available`에 새로운 파일을 생성/덮어쓰기한다.
<br>
이를 위해서는 현재 파이프라인 실행 주체인 runners가 소유 권한이 있어야 한다.
<br>
따라서 아래 명령어를 통해 소유자를 변경해주자

```bash
sudo chown -R <유저명> /etc/nginx/sites-available
```

<br>

## runners의 sudo 권한 설정

**관련 스크립트 코드**

```shell
if ! sudo nginx -t; then
  echo "❌ nginx 설정 오류"
  docker-compose stop "$SERVICE_NAME"
  exit 1
fi

sudo nginx -s reload
```

`sudo` 명령어는 비밀번호가 필요한데 이를 무시하고 작업할 수 있도록 설정이 필요하다.
<br>
`sudo visudo`로 설정 파일에 진입한 후 맨아래에 다음 문장을 추가한다.

```
<runners실행한 유저명> ALL=(ALL) NOPASSWD: /usr/sbin/nginx
```

<br>

## 파이프라인 스크립트 수정

이전 포스팅에서 사용했던 workflows 스크립트에서 도커 컴포즈 실행 대신 짜여진 `deploy.sh`를 실행하도록 수정하면 된다.

{% raw %}
```yaml
name: Manitto CI/CD 파이프라인

on:
  push:
    branches:
      - main

concurrency:
  group: jg-runner-group
  cancel-in-progress: false

jobs:
  build-and-deploy:
    runs-on: self-hosted
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Build Spring Boot Application
        run: |
          chmod +x gradlew
          ./gradlew clean build -x test

      - name: Build Docker Image
        run: |
          docker build --no-cache -t ${{ secrets.DOCKER_IMAGE_NAME }}:latest .

      - name: Add Private Files
        run: |
          echo "${{ secrets.APPLICATION_ENV }}" | base64 --decode > .env
          echo "${{ secrets.DOCKER_COMPOSE }}" | base64 --decode > docker-compose.yml

      - name: Run Deployment Script # 도커 컴포즈 실행 대신 deploy.sh를 실행한다.
        run: ../deploy.sh # 경로에 유의한다.

  clean:
    runs-on: self-hosted
    needs: build-and-deploy
    steps:
      - name: Cleanup Previous Workspace
        run: rm -rf ./*

      - name: Cleanup Docker and Cache
        run: |
          docker system prune -af --volumes
          rm -rf ~/.gradle/caches
```
{% endraw %}

<br>

## 결과

잘 설정했다면 아래처럼 애플리케이션이 API를 처리할 수 있을 때까지 헬스체크를 시도한 후 새로운 컨테이너로 재연결 될 것이다.

<img src="/assets/img/25/04/11/success.png" style="border-radius:5px" alt="success" width="800">
