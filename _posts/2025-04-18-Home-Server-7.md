---
title: 홈서버 구축 7 - MongoDB 안전하게 연결하기
date: 2025-04-18 22:37:00 +09:00
categories: [홈서버]
tags:
  [
    홈서버,
    미니pc,
    nginx,
    mongoDB
  ]
---

## 들어가며

DB를 관리하기 위해 pgAdmin, MongoDBCompass, DataGrip 등 다양한 도구들을 사용한다.
<br>
편하게 관리하기 위해서는 외부에 DB 엔드포인트가 공개되어야 하는데, 보안을 위해 여러 설정들이 필요하다.
<br>
이번 포스팅에서는 MongoDB 컨테이너를 띄우고 안전하게 연결할 수 있는 환경을 만들어보겠다.

<br>

## 초기 파일

MongoDB 또한 이식성을 위해 컨테이너로 관리한다.
<br>
이때, 애플리케이션이 도커 네트워크를 통해 DB에 연결할 수 있도록 애플리케이션과 같은 네트워크로 묶는 것을 추천한다.

<br>

#### docker-compose.yml

```yaml
services:
  mongodb:
    image: mongo:latest
    container_name: mongodb
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: <관리자계정>
      MONGO_INITDB_ROOT_PASSWORD: <비밀번호>
    entrypoint: [ "/bin/bash", "-c", "
          mkdir -p /var/log/mongodb &&
          touch /var/log/mongodb/access.log &&
          chown -R mongodb:mongodb /data/mongodb /var/log/mongodb &&
          exec docker-entrypoint.sh mongod --config /etc/mongo/mongod.conf
        " ]
    volumes:
      - /etc/mongo/mongod.conf:/etc/mongo/mongod.conf # 설정 파일 마운트
      - /data/mongodb:/data/mongodb # 데이터 마운트
      - /var/log/mongodb:/var/log/mongodb # 로그 마운트
    networks:
      - <네트워크 이름>

networks:
  <네트워크 이름>:
    external: true
```

**설정 목록**
- 관리자계정
- 비밀번호
- mongod.conf
- 마운트할 호스트 디렉토리 미리 생성
- 도커 네트워크 설정

<br>

#### mongod.conf

```conf
systemLog: # 접근 로그만 저장
  destination: file
  path: /var/log/mongodb/access.log  # 접근 로그 전용 파일
  logAppend: true
  verbosity: 0  # 일반 로그 최소화
  component:
    accessControl: { verbosity: 1 }  # 인증/접근 로그 활성화

setParameter:
  logLevel: 0
  
security:
  authorization: enabled

storage:
  dbPath: /data/mongodb
```

**security.authorization**
- 활성화하지 않으면 누구나 DB에 접속 가능하게 된다.
- 사용자 인증과 권한을 관리하기 위해 활성화 한다.

<br>

## DB 툴 연결 아키텍처

<img src="/assets/img/250418/connect.png" style="border-radius:5px" alt="connect" width="800">

TLS 인증서로 인증된 클라이언트인지 확인하지만, 실제 연결은 Nginx의 `SSL Termination`으로 일반 TCP로 연결된다.

> 이미지에는 생략되었지만, TLS는 기본적으로 클라이언트가 서버의 인증서를 검증한다.

<br>

## Nginx 설정

