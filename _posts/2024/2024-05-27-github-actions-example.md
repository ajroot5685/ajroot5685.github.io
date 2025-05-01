---
title: GitHub Actions로 스프링 CI/CD 구축하기
date: 2024-05-27 15:41:00 +09:00
categories: [CI/CD]
tags:
  [
    CI/CD,
    github actions,
    spring,
    NCP
  ]
---

## CI/CD 구축
[CI/CD의 새로운 패러다임 - Github Actions](https://ajroot5685.github.io/posts/what-is-gihub-actions/)

- 지난 포스팅에서 다루었던 GitHub Actions 개념들을 바탕으로 스프링 프로젝트를 도커 허브에 올리고(`CI`), Naver Cloud Platform의 서버 인스턴스에서 배포하는 과정(`CD`)를 구축해보겠다.
- 아키텍쳐로 나타내면 아래 그림과 같다.

<img src="/assets/img/24/05/27/github%20actions.png" alt="architcture" width=600>


<br>

## workflow 템플릿
- workflow는 처음부터 직접 작성할 수도 있지만, 사용 언어나 프레임워크에 맞는 기본 템플릿을 제공한다.
- CI/CD를 구축하고자 하는 레포지토리의 Actions 탭에서 여러 템플릿을 사용할 수 있다.
- 여기서는 Java with Gradle 템플릿을 기본으로 사용한다.

<img src="/assets/img/24/05/27/workflow template.png" alt="architcture" width=700>

<br>

## 완성 코드
- 완성 코드를 먼저 공개하고, 각 작업들을 단계별로 설명하겠다.

```yaml
name: Java CI with Gradle

# 동작 조건 설정 : main 브랜치에 push가 발생할 경우 동작한다.
on:
  push:
    branches: [ "main" ]

jobs:
  # Spring Boot 애플리케이션을 빌드하여 도커허브에 푸시하는 과정
  build-docker-image:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    # 1. Java 17 세팅
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    # 2. application.yml 설정
    - run: mkdir ./src/main/resources
    - run: echo "${{ secrets.APPLICATION_YML }}" | base64 --decode > ./src/main/resources/application.yml

    # 3-1. Gradle 권한 부여
    - name: Grant permission for gradlew
      run: chmod +x ./gradlew
        
    # 3-2. Spring Boot 애플리케이션 빌드
    - name: Build with Gradle
      run: ./gradlew clean build -x test

    # 4. Docker 이미지 빌드
    - name: docker image build
      run: docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/moonduck .

    # 5. DockerHub 로그인
    - name: docker login
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_PASSWORD }}

    # 6. Docker Hub 이미지 푸시
    - name: docker Hub push
      run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/moonduck

  deploy:
    # build-docker-image (위)과정이 완료되어야 실행됩니다.
    needs: build-docker-image
    runs-on: ubuntu-latest

    steps:
      # NCP 로그인
    - name: NCP login and docker image pull and run
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.NCP_HOST }}
        username: ${{ secrets.NCP_USERNAME }}
        password: ${{ secrets.NCP_PASSWORD }}
        port: ${{ secrets.NCP_PORT }}
        script: |
          echo "${{ secrets.DOCKERHUB_PASSWORD }}" | sudo docker login -u ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin
          
          # 컨테이너가 존재할 때만 컨테이너 제거 작업 수행
          CONTAINERS=$(sudo docker ps -aq)
          if [ -n "$CONTAINERS" ]; then
            sudo docker stop $CONTAINERS
            sudo docker rm -f $CONTAINERS
          fi
          
          sudo docker pull ${{ secrets.DOCKERHUB_USERNAME }}/moonduck
          sudo docker run -d -p 8080:8081 ${{ secrets.DOCKERHUB_USERNAME }}/moonduck
          sudo docker image prune -f
          ps -ef | grep docker
```

<br>

## secrets
- 비밀 키나, 공개하고 싶지 않은 값들은 깃허브에 올리면 안된다.
- 이런 비밀 값들을 사용하기 위해서는 레포지토리의 `Settings - Secrets and variables - actions` 에서 등록해주어야 한다.

<img src="/assets/img/24/05/27/secrets.png" alt="architcture" width=700>

<br>

#### [application.yml]
- 다른 키들과 다르게 `application.yml` 파일을 등록하는 경우, `base64` 인코딩이 필요하다.
- 공백, 줄 바꿈, 콜론 등 다양한 문자들을 포함한 문법이 사용되는데 직접 저장하는 경우 문법이 깨질 수 있기 때문이다.
- 따라서 secrets 값에는 인코딩한 값을 저장하고 사용할 때는 디코딩하여 사용해야 한다.

<br>

## on 설정
- CI/CD를 적용할 브랜치는 `main` 브랜치이고, `push` 작업이 발생할 때 적용시킬 것이다.

```yaml
name: Java CI with Gradle
# name은 workflow의 이름이다.

# 동작 조건 설정 : main 브랜치에 push가 발생할 경우 동작한다.
on:
  push:
    branches: [ "main" ]
```

<br>

## jobs 설정
- 작업은 총 2개이다.
    - **빌드** : 스프링 부트 애플리케이션을 빌드하여 도커 허브에 push하는 작업
    - **배포** : 클라우드 서버에서 도커 허브 이미지를 pull 받아 인스턴스를 띄우는 작업
- 배포 작업은 빌드 과정이 끝난 후에 실행되어야 하므로 `needs` 를 통해 작업의 실행 순서를 지정해준다.

```yaml
jobs:
  # Spring Boot 애플리케이션을 빌드하여 도커허브에 푸시하는 과정
  build-docker-image:
    runs-on: ubuntu-latest
    steps:
        .
        .

  deploy:
    # build-docker-image (위)과정이 완료되어야 실행됩니다.
    needs: build-docker-image
    runs-on: ubuntu-latest

    steps:
        .
        .
```

<br>

## build-docker-image
- 빌드하고, 도커 허브에 push 하는 작업을 단계별로 살펴보겠다.

<br>

#### [체크아웃]
- 현재 레포지토리의 코드를 github actions를 실행하는 인스턴스로 가져오는 작업이다.
- 커스터마이징 할 필요 없이, actions를 이용하면 된다.

```yaml
- uses: actions/checkout@v3
```

<br>

#### [Java 17 세팅]
- 스프링 부트 애플리케이션의 빌드를 위해 JAVA가 필요하다.
- 마찬가지로 actions로 간단하게 짤 수 있고, 추가 인자를 통해 버전 지정이 가능하다.

```yaml
- name: Set up JDK 17
  uses: actions/setup-java@v3
  with:
    java-version: '17'
    distribution: 'temurin'
```

<br>

#### [application.yml 설정]
- 위에서 언급했듯이, `application.yml` 을 secrets으로 넣을 때는 `base64` 로 인코딩을 해주어야 했다.
- `application.yml` 파일이 들어갈 디렉토리를 만들고 `echo` 명령어를 통해 넣는다.
    > `echo` 명령어는 해당 파일이 존재하지 않으면 자동으로 파일을 생성한다.

```yaml
- run: mkdir ./src/main/resources
- run: echo "${{ secrets.APPLICATION_YML }}" | base64 --decode > ./src/main/resources/application.yml
```

<br>

#### [Gradle 권한 부여]
- ubuntu 에서 `./gradlew`을 실행하려면 실행 권한을 부여 해줘야 한다.

```yaml
- name: Grant permission for gradlew
  run: chmod +x ./gradlew
```

<br>

#### [Spring Boot 애플리케이션 빌드]
- 로컬 환경에서 제대로 작동이 되는 것을 확인했다는 전제로, 빌드 과정에서 테스트는 수행하지 않는다.

```yaml
- name: Build with Gradle
  run: ./gradlew clean build -x test
```

<br>

#### [Docker 이미지 빌드]
- 빌드 결과물과 설계한 `Dockerfile` 을 바탕으로 Docker 이미지를 빌드한다.

```yaml
- name: docker image build
  run: docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/moonduck .
```

<br>

#### [DockerHub 로그인]
- 도커 허브에 도커 이미지를 올리기 위해 로그인을 수행한다.
- 도커가 공개한 action을 사용한다.

```yaml
- name: docker login
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_PASSWORD }}
```

<br>

#### [Docker Hub 이미지 푸시]
- 만든 도커 이미지를 push 한다.

```yaml
- name: docker Hub push
  run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/moonduck
```

<br>

## deploy
- 도커 허브에 이미지를 올렸으므로, 클라우드 인스턴스에 접속하여 배포하는 작업을 살펴보겠다.
- 여러 가지 방법이 있지만, 간단하게 ssh 접속을 통해 배포하는 방법을 사용했다.
- ssh는 action으로 간단하게 구성하고, 이어지는 스크립트 작성을 통해 ssh 접속 후 실행할 명령들을 지정하였다.

```yaml
- name: NCP login and docker image pull and run
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets.NCP_HOST }}
    username: ${{ secrets.NCP_USERNAME }}
    password: ${{ secrets.NCP_PASSWORD }}
    port: ${{ secrets.NCP_PORT }}
    script: |
      echo "${{ secrets.DOCKERHUB_PASSWORD }}" | sudo docker login -u ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin
          
      # 컨테이너가 존재할 때만 컨테이너 제거 작업 수행
      CONTAINERS=$(sudo docker ps -aq)
      if [ -n "$CONTAINERS" ]; then
        sudo docker stop $CONTAINERS
        sudo docker rm -f $CONTAINERS
      fi
          
      sudo docker pull ${{ secrets.DOCKERHUB_USERNAME }}/moonduck
      sudo docker run -d -p 8080:8081 ${{ secrets.DOCKERHUB_USERNAME }}/moonduck
      sudo docker image prune -f
      ps -ef | grep docker
```

<br>

## 결과
<img src="/assets/img/24/05/27/github action result.png" alt="result" width=500>

- 이렇게 설계를 완료한 후 `main` 브랜치에 push 작업을 수행할 때마다 정상적으로 CI/CD가 자동화된 것을 볼 수 있다.