TLS 설정을 하기 전에 포트포워딩, Nginx 설정부터 해줘야 한다.
<br>
[Nginx로 트래픽 분배하기](https://ajroot5685.github.io/posts/Home-Server-4/)에서 `Debian-style Nginx Config` 스타일에 따라 스트림 블록을 설정해줄 것이다.

<br>

#### nginx-full 설치

mongoDB는 HTTP같은 프로토콜이 아니라 고유한 **mongo 프로토콜**을 사용하므로 TCP로 연결해야 한다.
<br>
기본 nginx가 아닌 `nginx-full`로 설치해야 4계층 로드밸런서 역할을 수행하는 `stream` 블록을 사용할 수 있다.

```bash
sudo apt update
sudo apt install nginx-full -y
```

<br>

#### 스트림 설정

TLS 설정은 뒤에서 추가한다
<br>
지금은 nginx를 통해 DB에 연결할 수 있는지만 확인하자

```nginx
# sudo vim /etc/nginx/streams-available/mongo

upstream mongodb {
    server localhost:<mongo 포트>;
}

server {
    listen 20000;
    proxy_pass mongodb;
}
```

<br>

#### 설정 적용

설정을 적용하고 일반 TCP 연결이 잘 되는지 확인한다.

```bash
sudo ln -s /etc/nginx/streams-available/mongo /etc/nginx/streams-enabled/mongo # 상대 경로가 아닌 절대 경로를 설정해야함

sudo nginx -t # 설정 테스트, 오류 있을 시 고친 후 진행

sudo systemctl reload nginx # 설정 반영, 메모리로 올림

sudo ufw allow 20000 # 해당 사이트에 대한 방화벽 허용
```

<br>

## mTLS

클라이언트가 서버의 인증서를 검증하기만 하는 `1-way TLS`에서 서버가 클라이언트의 인증서를 검증하는 작업이 추가되면 `mutual TLS`이다.
<br>
서버가 아무에게나 개방하지 않고, 인증된 인증서를 가지고 있는 클라이언트만 허용하여 보안이 향상시킬 수 있다.

<br>

## 클라이언트 인증서 생성

```bash
# 클라이언트 인증서 검증을 위한 CA 생성
sudo openssl genpkey -algorithm RSA -out mongoCA.key # CA용 개인 키
sudo openssl req -new -x509 -key mongoCA.key -days 3650 -out mongoCA.crt # CA 자체 인증서
sudo cat mongoCA.key mongoCA.crt | sudo tee mongoCA.pem > /dev/null # mongoDB에 지정할 

# 클라이언트 인증서 생성 및 CA 서명
sudo openssl genpkey -algorithm RSA -out client.key # 클라이언트용 개인 키
sudo openssl req -new -key client.key -out client.csr # 인증서 발급 요청서
sudo openssl x509 -req -in client.csr -CA mongoCA.crt -CAkey mongoCA.key -CAcreateserial -days 1825 -out client.crt # mongoCA.crt로 인증
sudo cat client.key client.crt | sudo tee client.pem > /dev/null # 클라이언트 인증서와 개인 키를 하나의 PEM파일로 합치기
```

위 명령어를 통해 클라이언트가 연결에 사용할 `pem` 파일을 연결이 필요한 사람에게 전달하자
> 당연하지만, 노출이 되면 안 된다.

바로 아래에서 `pem`으로 어떻게 TLS 통신이 이루어지는지 설명한다.

<br>

## 서명

TLS 프로토콜에서 어떻게 서로를 신뢰할까?
<br>
그것은 서명 메커니즘 덕분이다.
<br>
여기서는 외부 인증기관을 이용하지 않고, 자체 서명을 통해 클라이언트의 신뢰성을 검증한다.
<br>
서명을 이해한 사람이라도 쉽지 않은 내용이니 천천히 살펴보자

<br>

#### 외부 CA(인증 기관)를 이용한 서명

먼저 대표적인 CA인 `Let's encrypt`를 이용한 서명 과정을 살펴보겠다.

<br>

<img src="/assets/img/250418/lets.png" style="border-radius:5px" alt="lets" width="400">

1. 자신의 비밀 키를 사용해서 서명이 필요한 인증서 파일을 만든다.
2. 외부 CA가 서명을 함으로써 정상적인 인증서가 된다. 이때 CA의 비밀 키를 사용하여 서명을 한다.

<br>

<img src="/assets/img/250418/check1.png" style="border-radius:5px" alt="check1" width="600">

<ol start="3">
  <li>클라이언트는 서명된 인증서를 서버로 전달한다.
    <blockquote><p>서버가 클라이언트에 전달할 때도 똑같다.</p></blockquote>
  </li>
  <li>전달받은 서버는 이미 로컬에 내장된 CA 공개키로 검증한다.</li>
  <li>검증이 성공하면 해당 인증서는 정상이라고 판단하고, 클라이언트를 신뢰한다.</li>
</ol>

<br>

#### 자체 서명 CA를 이용한 서명

외부 CA와 흐름은 같다.
<br>
다만 자체 서명으로 생성된 인증서는 공개키 역할을 수행하여 비밀키로 서명된 다른 인증서를 검증할 수 있다.

<br>

<img src="/assets/img/250418/self.png" style="border-radius:5px" alt="self" width="400">

1. 비밀키 `mongoCA.key`를 이용해 자체 서명한 `mongoCA.crt`를 생성한다.
2. 클라이언트가 사용할 비밀키로 똑같이 서명이 필요한 인증서 파일을 만든다.(`client.csr`)
3. 자체 서명 CA의 비밀키와 인증서를 지정하여 클라이언트 인증서를 서명한다.
  - CA의 비밀키로 서명한다.
  - CA의 인증서를 지정하는 이유는 누가 서명했는지를 기록하기 위함이다.
  - 즉, 서명한 주체가 `mongoCA.crt` 인증서가 되는 것이다.

<br>

<img src="/assets/img/250418/merge.png" style="border-radius:5px" alt="merge" width="400">

<ol start="4">
  <li>편의를 위해 클라이언트 키와 클라이언트 인증서를 묶어 <code class="language-plaintext highlighter-rouge">client.pem</code>으로 만든다.
    <blockquote>
      <p>서버에는 인증서만 보내진다.</p>
      <p>비밀키는 <strong>SSL 핸드셰이크</strong> 과정에만 사용되고, 서버로 전달되지 않는다. 핸드셰이크 과정도 복잡하므로 다루지 않겠다.</p>
    </blockquote>
  </li>
  <li>전달받은 서버는 이미 로컬에 내장된 CA 공개키로 검증한다.</li>
  <li>검증이 성공하면 해당 인증서는 정상이라고 판단하고, 클라이언트를 신뢰한다.</li>
</ol>

<br>

<img src="/assets/img/250418/check2.png" style="border-radius:5px" alt="check2" width="600">

<ol start="5">
  <li>클라이언는 서명된 인증서를 서버로 전달한다.</li>
  <li>전달받은 서버는 인증서에 포함되어 있는 공개키로 검증한다.</li>
  <li>검증이 성공하면 해당 인증서는 정상이라고 판단하고, 클라이언트를 신뢰한다.</li>
</ol>

<br>

## TLS 통신 설정

이제 TLS 통신을 위한 추가 작업들을 해주어야 한다.
<br>
애플리케이션도 일반 연결을 사용하고, DB 툴에도 SSL Termination을 통해 일반 연결을 사용하므로 mongoDB에 별다른 TLS 설정은 필요없다.
<br>
대신 리버스 프록시를 통해서만 접근할 수 있도록 방화벽/포트포워딩 설정을 잘 해주자.
<br>
또한 nginx가 검증할 수 있도록 nginx에 추가 설정을 해준다.

<br>

#### Nginx 추가 설정

TLS는 기본적으로 클라이언트가 서버의 인증서를 검증하는데, 이전에 설정했던 도메인 ssl 정보를 그대로 사용했다.

> 이것이 바로 리버스 프록시의 장점이다!<br>
> SSL의 중앙화된 관리로 추가 작업이 없어졌다.

nginx가 클라이언트의 인증서에 서명해줬던 `mongoCA.crt`를 통해 검증한다.

```nginx
# sudo vim /etc/nginx/streams-available/mongo

upstream mongodb {
    server localhost:<mongo 포트>;
}

server {
    listen 20000 ssl;
    proxy_pass mongodb;

    # 클라이언트가 검증할 서버의 인증서
    ssl_certificate /etc/letsencrypt/live/domain.mooo.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain.mooo.com/privkey.pem;

    ssl_verify_client on; # 클라이언트 인증서 검증, ssl_client_certificate로 지정된 CA로 검증한다.
    ssl_client_certificate /etc/mongo/ssl/mongoCA.crt; # 클라이언트 인증서를 검증할 인증서
}
```

<br>

## MongoDBCompass 연결

이제 `client.pem` 파일을 가지고 있는 사람만 DB에 접속할 수 있도록 보안이 강화되었다.
<br>
공식 DB 관리 도구인 MongoDBCompass로 TLS 연결을 진행해보자
<br><br>

새로운 커넥션 창을 열어서 `Advanced Connection Options`를 눌러 설정한다.

<br>

<img src="/assets/img/250418/connect1.png" style="border-radius:5px" alt="connect1" width="800">

먼저 `General` 탭에서 연결할 호스트를 넣어준다.

<br>

<img src="/assets/img/250418/connect2.png" style="border-radius:5px" alt="connect2" width="800">

`Authentication` 탭에서 유저명, 비밀번호, DB명을 넣어준다.

<br>

<img src="/assets/img/250418/connect3.png" style="border-radius:5px" alt="connect3" width="800">

`TLS/SSL` 탭에서 `SSL/TLS Connection`을 On으로 설정하고, `Client Certificate and Key(.pem)` 에 생성한 `client.pem` 파일을 넣어준다.
<br>
이후 `Save&Connect`로 연결해보면 정상 연결됨을 알 수 있다.